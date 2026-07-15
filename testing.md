# Testing

Kiểm thử để code đổi mà không sợ vỡ.

> Đi sâu riêng cho E2E/automation với Playwright: [automation-testing-playwright.md](automation-testing-playwright.md)

## 3 tầng test

- **Unit** — test 1 hàm/1 logic nhỏ, chạy nhanh nhất. Nhiều nhất.
- **Component / Integration** — test component render + tương tác đúng.
- **E2E (end-to-end)** — mô phỏng người dùng thật trên trình duyệt. Ít nhất, chậm nhất.

## Công cụ

- **[Vitest](https://vitest.dev)** — test runner nhanh, hợp Vite/React (thay Jest)
- **[Jest](https://jestjs.io)** — test runner kinh điển, nhiều dự án dùng
- **[Testing Library](https://testing-library.com)** — test component "như người dùng" (React Testing Library)
- **[Playwright](https://playwright.dev)** — E2E mạnh, đa trình duyệt (khuyên dùng)
- **[Cypress](https://cypress.io)** — E2E, trực quan, dễ bắt đầu

## Cách dùng

### HTML tĩnh

- JS thuần → test bằng **Vitest** (import hàm ra test), không cần framework.
- Luồng người dùng trên trang → **Playwright** mở trang thật và thao tác:

```js
import { test, expect } from '@playwright/test';
test('mở trang', async ({ page }) => {
  await page.goto('http://localhost:5500');
  await expect(page.getByRole('heading')).toBeVisible();
});
```

### React

**Vitest + React Testing Library** — test component theo hành vi:

```bash
npm install -D vitest @testing-library/react @testing-library/user-event
```

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('bấm nút tăng số', async () => {
  render(<Counter />);
  await userEvent.click(screen.getByRole('button', { name: /tăng/i }));
  expect(screen.getByText('1')).toBeInTheDocument();
});
```

Nguyên tắc: test **cái người dùng thấy/làm** (text, role, click), đừng test chi tiết nội bộ.

### Next.js

- Unit/component: Vitest + Testing Library như React.
- E2E: **Playwright** — chạy `next build && next start` rồi test trên URL thật:

```js
test('đăng nhập', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('a@b.com');
  await page.getByRole('button', { name: 'Đăng nhập' }).click();
  await expect(page).toHaveURL('/dashboard');
});
```

## Mẹo

- Ưu tiên **query theo role/label** (`getByRole`) → gần với accessibility + bền hơn `getByTestId`.
- Đừng chạy theo 100% coverage — test **luồng quan trọng** (thanh toán, đăng nhập) trước.
- E2E ít mà chắc; unit nhiều mà nhanh (mô hình "kim tự tháp test").
- Mock API bằng **[MSW](https://mswjs.io)** để test không phụ thuộc backend thật.
