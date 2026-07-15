# Security

Các lỗ hổng web phổ biến và cách phòng.

## Các mối nguy chính

| Lỗ hổng | Là gì | Phòng |
| --- | --- | --- |
| **XSS** | Chèn script độc qua input hiển thị lại | Escape output, sanitize HTML, CSP |
| **CSRF** | Giả request từ user đã đăng nhập | Token CSRF, cookie `SameSite` |
| **SQL Injection** | Chèn SQL qua input | Dùng ORM / query tham số hoá |
| **Rò rỉ secret** | Lộ API key/token ra client/git | Chỉ để ở server, `.gitignore`, biến env |
| **Broken auth** | Session/quyền yếu | Xem [auth.md](auth.md) |

## Cách dùng

### HTML tĩnh

- **Không nhét `innerHTML`** từ dữ liệu người dùng → dùng `textContent`:

```js
el.textContent = userInput;      // an toàn
// el.innerHTML = userInput;     // NGUY HIỂM (XSS)
```

- Cần render HTML người dùng → sanitize bằng **[DOMPurify](https://github.com/cure53/DOMPurify)**.
- Thêm **CSP** qua thẻ meta để chặn script lạ:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```

### React

- JSX **tự escape** `{biến}` → an toàn mặc định (đây là lý do ít XSS hơn).
- **Chỗ duy nhất nguy hiểm**: `dangerouslySetInnerHTML` → phải DOMPurify trước.

```jsx
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />
```

### Next.js

- Đặt **security headers** trong `next.config.js` (CSP, `X-Frame-Options`, HSTS).
- **Server Action / Route Handler**: validate mọi input bằng **Zod**, kiểm tra quyền trước khi thao tác DB.
- **`NEXT_PUBLIC_`** mới lộ ra client → secret KHÔNG được đặt prefix này.
- Chống spam/brute-force → **rate limit** (vd Upstash Ratelimit) ở API.

## Checklist nhanh

- [ ] Secret/API key **không** ở client, **không** commit lên git.
- [ ] Validate + sanitize **mọi input** ở server.
- [ ] Query DB qua **ORM/tham số hoá** (không nối chuỗi SQL).
- [ ] Cookie: `httpOnly` + `Secure` + `SameSite`.
- [ ] HTTPS toàn site.
- [ ] Rate limit endpoint nhạy cảm (login, gửi mail).
- [ ] Cập nhật dependency (`npm audit`), gỡ package không dùng.
- [ ] Không log thông tin nhạy cảm (xem [error-monitoring.md](error-monitoring.md)).
