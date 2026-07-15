# HTTP fundamentals

Nền của mọi thứ backend: request/response thực sự trông như thế nào.

## Một request/response thật

```
GET /api/products?page=2 HTTP/1.1        ← method + path + version
Host: shop.com
Accept: application/json                  ← headers
Cookie: session=abc123

HTTP/1.1 200 OK                           ← status line
Content-Type: application/json
Cache-Control: max-age=60

{"items": [...]}                          ← body
```

HTTP là **text + stateless**: server không tự nhớ gì giữa 2 request — mọi "trạng thái" (đăng nhập, giỏ hàng) phải gửi kèm mỗi lần (cookie/token) hoặc lưu ở DB.

## Methods — ngữ nghĩa thật sự

| Method | Nghĩa | Idempotent? |
| --- | --- | --- |
| GET | Đọc, không đổi gì | ✅ |
| POST | Tạo mới / hành động | ❌ |
| PUT | Thay thế toàn bộ resource | ✅ |
| PATCH | Sửa một phần | ❌ (thường) |
| DELETE | Xoá | ✅ |

**Idempotent** = gọi 10 lần kết quả như gọi 1 lần. Quan trọng vì: client/proxy được phép **retry** request idempotent khi mạng chập chờn; POST retry là ra 2 đơn hàng (vì vậy mới cần idempotency key trong [payment.md](payment.md)).

## Status codes — nhóm là đủ nhớ

- **2xx thành công**: `200` OK, `201` đã tạo, `204` xong-không-có-body.
- **3xx chuyển hướng**: `301` chuyển vĩnh viễn, `302` tạm, `304` chưa đổi (cache).
- **4xx lỗi phía client**: `400` request sai, `401` chưa đăng nhập, `403` không có quyền, `404` không có, `409` xung đột, `422` data không hợp lệ, `429` gọi quá nhiều.
- **5xx lỗi phía server**: `500` server toang, `502/504` proxy không gọi được app phía sau.

Cặp hay nhầm: **401 = "bạn là ai?"** (thiếu/sai đăng nhập) vs **403 = "biết bạn là ai, nhưng không được phép"**.

## Headers đáng biết

- `Content-Type` — body là gì (`application/json`, `multipart/form-data`).
- `Authorization: Bearer <token>` — chuẩn gửi token.
- `Cache-Control`, `ETag` — cache (xem [caching.md](caching.md)).
- `Set-Cookie` với `HttpOnly; Secure; SameSite` — xem [security.md](security.md).
- `X-Request-Id` — dán mã theo dõi vào request để lần log (xem [logging-observability.md](logging-observability.md)).

## Thử bằng tay — curl

```bash
curl -i https://api.example.com/products          # -i: xem cả headers
curl -X POST https://api.example.com/orders \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $TOKEN" \
     -d '{"productId": 1}'
```

Debug API mà chưa biết curl (hoặc Postman/Bruno) thì mọi thứ thành đoán mò.

## HTTP/1.1 vs 2 vs 3 (đủ dùng)

- **1.1** — mỗi kết nối 1 request một lúc → trình duyệt phải mở nhiều kết nối.
- **2** — multiplex nhiều request trên 1 kết nối, header nén. Phổ biến hiện nay.
- **3 (QUIC)** — chạy trên UDP, bắt tay nhanh hơn, tốt cho mạng di động.

Là dev app, bạn hầu như không phải làm gì — server/CDN lo; chỉ cần biết để đọc hiểu.

## Mẹo

- Trả **đúng status code** — client (và [api-layer.md](api-layer.md)) xử lý theo code, không theo message.
- Body của lỗi cũng nên là **JSON có cấu trúc** (`{ "error": { "code": "...", "message": "..." } }`) — đừng trả HTML lỗi cho API.
- GET **không được có side effect** — crawler/prefetch có thể gọi GET bất cứ lúc nào.
- Mở DevTools → tab **Network** và đọc từng request là cách học HTTP nhanh nhất.
