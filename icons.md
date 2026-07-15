# Hướng dẫn dùng Icônes (icones.js.org)

> Nguồn: <https://icones.js.org>

## Icônes là gì?

**Icônes** gom hơn **200.000+ icon** từ ~150 bộ icon (Material Design Icons, Tabler,
Lucide, Font Awesome, Heroicons, Simple Icons...) vào một chỗ để tìm, copy và export
dễ dàng. Nó chạy trên nền [Iconify](https://iconify.design/).

Mỗi icon có một **ID dạng `prefix:name`** — ví dụ `mdi:home`, `lucide:settings`,
`tabler:user`. ID này là chìa khóa để dùng ở mọi nơi.

## Cách dùng cơ bản (trên web)

1. Vào <https://icones.js.org>
2. **Search** tên icon (tiếng Anh): `home`, `arrow`, `github`...
3. **Lọc theo bộ icon** ở sidebar trái nếu muốn (chỉ Tabler, chỉ Lucide...).
4. **Click 1 icon** → panel hiện các tùy chọn copy:
   - **Copy SVG** — dán thẳng SVG vào code
   - **Copy Component** — dạng component (React/Vue/Svelte...)
   - **Copy iconify name** — dạng `mdi:home` (dùng với Iconify)
   - **Download** PNG/SVG
   - Đổi **màu**, **size** ngay trên panel
5. **Chọn nhiều icon** (thêm vào collection dưới màn hình) → export cả loạt một lần.

## Các usecase phổ biến

### 1. Dán thẳng SVG (bất cứ đâu — HTML tĩnh, Learning Hub, template)

Đơn giản nhất, không cần cài gì. Chọn icon → **Copy SVG** → dán vào HTML:

```html
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
  <path fill="currentColor" d="M10 20v-6h4v6h5v-8h3L12 3L2 12h3v8z"/>
</svg>
```

Mẹo: dùng `fill="currentColor"` để icon **tự đổi màu theo màu chữ** (CSS `color`).

### 2. Iconify component (khuyên dùng cho Next.js / React)

Cài một lần, xài mọi icon trong 200k+ bộ mà **không cần import từng cái**:

```bash
npm install @iconify/react
```

```jsx
import { Icon } from '@iconify/react';

<Icon icon="lucide:home" />
<Icon icon="tabler:arrow-right" width="20" color="#3b82f6" />
```

→ Trên Icônes chỉ cần copy tên `lucide:home`. Đổi bộ icon = đổi chuỗi tên,
không cần cài thêm package.

### 3. Iconify qua CDN (web tĩnh, không build tool)

Cho các trang HTML thuần (kiểu Learning Hub hiện tại):

```html
<script src="https://code.iconify.design/iconify-icon/2.1.0/iconify-icon.min.js"></script>

<iconify-icon icon="mdi:github"></iconify-icon>
<iconify-icon icon="tabler:sun" width="24"></iconify-icon>
```

### 4. Icon component thư viện có sẵn (Lucide / Heroicons / Tabler)

Nếu chỉ dùng 1 bộ và muốn tree-shaking chuẩn, dùng Icônes để **tra đúng tên icon**
rồi import từ package gốc:

```jsx
// Lucide (rất hợp Next.js)
import { Home, Settings } from 'lucide-react';
<Home size={20} />
```

### 5. VS Code extension (Dev Dashboard)

Trong webview, dùng **SVG inline** (usecase 1) hoặc **CDN** (usecase 3) là gọn nhất,
vì webview không cần bundle riêng cho icon.

## Chọn cách nào?

| Tình huống                                    | Nên dùng                          |
| --------------------------------------------- | --------------------------------- |
| HTML tĩnh, không build (Learning Hub)         | Copy SVG hoặc CDN `iconify-icon`  |
| Next.js / React, muốn xài nhiều bộ icon       | `@iconify/react`                  |
| Chỉ dùng 1 bộ, tối ưu bundle                  | Lucide/Heroicons package + Icônes |
| VS Code webview                               | SVG inline / CDN                  |
