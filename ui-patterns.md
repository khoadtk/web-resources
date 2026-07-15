# UI Patterns (Toast, Loading, Empty state...)

Các mẫu UI hay lặp lại: thông báo, trạng thái tải, trạng thái rỗng, lỗi.

## Thư viện

- **[Sonner](https://sonner.emilkowal.ski)** — toast đẹp cho React, cực gọn (khuyên dùng)
- **[react-hot-toast](https://react-hot-toast.com)** — toast nhẹ, dễ dùng
- **[nProgress](https://ricostacruz.com/nprogress/)** — thanh loading trên đầu trang khi chuyển trang
- Skeleton: tự làm bằng CSS hoặc component của shadcn/ui

## Các pattern & cách dùng

### 1. Toast / Notification (báo thành công/lỗi)

**HTML tĩnh** — tự dựng bằng div + CSS + setTimeout:

```js
function toast(msg) {
  const el = document.createElement('div');
  el.className = 'toast';
  el.textContent = msg;
  document.body.appendChild(el);
  setTimeout(() => el.remove(), 3000);
}
```

**React / Next.js** — dùng Sonner:

```bash
npm install sonner
```

```jsx
import { Toaster, toast } from 'sonner';
// đặt <Toaster /> 1 lần ở layout gốc
toast.success('Lưu thành công');
toast.error('Có lỗi xảy ra');
```

(Next.js: đặt `<Toaster />` trong `app/layout.tsx`, component gọi `toast()` cần `'use client'`.)

### 2. Loading / Skeleton (đang tải)

- **Spinner** — cho tải nhanh, ngắn.
- **Skeleton** (khung xám nhấp nháy) — cho tải nội dung dạng list/card, đỡ "giật".

```css
.skeleton {
  background: linear-gradient(90deg, #eee 25%, #f5f5f5 50%, #eee 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer { to { background-position: -200% 0; } }
```

**Next.js** có sẵn file `loading.tsx` → tự hiện khi trang/route đang tải.

### 3. Empty state (chưa có dữ liệu)

Đừng để trang trắng. Hiện: icon/illustration + 1 câu + nút hành động.

```jsx
<div className="empty">
  <img src="/illustrations/empty.svg" alt="" width={160} />
  <p>Chưa có ghi chú nào</p>
  <button>Tạo ghi chú đầu tiên</button>
</div>
```

(Illustration lấy từ [illustrations.md](illustrations.md) — unDraw/Storyset.)

### 4. Error state (khi lỗi)

- Báo lỗi rõ ràng + nút **thử lại**, đừng chỉ log console.
- **Next.js** có `error.tsx` → tự bắt lỗi của route và hiện UI dự phòng.

## Mẹo

- 4 trạng thái mọi màn có data đều cần: **loading / có data / rỗng / lỗi** — làm đủ cả 4.
- Toast dùng cho thông báo **thoáng qua**; lỗi nghiêm trọng → hiện ngay tại chỗ (inline).
- Skeleton nên **giống hình dạng nội dung thật** → cảm giác nhanh hơn.
