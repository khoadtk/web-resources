# Automation testing với Playwright (deep-dive)

> Đào sâu từ [testing.md](testing.md) — tập trung vào E2E/automation testing bằng **Playwright**.
> Kết hợp BDD/Gherkin: [playwright-cucumber.md](playwright-cucumber.md)

## Setup

```bash
npm init playwright@latest
# tạo: playwright.config.ts, tests/, .github/workflows (nếu chọn), cài browser
```

```
tests/
├─ example.spec.ts
playwright.config.ts        # baseURL, projects (browser), retries, reporter
```

```ts
// playwright.config.ts — các option quan trọng nhất
export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'http://localhost:3000',   // → page.goto('/') thay vì URL đầy đủ
    trace: 'on-first-retry',            // ghi trace khi test fail rồi retry
  },
  webServer: {                          // tự bật app trước khi test
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

`webServer` là option đáng giá nhất: chạy `npx playwright test` là nó tự start app, khỏi mở 2 terminal.

## Cấu trúc một test

```ts
import { test, expect } from '@playwright/test';

test('đăng nhập thành công', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('a@b.com');
  await page.getByLabel('Mật khẩu').fill('secret123');
  await page.getByRole('button', { name: 'Đăng nhập' }).click();

  await expect(page).toHaveURL('/dashboard');
  await expect(page.getByRole('heading', { name: 'Xin chào' })).toBeVisible();
});
```

Hai điều làm Playwright "ít flaky" hơn Selenium/Cypress đời cũ — hiểu rõ để tận dụng:

1. **Auto-waiting** — `click()`/`fill()` tự chờ element hiện ra, enable, ổn định. Không cần `sleep`.
2. **Web-first assertions** — `expect(...).toBeVisible()` tự retry tới khi đúng (mặc định 5s), không chụp trạng thái 1 lần rồi fail.

## Locator — chọn element đúng cách

Thứ tự ưu tiên (giống Testing Library, tốt cho cả a11y — xem [accessibility.md](accessibility.md)):

```ts
page.getByRole('button', { name: 'Lưu' })   // ① tốt nhất — theo vai trò + tên
page.getByLabel('Email')                     // ② form field theo label
page.getByPlaceholder('Tìm kiếm...')
page.getByText('Chưa có dữ liệu')
page.getByTestId('cart-total')               // ③ khi không còn cách nào sạch hơn
page.locator('.btn-primary')                 // ④ tránh — vỡ khi đổi CSS
```

- Lọc & kết hợp: `page.getByRole('listitem').filter({ hasText: 'iPhone' }).getByRole('button', { name: 'Xoá' })`
- **Strict mode**: locator khớp ≥2 element là lỗi ngay — dùng `.first()`/`.nth()` có chủ đích, đừng để tình cờ.

## Chạy & debug

```bash
npx playwright test                    # headless, tất cả browser trong config
npx playwright test --ui               # UI mode — xem từng bước, time-travel (dùng nhiều nhất khi dev)
npx playwright test --headed --project=chromium
npx playwright test -g "đăng nhập"     # lọc theo tên test
npx playwright codegen localhost:3000  # thao tác trên browser → sinh code test
npx playwright show-report             # mở HTML report
npx playwright show-trace trace.zip    # mổ xẻ test fail: DOM, network, console từng bước
```

**Trace viewer** là killer feature: CI fail mà không tái hiện được ở máy mình → tải trace từ CI về, xem lại từng bước như video có DOM thật.

## Fixtures & Page Object Model

### POM — gom thao tác 1 trang vào 1 class

```ts
// pages/login.page.ts
export class LoginPage {
  constructor(private page: Page) {}
  async login(email: string, pass: string) {
    await this.page.goto('/login');
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Mật khẩu').fill(pass);
    await this.page.getByRole('button', { name: 'Đăng nhập' }).click();
  }
}
```

### Custom fixture — inject POM vào test

```ts
// fixtures.ts
export const test = base.extend<{ loginPage: LoginPage }>({
  loginPage: async ({ page }, use) => { await use(new LoginPage(page)); },
});
```

```ts
test('vào được dashboard', async ({ loginPage, page }) => {
  await loginPage.login('a@b.com', 'secret123');
  await expect(page).toHaveURL('/dashboard');
});
```

POM cho app nhiều màn; app nhỏ thì hàm helper thường là đủ — đừng over-engineer.

## Auth — đăng nhập 1 lần cho cả suite

Đăng nhập trong từng test vừa chậm vừa lặp. Chuẩn Playwright: **setup project + storageState**:

```ts
// auth.setup.ts — chạy 1 lần trước cả suite
setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  // ...đăng nhập...
  await page.context().storageState({ path: '.auth/user.json' }); // lưu cookie + localStorage
});
```

```ts
// playwright.config.ts
projects: [
  { name: 'setup', testMatch: /auth\.setup\.ts/ },
  { name: 'chromium', use: { storageState: '.auth/user.json' }, dependencies: ['setup'] },
]
```

Mọi test sau đó mở ra là đã đăng nhập sẵn. (`.auth/` nhớ cho vào `.gitignore` — xem [env-config.md](env-config.md).)

## Mock network — test không phụ thuộc backend

```ts
await page.route('**/api/products', route =>
  route.fulfill({ json: [{ id: 1, name: 'Test product' }] })
);
```

- Test UI state hiếm (API lỗi 500, list rỗng) mà không cần dựng data thật:

```ts
await page.route('**/api/products', route => route.fulfill({ status: 500 }));
await expect(page.getByText('Có lỗi xảy ra')).toBeVisible();  // test error state (ui-patterns.md)
```

- E2E "thật" thì đừng mock — mock nhiều quá thì không còn là end-to-end nữa. Cân bằng: luồng chính chạy backend thật, edge case dùng mock.

## Chạy trên CI (GitHub Actions)

```yaml
# .github/workflows/e2e.yml
- uses: actions/setup-node@v4
- run: npm ci
- run: npx playwright install --with-deps chromium
- run: npx playwright test
- uses: actions/upload-artifact@v4       # giữ report + trace khi fail
  if: failure()
  with: { name: playwright-report, path: playwright-report/ }
