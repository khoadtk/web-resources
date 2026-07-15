# Caching & CDN

Lưu tạm kết quả để trả nhanh hơn và giảm tải.

## Các tầng cache

- **Browser cache** — trình duyệt giữ asset (ảnh, JS, CSS) theo header.
- **CDN** — máy chủ gần user giữ bản sao asset tĩnh (Cloudflare, Vercel Edge).
- **Server / app cache** — Redis, in-memory, hoặc cache của framework.
- **Data cache (client)** — TanStack Query giữ data đã fetch (xem [data-state.md](data-state.md)).

## Công cụ

- **[Redis](https://redis.io)** / **[Upstash](https://upstash.com)** — cache key-value (Upstash: serverless, hợp Vercel)
- **[Cloudflare](https://cloudflare.com)** — CDN + cache edge, free tốt
- Next.js có **cache tích hợp** (fetch cache, Route/Data cache).

## Cách dùng

### HTML tĩnh

Đặt **Cache-Control header** cho asset (cấu hình ở nền tảng host):

```
Cache-Control: public, max-age=31536000, immutable   # asset có hash tên file
Cache-Control: no-cache                               # HTML (luôn kiểm tra bản mới)
```

Kỹ thuật **cache busting**: đặt hash vào tên file (`app.a1b2.js`) → đổi nội dung = đổi tên = tải bản mới.

### React (SPA)

- Build tool (Vite) tự hash tên file → cache dài hạn an toàn.
- Data cache → **TanStack Query** với `staleTime`:

```jsx
useQuery({ queryKey: ['users'], queryFn, staleTime: 60_000 }); // 60s coi là còn tươi
```

### Next.js

- `fetch` được cache mặc định; chỉnh bằng option:

```jsx
fetch(url, { next: { revalidate: 60 } });   // ISR: làm mới sau 60s
fetch(url, { cache: 'no-store' });          // không cache (data động)
```

- App cache nặng → **Redis/Upstash**:

```jsx
const cached = await redis.get(key);
if (cached) return cached;
const data = await queryDB();
await redis.set(key, data, { ex: 300 }); // TTL 5 phút
```

## Mẹo

- **Cache cái ít đổi** (asset, danh mục), **đừng cache cái riêng tư/động** (giỏ hàng, trang cá nhân).
- Luôn có cách **invalidate** khi data đổi (revalidate, xoá key) → tránh trả bản cũ.
- **Cache busting bằng hash tên file** cho JS/CSS → vừa cache lâu vừa update được.
- Đo trước khi tối ưu — cache sai chỗ gây bug "sao dữ liệu không cập nhật".
