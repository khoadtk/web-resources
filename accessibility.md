# Accessibility (a11y)

Làm web dùng được cho mọi người: đọc màn hình (screen reader), bàn phím, người mù màu.

## Công cụ

- **[axe DevTools](https://www.deque.com/axe/devtools/)** — extension quét lỗi a11y
- **[WAVE](https://wave.webaim.org)** — kiểm tra a11y online
- **[Contrast Checker](https://webaim.org/resources/contrastchecker/)** — tương phản màu
- **Lighthouse** (trong Chrome DevTools) — chấm điểm Accessibility

## Nguyên tắc cốt lõi

- **HTML ngữ nghĩa**: dùng `<button>`, `<nav>`, `<header>`, `<main>` thay vì `<div>` bừa.
- **Ảnh có `alt`**; ảnh trang trí → `alt=""`.
- **Form có `<label>`** gắn với input (`htmlFor` / `for`).
- **Bàn phím**: mọi thứ bấm được bằng chuột phải bấm được bằng `Tab` + `Enter`.
- **Tương phản** chữ/nền ≥ 4.5:1.
- **Đừng chỉ dùng màu** để báo trạng thái — thêm icon/text.

## Cách dùng

### HTML tĩnh

Dùng ARIA khi HTML thường không đủ diễn đạt:

```html
<button aria-label="Đóng">✕</button>
<nav aria-label="Điều hướng chính">...</nav>
<div role="alert">Lưu thành công</div>
```

Nút chỉ có icon → luôn thêm `aria-label` mô tả.

### React

- Dùng đúng thẻ ngữ nghĩa như HTML thuần (JSX vẫn là HTML).
- Lint tự động với **eslint-plugin-jsx-a11y** để bắt lỗi khi code:

```bash
npm install -D eslint-plugin-jsx-a11y
```

- Component phức tạp (modal, dropdown) → dùng Radix/Headless UI (đã lo a11y sẵn).

### Next.js

- Kế thừa mọi thứ của React.
- `next/link` + `next/image` đã xử lý focus/alt đúng chuẩn.
- Chạy **Lighthouse** trên bản build để chấm điểm a11y trước khi deploy.

## Mẹo test nhanh

1. **Rút chuột ra**, chỉ dùng `Tab` + `Enter` — có dùng được cả trang không?
2. Bật screen reader (VoiceOver trên Mac: `Cmd+F5`) — nghe có hiểu không?
3. Chạy axe/Lighthouse — sửa lỗi đỏ trước.
