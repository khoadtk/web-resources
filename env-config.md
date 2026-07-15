# Env & Config

Quản lý biến môi trường, cấu hình, và secret an toàn.

## Nguyên tắc

- **Không hard-code** URL/khoá trong code → đưa vào biến env.
- **Không commit secret** → `.env` vào `.gitignore`, chỉ commit `.env.example`.
- **Phân biệt** biến công khai (lộ ra client được) và biến bí mật (chỉ server).
- Mỗi môi trường (dev / staging / production) có bộ env riêng.

## File chuẩn

```
.env                # mặc định (thường KHÔNG commit nếu có secret)
.env.local          # override cục bộ, KHÔNG commit
.env.example        # mẫu, commit được (không có giá trị thật)
```

```bash
# .env.example
DATABASE_URL=
NEXT_PUBLIC_API_URL=
STRIPE_SECRET_KEY=
```

## Cách dùng

### HTML tĩnh

Không có "env" thật lúc chạy → giá trị nhúng vào lúc build, hoặc gọi qua backend.

- **Đừng nhét secret** vào JS client (ai cũng xem được).
- Config công khai → có thể để trong 1 file `config.js`.

### React (Vite)

Chỉ biến có tiền tố **`VITE_`** mới lộ ra client:

```bash
# .env
VITE_API_URL=https://api.example.com
```

```js
import.meta.env.VITE_API_URL
```

→ **secret không đặt `VITE_`** (và vẫn nhớ: mọi thứ tới client đều công khai).

### Next.js

- **`NEXT_PUBLIC_`** → lộ ra client; không prefix → **chỉ server**:

```bash
NEXT_PUBLIC_API_URL=https://api.example.com   # client dùng được
DATABASE_URL=postgres://...                   # chỉ server
```

```js
process.env.DATABASE_URL          // server only
process.env.NEXT_PUBLIC_API_URL   // client + server
```

- Trên Vercel/Netlify: khai lại env trong **dashboard** theo từng môi trường.

## Validate env (nên có)

Dùng **Zod** kiểm tra env lúc khởi động → thiếu là báo ngay, không lỗi mơ hồ:

```js
import { z } from 'zod';
const env = z.object({
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().min(1),
}).parse(process.env);
```

(Hoặc dùng **[@t3-oss/env-nextjs](https://env.t3.gg)** cho Next.js — tách rõ client/server.)

## Mẹo

- **Lỡ commit secret = phải xoay key mới** (xem [git-workflow.md](git-workflow.md), [security.md](security.md)).
- Giữ **`.env.example` luôn cập nhật** → người mới clone biết cần biến gì.
- Đừng test bằng cách **sửa `.env` thật** của mình → override qua shell env (đây là ghi chú [[no-mutate-real-config-when-testing]]).
- Secret thật nên ở **secret manager** của nền tảng, không nằm trong repo.
