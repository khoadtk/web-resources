# Logging & Observability (BE)

Khi production có sự cố lúc 2h sáng, thứ duy nhất bạn có là log và metrics — nếu đã ghi đúng.

> Phía frontend (Sentry, error boundary) ở [error-monitoring.md](error-monitoring.md). File này là phía server.

## 3 trụ của observability

- **Logs** — chuyện gì đã xảy ra, từng sự kiện ("user 42 tạo đơn #17 thất bại: hết hàng").
- **Metrics** — con số theo thời gian (request/giây, p95 latency, % lỗi, RAM).
- **Traces** — 1 request đi qua những đâu, mỗi chặng tốn bao lâu (API → DB → gọi service khác).

App nhỏ: làm tử tế **logs** trước; metrics và traces thêm khi hệ thống phình ra.

## Structured logging — log là JSON, không phải câu văn

```ts
// ❌ log kiểu văn xuôi — chỉ người đọc được, không lọc được
console.log(`User ${userId} checkout failed: ${err.message}`);

// ✅ structured — máy lọc/đếm/cảnh báo được
logger.error({ event: 'checkout_failed', userId, orderId, err: err.message });
```

Node: dùng **pino** (nhanh, JSON mặc định) — dev thì thêm `pino-pretty` cho dễ đọc:

```ts
import pino from 'pino';
const logger = pino();
logger.info({ event: 'order_created', orderId, userId, total });
```

### Log level — dùng đúng tầng

- `error` — hỏng thật, cần người xử lý. `warn` — bất thường nhưng tự vượt qua (retry thành công).
- `info` — sự kiện nghiệp vụ chính (đơn tạo, user đăng ký). `debug` — chi tiết dò bug, tắt ở production.
- Production chạy mức `info`; điều tra sự cố mới hạ xuống `debug` (qua env — [env-config.md](env-config.md)).

## Request ID — sợi chỉ xuyên suốt

Mỗi request vào hệ thống gán 1 id, **mọi dòng log của request đó đều mang id này**:

```ts
// middleware (mọi framework đều có chỗ tương đương)
const requestId = req.headers['x-request-id'] ?? crypto.randomUUID();
// gắn vào logger con: mọi log sau đó tự kèm requestId
const log = logger.child({ requestId });
```

User báo "lúc 14:03 em bấm thanh toán bị lỗi" → tìm 1 request id → thấy **toàn bộ hành trình** của đúng request đó, kể cả khi nó đi qua nhiều service (truyền tiếp header `X-Request-Id`).

## Health check — cho máy hỏi "mày sống không?"

```
GET /healthz          → 200 {"status":"ok"}                    (liveness: process sống)
GET /readyz           → kiểm tra DB/Redis nối được rồi mới 200  (readiness: sẵn sàng nhận traffic)
```

Load balancer/Docker/K8s gọi endpoint này để quyết định gửi traffic hay khởi động lại app. Phân biệt 2 loại: app sống nhưng DB đứt → liveness OK, readiness FAIL → tạm rút khỏi load balancer thay vì restart vô ích.

## Metrics tối thiểu đáng theo dõi

- **Rate** — request/giây. **Errors** — % 5xx. **Duration** — latency p50/p95/p99 (nhìn **p95/p99**, đừng nhìn average — average che mất 5% user khổ nhất).
- Cộng thêm: kết nối DB đang mở, độ dài queue ([queue-cron.md](queue-cron.md)), RAM/CPU.
- Stack phổ biến: **Prometheus + Grafana** (self-host), hoặc dùng sẵn của nền tảng (Vercel/Railway metrics). Trace phân tán khi nhiều service: **OpenTelemetry**.

## Mẹo

- **Đừng log secret/PII**: mật khẩu, token, số thẻ — lọc trước khi ghi ([security.md](security.md)). Log là chỗ rò dữ liệu bị bỏ quên nhiều nhất.
- Log ra **stdout**, để hạ tầng (Docker, nền tảng deploy) gom — đừng tự viết file + xoay file trong app.
- `error` phải kèm **stack trace + context** (input gây lỗi) — "Error: failed" trơ trọi bằng không có.
- Đặt **alert theo triệu chứng user thấy** (% lỗi tăng, p95 vọt) chứ đừng alert mọi thứ — alert kêu suốt ngày = không ai đọc alert nữa.
- Sự kiện nghiệp vụ quan trọng (thanh toán!) log ở mức `info` có cấu trúc — chính là data điều tra + đối soát sau này.
