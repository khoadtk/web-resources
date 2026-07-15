# Date & Time

Xử lý ngày giờ: format, tính toán, timezone, "5 phút trước".

## Thư viện

- **[Day.js](https://day.js.org)** — 2KB, API giống Moment (khuyên dùng thay Moment)
- **[date-fns](https://date-fns.org)** — hàm rời, tree-shake tốt, immutable
- **[Luxon](https://moment.github.io/luxon/)** — timezone mạnh
- **`Intl.DateTimeFormat`** — có sẵn trình duyệt, format theo locale, **không cần lib**
- ⚠️ **Moment.js đã ngừng phát triển** — dự án mới đừng dùng.

## Nguyên tắc quan trọng nhất

- **Lưu trữ & truyền đi: UTC (ISO 8601)** — `2026-07-15T03:00:00Z`.
- **Hiển thị: theo timezone người xem.**
- DB dùng kiểu `timestamptz` (Postgres); đừng lưu chuỗi ngày tự chế.

## Cách dùng

### HTML tĩnh

`Intl` có sẵn, format tiếng Việt không cần thư viện:

```js
new Intl.DateTimeFormat('vi-VN', { dateStyle: 'full', timeStyle: 'short' })
  .format(new Date());          // "Thứ Tư, 15 tháng 7, 2026 lúc 10:00"

new Intl.RelativeTimeFormat('vi').format(-5, 'minute'); // "5 phút trước"
```

### React

**Day.js** cho tính toán + format gọn:

```bash
npm install dayjs
```

```jsx
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
import 'dayjs/locale/vi';
dayjs.extend(relativeTime); dayjs.locale('vi');

dayjs('2026-07-15').format('DD/MM/YYYY');   // 15/07/2026
dayjs(post.createdAt).fromNow();            // "5 phút trước"
dayjs().add(7, 'day');                      // tuần sau
```

Date picker: **react-day-picker** (shadcn/ui dùng nó) hoặc `<input type="date">`.

### Next.js

- Cẩn thận **hydration mismatch**: server và client có thể ở timezone khác → giờ render 2 nơi khác nhau:

```jsx
// cách an toàn: format ở client sau khi mount, hoặc truyền chuỗi đã format từ server
const [time, setTime] = useState('');
useEffect(() => setTime(dayjs(iso).format('HH:mm DD/MM')), [iso]);
```

- Server Action/API: nhận & trả **chuỗi ISO UTC**, để client tự format.

## Mẹo

- "5 phút trước" phải **tự cập nhật** → re-render theo interval, hoặc kèm tooltip giờ đầy đủ.
- Đừng cộng trừ ngày bằng tay (`+ 86400000`) — dính DST/timezone là sai; dùng lib.
- Input của user (date picker) là **giờ địa phương** → convert sang UTC trước khi gửi server.
- So sánh/sắp xếp: so bằng timestamp/ISO, đừng so chuỗi đã format.
