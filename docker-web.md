# Docker cho web

Đóng gói app web thành container: chạy đâu cũng giống nhau, deploy dễ.

## Khi nào cần

- Deploy lên **VPS/Cloud Run/Fly.io** (không phải Vercel/Netlify — mấy nền tảng đó tự lo).
- Chạy **dịch vụ phụ khi dev**: Postgres, Redis, Meilisearch... bằng `docker compose` — không cần cài vào máy.
- Đảm bảo "máy tôi chạy được" = mọi nơi chạy được.

## Khái niệm 30 giây

- **Image** — bản đóng gói app (code + runtime).
- **Container** — image đang chạy.
- **Dockerfile** — công thức build image.
- **docker compose** — chạy nhiều container cùng lúc (app + db).

## Cách dùng

### HTML tĩnh

Serve bằng nginx — Dockerfile 2 dòng:

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

```bash
docker build -t mysite . && docker run -p 8080:80 mysite
```

### React (SPA)

**Multi-stage**: build bằng Node, serve bằng nginx (image cuối nhỏ):

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
# SPA route: cần nginx.conf với try_files $uri /index.html;
```

### Next.js

Bật `output: 'standalone'` trong `next.config.js` → image gọn:

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-alpine
WORKDIR /app
COPY --from=build /app/.next/standalone ./
COPY --from=build /app/.next/static ./.next/static
COPY --from=build /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

### docker compose cho môi trường dev

Chạy DB/Redis phục vụ dev mà không cài vào máy:

```yaml
# compose.yaml
services:
  db:
    image: postgres:17-alpine
    environment: { POSTGRES_PASSWORD: dev }
    ports: ['5432:5432']
    volumes: ['dbdata:/var/lib/postgresql/data']
  redis:
    image: redis:7-alpine
    ports: ['6379:6379']
volumes: { dbdata: {} }
```

```bash
docker compose up -d     # bật
docker compose down      # tắt
```

## Mẹo

- **`.dockerignore`** (node_modules, .next, .git, .env) — build nhanh + không rò secret vào image.
- Copy `package.json` + install **trước** khi copy code → tận dụng layer cache, build lại nhanh.
- Secret truyền qua **env lúc chạy** (`-e`, compose `environment`), đừng nướng vào image (xem [env-config.md](env-config.md)).
- Dùng image **alpine** + multi-stage → image nhỏ hơn nhiều lần.
- App trong container phải nghe `0.0.0.0`, không phải `localhost`.
