# Dark mode

Chế độ sáng / tối, và nhớ lựa chọn của người dùng.

## Nguyên tắc

- Dùng **CSS variable** cho màu, đổi bộ biến theo theme (xem [colors.md](colors.md)).
- Gắn theme bằng **thuộc tính trên `<html>`**: `data-theme="dark"` hoặc class `.dark`.
- 3 lựa chọn nên có: **Sáng / Tối / Theo hệ thống** (`prefers-color-scheme`).
- Tránh **nhấp nháy** (FOUC): set theme **trước khi** trang render.

## Cách dùng

### HTML tĩnh

```css
:root { --bg: #fff; --text: #111; }
:root[data-theme="dark"] { --bg: #0a0a0a; --text: #eee; }
body { background: var(--bg); color: var(--text); }
```

```html
<!-- đặt NGAY đầu <head> để tránh nhấp nháy -->
<script>
  const t = localStorage.theme
    || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
  document.documentElement.dataset.theme = t;
</script>

<button onclick="
  const d = document.documentElement;
  d.dataset.theme = d.dataset.theme === 'dark' ? 'light' : 'dark';
  localStorage.theme = d.dataset.theme;
">Đổi theme</button>
```

### React

Quản lý bằng context/state + set attribute trên `<html>`:

```jsx
const [theme, setTheme] = useState(() => localStorage.theme ?? 'light');
useEffect(() => {
  document.documentElement.dataset.theme = theme;
  localStorage.theme = theme;
}, [theme]);
```

### Next.js

Dùng **next-themes** — lo sẵn nhớ lựa chọn + chống nhấp nháy:

```bash
npm install next-themes
```

```jsx
// app/providers.tsx
'use client';
import { ThemeProvider } from 'next-themes';
export function Providers({ children }) {
  return (
    <ThemeProvider attribute="data-theme" defaultTheme="system" enableSystem>
      {children}
    </ThemeProvider>
  );
}
```

```jsx
// nút toggle
'use client';
import { useTheme } from 'next-themes';
const { theme, setTheme } = useTheme();
<button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>Đổi</button>
```

## Mẹo

- Tailwind: bật `darkMode: 'class'` (hoặc `[data-theme="dark"]`) → viết `dark:bg-black`.
- Đừng dùng đen tuyền `#000` cho nền tối → dùng `#0a0a0a`/`#111` cho dịu mắt.
- Ảnh/logo cần bản riêng cho nền tối → dùng `<picture>` hoặc filter.
- Cùng cơ chế "chuyển chế độ toàn app" với [i18n.md](i18n.md).
