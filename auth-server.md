# Auth phía server

Bên dưới các dịch vụ auth (Clerk, NextAuth) là những cơ chế này — hiểu để dùng đúng và debug được.

> Tổng quan + thư viện ở [auth.md](auth.md). File này đào phần nền, không gắn framework.

## Lưu mật khẩu — hash, không phải mã hoá

- **Không bao giờ lưu plaintext**, cũng không "mã hoá" (mã hoá giải ngược được — DB lộ là lộ hết).
- **Hash một chiều + salt** bằng thuật toán *cố tình chậm*: **bcrypt** hoặc **argon2** (không dùng MD5/SHA-256 trần — nhanh quá, brute-force dễ).

```ts
import bcrypt from 'bcrypt';
const hash = await bcrypt.hash(password, 12);        // 12 = cost, chậm ~100ms có chủ đích
const ok = await bcrypt.compare(inputPassword, hash); // so sánh, không decode
```

- Salt (bcrypt tự sinh, nằm trong chuỗi hash) → 2 user cùng mật khẩu vẫn ra hash khác → chặn tra bảng sẵn (rainbow table).

## Session vs JWT — khác nhau ở chỗ ai giữ trạng thái

### Session (stateful)

```
Đăng nhập → server tạo bản ghi session (DB/Redis) → gửi cookie chứa sessionId
Request sau → server tra sessionId trong store → biết user là ai
```

- **Thu hồi tức thì**: xoá bản ghi là logout ngay (đổi mật khẩu, khoá tài khoản).
- Giá: mỗi request tốn 1 lượt tra store; nhiều instance thì store phải chung (Redis — xem [networking-basics.md](networking-basics.md) phần stateless).

### JWT (stateless)

```
Đăng nhập → server ký token chứa { sub: userId, exp: ... } → client giữ
Request sau → server chỉ VERIFY chữ ký, không tra DB
```

- Nhanh, không cần store, service khác verify được (chỉ cần secret/public key) — hợp API/microservice.
- Giá: **không thu hồi được trước khi hết hạn** — token lộ là dùng được tới `exp`.

```ts
import jwt from 'jsonwebtoken';
const token = jwt.sign({ sub: user.id }, process.env.JWT_SECRET!, { expiresIn: '15m' });
const payload = jwt.verify(token, process.env.JWT_SECRET!); // throw nếu sai/hết hạn
```

- JWT chỉ **ký, không mã hoá** — ai cũng decode đọc được payload (thử ở jwt.io). Đừng nhét dữ liệu nhạy cảm vào.

### Chọn cái nào?

Web app truyền thống, cần logout tức thì → **session + cookie httpOnly**. API cho nhiều client/service → **JWT ngắn hạn + refresh token**. Đừng chọn JWT chỉ vì "nghe hiện đại".

## Refresh token — vá điểm yếu của JWT

```
Access token  : JWT sống NGẮN (15 phút) — gửi kèm mọi request
Refresh token : chuỗi random sống DÀI (30 ngày) — LƯU TRONG DB, chỉ dùng để xin access token mới
```

- Access token lộ → thiệt hại tối đa 15 phút. Refresh token **nằm trong DB nên thu hồi được** — đây là chỗ lấy lại quyền kiểm soát.
- Nâng cao: **rotation** — mỗi lần dùng refresh token thì cấp cái mới + vô hiệu cái cũ; cái cũ bị dùng lại = dấu hiệu token bị trộm → thu hồi cả chuỗi.

## OAuth 2.0 — "Đăng nhập bằng Google" hoạt động thế nào

```
1. App đẩy user sang Google kèm client_id + redirect_uri + state
2. User đồng ý trên trang Google
3. Google gọi lại redirect_uri của bạn kèm ?code=...
4. SERVER bạn đổi code + client_secret lấy access token (server-to-server)
5. Lấy profile (email, tên) → tạo/tìm user trong DB CỦA BẠN → tạo session của bạn
```

- Điều hay hiểu nhầm: sau bước 5, phiên đăng nhập là **session/JWT của app bạn** như thường — OAuth chỉ là cách *xác minh danh tính ban đầu* thay cho mật khẩu.
- `state` chống CSRF cho flow; `client_secret` chỉ ở server ([env-config.md](env-config.md)).

## Checklist an toàn

- [ ] Hash bằng bcrypt/argon2, không giới hạn độ dài mật khẩu kiểu "tối đa 20 ký tự".
- [ ] Cookie phiên: `HttpOnly; Secure; SameSite=Lax` ([security.md](security.md)).
- [ ] Thông báo lỗi đăng nhập **mơ hồ có chủ đích**: "email hoặc mật khẩu sai" — không tiết lộ email tồn tại hay không.
- [ ] Rate limit endpoint login/register ([api-security.md](api-security.md)) — chặn brute-force.
- [ ] Reset mật khẩu: token random 1 lần, hết hạn nhanh (15–60 phút), **vô hiệu mọi session cũ** sau khi đổi.
- [ ] Đổi mật khẩu/quyền → thu hồi session/refresh token đang sống.
