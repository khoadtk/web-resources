# Web Resources

Kho ghi chú các chủ đề tài nguyên & nền tảng liên quan tới web — dạng "tra công cụ +
cách dùng + copy vào code". Mỗi chủ đề là một file riêng.

> 🔍 **Tra cứu nhanh**: mở [index.html](index.html) trong trình duyệt — có search
> (không cần gõ dấu) + lọc theo nhóm cho toàn bộ chủ đề.

## Assets & Design resources

- [Icons](icons.md) — Icônes, Iconify, Lucide, copy SVG
- [Fonts](fonts.md) — Google Fonts, Fontsource, self-host, `font-display`
- [Colors](colors.md) — palette, HSL, contrast checker (a11y)
- [Images & Photos](images.md) — Unsplash/Pexels, WebP/AVIF, lazy load
- [Illustrations](illustrations.md) — unDraw, Storyset, DiceBear
- [Shadows & Gradients](shadows-gradients.md) — generator, CSS

## Visualization

- [Charts & Data viz](charts.md) — Recharts, Chart.js, ECharts
- [Maps](maps.md) — Leaflet, MapLibre, Mapbox, react-leaflet

## UI / Component

- [Component libraries](component-libraries.md) — shadcn/ui, Radix, Headless UI
- [CSS & Layout](css-layout.md) — Flexbox, Grid, container queries, Tailwind
- [Animations](animations.md) — CSS animation, Framer Motion, Lottie, GSAP
- [Forms & Validation](forms-validation.md) — React Hook Form, Zod, error state
- [UI Patterns](ui-patterns.md) — toast, loading/skeleton, empty & error state
- [i18n](i18n.md) — đa ngôn ngữ (next-intl, react-i18next), Intl format
- [Dark mode](dark-mode.md) — sáng/tối, next-themes, chống nhấp nháy
- [Rich text editor](rich-text-editor.md) — Tiptap, Lexical, Quill, sanitize
- [Drag & Drop](drag-drop.md) — dnd-kit, SortableJS, kéo file
- [Tables & Data grid](tables.md) — TanStack Table, AG Grid, virtualize
- [Video & Audio](video-audio.md) — player, HLS, Mux, embed YouTube
- [Design system & Figma](design-system.md) — token, Storybook, Figma Dev Mode

## Data & Backend

- [Data & State](data-state.md) — fetch/axios, TanStack Query, Zustand/Redux
- [API layer](api-layer.md) — REST, GraphQL, tRPC, mutation
- [Database & ORM](database-orm.md) — Prisma, Drizzle, Supabase, Postgres
- [Realtime](realtime.md) — WebSocket, SSE, Socket.IO, Pusher/Ably
- [File upload & Storage](file-upload.md) — S3/R2, Cloudinary, UploadThing
- [Payment](payment.md) — Stripe, webhook, VNPay/MoMo
- [Email](email.md) — Resend, React Email, Nodemailer, SPF/DKIM
- [Caching & CDN](caching.md) — browser/CDN/Redis, cache busting, revalidate
- [Queue & Cron](queue-cron.md) — background job, BullMQ, QStash, cron
- [Search](search.md) — Fuse.js, Meilisearch, Algolia, full-text
- [AI integration](ai-integration.md) — chatbot, embeddings, RAG, Vercel AI SDK

## Backend fundamentals

> Nền tảng BE không gắn framework — dùng được dù code NestJS, Express hay gì khác.

- [HTTP fundamentals](http-fundamentals.md) — method, status code, header, idempotent, curl
- [REST API design](rest-api-design.md) — đặt tên endpoint, pagination, lỗi thống nhất, OpenAPI
- [SQL fundamentals](sql-fundamentals.md) — JOIN, index, transaction, N+1, ràng buộc
- [Networking basics](networking-basics.md) — DNS, TLS, port, reverse proxy, CORS
- [Logging & Observability](logging-observability.md) — structured log, request id, health check, metrics
- [Auth phía server](auth-server.md) — hash mật khẩu, session vs JWT, refresh token, OAuth flow
- [Phân quyền (Authorization)](authorization.md) — ACL, RBAC, ABAC, ReBAC, policy-as-code + usecase
- [API security](api-security.md) — validate biên, BOLA/IDOR, rate limit, OWASP API Top 10
- [Concurrency & Transaction](concurrency-transactions.md) — race condition, lock, isolation, idempotency
- [Message queue & Event](message-queue.md) — queue vs pub/sub, at-least-once, DLQ, outbox
- [Server caching sâu](server-caching.md) — cache-aside, invalidation, thundering herd
- [Webhooks design](webhooks.md) — verify chữ ký, trả 200 nhanh, idempotent, gửi webhook
- [File & Streaming](file-streaming.md) — stream, backpressure, range request, export lớn
- [Deployment & Process](deployment-process.md) — graceful shutdown, zero-downtime, expand/contract migration

## Web fundamentals

- [SEO & meta tags](seo-meta.md) — Open Graph, favicon, sitemap
- [Accessibility (a11y)](accessibility.md) — ARIA, keyboard, contrast
- [Performance](performance.md) — Core Web Vitals, lazy load, code splitting
- [Auth](auth.md) — đăng nhập, session, JWT, OAuth (NextAuth, Clerk)
- [Security](security.md) — XSS, CSRF, CSP, rate limit, checklist
- [Error & Monitoring](error-monitoring.md) — Sentry, error boundary, analytics
- [PWA & Offline](pwa.md) — manifest, service worker, cài được/offline
- [Feature flags & A/B test](feature-flags.md) — PostHog, rollout dần, kill switch
- [Web Workers & WASM](web-workers-wasm.md) — tính toán nặng không đơ UI, ffmpeg.wasm
  - [Deep-dive: trong Next.js](web-workers-wasm-nextjs.md) — SSR-safe, Comlink, COOP/COEP, transferable

## Nền tảng khác

- [Browser extension](browser-extension.md) — Manifest V3, WXT, content script

## Tooling

- [Dev utilities](dev-utilities.md) — regex101, transform.tools, caniuse
- [Testing](testing.md) — Vitest, Testing Library, Playwright (unit + e2e)
  - [Deep-dive: Automation testing với Playwright](automation-testing-playwright.md) — locator, fixtures, auth state, trace, CI
  - [Playwright + Cucumber-js (BDD)](playwright-cucumber.md) — Gherkin, World, step definitions, playwright-bdd
- [Deploy & Hosting](deploy-hosting.md) — Vercel, Netlify, Cloudflare, env, CI/CD
- [Git workflow](git-workflow.md) — branch, commit convention, PR, .gitignore
- [Env & Config](env-config.md) — biến env, secret, .env.example, validate
- [TypeScript tips](typescript-tips.md) — utility types, as const, Zod infer
- [Date & Time](date-time.md) — Day.js, Intl, UTC/timezone, "5 phút trước"
- [Monorepo](monorepo.md) — pnpm workspaces, Turborepo, shared packages
- [Docker cho web](docker-web.md) — Dockerfile, multi-stage, compose dev

---

> Cứ thêm chủ đề mới = tạo file `.md` + thêm 1 dòng vào mục lục này.
