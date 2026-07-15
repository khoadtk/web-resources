# Shadows & Gradients

## Shadow (đổ bóng)

### Công cụ

- **[Shadows Brumm](https://shadows.brumm.af)** — tạo bóng nhiều lớp mượt như thật
- **[Smooth Shadow](https://smoothshadow.com)** — bóng mềm, chỉnh trực quan
- **[Tailwind Shadow](https://tailwindcss.com/docs/box-shadow)** — bộ `shadow-sm → shadow-2xl` có sẵn

### Mẹo

Bóng đẹp = **nhiều lớp bóng nhẹ** thay vì 1 bóng đậm:

```css
box-shadow:
  0 1px 2px rgba(0,0,0,0.04),
  0 2px 4px rgba(0,0,0,0.04),
  0 4px 8px rgba(0,0,0,0.04);
```

- Dùng màu bóng theo tông màu nền (không dùng đen tuyền 100%).
- Dark mode: bóng ít tác dụng → dùng viền sáng (`border`) hoặc glow thay bóng.

## Gradient (chuyển màu)

### Công cụ

- **[CSS Gradient](https://cssgradient.io)** — tạo gradient, xuất CSS
- **[Mesh Gradient](https://meshgradient.com)** — gradient dạng "mesh" hiện đại
- **[Hypercolor](https://hypercolor.dev)** — bộ gradient Tailwind đẹp sẵn
- **[Gradienta](https://gradienta.io)** — gradient tải về dạng ảnh/CSS

### Cú pháp

```css
/* Linear */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

/* Radial */
background: radial-gradient(circle at top, #f093fb, #f5576c);

/* Text gradient */
.title {
  background: linear-gradient(90deg, #3b82f6, #8b5cf6);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
```

### Mẹo

- Gradient nền nên **nhẹ, ít tương phản** để không lấn text.
- Text gradient hợp với tiêu đề lớn, tránh chữ nhỏ (khó đọc).
