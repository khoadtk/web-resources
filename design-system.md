# Design system & Figma

Giữ UI nhất quán: token, style guide, component có tài liệu, từ Figma ra code.

## Thành phần của một design system

- **Design tokens** — biến nền tảng: màu, spacing, font-size, radius, shadow.
- **Component** — button, input, card... dùng lại, có quy tắc rõ.
- **Tài liệu** — khi nào dùng gì, do's & don'ts.

## Công cụ

- **[Figma](https://figma.com)** — thiết kế + prototype, chuẩn ngành
- **[Storybook](https://storybook.js.org)** — "catalog" component: xem, thử prop, tài liệu hoá
- **[Style Dictionary](https://styledictionary.com)** — biến token (JSON) thành CSS/TS/iOS/Android
- **[Tokens Studio](https://tokens.studio)** — plugin Figma quản lý token, export JSON
- Tham khảo hệ có sẵn: [Material](https://m3.material.io), [Radix](https://www.radix-ui.com/themes), [shadcn/ui](https://ui.shadcn.com)

## Cách dùng

### HTML tĩnh

Design token = **CSS variables** đặt 1 chỗ, cả site dùng chung:

```css
:root {
  /* token gốc */
  --color-primary: #3b82f6;
  --space-2: 8px; --space-4: 16px;
  --radius: 8px;
  --font-size-body: 1rem;
}
.btn {
  background: var(--color-primary);
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius);
}
```

→ đổi 1 biến, cả site đổi theo (cùng cơ chế với [dark-mode.md](dark-mode.md)).

### React

- Token → file `theme.ts` hoặc Tailwind config (single source):

```js
// tailwind.config.js — map token vào Tailwind
theme: { extend: { colors: { primary: 'var(--color-primary)' } } }
```

- Component + **Storybook** để xem/tài liệu hoá từng trạng thái:

```bash
npx storybook@latest init
```

```jsx
// Button.stories.jsx
export default { component: Button };
export const Primary = { args: { variant: 'primary', children: 'Lưu' } };
```

### Next.js

- Token trong `globals.css` + Tailwind; component nền shadcn/ui rồi chỉnh theo token của bạn (xem [component-libraries.md](component-libraries.md)).
- Monorepo: tách `packages/ui` chứa design system dùng chung nhiều app (xem [monorepo.md](monorepo.md)).

## Figma → code

- Bật **Dev Mode** trong Figma: đo spacing, copy CSS/Tailwind của từng layer.
- Token đồng bộ 2 chiều: **Tokens Studio** (Figma) export JSON → **Style Dictionary** build ra CSS variables.
- Đừng kỳ vọng "bấm nút ra code xài được" — công cụ Figma-to-code sinh khung, bạn vẫn phải ráp vào hệ component thật.

## Mẹo

- **Bắt đầu bằng token, không phải component** — màu/spacing nhất quán đã giải quyết 80% "UI lệch".
- Spacing theo **thang 4px/8px** (4, 8, 12, 16, 24, 32...) — hết cãi nhau 13px hay 15px.
- Đặt tên token theo **vai trò** (`--color-primary`, `--color-danger`) thay vì theo màu (`--blue`) → đổi theme không gãy nghĩa.
- Ít mà nghiêm: 1 button 3 variant được dùng đúng > 10 variant mỗi nơi một kiểu.