```

- Config cho CI: `retries: process.env.CI ? 2 : 0`, `workers: 1` nếu app không chịu được song song.
- Gắn vào luồng PR như [git-workflow.md](git-workflow.md): E2E pass mới cho merge.

## Test với Next.js

- `webServer.command: 'npm run build && npm run start'` trên CI — test bản **production build**, không phải dev server (dev chậm + hành vi khác: không cache, compile theo request).
- Trang static/ISR: nhớ hành vi stale-while-revalidate khi assert nội dung "mới nhất" (xem bài học rendering trong `learning/nextjs-rendering.html`).
- Server Action/form: test như user thật — fill + click + assert kết quả; không cần biết bên dưới là action hay API.

## Chống flaky — checklist

- [ ] **Cấm `page.waitForTimeout(3000)`** — nguồn flaky số 1; thay bằng assertion tự retry (`toBeVisible`, `toHaveURL`...).
- [ ] Assert bằng **web-first assertion**, không lấy giá trị ra rồi so (`expect(await el.textContent()).toBe(...)` là chụp 1 lần — dễ fail sớm).
- [ ] **Test độc lập nhau** — không test nào phụ thuộc test trước chạy xong; Playwright chạy song song mặc định, thứ tự không đảm bảo.
- [ ] Data test **tự tạo tự dọn** (qua API/DB seed trong `beforeEach`), đừng dựa vào data có sẵn trên môi trường.
- [ ] Locator theo **role/label**, không theo class CSS.
- [ ] Bật `trace: 'on-first-retry'` — fail là có bằng chứng, khỏi đoán.
- [ ] Ít mà chắc: E2E chỉ phủ **luồng quan trọng** (đăng nhập, thanh toán, luồng chính) — phần còn lại để unit/component test (kim tự tháp trong [testing.md](testing.md)).
