# Colors

## Công cụ

- **[Coolors](https://coolors.co)** — tạo bảng màu (palette) nhanh, bấm space để random
- **[Tailwind Colors](https://tailwindcss.com/docs/customizing-colors)** — bảng màu chuẩn, có sẵn 50→950
- **[Realtime Colors](https://realtimecolors.com)** — thử màu ngay trên 1 layout web mẫu
- **[Radix Colors](https://www.radix-ui.com/colors)** — thang màu có accessibility + dark mode sẵn
- **[Contrast Checker (WebAIM)](https://webaim.org/resources/contrastchecker/)** — kiểm tra tương phản chữ/nền (a11y)

## Hệ màu nên dùng

- **HEX** — `#3b82f6` (phổ biến, copy từ design)
- **HSL** — `hsl(217 91% 60%)` — dễ chỉnh sáng/tối bằng cách đổi số L
- **OKLCH** — `oklch(0.6 0.2 250)` — màu đều mắt hơn, đang thành chuẩn mới (Tailwind v4 dùng)

## Cách dùng

### HTML tĩnh

CSS variable cho theme (dark/light) — bật/tắt bằng `data-theme` trên `<html>`:

```css
:root {
  --bg: #ffffff;
  --text: #1a1a1a;
  --primary: #3b82f6;
}
[data-theme="dark"] {
  --bg: #0a0a0a;
  --text: #ededed;
}
body { background: var(--bg); color: var(--text); }
```

### React

Đọc màu từ 1 file token dùng chung → tránh hard-code rải rác:

```js
// theme.js
export const colors = { primary: '#3b82f6', bg: '#ffffff', text: '#1a1a1a' };
```

```jsx
import { colors } from './theme';
<button style={{ background: colors.primary }}>OK</button>
```

Hoặc dùng CSS variable + class `.dark` toggle bằng state.

### Next.js

Dùng CSS variable trong `globals.css` + Tailwind map vào token. Toggle dark mode
gọn nhất với **next-themes**:

```bash
npm install next-themes
```

```jsx
// app/providers.tsx
'use client';
import { ThemeProvider } from 'next-themes';
export function Providers({ children }) {
  return <ThemeProvider attribute="data-theme">{children}</ThemeProvider>;
}
```

## Mẹo

### Accessibility (bắt buộc để ý)

- Chữ thường cần tỉ lệ tương phản **≥ 4.5:1** so với nền.
- Chữ lớn (≥24px hoặc 18px bold) cần **≥ 3:1**.
- Đừng chỉ dùng màu để truyền thông tin (vd đỏ/xanh) — thêm icon/text cho người mù màu.

### Tạo thang màu nhanh

Chọn 1 màu chính → dùng Radix/Tailwind để lấy các bậc sáng-tối thay vì tự chỉnh tay.
