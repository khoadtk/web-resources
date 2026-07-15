# SQL fundamentals

ORM che SQL đi, nhưng lúc chậm/sai thì phải đọc được SQL mới cứu được.

> ORM & chọn database ở [database-orm.md](database-orm.md). File này là phần nền bên dưới.

## Truy vấn cốt lõi

```sql
SELECT name, price FROM products
WHERE status = 'active' AND price > 100
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;

-- JOIN: ghép bảng theo khoá
SELECT o.id, u.email
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE o.total > 500;

-- GROUP BY: gom nhóm + tính
SELECT user_id, COUNT(*) AS order_count, SUM(total) AS revenue
FROM orders
GROUP BY user_id
HAVING SUM(total) > 1000;   -- lọc SAU khi gom (WHERE là lọc TRƯỚC)
```

`INNER JOIN` chỉ lấy dòng khớp cả 2 bên; `LEFT JOIN` giữ hết bên trái, bên phải không khớp thì NULL — đếm "user chưa có đơn nào" là việc của LEFT JOIN.

## Index — thứ quyết định nhanh hay chậm

Index như **mục lục sách**: không có thì mỗi query là lật từng trang (full table scan).

```sql
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at); -- composite
```

- Đánh index cột hay xuất hiện trong **WHERE / JOIN / ORDER BY**.
- Composite index dùng được từ **trái sang**: index `(user_id, created_at)` phục vụ được lọc theo `user_id`, nhưng không phục vụ lọc chỉ theo `created_at`.
- Index không miễn phí: mỗi INSERT/UPDATE phải cập nhật index → đừng rải bừa.
- Kiểm tra query có dùng index không:

```sql
EXPLAIN ANALYZE SELECT ... ;   -- thấy "Seq Scan" trên bảng lớn = thiếu index
```

## Transaction — tất cả hoặc không gì cả

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;    -- lỗi giữa chừng thì ROLLBACK: không ai mất tiền lơ lửng
```

- Bất kỳ chuỗi thao tác nào "phải đi cùng nhau" (trừ kho + tạo đơn) → bọc transaction.
- Giữ transaction **ngắn** — đừng gọi API ngoài/chờ user bên trong transaction (giữ lock lâu, nghẽn cả bảng).

## N+1 — bug hiệu năng kinh điển nhất

```ts
const users = await db.user.findMany();          // 1 query
for (const u of users) {
  u.orders = await db.order.findMany({ where: { userId: u.id } }); // +N query!
}
```

100 user = **101 query**. Chữa: JOIN hoặc bảo ORM load kèm (`include`/`with`) — thành 1–2 query. Dấu hiệu nhận biết: bật log query của ORM, thấy hàng loạt query giống hệt nhau chỉ khác id.

## Ràng buộc — để DB tự bảo vệ data

```sql
CREATE TABLE orders (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id    BIGINT NOT NULL REFERENCES users(id),   -- foreign key
  email      TEXT UNIQUE,                             -- không trùng
  total      NUMERIC NOT NULL CHECK (total >= 0),     -- không âm
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Validate ở app vẫn cần, nhưng **ràng buộc DB là hàng rào cuối** — app có bug thì data vẫn không nát. `TIMESTAMPTZ` + UTC cho thời gian (xem [date-time.md](date-time.md)); `NUMERIC` cho tiền (FLOAT làm tròn sai).

## Mẹo

- **Không bao giờ nối chuỗi SQL từ input** — parameterized query/ORM (SQL injection, xem [security.md](security.md)).
- `SELECT *` tiện lúc dev, tốn lúc chạy thật — lấy đúng cột cần, nhất là bảng có cột to (text/json).
- OFFSET lớn chậm → phân trang cursor (`WHERE created_at < $last ORDER BY created_at DESC LIMIT 20`, xem [rest-api-design.md](rest-api-design.md)).
- Học nhanh nhất: mở `psql`/TablePlus, bật log query của ORM lên và đọc SQL nó sinh ra.
- Soft delete (`deleted_at`) thay vì DELETE thật khi data quan trọng — nhớ thêm `WHERE deleted_at IS NULL` vào index/query.
