# Monorepo

Nhiều app/package trong 1 repo: web + api + shared code dùng chung.

## Khi nào cần

- Có **≥2 app dùng chung code** (web Next.js + API NestJS + package `shared` chứa type/util).
- Muốn đổi code shared → mọi app thấy ngay, không cần publish npm.
- **1 app đơn lẻ thì KHÔNG cần** — thêm monorepo chỉ thêm phức tạp.

## Công cụ

- **npm/pnpm workspaces** — nền tảng: nhiều package trong 1 repo (pnpm nhanh + tiết kiệm disk nhất)
- **[Turborepo](https://turborepo.dev)** — chạy task có cache + song song trên workspace (khuyên dùng, hợp Vercel/Next)
- **[Nx](https://nx.dev)** — mạnh hơn, nhiều plugin (NestJS, React...), hợp repo lớn

## Cấu trúc điển hình

```
my-repo/
├─ package.json          # "workspaces": ["apps/*", "packages/*"]
├─ turbo.json
├─ apps/
│  ├─ web/               # Next.js
│  └─ api/               # NestJS/Express
└─ packages/
   ├─ shared/            # type + util dùng chung
   └─ ui/                # component dùng chung
```

## Cách dùng

### Setup nhanh (pnpm + Turborepo)

```bash
npx create-turbo@latest        # scaffold sẵn apps/ + packages/
```

```yaml
# pnpm-workspace.yaml
packages: ['apps/*', 'packages/*']
```

Dùng package nội bộ trong app:

```json
// apps/web/package.json
{ "dependencies": { "@repo/shared": "workspace:*" } }
```

```ts
import { formatPrice } from '@repo/shared';
```

### Chạy task

```bash
turbo dev              # chạy dev mọi app song song
turbo build            # build theo thứ tự phụ thuộc, có cache
turbo lint --filter=web   # chỉ chạy cho app web
```

### Với React/Next.js

- Package `ui` chứa component dùng chung; Next.js cần `transpilePackages: ['@repo/ui']` trong `next.config.js`.
- Deploy trên Vercel: chọn root là `apps/web`, Vercel tự hiểu Turborepo.

## Mẹo

- Bắt đầu **tối giản**: workspaces + 1 package `shared`, thêm Turborepo khi build chậm.
- **Type dùng chung** (API request/response) để trong `packages/shared` → frontend/backend không lệch nhau (xem [typescript-tips.md](typescript-tips.md)).
- Thống nhất **1 version** cho dep quan trọng (TS, React) giữa các package — lệch version là nguồn bug khó hiểu.
- CI: dùng `turbo run build --filter=...[origin/main]` để chỉ build phần bị ảnh hưởng.
