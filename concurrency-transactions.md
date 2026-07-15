# Concurrency & Transaction

Vì sao data sai khi 2 request tới cùng lúc — và các công cụ để nó không sai.

> Transaction cơ bản ở [sql-fundamentals.md](sql-fundamentals.md). File này về phần khó: nhiều việc xảy ra *đồng thời*.

## Race condition — bug chỉ xuất hiện khi đông người

```ts
// Kho còn 1 món. Hai request mua CÙNG LÚC:
const p = await db.product.findUnique({ where: { id } });   // cả 2 đọc: stock = 1
if (p.stock > 0) {                                          // cả 2 thấy > 0 ✓
  await db.product.update({ data: { stock: p.stock - 1 } });// cả 2 ghi stock = 0
}
// → bán được 2 món trong khi kho có 1. Test tay không bao giờ thấy bug này.
```

Khuôn chung của lỗi: **đọc → suy nghĩ → ghi** dựa trên giá trị đã đọc, trong khi kẻ khác chen vào giữa. Mọi giải pháp bên dưới đều nhằm làm cụm này **nguyên tử** (atomic).

## Giải pháp theo thang — từ nhẹ tới nặng

### 1. Đẩy phép toán xuống DB (đủ cho đa số case)

```sql
UPDATE products SET stock = stock - 1
WHERE id = $1 AND stock > 0
RETURNING stock;   -- 0 dòng bị ảnh hưởng = hết hàng, báo lỗi
```

Đọc-kiểm tra-ghi gộp thành **một câu lệnh** — DB tự xử lý tranh chấp. Nhanh, không lock thủ công.

### 2. Optimistic lock — kèm version, ghi đè thì thử lại

```sql
UPDATE products SET stock = 0, version = version + 1
WHERE id = $1 AND version = $2;   -- version đã đổi (ai đó sửa trước) → 0 dòng → retry
```

Hợp khi **hiếm khi va nhau** (form sửa profile, CMS): không ai phải chờ, va thì thử lại/báo "bản ghi đã bị sửa".

### 3. Pessimistic lock — khoá dòng, người sau xếp hàng

```sql
BEGIN;
SELECT * FROM products WHERE id = $1 FOR UPDATE;  -- dòng bị khoá tới hết transaction
-- kiểm tra + tính toán phức tạp...
UPDATE products SET stock = ... WHERE id = $1;
COMMIT;
```

Hợp khi **va nhau thường xuyên** hoặc logic giữa đọc-ghi phức tạp (chuyển tiền). Giá: giữ lock lâu là nghẽn — transaction phải ngắn, **không gọi API ngoài bên trong**.

### 4. Ngoài một DB — queue hoặc distributed lock

Việc đụng nhiều hệ thống (trừ kho + gọi cổng thanh toán) → đưa vào **queue xử lý tuần tự** ([queue-cron.md](queue-cron.md)) hoặc Redis lock (Redlock). Đừng với tới đây khi mục 1–3 còn giải được.

## Isolation level — transaction "nhìn thấy" gì của nhau

Mặc định Postgres/MySQL là **Read Committed**: chỉ thấy dữ liệu đã commit — nhưng 2 lần đọc trong cùng transaction vẫn có thể ra khác nhau (kẻ khác commit xen giữa). Mức cao hơn (`Repeatable Read`, `Serializable`) chặt hơn, đổi lại chậm hơn và có thể phải retry transaction bị huỷ.

Thực dụng: **giữ mặc định + dùng lock/atomic update đúng chỗ** (mục trên); chỉ nâng isolation khi hiểu rõ vì sao. Đừng nâng lên Serializable toàn cục để "cho chắc" — trả giá hiệu năng cả hệ thống.

## Idempotency — chạy lại không sai

Retry là chuyện thường (mạng chập chờn, webhook gọi lần 2, user bấm đúp). Thao tác ghi cần **chạy 2 lần mà kết quả như 1 lần**:

```ts
// Client gửi kèm khoá duy nhất cho "ý định" này
POST /orders
Idempotency-Key: order-attempt-8f3a...

// Server: khoá đã xử lý rồi → trả lại KẾT QUẢ CŨ, không tạo đơn mới
const existing = await db.idempotencyKey.findUnique({ where: { key } });
if (existing) return json(existing.response);
```

- DB hỗ trợ sẵn dạng đơn giản: `INSERT ... ON CONFLICT DO NOTHING` với unique constraint.
- Webhook handler ([payment.md](payment.md)) và job trong queue **bắt buộc** idempotent — chúng chắc chắn sẽ bị gọi lặp.

## Mẹo

- Nghi ngờ race condition? **Viết test bắn N request song song** (`Promise.all`) vào endpoint — bug hiện ra ngay thay vì chờ production đông khách.
- Unique constraint trong DB là "lưới cuối" rẻ nhất: 2 request cùng tạo `email` trùng → 1 cái fail sạch sẽ thay vì 2 bản ghi.
- Transaction ngắn, lock ít, **thứ tự lock cố định** (luôn khoá account id nhỏ trước) — tránh deadlock.
- Deadlock vẫn xảy ra được → code phải sẵn sàng **retry** transaction bị DB huỷ.
- Số đếm/số dư: đừng đọc-cộng-ghi từ app; dùng `UPDATE ... SET x = x + 1` hoặc event + tính lại.
