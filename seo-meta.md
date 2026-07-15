# SEO & meta tags

Thẻ meta giúp Google hiểu trang + hiện đẹp khi share lên Facebook/Zalo/Twitter.

## Công cụ

- **[Metatags.io](https://metatags.io)** — sinh thẻ meta + xem trước khi share
- **[OpenGraph.xyz](https://www.opengraph.xyz)** — preview OG card
- **[Favicon Generator](https://realfavicongenerator.net)** — tạo đủ bộ favicon mọi thiết bị
- **[Google Rich Results Test](https://search.google.com/test/rich-results)** — test structured data

## Các thẻ cốt lõi

```html
<title>Tiêu đề trang (50-60 ký tự)</title>
<meta name="description" content="Mô tả ~150 ký tự, hiện dưới tiêu đề trên Google">

<!-- Open Graph (Facebook, Zalo, LinkedIn) -->
<meta property="og:title" content="Tiêu đề khi share">
<meta property="og:description" content="Mô tả khi share">
<meta property="og:image" content="https://site.com/og-image.png">
<meta property="og:url" content="https://site.com/trang">

<!-- Twitter -->
<meta name="twitter:card" content="summary_large_image">
```

- Ảnh OG chuẩn: **1200×630px**.

## Cách dùng

### HTML tĩnh

Dán trực tiếp các thẻ trên vào `<head>` của từng trang.

### React

React SPA render bằng JS → crawler cũ có thể không thấy meta. Dùng
**[react-helmet-async](https://github.com/staylor/react-helmet-async)** để set meta theo trang:

```jsx
import { Helmet } from 'react-helmet-async';
<Helmet>
  <title>Trang chủ</title>
  <meta name="description" content="..." />
</Helmet>
```

(Muốn SEO chuẩn với React → cân nhắc SSR/Next.js.)

### Next.js

Dùng **Metadata API** (App Router) — SSR nên crawler luôn thấy:

```jsx
// app/page.tsx
export const metadata = {
  title: 'Trang chủ',
  description: '...',
  openGraph: { title: '...', images: ['/og.png'] },
};
```

Hoặc `generateMetadata()` cho trang động (blog theo slug...).

## Nền tảng khác

- **`sitemap.xml`** — liệt kê URL cho Google (Next.js: `app/sitemap.ts`).
- **`robots.txt`** — cho phép/chặn crawler (Next.js: `app/robots.ts`).
- **URL sạch, có nghĩa** — `/blog/huong-dan-icon` tốt hơn `/p?id=123`.
