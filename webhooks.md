# Webhooks design

Webhook = "API ngược": thay vì bạn hỏi (polling), bên kia **gọi bạn** khi có chuyện. File này gộp cả hai vai: **nhận** webhook (Stripe gọi bạn) và **gửi** webhook (bạn gọi hệ thống của khách).

## Nhận webhook cho đúng

### 1. Verify chữ ký — ai gọi cũng được, nên phải kiểm

Endpoint webhook là URL công khai — kẻ xấu POST payload giả được. Nhà cung cấp ký payload bằng secret; bạn verify:

```ts
// kiểu Stripe/GitHub: HMAC của raw body
import crypto from 'node:crypto';

const expected = crypto.createHmac('sha256', process.env.WEBHOOK_SECRET!)
  .update(rawBody)                         // RAW body — chưa qua JSON.parse!
  .digest('hex');
if (!crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expected))) return json(401);
```

Bẫy số 1: framework parse JSON trước → body bị format lại → chữ ký sai. Route webhook phải đọc **raw body** (Express: `express.raw()`; Next.js Route Handler: `await req.text()`).

### 2. Trả 200 nhanh, xử lý sau

Bên gửi thường timeout 5–10s và sẽ **retry nếu không nhận 2xx**. Xử lý nặng trong handler → timeout → bị gọi lại → xử lý đúp.

```
Nhận → verify chữ ký → LƯU event vào DB/queue → trả 200 ngay
                          ↓
              worker xử lý thật ở nền (queue-cron.md)
```

### 3. Idempotent — chắc chắn sẽ nhận trùng

Retry + at-least-once ([message-queue.md](message-queue.md)) → cùng event tới ≥2 lần là chuyện thường:

```sql
INSERT INTO webhook_events (event_id, payload) VALUES ($1, $2)
ON CONFLICT (event_id) DO NOTHING;   -- event_id nhà cung cấp gửi kèm; trùng thì bỏ qua
```

### 4. Đừng tin thứ tự

`order.updated` có thể tới **trước** `order.created` (retry, mạng). Xử lý theo **trạng thái hiện tại** (gọi API bên kia đọc bản mới nhất, hoặc so timestamp) thay vì áp event mù quáng theo thứ tự nhận.

## Gửi webhook cho tử tế (khi bạn là nhà cung cấp)

Ngược lại tất cả điều trên, cộng thêm:

- **Ký payload**: gửi kèm `X-Signature: hmac-sha256(secret, body)` + timestamp (chống replay: từ chối chữ ký quá 5 phút tuổi).
- **Retry có backoff**: 2xx là xong; còn lại retry lịch thưa dần (1m, 5m, 30m, 2h...) tối đa vài ngày; gửi qua queue chứ đừng gửi inline trong request user.
- **Event có `id` duy nhất + `type` + `created_at`** — để bên nhận làm idempotency.
- Cho user **xem log + bấm gửi lại** event trên dashboard — tính năng được yêu quý nhất của mọi hệ thống webhook.
- Vô hiệu endpoint fail liên tục nhiều ngày (kèm email báo) — đừng retry vào hư không mãi mãi.

## Test webhook khi dev

- **Đưa localhost ra internet**: `ngrok http 3000` / `cloudflared tunnel` — lấy URL tạm đăng ký với nhà cung cấp.
- Stripe có CLI giả lập: `stripe listen --forward-to localhost:3000/api/webhook` + `stripe trigger payment_intent.succeeded`.
- Viết test bắn payload thật (lưu từ log) vào handler — kèm case chữ ký sai, event trùng, event sai thứ tự.

## Checklist

- [ ] Verify chữ ký trên **raw body**, so bằng `timingSafeEqual`.
- [ ] Trả 2xx nhanh; xử lý thật ở worker/queue.
- [ ] Idempotent theo `event_id` (unique constraint).
- [ ] Không giả định thứ tự event.
- [ ] Secret webhook riêng biệt, xoay được ([env-config.md](env-config.md)).
- [ ] Log mọi event nhận được kèm kết quả xử lý ([logging-observability.md](logging-observability.md)) — đối soát khi khách kêu "sao đơn chưa cập nhật".
- [ ] Gửi đi: ký + retry backoff + dashboard xem lại ([payment.md](payment.md) là ví dụ phía nhận).
