# Data & State

Lấy dữ liệu từ API (data fetching) và quản lý trạng thái (state) trong app.

## Phân biệt 2 loại state

- **Server state** — data từ API (user, sản phẩm...). Cần cache, refetch, loading/error.
  → dùng **TanStack Query** / SWR.
- **Client state** — state UI trong app (theme, modal mở/đóng, giỏ hàng tạm).
  → dùng `useState`, **Zustand**, Redux.

Đừng nhét data API vào Redux thủ công — để TanStack Query lo.

## Thư viện

- **[TanStack Query](https://tanstack.com/query)** — fetch + cache + refetch data server (mạnh nhất)
- **[SWR](https://swr.vercel.app)** — như trên, của Vercel, nhẹ hơn
- **[Zustand](https://zustand-demo.pmnd.rs)** — quản lý client state, cực gọn
- **[Redux Toolkit](https://redux-toolkit.js.org)** — client state cho app lớn, nhiều dev
- **[Axios](https://axios-http.com)** — thay `fetch`, tiện hơn (interceptor, JSON tự parse)

## Cách dùng

### HTML tĩnh

`fetch` thuần rồi tự cập nhật DOM:

```html
<script>
  fetch('/api/users')
    .then(r => r.json())
    .then(users => {
      document.querySelector('#list').innerHTML =
        users.map(u => `<li>${u.name}</li>`).join('');
    });
</script>
```

### React

**TanStack Query** cho data server — tự lo loading/error/cache:

```bash
npm install @tanstack/react-query
```

```jsx
import { useQuery } from '@tanstack/react-query';

function Users() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(r => r.json()),
  });
  if (isLoading) return <p>Đang tải...</p>;
  if (error) return <p>Lỗi</p>;
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

Client state → **Zustand**:

```jsx
import { create } from 'zustand';
const useCart = create(set => ({
  items: [],
  add: item => set(s => ({ items: [...s.items, item] })),
}));
```

### Next.js

- **Server Components** (mặc định) fetch thẳng trên server, không cần lib:

```jsx
// app/users/page.tsx  (Server Component)
export default async function Page() {
  const users = await fetch('https://api/...').then(r => r.json());
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

- Cần cache/refetch/mutation phía client → vẫn dùng TanStack Query trong Client Component.
- Client state nhẹ → Zustand (nhớ `'use client'`).

## Mẹo

- Mặc định fetch ở **server** (Next) → nhanh + SEO tốt; chỉ lên client khi cần tương tác.
- **Không lạm dụng global state** — state để càng gần nơi dùng càng tốt.
- Đặt **`queryKey`** rõ ràng để cache/invalidate đúng.
