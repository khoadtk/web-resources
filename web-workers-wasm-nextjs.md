# Web Workers & WASM trong Next.js (deep-dive)

> Đào sâu từ [web-workers-wasm.md](web-workers-wasm.md) — riêng cho Next.js App Router.

## Vì sao trong Next.js lại "khó" hơn?

Next.js chạy code ở **2 nơi**: server (SSR/Server Component) và client. Worker & WASM client chỉ tồn tại trên trình duyệt, nên mọi rắc rối đều quy về 3 điều:

1. **SSR không có `window`/`Worker`** → tạo worker lúc render server là crash (`Worker is not defined`).
2. **Bundler phải "nhìn thấy" file worker** để đóng gói riêng nó thành 1 chunk.
3. **WASM là file nhị phân** → cần config để import, và file nặng → phải lazy load.

## 1. Tạo Web Worker đúng cách

### Cú pháp bundler hiểu được

Webpack/Turbopack chỉ tách worker thành chunk riêng khi bạn viết **đúng dạng tĩnh** này:

```ts
// ✅ đúng — URL literal + import.meta.url, bundler phân tích được
const worker = new Worker(new URL('../workers/heavy.worker.ts', import.meta.url));

// ❌ sai — đường dẫn động, bundler không đóng gói được
const path = '../workers/' + name;
const worker = new Worker(new URL(path, import.meta.url));
```

### Cấu trúc file gợi ý

```
src/
├─ workers/
│  └─ heavy.worker.ts      # code chạy luồng riêng
├─ hooks/
│  └─ useHeavyWorker.ts    # hook bọc worker
└─ app/tools/page.tsx      # 'use client' component dùng hook
```

```ts
// workers/heavy.worker.ts
self.onmessage = (e: MessageEvent<number[]>) => {
  const result = e.data.reduce((a, b) => a + b, 0); // việc nặng
  self.postMessage(result);
};
export {}; // để TS coi đây là module
```

### Hook pattern — tạo trong effect, huỷ khi unmount

```tsx
'use client';
import { useEffect, useRef } from 'react';

export function useHeavyWorker(onResult: (n: number) => void) {
  const workerRef = useRef<Worker | null>(null);

  useEffect(() => {
    // chỉ chạy trên client sau khi mount → không đụng SSR
    workerRef.current = new Worker(
      new URL('../workers/heavy.worker.ts', import.meta.url)
    );
    workerRef.current.onmessage = (e) => onResult(e.data);
    return () => workerRef.current?.terminate(); // tránh leak + HMR nhân đôi worker
  }, []);

  return (data: number[]) => workerRef.current?.postMessage(data);
}
```

Điểm chốt:

- Tạo worker **trong `useEffect`**, không phải ở module scope → SSR an toàn.
- **`terminate()` trong cleanup** — thiếu dòng này, mỗi lần HMR khi dev là thêm 1 worker chạy nền.
- File dùng hook phải là **`'use client'`**.

## 2. Comlink — gọi worker như hàm async

Tự viết `postMessage` + match message id rất nhanh rối. Comlink biến worker thành object có type:

```bash
npm install comlink
```

```ts
// workers/image.worker.ts
import * as Comlink from 'comlink';

const api = {
  async compress(file: File, quality: number): Promise<Blob> {
    // ...việc nặng: decode → resize → encode
    return compressed;
  },
};
export type ImageWorkerApi = typeof api;
Comlink.expose(api);
```

```tsx
// hooks/useImageWorker.ts  ('use client' nơi dùng)
import * as Comlink from 'comlink';
import type { ImageWorkerApi } from '../workers/image.worker';

const worker = () =>
  Comlink.wrap<ImageWorkerApi>(
    new Worker(new URL('../workers/image.worker.ts', import.meta.url))
  );

// dùng: const blob = await api.compress(file, 0.8);  ← type-safe, như gọi hàm thường
```

Type `ImageWorkerApi` import bằng `import type` → chỉ lấy kiểu, không kéo code worker vào bundle chính.

## 3. Truyền dữ liệu lớn: Transferable

`postMessage` mặc định **copy** dữ liệu (structured clone). Mảng/ảnh lớn → copy tốn thời gian hơn cả việc tính. Chuyển "quyền sở hữu" bằng transferable:

```ts
const buffer = new ArrayBuffer(50_000_000);
worker.postMessage(buffer, [buffer]);   // chuyển, không copy — gần như 0ms
// sau dòng này buffer ở luồng chính rỗng (đã chuyển đi)
```

