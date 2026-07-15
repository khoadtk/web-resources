# Web Workers & WASM

Chạy việc nặng trong trình duyệt mà **không đơ UI**, và chạy code "không phải JS" trên web.

> Đi sâu riêng cho Next.js: [web-workers-wasm-nextjs.md](web-workers-wasm-nextjs.md)

## Khi nào cần

- JS chạy **1 luồng** — tính toán nặng (parse file lớn, nén ảnh, mã hoá) làm trang đứng hình → đẩy sang **Web Worker**.
- Cần thư viện gốc C/C++/Rust ngay trong trình duyệt (ffmpeg, sqlite, pdf...) → **WebAssembly (WASM)**.
- Lưu ý: việc nặng phía **server** thì dùng queue ([queue-cron.md](queue-cron.md)); Workers/WASM là cho phía **client**.

## Công cụ / thư viện WASM sẵn dùng

- **[Comlink](https://github.com/GoogleChromeLabs/comlink)** — gọi worker như gọi hàm thường (khỏi tự viết postMessage)
- **[ffmpeg.wasm](https://ffmpegwasm.netlify.app)** — xử lý video/audio trong trình duyệt
- **[sql.js](https://sql.js.org)** / **[wa-sqlite](https://github.com/rhashimoto/wa-sqlite)** — SQLite chạy client
- **[Squoosh codecs](https://github.com/GoogleChromeLabs/squoosh)** — nén ảnh WASM
- Viết WASM riêng: **Rust + wasm-bindgen** hoặc **AssemblyScript** (giống TS)

## Cách dùng

### HTML tĩnh

Worker API có sẵn:

```js
// main.js
const worker = new Worker('worker.js');
worker.postMessage({ numbers: bigArray });
worker.onmessage = e => console.log('kết quả:', e.data);

// worker.js — chạy luồng riêng, không chặn UI
self.onmessage = e => {
  const sum = e.data.numbers.reduce((a, b) => a + b, 0); // việc nặng
  self.postMessage(sum);
};
```

### React

- Vite hỗ trợ import worker sẵn:

```js
const worker = new Worker(new URL('./heavy.worker.js', import.meta.url), { type: 'module' });
```

- Dùng **Comlink** cho sạch:

```js
// heavy.worker.js
import * as Comlink from 'comlink';
Comlink.expose({ compress: (file) => {/* nặng */} });

// component
const api = Comlink.wrap(worker);
const result = await api.compress(file);   // như gọi async function
```

- Nhớ `worker.terminate()` khi unmount.

### Next.js

- Worker/WASM chỉ chạy client → component `'use client'`, tạo worker trong `useEffect`.
- WASM file `.wasm` được Next hỗ trợ import; lib như ffmpeg.wasm → load động khi cần:

```js
const { FFmpeg } = await import('@ffmpeg/ffmpeg'); // chỉ tải khi user bấm
```

- Đừng load WASM nặng lúc vào trang — chờ user thực sự dùng tính năng (xem [performance.md](performance.md)).

## Mẹo

- **Đo trước**: UI lag thật mới cần worker — thêm worker cho việc 5ms là phí công.
- Dữ liệu gửi qua worker bị **copy** — mảng lớn thì dùng `Transferable` (ArrayBuffer) để chuyển không copy.
- Worker **không đụng được DOM** — chỉ tính toán rồi gửi kết quả về.
- WASM file khá nặng (ffmpeg ~25MB) → lazy load + báo tiến trình tải cho user.
- Use case đẹp: nén ảnh client trước khi upload ([file-upload.md](file-upload.md)) — tiết kiệm băng thông cả hai đầu.
