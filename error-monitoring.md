# Error handling & Monitoring

Bắt lỗi khi chạy thật, ghi log, và theo dõi hành vi người dùng.

## Công cụ

- **[Sentry](https://sentry.io)** — bắt lỗi runtime + stack trace + báo về dashboard (phổ biến nhất)
- **[LogRocket](https://logrocket.com)** — quay lại phiên user + lỗi
- **[Vercel Analytics](https://vercel.com/analytics)** — analytics nhẹ, riêng tư (hợp Next.js)
- **[PostHog](https://posthog.com)** — analytics + session replay + feature flag (mã nguồn mở)
- **[Plausible](https://plausible.io)** — analytics đơn giản, không cookie, GDPR

## Cách dùng

### HTML tĩnh

Bắt lỗi toàn cục bằng event của `window`:

```html
<script>
  window.addEventListener('error', e => {
    // gửi về server/logging
    fetch('/log', { method: 'POST', body: JSON.stringify({ msg: e.message }) });
  });
  window.addEventListener('unhandledrejection', e => {
    console.error('Promise lỗi:', e.reason);
  });
</script>
```

Analytics nhẹ → nhúng snippet **Plausible/PostHog** vào `<head>`.

### React

- **Error Boundary** — chặn lỗi render, hiện UI dự phòng thay vì trắng trang:

```jsx
import { ErrorBoundary } from 'react-error-boundary';
<ErrorBoundary fallback={<p>Có lỗi, thử lại sau.</p>}>
  <App />
</ErrorBoundary>
```

- **Sentry** tự bắt lỗi + gửi về:

```bash
npm install @sentry/react
```

```jsx
import * as Sentry from '@sentry/react';
Sentry.init({ dsn: '...' });
```

### Next.js

- File **`error.tsx`** (App Router) — tự bắt lỗi của route, hiện UI + nút thử lại:

```jsx
// app/error.tsx
'use client';
export default function Error({ error, reset }) {
  return <button onClick={reset}>Thử lại</button>;
}
```

- **Sentry** có SDK riêng cho Next (bắt cả server + client):

```bash
npx @sentry/wizard@latest -i nextjs
```

## Mẹo

- Phân biệt **lỗi mong đợi** (form sai → báo tại chỗ) vs **lỗi bất ngờ** (crash → Error Boundary + Sentry).
- **Đừng nuốt lỗi** (`catch {}` rỗng) — ít nhất log lại.
- Che **thông tin nhạy cảm** trước khi gửi log (mật khẩu, token).
- Gắn **user id / release version** vào Sentry để lần ra lỗi nhanh.
- Analytics: chọn loại **không cookie** (Plausible) nếu ngại banner consent.
