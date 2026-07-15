# Component libraries

Thư viện component dựng sẵn (button, dialog, dropdown...) để không phải code lại từ đầu.

## Các lựa chọn

| Thư viện | Đặc điểm |
| --- | --- |
| **[shadcn/ui](https://ui.shadcn.com)** | Copy code vào project, bạn sở hữu & sửa được — hợp Next.js |
| **[Radix UI](https://radix-ui.com)** | Component "không style" (headless), lo sẵn logic + a11y |
| **[Headless UI](https://headlessui.com)** | Của team Tailwind, headless, React + Vue |
| **[MUI](https://mui.com)** | Material Design, đầy đủ, style sẵn |
| **[Chakra UI](https://chakra-ui.com)** | Dễ dùng, style bằng prop |
| **[Ark UI](https://ark-ui.com)** | Headless, đa framework (React/Vue/Solid) |

## Cách dùng

### HTML tĩnh

Các lib trên đều cần React/Vue. Với HTML thuần, dùng loại **không cần build**:

- **[Preline](https://preline.co)** / **[Flowbite](https://flowbite.com)** — component Tailwind, chỉ cần copy HTML + 1 file JS.

```html
<link href="https://cdn.jsdelivr.net/npm/flowbite@2/dist/flowbite.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/flowbite@2/dist/flowbite.min.js"></script>
<!-- rồi copy markup component từ docs -->
```

### React

Radix (headless) — bạn tự style, nó lo logic + accessibility:

```bash
npm install @radix-ui/react-dialog
```

```jsx
import * as Dialog from '@radix-ui/react-dialog';
<Dialog.Root>
  <Dialog.Trigger>Mở</Dialog.Trigger>
  <Dialog.Portal>...</Dialog.Portal>
</Dialog.Root>
```

### Next.js

**shadcn/ui** là lựa chọn phổ biến nhất — nó cài component vào code của bạn:

```bash
npx shadcn@latest init
npx shadcn@latest add button dialog
```

```jsx
import { Button } from '@/components/ui/button';
<Button variant="outline">Click</Button>
```

## Mẹo chọn

- Muốn **kiểm soát style tối đa** + Next.js → **shadcn/ui** (trên nền Radix).
- Muốn **build UI riêng từ đầu** → Radix / Headless UI (headless).
- Muốn **nhanh, có sẵn nhiều** → MUI / Chakra.
- **HTML tĩnh** → Flowbite / Preline.
