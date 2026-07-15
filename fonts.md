# Fonts

## Nguồn font

- **[Google Fonts](https://fonts.google.com)** — kho font web miễn phí lớn nhất
- **[Fontsource](https://fontsource.org)** — cài Google Fonts qua npm (self-host, không gọi CDN Google)
- **[Fontshare](https://fontshare.com)** — font chất lượng cao, miễn phí thương mại
- **[Bunny Fonts](https://fonts.bunny.net)** — thay Google Fonts, không tracking (GDPR)

## Cách dùng

### HTML tĩnh

Google Fonts qua `<link>` — nhanh, không cần build:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
```

```css
body { font-family: 'Inter', sans-serif; }
```

### React

Self-host qua Fontsource — không gọi CDN Google, import vào entry (`main.jsx`):

```bash
npm install @fontsource/inter
```

```js
import '@fontsource/inter/400.css';
import '@fontsource/inter/700.css';
// rồi dùng trong CSS: font-family: 'Inter', sans-serif;
```

### Next.js

`next/font` — chuẩn nhất: tự host, không layout shift, tối ưu sẵn:

```jsx
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'] });

// app/layout.tsx
<body className={inter.className}>
```

## Mẹo quan trọng

- **`font-display: swap`** — hiện text bằng font hệ thống trước, tránh "màn trắng chữ".
- **Chỉ load weight cần dùng** (400/700) — mỗi weight là một file, tải hết = chậm.
- **`subsets`** — chỉ lấy bảng chữ cần (latin) để nhẹ hơn.
- Ưu tiên **variable font** nếu có (1 file cho mọi độ đậm).
