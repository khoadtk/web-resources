# Deployment & Process (BE)

Deploy backend không rớt request: graceful shutdown, zero-downtime, migration.

> Chọn nền tảng + CI/CD cơ bản ở [deploy-hosting.md](deploy-hosting.md); đóng gói ở [docker-web.md](docker-web.md). File này về *quy trình* để deploy an toàn.

## Graceful shutdown — chết cho đàng hoàng

Khi deploy, process cũ bị gửi `SIGTERM`. App lờ đi → bị `SIGKILL` sau grace period → **request đang chạy dở bị cắt ngang** (transaction lửng, response đứt).

```ts
const server = app.listen(3000);

process.on('SIGTERM', async () => {
  server.close(async () => {          // 1. ngừng NHẬN request mới, chờ request đang chạy xong
    await queueWorker.close();        // 2. dừng nhận job mới, chờ job dở xong
    await db.$disconnect();           // 3. đóng kết nối DB/Redis
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 25_000); // chốt chặn: không thoát nổi thì tự thoát
});
```

Thứ tự chuẩn: **ngừng nhận việc mới → chờ việc dở → đóng kết nối → thoát**. Kèm theo: readiness probe ([logging-observability.md](logging-observability.md)) phải **báo fail ngay khi nhận SIGTERM** để load balancer ngừng gửi request tới trước khi bạn đóng cửa.

## Zero-downtime deploy

```
Rolling:     [v1] [v1] [v1] → [v2] [v1] [v1] → [v2] [v2] [v1] → xong
             thay từng instance, LB chỉ gửi traffic vào instance healthy

Blue-green:  dựng nguyên cụm v2 song song → healthcheck OK → gạt traffic sang → giữ v1 để rollback tức thì
```

Điều kiện để mọi kiểu trên chạy được — và là **2 quy tắc quan trọng nhất file này**:

1. **App stateless** — session/data ở Redis/DB, không ở RAM/disk của instance ([networking-basics.md](networking-basics.md)).
2. **v1 và v2 chạy song song được một lúc** → mọi thay đổi (DB schema, message format, API) phải **tương thích lùi ít nhất 1 phiên bản**.

## Migration lúc deploy — expand → migrate → contract

Hệ quả trực tiếp của quy tắc 2. Đổi tên cột `name` → `full_name` mà làm 1 phát: code v1 còn sống query `name` → nổ ngay giữa lúc deploy.

```
Bước 1 (expand):   THÊM cột full_name; code v2 ghi CẢ HAI, đọc full_name ưu tiên  → deploy
Bước 2 (migrate):  backfill data cũ sang full_name (script chạy nền, theo batch)
Bước 3 (contract): code v3 chỉ dùng full_name → deploy → XOÁ cột name (sau vài ngày yên ổn)
```

- Mọi migration phải **thêm trước, xoá sau**, tách nhiều lần deploy. Không bao giờ `DROP`/`RENAME` trong cùng deploy với code đang dùng nó.
- Chạy migration **trước khi** bật code mới (bước riêng trong CI: `migrate deploy` → rồi mới rollout), và migration phải **an toàn khi chạy lại** (idempotent).
- Migration khoá bảng lâu (thêm index bảng to) → dùng dạng không khoá (`CREATE INDEX CONCURRENTLY` trong Postgres), chạy giờ vắng.

## Env theo stage & release hygiene

- Tối thiểu 3 stage: **dev → staging → production**, mỗi stage bộ env riêng ([env-config.md](env-config.md)); staging giống production nhất có thể (cùng version DB, cùng cách deploy).
- **Artifact build một lần, promote dần**: image đã test ở staging là *chính image đó* lên production — không build lại (build lại = thứ lên prod chưa từng được test).
- Gắn **version/commit SHA vào app** (env `RELEASE`) → hiện trong log + Sentry, biết lỗi thuộc bản nào.
- Feature chưa xong vẫn merge được → giấu sau **feature flag** ([feature-flags.md](feature-flags.md)); deploy ≠ release.

## Rollback — nghĩ trước khi cần

- Rollback code: quay lại image cũ — 1 phút nếu làm đúng "build once".
- Rollback **migration thì đừng**: DB đã có data mới; đúng bài là expand/contract khiến **code cũ vẫn chạy được trên schema mới** — rollback code là đủ, schema để yên.
- Định nghĩa trước tín hiệu abort: % lỗi 5xx tăng, p95 vọt ([logging-observability.md](logging-observability.md)) → tự động dừng rollout (canary).

## Checklist deploy an toàn

- [ ] Bắt `SIGTERM`: đóng cửa nhận request → chờ việc dở → đóng kết nối.
- [ ] Readiness fail ngay khi bắt đầu shutdown.
- [ ] Schema change theo expand → migrate → contract; không DROP cùng deploy.
- [ ] Migration chạy như bước riêng, idempotent, không khoá bảng dài.
- [ ] Build một lần, promote artifact qua các stage.
- [ ] Có đường rollback code 1 lệnh + tín hiệu abort tự động.
- [ ] Worker/queue cũng graceful shutdown, job đang dở không mất ([message-queue.md](message-queue.md) — at-least-once lo phần chạy lại).
