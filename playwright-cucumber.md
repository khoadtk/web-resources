# Playwright + Cucumber-js (BDD)

> Nối tiếp [automation-testing-playwright.md](automation-testing-playwright.md) — viết test theo kiểu BDD: kịch bản Gherkin cho người đọc, Playwright chạy bên dưới.

## BDD giải quyết gì?

Test thường chỉ dev đọc được. **BDD (Behavior-Driven Development)** tách test làm 2 lớp:

- **Feature file (Gherkin)** — kịch bản viết bằng ngôn ngữ tự nhiên: `Given / When / Then`. QA, BA, PO đọc và góp ý được.
- **Step definitions** — code Playwright "dịch" từng câu Gherkin thành thao tác thật.

```gherkin
Feature: Đăng nhập
  Scenario: Đăng nhập thành công
    Given tôi đang ở trang đăng nhập
    When tôi đăng nhập với email "a@b.com" và mật khẩu "secret123"
    Then tôi thấy trang dashboard
```

**Chỉ đáng dùng khi feature file có người-không-code đọc thật** (QA viết kịch bản, PO review acceptance criteria). Nếu chỉ dev đọc test → Playwright thuần ít tốn công hơn; Gherkin lúc đó chỉ là một tầng gián tiếp thêm việc.

## Hai con đường — chọn trước khi setup

| | Cách chạy | Được | Mất |
| --- | --- | --- | --- |
| **1. cucumber-js làm runner** | `npx cucumber-js` | Hệ sinh thái Cucumber chuẩn, report Cucumber | Mất toàn bộ tính năng runner của Playwright: fixtures, `--ui`, trace config, `webServer`, parallel thông minh |
| **2. playwright-bdd** (khuyên dùng) | Sinh spec từ `.feature` rồi `npx playwright test` | Giữ nguyên mọi thứ của Playwright runner + vẫn viết Gherkin | Thêm 1 bước generate |

Dưới đây là cách 1 (chuẩn Cucumber, gặp nhiều ở dự án/công ty), rồi tới cách 2.

## Cách 1: cucumber-js làm runner

### Setup

```bash
npm i -D @cucumber/cucumber playwright tsx @playwright/test
# @playwright/test chỉ để mượn expect; runner là cucumber-js
```

```js
// cucumber.js — config
module.exports = {
  default: {
    paths: ['features/**/*.feature'],
    require: ['features/steps/**/*.ts', 'features/support/**/*.ts'],
    requireModule: ['tsx/cjs'],          // chạy TS trực tiếp
    format: ['progress', 'html:reports/cucumber.html'],
  },
};
```

```
features/
├─ login.feature
├─ steps/
│  └─ login.steps.ts
└─ support/
   └─ world.ts          # quản lý browser/page
```

### World — chỗ giữ browser & page

Cucumber tạo **một World instance cho mỗi scenario** — đây là "this" trong step, nơi giữ `page`:

```ts
// features/support/world.ts
import { setWorldConstructor, World, Before, After, BeforeAll, AfterAll } from '@cucumber/cucumber';
import { chromium, Browser, BrowserContext, Page } from 'playwright';

let browser: Browser;

export class CustomWorld extends World {
  context!: BrowserContext;
  page!: Page;
}
setWorldConstructor(CustomWorld);

BeforeAll(async () => { browser = await chromium.launch(); });        // 1 browser cho cả suite
Before(async function (this: CustomWorld) {                           // mỗi scenario 1 context sạch
  this.context = await browser.newContext({ baseURL: 'http://localhost:3000' });
  this.page = await this.context.newPage();
});
After(async function (this: CustomWorld) { await this.context.close(); });
AfterAll(async () => { await browser.close(); });
```

Lưu ý: đây là **Playwright library API** (`chromium.launch()`), không phải `@playwright/test` — vì runner giờ là Cucumber.

### Step definitions

