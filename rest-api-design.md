# REST API design

Thiết kế API để người khác (và chính bạn 6 tháng sau) dùng không phải đoán.

> Nền tảng HTTP ở [http-fundamentals.md](http-fundamentals.md); cách gọi từ client ở [api-layer.md](api-layer.md).

## Đặt tên endpoint

```
GET    /products              # danh sách
POST   /products              # tạo
GET    /products/42           # chi tiết
PATCH  /products/42           # sửa một phần
DELETE /products/42           # xoá
GET    /products/42/reviews   # resource lồng nhau (1 cấp là đủ)
```

Quy tắc:

- **Danh từ số nhiều**, không động từ: `/products`, không phải `/getProducts` (động từ đã nằm ở method).
- Hành động không map được vào CRUD → dùng "sub-action" rõ ràng: `POST /orders/42/cancel`.
- Lồng tối đa **1 cấp** — `/users/1/orders/2/items/3` là mùi thiết kế xấu; item có id riêng thì `/order-items/3`.

## Pagination — chọn 1 trong 2

```
GET /products?page=2&limit=20          # offset-based: đơn giản, nhảy trang được
GET /products?cursor=abc123&limit=20   # cursor-based: ổn định khi data đổi, hợp infinite scroll
```

Response nên kèm metadata:

```json
{ "items": [...], "total": 512, "nextCursor": "def456" }
```

Offset chậm dần với bảng lớn (`OFFSET 100000` phải đếm bỏ 100k dòng — xem [sql-fundamentals.md](sql-fundamentals.md)); data hay thêm/xoá thì cursor tránh trùng/lọt item.

## Filter, sort, chọn field

```
GET /products?status=active&sort=-createdAt&fields=id,name,price
```

- Filter = query param theo tên field; `-` prefix cho sort giảm dần là convention phổ biến.
- Đừng chế endpoint mới cho mỗi kiểu lọc (`/active-products`) — param hoá.

## Định dạng lỗi thống nhất

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Email không hợp lệ",
    "details": [{ "field": "email", "issue": "invalid_format" }]
  }
}
```

- **`code` cho máy** (client switch theo nó, không parse message), **`message` cho người**.
- Cùng một khuôn lỗi cho **mọi** endpoint — client viết 1 hàm xử lý là xong.
- Validation lỗi nhiều field → trả **hết một lượt** trong `details`, đừng bắt client sửa từng cái.

## Versioning

```
GET /v1/products      # đường dẫn — dễ thấy, dễ route (phổ biến nhất)
```

- Chỉ tăng version khi **breaking change** (đổi/xoá field, đổi ngữ nghĩa). Thêm field mới không cần.
- Tránh breaking ngay từ đầu: client phải **bỏ qua field lạ** (tolerant reader).

## Tài liệu hoá — OpenAPI

Mô tả API bằng **OpenAPI (Swagger)** — 1 file YAML/JSON sinh ra được docs UI, client SDK, mock server:

```yaml
paths:
  /products/{id}:
    get:
      parameters: [{ name: id, in: path, required: true, schema: { type: integer } }]
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Product" } } } }
```

- Viết schema trước rồi code theo (design-first), hoặc sinh từ code (NestJS/Fastify có decorator/plugin sẵn).
- Docs UI: Swagger UI, Redoc, [Scalar](https://scalar.com).

## Mẹo

- **Nhất quán > "đúng chuẩn"**: chọn convention (snake_case hay camelCase, cấu trúc lỗi...) rồi giữ nguyên toàn API — dev dùng API học 1 lần là đoán được phần còn lại.
- Response list **luôn là object bọc ngoài** (`{ items: [] }`), đừng trả mảng trần — sau này thêm `total`/`cursor` không breaking.
- Timestamps: **ISO 8601 UTC** (`2026-07-16T03:00:00Z`) — xem [date-time.md](date-time.md).
- Validate input ở server bằng schema ([forms-validation.md](forms-validation.md) — Zod dùng chung được cho cả docs + validate).
- Đừng expose id tự tăng nếu ngại lộ quy mô (`/orders/17` → biết bạn có 17 đơn) — dùng UUID/nanoid.
