# Animations

Chuyển động UI: hover, xuất hiện, chuyển trang, micro-interaction.

## Công cụ & thư viện

- **[Framer Motion](https://www.framer.com/motion/)** — animation cho React, API dễ, phổ biến nhất
- **[GSAP](https://gsap.com)** — mạnh nhất cho animation phức tạp, timeline
- **[Lottie](https://lottiefiles.com)** — chạy animation vẽ từ After Effects (file JSON)
- **[Auto-Animate](https://auto-animate.formkit.com)** — thêm 1 dòng, tự animate khi list thay đổi
- **[Animate.css](https://animate.style)** — thư viện class animation CSS sẵn
- **[Cubic-bezier](https://cubic-bezier.com)** — chỉnh đường cong easing

## Cách dùng

### HTML tĩnh

CSS `transition` + `@keyframes` — đủ cho phần lớn nhu cầu, không cần JS:

```css
.btn { transition: transform 0.2s ease; }
.btn:hover { transform: scale(1.05); }

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
.card { animation: fadeIn 0.4s ease; }
```

Cần animation vẽ sẵn → nhúng **Lottie** qua web component:

```html
<script src="https://unpkg.com/@lottiefiles/dotlottie-wc@0.6.2/dist/dotlottie-wc.js" type="module"></script>
<dotlottie-wc src="anim.lottie" autoplay loop></dotlottie-wc>
```

### React

**Framer Motion** — animate bằng prop, gọn:

```bash
npm install framer-motion
```

```jsx
import { motion } from 'framer-motion';
<motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }}>
  Nội dung
</motion.div>
```

List tự animate khi thêm/xóa item → **Auto-Animate**:

```jsx
import { useAutoAnimate } from '@formkit/auto-animate/react';
const [parent] = useAutoAnimate();
<ul ref={parent}>{items.map(...)}</ul>
```

### Next.js

- Framer Motion cần chạy client → thêm `'use client'` ở component có animation.
- Chuyển trang mượt: dùng **View Transitions API** (Next 15+) hoặc `AnimatePresence` của Framer Motion.

## Mẹo

- **Chỉ animate `transform` + `opacity`** → mượt (chạy trên GPU). Tránh animate `width`/`top`/`margin`.
- Thời lượng ngắn: **150–300ms** cho micro-interaction.
- Tôn trọng `prefers-reduced-motion` (người dùng tắt hiệu ứng):

```css
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
```
