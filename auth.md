# Auth

Đăng nhập / đăng ký, phiên (session), phân quyền.

## Khái niệm cốt lõi

- **Authentication** — bạn là ai (đăng nhập).
- **Authorization** — bạn được làm gì (phân quyền).
- **Session (cookie)** — server giữ phiên, gửi cookie cho client. An toàn cho web.
- **JWT (token)** — token tự chứa thông tin, client giữ. Hợp API/mobile.
- **OAuth / social login** — đăng nhập qua Google, GitHub, Facebook.

## Dịch vụ / thư viện

- **[Clerk](https://clerk.com)** — auth "cắm là chạy", UI sẵn, free tier tốt (hợp Next.js)
- **[Auth.js / NextAuth](https://authjs.dev)** — auth mã nguồn mở cho Next.js, nhiều provider
- **[Supabase Auth](https://supabase.com/auth)** — auth + database chung một chỗ
- **[Lucia](https://lucia-auth.com)** — thư viện auth tự kiểm soát (đọc thêm để hiểu bản chất)
- **[Better Auth](https://better-auth.com)** — framework-agnostic, đang lên

## Cách dùng

### HTML tĩnh

Web tĩnh thuần không có server → gọi tới **auth service** qua JS SDK, hoặc gọi API backend riêng:

```html
<script>
  // ví dụ gọi API backend tự viết
  const res = await fetch('/api/login', {
    method: 'POST',
    body: JSON.stringify({ email, password }),
  });
  // server set cookie httpOnly để giữ session
</script>
```

Với trang tĩnh có phần riêng tư → thường cần backend hoặc dịch vụ (Supabase/Clerk).

### React (SPA)

Dùng SDK của dịch vụ; SPA thường lưu token và gắn vào header request:

```jsx
// ví dụ Clerk
import { SignedIn, SignedOut, SignInButton, UserButton } from '@clerk/clerk-react';
<SignedOut><SignInButton /></SignedOut>
<SignedIn><UserButton /></SignedIn>
```

- Chặn route riêng tư bằng "protected route" (kiểm tra đăng nhập trước khi render).

### Next.js

**Auth.js (NextAuth)** hoặc **Clerk** là phổ biến nhất.

```bash
npm install next-auth
```

```jsx
// bảo vệ trong middleware.ts → chặn cả trang chưa đăng nhập
// đọc session trong Server Component:
import { auth } from '@/auth';
const session = await auth();
if (!session) redirect('/login');
```

- Ưu tiên **session cookie httpOnly** (SSR) → an toàn hơn lưu token trong localStorage.

## Mẹo bảo mật

- **Không lưu token nhạy cảm trong `localStorage`** (dễ bị XSS) → dùng cookie `httpOnly`.
- Luôn kiểm tra quyền lại ở **server**, đừng tin client.
- Dùng **HTTPS**, bật `Secure` + `SameSite` cho cookie.
- Băm mật khẩu bằng **bcrypt/argon2**, không lưu plaintext (nếu tự làm backend).
- Cân nhắc dịch vụ sẵn (Clerk/Auth.js) thay vì tự viết — auth dễ sai.
