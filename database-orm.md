# Database & ORM

Lưu trữ dữ liệu và cách truy vấn từ code.

## Chọn database

- **PostgreSQL** — mặc định tốt cho phần lớn app (quan hệ, mạnh, miễn phí).
- **MySQL / MariaDB** — quan hệ, phổ biến.
- **SQLite** — nhẹ, 1 file, hợp app nhỏ / dev / local.
- **MongoDB** — NoSQL, dữ liệu linh hoạt (dùng khi thật sự cần).

## ORM / công cụ (viết query bằng code thay vì SQL tay)

- **[Prisma](https://prisma.io)** — ORM phổ biến nhất, type-safe, có migration (khuyên dùng)
- **[Drizzle](https://orm.drizzle.team)** — ORM nhẹ, gần SQL, type-safe, đang lên
- **[Supabase](https://supabase.com)** — Postgres + auth + API + realtime dựng sẵn (backend-as-a-service)
- **[Kysely](https://kysely.dev)** — query builder type-safe, sát SQL

## Cách dùng

### HTML tĩnh

Trang tĩnh **không nối trực tiếp** vào DB (lộ thông tin kết nối). Cách đúng:

- Gọi qua **API backend** riêng, hoặc
- Dùng **Supabase JS SDK** (an toàn nhờ Row Level Security):

```html
<script type="module">
  import { createClient } from 'https://esm.sh/@supabase/supabase-js';
  const db = createClient(URL, ANON_KEY);
  const { data } = await db.from('posts').select('*');
</script>
```

### React (SPA)

Tương tự: SPA gọi tới **API** hoặc **Supabase SDK**, không nhúng chuỗi kết nối DB thật vào client.

### Next.js

Có server → query DB thẳng trong **Server Component / API route / Server Action** bằng Prisma:

```bash
npm install prisma @prisma/client
npx prisma init
```

```prisma
// schema.prisma
model Post {
  id    Int    @id @default(autoincrement())
  title String
}
```

```jsx
// Server Component
import { prisma } from '@/lib/prisma';
export default async function Page() {
  const posts = await prisma.post.findMany();
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

```bash
npx prisma migrate dev --name init   # tạo bảng theo schema
```

## Mẹo

- **Không bao giờ để chuỗi kết nối DB (`DATABASE_URL`) lộ ra client** — chỉ dùng ở server (xem cảnh báo secret trong [[personal-workspace-map]]).
- Dùng **migration** (Prisma/Drizzle) để đổi schema có kiểm soát, đừng sửa DB tay.
- Tạo **1 instance Prisma dùng chung** (tránh tạo mới mỗi request khi dev → hết connection).
- Bắt đầu nhanh → **Supabase** (khỏi dựng backend); cần kiểm soát → Postgres + Prisma tự host.
