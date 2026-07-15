# Realtime

Dữ liệu cập nhật tức thời: chat, thông báo, "đang gõ...", giá live.

## Chọn công nghệ

| Cách | Khi nào dùng |
| --- | --- |
| **WebSocket** | 2 chiều, liên tục: chat, game, collaborate |
| **SSE (Server-Sent Events)** | 1 chiều server→client: notify, feed, log stream |
| **Polling** | Đơn giản nhất: gọi lại API sau mỗi vài giây (khi realtime không gắt) |

## Dịch vụ / thư viện

- **[Socket.IO](https://socket.io)** — WebSocket dễ dùng (tự reconnect, room, fallback)
- **[Pusher](https://pusher.com)** / **[Ably](https://ably.com)** — realtime as-a-service (khỏi tự dựng server)
- **[Supabase Realtime](https://supabase.com/realtime)** — nghe thay đổi DB realtime
- **[PartyKit](https://partykit.io)** — realtime chạy trên edge, dễ deploy

## Cách dùng

### HTML tĩnh

WebSocket / SSE có sẵn trong trình duyệt, không cần lib:

```html
<script>
  // WebSocket
  const ws = new WebSocket('wss://server/chat');
  ws.onmessage = e => console.log('nhận:', e.data);
  ws.send('xin chào');

  // SSE (chỉ nhận)
  const es = new EventSource('/api/stream');
  es.onmessage = e => console.log(e.data);
</script>
```

### React

Kết nối trong `useEffect`, **nhớ đóng khi unmount**:

```jsx
useEffect(() => {
  const ws = new WebSocket('wss://server/chat');
  ws.onmessage = e => setMessages(m => [...m, e.data]);
  return () => ws.close();   // dọn dẹp
}, []);
```

Dùng **Socket.IO** hoặc SDK Pusher/Ably cho tính năng phức tạp (room, presence).

### Next.js

- Serverless (Vercel) **không giữ WebSocket lâu** → dùng dịch vụ ngoài (**Pusher/Ably/Supabase**) hoặc PartyKit.
- **SSE** thì viết được bằng Route Handler trả `ReadableStream`.
- Component realtime cần `'use client'`.

## Mẹo

- Luôn xử lý **mất kết nối + tự reconnect** (Socket.IO lo sẵn).
- **Không realtime = không cần** — polling/refetch (TanStack Query) đủ cho phần lớn app.
- Chống lạm dụng: **throttle** sự kiện "đang gõ", giới hạn tần suất gửi.
- Xác thực user khi mở kết nối (token) — đừng để ai cũng nghe được.