Các loại transfer được: `ArrayBuffer`, `MessagePort`, `ImageBitmap`, `OffscreenCanvas`. Với Comlink: `Comlink.transfer(value, [buffer])`.

## 4. WASM phía client

### Import file .wasm trong Next

Bật thí nghiệm webpack trong `next.config.js`:

```js
// next.config.js
module.exports = {
  webpack: (config) => {
    config.experiments = { ...config.experiments, asyncWebAssembly: true };
    return config;
  },
};
```

Nhưng thực tế **ít khi cần** — các thư viện WASM phổ biến (ffmpeg.wasm, sql.js...) tự lo việc load file `.wasm` qua `fetch`, bạn chỉ cần:

```tsx
'use client';
// chỉ tải khi user thật sự dùng (25MB!)
async function loadFFmpeg() {
  const { FFmpeg } = await import('@ffmpeg/ffmpeg');
  const ffmpeg = new FFmpeg();
  await ffmpeg.load(); // fetch file .wasm
  return ffmpeg;
}
```

### SharedArrayBuffer — bẫy lớn nhất của ffmpeg.wasm

Bản multithread của ffmpeg.wasm cần `SharedArrayBuffer`, mà trình duyệt chỉ bật khi trang có **COOP/COEP header**:

```js
// next.config.js
module.exports = {
  async headers() {
    return [{
      source: '/(.*)',
      headers: [
        { key: 'Cross-Origin-Opener-Policy', value: 'same-origin' },
        { key: 'Cross-Origin-Embedder-Policy', value: 'require-corp' },
      ],
    }];
  },
};
```

⚠️ Đánh đổi: `require-corp` **chặn tài nguyên cross-origin không khai báo CORP** — iframe YouTube, ảnh CDN ngoài, script analytics có thể vỡ. Cách né:

- Chỉ đặt header cho **route dùng WASM** (`source: '/tools/:path*'`) thay vì cả site.
- Hoặc dùng bản **single-thread** (`@ffmpeg/core` thường) — chậm hơn nhưng khỏi cần header.

## 5. WASM phía server (bonus)

- **Node runtime** (Route Handler mặc định): load `.wasm` như file thường:

```ts
import fs from 'node:fs';
const wasm = await WebAssembly.instantiate(fs.readFileSync('./lib.wasm'));
```

- **Edge runtime** (Vercel Edge): hỗ trợ import WASM module trực tiếp — đây là cách chạy thư viện native trên edge (vd resize ảnh bằng `@vercel/og` chính là WASM bên dưới).
- Việc nặng chạy **lâu** ở server → đừng nhét vào request, đẩy sang queue ([queue-cron.md](queue-cron.md)).

## 6. Ví dụ end-to-end: nén ảnh trước khi upload

Use case thực tế nhất, ăn khớp với [file-upload.md](file-upload.md) — giảm 5MB ảnh điện thoại xuống ~300KB **trước** khi rời máy user:

```bash
npm install browser-image-compression   # lib này tự chạy trong web worker
```

```tsx
'use client';
import imageCompression from 'browser-image-compression';

async function handleFile(file: File) {
  const compressed = await imageCompression(file, {
    maxSizeMB: 0.3,
    maxWidthOrHeight: 1920,
    useWebWorker: true,      // nén trong worker → UI không đơ
  });
  // upload compressed thay vì file gốc
  const fd = new FormData();
  fd.append('file', compressed);
  await fetch('/api/upload', { method: 'POST', body: fd });
}
```

Được cả ba: UI mượt (worker), upload nhanh (file nhỏ), server nhẹ (ít băng thông + khỏi resize).

## 7. Các usecase thực tế khác

### Import file Excel/CSV lớn — parse không đơ UI

User kéo file 50k dòng vào ([drag-drop.md](drag-drop.md)) → parse trong worker → hiện preview bảng ([tables.md](tables.md)):

```ts
// workers/csv.worker.ts
import Papa from 'papaparse';
self.onmessage = (e: MessageEvent<File>) => {
  Papa.parse(e.data, {
    header: true,
    complete: (r) => self.postMessage({ rows: r.data, errors: r.errors }),
  });
};
```

(PapaParse còn có option `worker: true` tự lo — nhưng tự viết worker cho bạn thêm bước validate/transform trước khi trả về.) Excel thì dùng **SheetJS** cùng pattern.

### Cắt / convert video ngắn — ffmpeg.wasm

Trim video trước khi upload (đỡ upload 100MB để server cắt còn 10s):

