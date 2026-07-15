# Illustrations

Ảnh minh họa (illustration) — dùng cho landing page, empty state, onboarding, 404...

## Nguồn miễn phí

- **[unDraw](https://undraw.co)** — illustration SVG, **đổi màu chủ đạo được**, miễn phí
- **[Storyset](https://storyset.com)** — illustration có animation, chỉnh màu
- **[Humaaans](https://humaaans.com)** — ghép người (tư thế, quần áo) tùy biến
- **[Open Peeps](https://openpeeps.com)** — nhân vật vẽ tay, mix & match
- **[DiceBear](https://dicebear.com)** — sinh **avatar** tự động từ 1 chuỗi (seed)
- **[Blush](https://blush.design)** — illustration tùy biến từ nhiều nghệ sĩ

## Cách dùng

### unDraw / Storyset

Tải SVG → chỉnh màu chủ đạo cho khớp brand → dùng như ảnh thường:

```html
<img src="/illustrations/empty-box.svg" alt="Chưa có dữ liệu" width="300">
```

### DiceBear — avatar động (không cần lưu ảnh)

```html
<img src="https://api.dicebear.com/9.x/thumbs/svg?seed=Khoa" alt="avatar">
```

Đổi `seed` → ra avatar khác. Rất hợp làm avatar mặc định cho user.

## Usecase phổ biến

- **Empty state** — "Chưa có dữ liệu", giỏ hàng trống
- **404 / error page**
- **Onboarding / landing** — minh họa tính năng
- **Avatar mặc định** — DiceBear theo tên/email user

## Mẹo

- Giữ **1 phong cách nhất quán** trên toàn app (đừng trộn unDraw + Humaaans lung tung).
- Chỉnh **màu chủ đạo** khớp với `--primary` của theme.
- Ưu tiên **SVG** (nhẹ, không vỡ) hơn PNG.
