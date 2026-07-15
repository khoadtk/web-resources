# Performance

Web nhanh = giữ chân người dùng + SEO tốt hơn (Google xếp hạng theo tốc độ).

## Đo lường

- **[PageSpeed Insights](https://pagespeed.web.dev)** — chấm điểm + gợi ý sửa
- **Lighthouse** (Chrome DevTools) — Performance score
- **[WebPageTest](https://www.webpagetest.org)** — test chi tiết theo vùng
- **[Bundlephobia](https://bundlephobia.com)** — cân nặng 1 package npm trước khi cài

## Core Web Vitals (3 chỉ số Google quan tâm)

| Chỉ số | Nghĩa | Ngưỡng tốt |
| --- | --- | --- |
| **LCP** | Thời gian hiện nội dung chính | < 2.5s |
| **INP** | Độ trễ khi tương tác | < 200ms |
| **CLS** | Độ "nhảy layout" | < 0.1 |

## Kỹ thuật chung

- **Nén ảnh** + dùng WebP/AVIF (xem [images.md](images.md)).
- **Lazy load** ảnh + component dưới màn hình.
- **Chỉ load font/JS cần dùng** — bỏ thư viện nặng không cần.
- **Cache** + CDN cho asset tĩnh.
- Đặt `width`/`height` cho ảnh → tránh CLS.

## Cách dùng

### HTML tĩnh

- `defer` cho script để không chặn render:

```html
<script src="app.js" defer></script>
<img src="a.webp" loading="lazy" width="600" height="400" alt="">
```

- `preconnect`/`preload` cho tài nguyên quan trọng (font, ảnh hero).

### React

- **Code splitting** với `lazy` + `Suspense` → chỉ tải trang đang xem:

```jsx
import { lazy, Suspense } from 'react';
const Chart = lazy(() => import('./Chart'));
<Suspense fallback={<Spinner />}><Chart /></Suspense>
```

- Tránh render lại thừa: `memo`, `useMemo`, `useCallback` (khi thật sự cần).
- Kiểm tra bundle bằng `rollup-plugin-visualizer` / `vite-bundle-visualizer`.

### Next.js

- **Server Components** (mặc định) → gửi ít JS xuống client.
- `next/image`, `next/font` tối ưu sẵn ảnh + font.
- `next/dynamic` để lazy-load component nặng:

```jsx
import dynamic from 'next/dynamic';
const Chart = dynamic(() => import('./Chart'), { ssr: false });
```

- Ưu tiên **SSR/SSG** để LCP nhanh; phân tích bằng `@next/bundle-analyzer`.
