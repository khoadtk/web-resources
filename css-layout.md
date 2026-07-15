# CSS & Layout

Dàn trang: Flexbox, Grid, responsive, và cách viết CSS gọn.

## Học & công cụ

- **[Flexbox Froggy](https://flexboxfroggy.com)** — game học Flexbox
- **[Grid Garden](https://cssgridgarden.com)** — game học Grid
- **[CSS Grid Generator](https://cssgrid-generator.netlify.app)** — sinh grid trực quan
- **[Flexbox Cheatsheet](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)** — tra nhanh property

## Chọn Flexbox hay Grid?

- **Flexbox** — dàn theo **1 chiều** (hàng ngang hoặc dọc): navbar, hàng nút, list.
- **Grid** — dàn theo **2 chiều** (hàng + cột): layout trang, gallery, dashboard.

## Kỹ thuật cốt lõi

```css
/* Flex: căn giữa mọi thứ */
.center { display: flex; align-items: center; justify-content: center; }

/* Grid: cột tự co, tự xuống dòng — responsive không cần media query */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}
```

## Responsive

```css
/* Mobile-first: viết cho mobile trước, mở rộng lên desktop */
.box { padding: 1rem; }
@media (min-width: 768px) { .box { padding: 2rem; } }
```

- **Container query** — responsive theo bề rộng component (không phải màn hình):

```css
.card-wrap { container-type: inline-size; }
@container (min-width: 400px) { .card { flex-direction: row; } }
```

## Cách dùng

### HTML tĩnh

Viết CSS thuần như trên trong file `.css`, dùng `class`. Đủ cho mọi layout.

### React

- CSS Modules (`styles.module.css`) → class cục bộ, không đụng nhau:

```jsx
import s from './Card.module.css';
<div className={s.card}>...</div>
```

- Hoặc **Tailwind** — viết layout bằng utility class thẳng trong JSX:

```jsx
<div className="flex items-center justify-center gap-4">...</div>
```

### Next.js

- Hỗ trợ sẵn CSS Modules + `globals.css`.
- **Tailwind** là lựa chọn phổ biến nhất (setup sẵn khi `create-next-app`):

```jsx
<div className="grid grid-cols-1 md:grid-cols-3 gap-4">...</div>
```

## Mẹo

- **`gap`** dùng được cho cả Flex và Grid → khoảng cách đều, khỏi margin lẻ.
- Ưu tiên **mobile-first** (`min-width`) — dễ maintain hơn.
- `minmax(0, 1fr)` fix lỗi grid/flex item bị tràn khi có nội dung dài.