```ts
// features/steps/login.steps.ts
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';   // expect dùng độc lập được
import type { CustomWorld } from '../support/world';

Given('tôi đang ở trang đăng nhập', async function (this: CustomWorld) {
  await this.page.goto('/login');
});

When('tôi đăng nhập với email {string} và mật khẩu {string}',
  async function (this: CustomWorld, email: string, pass: string) {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Mật khẩu').fill(pass);
    await this.page.getByRole('button', { name: 'Đăng nhập' }).click();
  });

Then('tôi thấy trang dashboard', async function (this: CustomWorld) {
  await expect(this.page).toHaveURL('/dashboard');
});
```

`{string}`, `{int}`... là **Cucumber expressions** — tham số từ câu Gherkin rơi thẳng vào hàm.

### Chạy

```bash
npx cucumber-js                        # tất cả feature
npx cucumber-js --tags "@smoke"        # theo tag: @smoke trên Feature/Scenario
npx cucumber-js features/login.feature:3   # đúng 1 scenario theo dòng
```

Nhớ tự bật app trước (không còn `webServer` của Playwright) — dùng `start-server-and-test` hoặc script riêng.

### Gherkin nâng cao hay dùng

```gherkin
# Scenario Outline — 1 kịch bản chạy nhiều bộ dữ liệu
Scenario Outline: Đăng nhập sai
  When tôi đăng nhập với email "<email>" và mật khẩu "<pass>"
  Then tôi thấy lỗi "<error>"
  Examples:
    | email   | pass  | error                  |
    | a@b.com | sai   | Sai mật khẩu           |
    | x       | 12345 | Email không hợp lệ     |

# Background — bước chung chạy trước mọi scenario trong file
Background:
  Given tôi đã đăng nhập
```

Gherkin còn hỗ trợ **tiếng Việt chính thức**: thêm `# language: vi` đầu file rồi dùng `Tính năng / Kịch bản / Cho / Khi / Thì` — feature file thuần Việt cho team đọc.

## Cách 2: playwright-bdd (giữ Playwright runner)

```bash
npm i -D playwright-bdd
```

```ts
// playwright.config.ts
import { defineBddConfig } from 'playwright-bdd';
const testDir = defineBddConfig({
  features: 'features/**/*.feature',
  steps: 'features/steps/**/*.ts',
});
export default defineConfig({ testDir, /* webServer, trace... như thường */ });
```

```ts
// steps viết bằng fixtures của Playwright — không cần World tự chế
import { createBdd } from 'playwright-bdd';
const { Given, When, Then } = createBdd();

Given('tôi đang ở trang đăng nhập', async ({ page }) => {
  await page.goto('/login');
});
```

```bash
npx bddgen && npx playwright test    # sinh spec từ .feature rồi chạy như Playwright thường
```

→ Có lại đủ: `--ui`, trace viewer, `webServer`, storageState, parallel. Đây là lựa chọn tốt nhất nếu bạn bắt đầu mới và không bị ràng buộc tooling Cucumber có sẵn.

## Mẹo

- **Viết step theo hành vi, không theo UI**: "tôi đăng nhập với..." tốt hơn "tôi điền ô #email rồi bấm nút .submit" — UI đổi thì chỉ sửa step definition, feature file sống lâu.
- **Tái dùng step** giữa các feature — trước khi viết step mới, tìm step cũ khớp được; step trùng lặp là mùi của BDD xấu.
- Step definitions mỏng: gọi vào **Page Object** ([automation-testing-playwright.md](automation-testing-playwright.md)) thay vì nhét locator thẳng vào step.
- Tag chiến lược: `@smoke` chạy mỗi PR, full suite chạy nightly (cron trong CI — xem [deploy-hosting.md](deploy-hosting.md)).
- Checklist chống flaky bên file Playwright **áp dụng nguyên xi** ở đây — Cucumber không đổi bản chất auto-waiting/assertion.
- Đừng viết feature file kiểu "test script trá hình" (20 bước When liên tiếp) — mỗi scenario nên đọc như một câu chuyện nghiệp vụ ngắn.
