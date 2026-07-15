# Queue & Cron (Background jobs)

Chạy việc nền: gửi mail, xử lý ảnh, việc định kỳ — không bắt user chờ.

## Khi nào cần

- **Việc chậm** (gửi mail, resize ảnh, gọi AI) → đẩy vào **queue**, trả response ngay.
- **Việc định kỳ** (dọn dữ liệu, gửi report hằng ngày) → **cron**.
- **Việc nặng/nhiều** → tách **worker** xử lý dần.

## Công cụ

- **[BullMQ](https://docs.bullmq.io)** — queue trên Redis cho Node (mạnh, phổ biến)
- **[Upstash QStash](https://upstash.com/qstash)** — queue/cron serverless (hợp Vercel, không cần worker riêng)
- **[Inngest](https://inngest.com)** — background job + workflow, dev experience tốt
- **[Trigger.dev](https://trigger.dev)** — job dài, có UI theo dõi
- **Vercel Cron** / **GitHub Actions** — chạy định kỳ đơn giản

## Cách dùng

### HTML tĩnh / SPA

Frontend **không chạy background job** — chỉ **gọi API** để kích hoạt, rồi hiển thị trạng thái:

```js
await fetch('/api/export', { method: 'POST' }); // server đẩy vào queue
// poll trạng thái hoặc chờ realtime báo xong (xem realtime.md)
```

### Next.js

- **Cron**: khai báo trong `vercel.json` → Vercel gọi 1 Route Handler định kỳ:

```json
{ "crons": [{ "path": "/api/cron/daily", "schedule": "0 0 * * *" }] }
```

- **Queue**: serverless không giữ worker lâu → dùng **QStash/Inngest** (chúng gọi lại endpoint của bạn để xử lý job).

### Có server riêng (Node backend)

**BullMQ** + Redis: 1 process bỏ job vào queue, 1 worker xử lý:

```js
// thêm job
await queue.add('sendEmail', { to: 'a@b.com' });

// worker
new Worker('emails', async job => { await sendEmail(job.data); });
```

## Mẹo

- **Idempotent**: job có thể chạy lại → thiết kế để chạy 2 lần không sai (giống webhook [payment.md](payment.md)).
- **Retry + backoff** cho job lỗi (mạng chập chờn).
- **Cron syntax**: `phút giờ ngày tháng thứ` → dùng [crontab.guru](https://crontab.guru) để đọc.
- Serverless (Vercel) không hợp worker chạy lâu → chọn QStash/Inngest thay vì tự dựng BullMQ.
- Ghi log + theo dõi job (xem [error-monitoring.md](error-monitoring.md)).
