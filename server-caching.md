# Server caching sâu

Cache phía server: các pattern, và 2 bài toán khó thật sự — invalidation & thundering herd.

> Tổng quan các tầng cache (browser/CDN/Redis) ở [caching.md](caching.md). File này đào sâu tầng app/server.

## 3 pattern đọc/ghi cache

### Cache-aside (lazy) — phổ biến nhất

```ts
async function getProduct(id: string) {
  const hit = await redis.get(`product:${id}`);
  if (hit) return JSON.parse(hit);                       // cache hit

  const data = await db.product.findUnique({ where: { id } });  // miss → DB
  await redis.set(`product:${id}`, JSON.stringify(data), { EX: 300 });
  return data;
}
```

App tự quản: đọc cache → miss thì đọc DB rồi bỏ vào cache. Đơn giản, cache hỏng thì app vẫn chạy (chậm hơn). Nhược: request đầu sau khi expire luôn chậm (cold).

### Write-through / Write-behind

- **Write-through**: ghi DB **và** cache trong cùng thao tác → cache luôn tươi, giá là mỗi lần ghi chậm hơn.
- **Write-behind**: ghi cache trước, dồn ghi DB sau — nhanh nhưng rủi ro mất data khi cache sập; hiếm khi đáng dùng cho app thường.

Thực dụng: **cache-aside cho đọc + xoá key khi ghi** (invalidate-on-write) là combo đủ cho đa số app.

## Cache invalidation — "one of the two hard things"

Chiến lược theo thứ tự nên dùng:

1. **TTL ngắn** — chấp nhận cũ tối đa N giây. Rẻ nhất, đủ cho phần lớn data (danh mục, config). Đặt TTL là *quyết định nghiệp vụ*: "giá hiển thị được phép cũ bao lâu?"
2. **Xoá key khi ghi** — `DELETE product:42` ngay trong luồng update. Xoá (để lần đọc sau tự nạp) an toàn hơn *cập nhật* cache (dễ race: 2 luồng ghi cache chéo thứ tự → cache giữ bản cũ vĩnh viễn).
3. **Tag/nhóm key** — một thay đổi ảnh hưởng nhiều key (sửa category → mọi list chứa nó): gom key theo tag rồi xoá theo tag (Next.js `revalidateTag` là đúng ý tưởng này).

Bẫy hay gặp: **xoá cache rồi transaction rollback** → cache trống nhưng lát sau nạp lại data cũ — vẫn đúng! Ngược lại mới chết: *cập nhật* cache trước khi commit. Quy tắc: **invalidate sau khi commit thành công**.

## Thundering herd (cache stampede)

Key hot hết hạn → **1000 request cùng miss cùng lúc** → 1000 query giống hệt nhau đập vào DB → DB quỵ đúng lúc đông nhất.

### Chống — 3 kỹ thuật

```ts
// 1. Lock đơn giản: chỉ MỘT request đi nạp, số còn lại chờ/nhận bản cũ
const lock = await redis.set(`lock:${key}`, '1', { NX: true, EX: 10 });
if (!lock) { await sleep(100); return getCached(key); }  // người khác đang nạp
```

- **2. Stale-while-revalidate**: hết hạn thì vẫn trả bản cũ ngay + nạp lại ở nền (đúng cơ chế ISR trong bài rendering Next.js) — user không bao giờ chờ.
- **3. Jitter TTL**: `EX: 300 + random(0, 60)` — nghìn key không hết hạn *cùng một giây* sau lần deploy/warm-up.

## Cache gì, đừng cache gì

| Nên | Đừng |
| --- | --- |
| Data đọc nhiều ghi ít (danh mục, config, profile công khai) | Data riêng tư nhạy cảm mà key dễ nhầm (cache lẫn user!) |
| Kết quả tính toán đắt (aggregate, report) | Data cần đúng tuyệt đối từng giây (số dư khi giao dịch) |
| Response API bên ngoài chậm/giới hạn quota | Thứ query DB đã nhanh sẵn (<5ms) — thêm cache chỉ thêm bug |

Bug đáng sợ nhất của caching không phải chậm mà là **trả nhầm data của người khác** — key phải chứa đủ chiều (`user:{id}:orders`, kèm cả locale/version nếu response phụ thuộc).

## Mẹo

- **Đo hit rate** ([logging-observability.md](logging-observability.md)) — cache hit < 50% thường là TTL quá ngắn hoặc key quá phân mảnh; cache không ai hit là code chết.
- Prefix key theo domain + version schema: `v2:product:42` — đổi cấu trúc data thì đổi prefix, khỏi xoá tay.
- Redis sập không được làm app sập: bọc try/catch, coi như miss → về DB (graceful degradation).
- Đừng cache-hoá mọi thứ ngay từ đầu — thêm cache **sau khi profiling** chỉ ra query nào đắt ([performance.md](performance.md) tinh thần tương tự phía FE).
