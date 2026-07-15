# PWA & Offline

Biến web thành app **cài được** vào máy + **chạy offline**.

## Thành phần

- **Web App Manifest** — file `manifest.json`: tên, icon, màu → cho phép "Add to Home Screen".
- **Service Worker** — script chạy nền: cache, offline, push notification.
- **HTTPS** — bắt buộc để service worker hoạt động.

## Công cụ

- **[Vite PWA](https://vite-pwa-org.netlify.app)** — plugin PWA cho Vite/React
- **[next-pwa](https://github.com/shadowwalker/next-pwa)** / **[Serwist](https://serwist.pages.dev)** — PWA cho Next.js
- **[PWABuilder](https://www.pwabuilder.com)** — sinh manifest + đóng gói thành app store
- **[Workbox](https://developer.chrome.com/docs/workbox)** — thư viện quản lý cache của Google

## Cách dùng

### HTML tĩnh

Tạo `manifest.json` + đăng ký service worker:

```html
<link rel="manifest" href="/manifest.json">
<script>
  if ('serviceWorker' in navigator)
    navigator.serviceWorker.register('/sw.js');
</script>
```

```json
// manifest.json
{
  "name": "My App", "short_name": "App",
  "start_url": "/", "display": "standalone",
  "theme_color": "#3b82f6",
  "icons": [{ "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" }]
}
```

```js
// sw.js — cache đơn giản
self.addEventListener('fetch', e =>
  e.respondWith(caches.match(e.request).then(r => r || fetch(e.request)))
);
```

### React (Vite)

Dùng **Vite PWA** — tự sinh service worker + manifest:

```bash
npm install -D vite-plugin-pwa
```

```js
// vite.config.js
import { VitePWA } from 'vite-plugin-pwa';
export default { plugins: [VitePWA({ registerType: 'autoUpdate' })] };
```

### Next.js

Dùng **Serwist** (hoặc next-pwa). App Router hỗ trợ `app/manifest.ts`:

```jsx
// app/manifest.ts
export default function manifest() {
  return { name: 'My App', start_url: '/', display: 'standalone', icons: [...] };
}
```

## Mẹo

- Cần đủ **icon 192px + 512px** thì mới cài được.
- **Cache có chiến lược**: tĩnh (cache-first), API (network-first) — đừng cache tất cả bừa.
- Có **cơ chế update**: service worker mới → nhắc user reload (kẹt cache cũ rất khó chịu).
- Test bằng **Lighthouse → PWA** để biết còn thiếu gì.
