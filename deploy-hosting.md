# Deploy & Hosting

Đưa web lên mạng cho người khác truy cập.

## Nền tảng

| Nền tảng | Hợp với |
| --- | --- |
| **[Vercel](https://vercel.com)** | Next.js (cùng nhà làm ra) — deploy 1 click, tốt nhất cho Next |
| **[Netlify](https://netlify.com)** | Web tĩnh, React SPA, form + function |
| **[Cloudflare Pages](https://pages.cloudflare.com)** | Tĩnh + edge function, CDN nhanh, free hào phóng |
| **[GitHub Pages](https://pages.github.com)** | Web tĩnh thuần, miễn phí, hợp demo/portfolio |
| **[Render](https://render.com)** / **[Railway](https://railway.app)** | App có backend/database |

## Cách dùng

### HTML tĩnh

Chỉ cần đẩy folder chứa `index.html` lên:

- **GitHub Pages** — push repo → bật Pages trong Settings.
- **Netlify / Cloudflare Pages** — kéo-thả folder hoặc nối GitHub, không cần build.

### React (SPA)

Có bước build → cần cấu hình:

```
Build command:  npm run build
Output dir:     dist   (Vite)  hoặc  build  (CRA)
```

- Nối repo GitHub với Vercel/Netlify → mỗi lần push tự build & deploy.
- SPA nhiều route cần **rewrite về `index.html`** (Netlify: file `_redirects` với `/* /index.html 200`).

### Next.js

- **Vercel** là lựa chọn tự nhiên nhất: nối GitHub → tự nhận diện Next → deploy.
- Có SSR/API route → cần nền tảng chạy Node (Vercel/Netlify/Render), **không** deploy như tĩnh được (trừ khi `output: 'export'` cho site hoàn toàn tĩnh).

## Biến môi trường (env)

```bash
# .env.local  (KHÔNG commit)
DATABASE_URL=...
NEXT_PUBLIC_API_URL=...   # prefix NEXT_PUBLIC_ mới lộ ra client (Next.js)
```

- Khai báo lại env trên dashboard của nền tảng (Vercel/Netlify Settings).
- **Không commit** file `.env` chứa secret → thêm vào `.gitignore`.

## CI/CD (tự động)

- Nối GitHub → **push là tự deploy** (preview cho mỗi PR, production cho `main`).
- Muốn chạy test/lint trước khi deploy → **GitHub Actions** (`.github/workflows/`).

## Mẹo

- **Preview deploy**: mỗi PR có 1 URL riêng để review trước khi merge.
- Gắn **custom domain** trong dashboard, nền tảng lo SSL (HTTPS) tự động.
- Kiểm tra **build log** khi deploy fail — thường do thiếu env hoặc sai output dir.
