# API layer

Cách frontend nói chuyện với backend: gọi, thiết kế, và mutation.

## Kiểu API

- **REST** — endpoint theo resource (`GET /users`, `POST /users`). Phổ biến, đơn giản.
- **GraphQL** — 1 endpoint, client tự chọn field cần. Hợp data phức tạp, nhiều quan hệ.
- **tRPC** — gọi hàm backend như hàm local, **type-safe** đầu-cuối (chỉ TS, hợp Next.js).

## Công cụ

- **[Axios](https://axios-http.com)** — HTTP client tiện hơn `fetch`
- **[TanStack Query](https://tanstack.com/query)** — cache + mutation (xem [data-state.md](data-state.md))
- **[tRPC](https://trpc.io)** — RPC type-safe cho fullstack TS
- **[Apollo](https://www.apollographql.com)** / **[urql](https://urql.dev)** — client GraphQL
- **[Zod](https://zod.dev)** — validate dữ liệu vào/ra API

## Cách dùng

### HTML tĩnh

`fetch` thuần cho GET/POST:

```js
// GET
const users = await fetch('/api/users').then(r => r.json());

// POST
await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Khoa' }),
});
```

### React

**TanStack Query** cho cả đọc (query) và ghi (mutation):

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

const qc = useQueryClient();
const addUser = useMutation({
  mutationFn: (u) => fetch('/api/users', {
    method: 'POST', body: JSON.stringify(u),
  }).then(r => r.json()),
  onSuccess: () => qc.invalidateQueries({ queryKey: ['users'] }), // refetch list
});

addUser.mutate({ name: 'Khoa' });
```

### Next.js

- **Route Handler** (`app/api/.../route.ts`) — tự viết REST endpoint:

```jsx
// app/api/users/route.ts
export async function GET() {
  const users = await prisma.user.findMany();
  return Response.json(users);
}
```

- **Server Action** — gọi hàm server từ form/nút, không cần tự dựng endpoint.
- Muốn **type-safe tuyệt đối** frontend↔backend → **tRPC**.

## Mẹo

- **Validate input bằng Zod ở server** — không tin dữ liệu client gửi lên.
- Trả **HTTP status đúng** (200/201/400/401/404/500) → client xử lý dễ.
- Sau mutation, **invalidate query** liên quan để UI tự cập nhật.
- Xử lý đủ **loading / error** cho mọi lời gọi (xem [ui-patterns.md](ui-patterns.md)).
- Đặt base URL + token trong **interceptor** (Axios) để khỏi lặp.