```ts
// trong worker, sau khi ffmpeg.load()
await ffmpeg.writeFile('in.mp4', await fetchFile(file));
await ffmpeg.exec(['-i', 'in.mp4', '-ss', '0', '-t', '10', '-c', 'copy', 'out.mp4']);
const data = await ffmpeg.readFile('out.mp4');   // Blob → preview hoặc upload
```

Cùng luồng này: **tạo thumbnail video** (`-vf thumbnail`), **convert webm → mp4**, **tách audio**. Nhớ bẫy COOP/COEP ở mục 4.

### OCR — đọc chữ trong ảnh bằng tesseract.js

Quét hoá đơn/danh thiếp ngay trên máy user, không gửi ảnh đi đâu (privacy):

```ts
import { createWorker } from 'tesseract.js';   // tự chạy WASM trong worker sẵn
const w = await createWorker('vie');           // có tiếng Việt
const { data } = await w.recognize(file);
console.log(data.text);
```

### SQLite trong trình duyệt — app offline thật sự

**sql.js / wa-sqlite** + lưu vào OPFS → app ghi chú/flashcard dùng SQL đầy đủ, không cần server (hợp kiểu Learning Hub offline):

```ts
const SQL = await initSqlJs({ locateFile: f => `https://sql.js.org/dist/${f}` });
const db = new SQL.Database();
db.run('CREATE TABLE cards (front TEXT, back TEXT)');
```

### Hash file trước upload — dedupe & resume

Tính SHA-256 trong worker → hỏi server "file này có chưa?" trước khi tốn băng thông upload:

```ts
// workers/hash.worker.ts
self.onmessage = async (e: MessageEvent<ArrayBuffer>) => {
  const digest = await crypto.subtle.digest('SHA-256', e.data);
  self.postMessage(Array.from(new Uint8Array(digest)).map(b => b.toString(16).padStart(2, '0')).join(''));
};
// gửi bằng transferable: worker.postMessage(buf, [buf])
```

### Export ZIP/Excel lớn phía client

Gom 200 ảnh đã xử lý thành 1 file ZIP cho user tải, không cần server đóng gói:

```ts
import { zip } from 'fflate';   // fflate nhanh + hỗ trợ chạy trong worker
zip({ 'photos/a.jpg': aBytes, 'photos/b.jpg': bBytes }, (err, out) => {
  const url = URL.createObjectURL(new Blob([out]));
  // <a download href={url}>
});
```

### Search client trên dataset lớn

Fuse.js trên 50k bản ghi làm ô search khựng theo từng phím gõ → đẩy index + query vào worker, main thread chỉ gửi từ khoá (đã debounce, xem [search.md](search.md)) và nhận kết quả.

### Chọn nhanh theo nhu cầu

| Nhu cầu | Công cụ | Kiểu |
| --- | --- | --- |
| Nén ảnh trước upload | browser-image-compression | Worker |
| Parse CSV/Excel lớn | PapaParse / SheetJS | Worker |
| Cắt/convert video | ffmpeg.wasm | WASM (+COOP/COEP) |
| OCR ảnh → text | tesseract.js | WASM + Worker |
| DB offline có SQL | sql.js / wa-sqlite | WASM |
| Hash/dedupe file | crypto.subtle | Worker |
| ZIP/export lớn | fflate | Worker |
| Search dataset lớn | Fuse.js trong worker | Worker |

## 8. Checklist lỗi hay gặp

- [ ] `Worker is not defined` → đang tạo worker lúc SSR; chuyển vào `useEffect` / sau tương tác.
- [ ] Worker "không làm gì" → sai cú pháp `new URL(..., import.meta.url)` nên bundler không đóng gói.
- [ ] Dev một hồi máy nóng → thiếu `terminate()` trong cleanup, HMR chồng worker.
- [ ] `SharedArrayBuffer is not defined` → thiếu COOP/COEP header (mục 4).
- [ ] Ảnh iframe/CDN vỡ sau khi thêm header → COEP chặn; giới hạn header theo route.
- [ ] Bundle phình to → WASM/worker lib phải `await import()` khi dùng, không import đầu file.
- [ ] Gửi mảng lớn qua worker vẫn lag → quên transferable (mục 3).

## Khi nào KHÔNG cần

- Việc < ~50ms → cứ chạy luồng chính; worker thêm độ trễ giao tiếp + phức tạp.
- Việc nặng nhưng **không cần offline/client** → làm ở server + queue rẻ hơn nhiều so với ship 25MB WASM cho user.
- Cần "UI đỡ giật khi render list lớn" → đó là việc của virtualize ([tables.md](tables.md)) / `useTransition`, không phải worker.
