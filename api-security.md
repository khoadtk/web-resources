# API security

Bảo vệ API — phần nối dài của [security.md](security.md) (vốn nghiêng FE) sang phía server.

## Nguyên tắc gốc: đừng tin bất kỳ thứ gì client gửi

Client "của bạn" chỉ là một trong các client — ai cũng gọi API của bạn được bằng curl, với bất kỳ payload nào, bỏ qua mọi validate/ẩn nút trên UI. Mọi kiểm tra trên FE chỉ là UX; **kiểm tra thật nằm ở server**.

## Validate input bằng schema — ở biên, một lần

```ts
import { z } from 'zod';

const CreateOrder = z.object({
  productId: z.string().uuid(),
  quantity: z.number().int().min(1).max(100),
}).strict();   // .strict(): field lạ là lỗi — chặn kẻ gửi thêm field bậy

const parsed = CreateOrder.safeParse(await req.json());
if (!parsed.success) return json(422, { error: ... });
```

- **Allowlist, không blocklist**: khai báo cái được phép; đừng ngồi liệt kê cái cấm.
- Bẫy kinh điển — **mass assignment**: `db.user.update({ data: req.body })` → client gửi `{ "role": "admin" }` là lên admin. Luôn pick field rõ ràng, `.strict()` giúp chặn từ lớp validate.

## Authorization — lỗi bị khai thác nhiều nhất (BOLA/IDOR)

```ts
// ❌ đã đăng nhập là xem được MỌI đơn — chỉ cần đoán id
const order = await db.order.findUnique({ where: { id: params.id } });

// ✅ kiểm tra QUYỀN SỞ HỮU trên từng resource
const order = await db.order.findFirst({
  where: { id: params.id, userId: session.userId },
});
if (!order) return json(404);
```

Authentication trả lời "bạn là ai", nhưng từng endpoint vẫn phải hỏi "**bạn có được đụng resource NÀY không**". Đây là lỗi số 1 trong OWASP API Top 10. Trả `404` thay vì `403` khi không sở hữu — không xác nhận resource tồn tại.

## Rate limiting — chặn brute-force & lạm dụng

- Giới hạn theo **IP** (chống dò), theo **user** (chống lạm dụng), chặt hơn hẳn cho **login / gửi mail / gọi AI** (đắt tiền).
- Trả `429` kèm header `Retry-After`.
- Node: `rate-limiter-flexible` + Redis (nhiều instance phải đếm chung); serverless: Upstash Ratelimit; hoặc chặn từ tầng proxy/Cloudflare ([networking-basics.md](networking-basics.md)).

```ts
// ý tưởng: sliding window theo key
const key = `login:${ip}`;
if (await limiter.isOverLimit(key, { max: 5, windowSec: 60 })) return json(429);
```

## Chống lộ dữ liệu thừa

- **Response cũng cần "validate ra"**: trả DTO/schema rõ ràng, đừng `return user` nguyên bản ghi DB (lộ `passwordHash`, field nội bộ).
- Lỗi 500: trả message chung chung cho client, chi tiết + stack chỉ ghi vào log ([logging-observability.md](logging-observability.md)). Stack trace trong response = bản đồ tặng attacker.
- Tắt header khoe công nghệ (`X-Powered-By: Express`).

## Secrets & API key

- Secret chỉ ở server, xoay được, không commit ([env-config.md](env-config.md)).
- Cấp key cho bên ngoài gọi API của bạn → lưu **hash của key** (như mật khẩu), hiện key đúng 1 lần lúc tạo; gắn **scope + rate limit** theo key.
- Nhận **webhook** từ bên ngoài (Stripe...): verify chữ ký, xử lý idempotent ([payment.md](payment.md), [concurrency-transactions.md](concurrency-transactions.md)).

## OWASP API Top 10 — 5 cái gặp nhiều nhất

| # | Lỗi | Chống bằng |
| --- | --- | --- |
| 1 | BOLA/IDOR — xem resource người khác qua id | Check ownership từng resource (ở trên) |
| 2 | Broken authentication | [auth-server.md](auth-server.md): hash đúng, token ngắn, rate limit login |
| 3 | Lộ data thừa trong response | DTO rõ ràng, không trả raw record |
| 4 | Thiếu rate limit | Mục rate limiting |
| 5 | Mass assignment | `.strict()` + pick field |

## Checklist

- [ ] Mọi input qua schema validate (`.strict()`), mọi query DB tham số hoá ([sql-fundamentals.md](sql-fundamentals.md)).
- [ ] Mọi endpoint có resource theo id → check ownership/quyền, không chỉ check đăng nhập.
- [ ] Rate limit: mặc định toàn API + chặt riêng login/OTP/mail/AI.
- [ ] Response qua DTO; lỗi 500 không lộ stack; tắt `X-Powered-By`.
- [ ] HTTPS bắt buộc; CORS chỉ mở cho origin của mình ([networking-basics.md](networking-basics.md)).
- [ ] Log sự kiện auth thất bại (đăng nhập sai liên tục, 403 dồn dập) để phát hiện tấn công ([logging-observability.md](logging-observability.md)).
