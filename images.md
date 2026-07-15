# Images & Photos

## Nguồn ảnh miễn phí

- **[Unsplash](https://unsplash.com)** — ảnh chụp chất lượng cao, miễn phí thương mại
- **[Pexels](https://pexels.com)** — ảnh + video miễn phí
- **[Pixabay](https://pixabay.com)** — ảnh, vector, video
- **[placehold.co](https://placehold.co)** — ảnh placeholder `https://placehold.co/600x400`

## Định dạng nên dùng

| Định dạng | Khi nào dùng |
| --- | --- |
| **WebP** | Mặc định cho ảnh web — nhẹ hơn JPG ~30% |
| **AVIF** | Nhẹ nhất, chất lượng cao — nhưng encode chậm |
| **SVG** | Logo, icon, hình khối (vector, không vỡ) |
| **PNG** | Cần nền trong suốt, ảnh ít màu |
| **JPG** | Ảnh chụp khi không hỗ trợ WebP |

## Tối ưu ảnh

- **[Squoosh](https://squoosh.app)** — nén ảnh online (đổi format, chỉnh chất lượng)
- **[TinyPNG](https://tinypng.com)** — nén PNG/JPG nhanh

## Cách dùng

### HTML tĩnh

Lazy load + responsive image thủ công:

```html
<img src="photo.webp" loading="lazy" width="600" height="400" alt="mô tả">

<!-- responsive: chọn size theo màn hình -->
<img
  srcset="small.webp 480w, large.webp 1080w"
  sizes="(max-width: 600px) 480px, 1080px"
  src="large.webp" alt="mô tả">
```

- Luôn đặt `width` + `height` → tránh layout shift (CLS).
- Luôn có `alt` → SEO + accessibility.

### React

Không có tối ưu sẵn — vẫn dùng `<img loading="lazy">`. Muốn tự tối ưu thì
dùng lib như **[unpic](https://unpic.pics)** hoặc build-time (vite-imagetools):

```jsx
<img src={photo} loading="lazy" width={600} height={400} alt="mô tả" />
```

### Next.js

`next/image` — tự tối ưu, tự lazy load, tự chọn WebP/AVIF, tự responsive:

```jsx
import Image from 'next/image';
<Image src="/photo.jpg" width={600} height={400} alt="mô tả" />
```

Ảnh từ domain ngoài (Unsplash...) cần khai báo trong `next.config.js` → `images.remotePatterns`.
