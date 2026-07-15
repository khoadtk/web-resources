# Message queue & Event

Khái niệm nhắn tin bất đồng bộ giữa các phần của hệ thống — nền trước khi học Kafka/RabbitMQ.

> Cách chạy background job cụ thể ở [queue-cron.md](queue-cron.md). File này là lớp khái niệm bên dưới.

## Vì sao cần?

Gọi trực tiếp (HTTP/gRPC) nghĩa là: bên kia phải **sống, nhanh, và thành công ngay lúc đó**. Queue tách đôi ràng buộc này:

```
[Order service] --đăng "OrderCreated"--> [QUEUE] --> [Email service] (xử lý khi rảnh)
                                                 --> [Inventory service]
```

- **Producer không chờ** consumer — trả response cho user ngay.
- Consumer chết 5 phút? Message **nằm chờ trong queue**, sống lại xử lý tiếp — không mất việc.
- Lưu lượng dội đột ngột → queue phình ra hấp thụ, consumer xử lý theo sức mình (buffer).

## Hai mô hình — phân biệt cho rõ

| | Queue (point-to-point) | Pub/Sub (fan-out) |
| --- | --- | --- |
| Một message tới | **1 consumer** (chia việc) | **mọi subscriber** (loan tin) |
| Câu hỏi trong đầu | "việc này ai đó làm giùm" | "chuyện này xảy ra, ai quan tâm thì nghe" |
| Ví dụ | resize ảnh, gửi mail | OrderCreated → email + kho + analytics |
| Công cụ điển hình | RabbitMQ queue, BullMQ, SQS | Kafka topic, Redis pub/sub, SNS |

Kafka gộp cả hai: nhiều **consumer group** cùng đọc 1 topic (pub/sub giữa các group), trong 1 group thì chia nhau (queue).

## Ngữ nghĩa giao hàng — cái bẫy phải hiểu trước tiên

- **At-most-once** — gửi rồi quên: có thể **mất** message, không bao giờ lặp.
- **At-least-once** — retry tới khi consumer xác nhận (ack): không mất, nhưng **có thể lặp** ← mặc định thực tế của mọi hệ thống nghiêm túc.
- **Exactly-once** — nghe hứa hẹn nhưng gần như không tồn tại end-to-end; thứ bạn thật sự làm là *at-least-once + consumer idempotent*.

→ Hệ quả bắt buộc: **consumer phải idempotent** — nhận cùng message 2 lần, kết quả như 1 lần (xem [concurrency-transactions.md](concurrency-transactions.md)). Đây là quy tắc số 1 của cả chủ đề này.

## Ack, retry, dead letter

```
Consumer nhận message → xử lý → ACK (xong, xoá khỏi queue)
                     ↘ lỗi/timeout → message quay lại queue → RETRY (kèm backoff)
                        ↘ fail quá N lần → DEAD LETTER QUEUE (DLQ)
```

- **Ack sau khi xử lý xong**, không phải lúc vừa nhận — ack sớm thì consumer chết giữa chừng là mất việc.
- **DLQ** = "phòng cấp cứu": message hỏng (bug, data xấu) được cách ly để người xem, thay vì retry vô hạn chặn cả queue. Queue nào không có DLQ thì một message độc làm nghẽn mọi thứ phía sau ("poison message").
- Retry có **backoff + jitter** (1s, 4s, 16s...), đừng nện lại ngay lập tức.

## Ordering — thứ tự không miễn phí

- Queue có nhiều consumer chạy song song → **không đảm bảo thứ tự** xử lý.
- Cần thứ tự theo thực thể (các event của *cùng 1 đơn hàng* phải theo thứ tự) → dùng **partition key** (Kafka: cùng key vào cùng partition, 1 consumer đọc tuần tự). Thứ tự *toàn cục* thì gần như từ bỏ scale — hầu như không ai cần thật.

## Outbox pattern — gửi event & ghi DB không lệch nhau

Bug kinh điển: ghi DB xong, **crash trước khi kịp đăng event** (hoặc ngược lại) → hệ thống lệch nhau.

```
Trong CÙNG transaction: INSERT đơn hàng + INSERT event vào bảng "outbox"
→ một worker riêng đọc outbox, đăng lên queue, đánh dấu đã gửi
```

DB transaction đảm bảo "đơn + event" sống chết cùng nhau; worker retry tới khi đăng được (at-least-once → consumer idempotent lo phần lặp).

## Khi nào ĐỪNG dùng queue

- Cần **kết quả ngay trong response** (tính giá, validate) → gọi đồng bộ, queue chỉ thêm độ trễ và độ phức tạp.
- App nhỏ 1 service, việc nền đơn giản → BullMQ/QStash ([queue-cron.md](queue-cron.md)) là đủ, chưa cần Kafka. Kafka trả giá bằng vận hành (cluster, partition, rebalancing) — đáng khi có *nhiều* consumer/team và lưu lượng lớn.
- Nợ "đọc lại lịch sử event" (event sourcing, replay) → lúc đó Kafka mới thật sự khác biệt so với queue thường (log giữ lại theo retention, không xoá khi đọc).

## Mẹo

- Message nên là **sự kiện đã xảy ra** (`OrderCreated`) kèm đủ data, không phải mệnh lệnh mơ hồ — consumer mới tự xử được, khỏi gọi ngược.
- Đánh **version cho schema message** — producer nâng cấp mà consumer cũ vẫn đọc được (tolerant reader, giống [rest-api-design.md](rest-api-design.md)).
- Theo dõi **độ dài queue + tuổi message già nhất** ([logging-observability.md](logging-observability.md)) — queue phình liên tục nghĩa là consumer hụt hơi.
- Truyền `requestId` trong message để lần được dấu vết xuyên hệ thống.
