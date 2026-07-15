# Feature flags & A/B testing

Bật/tắt tính năng không cần deploy lại, và thử nghiệm 2 phiên bản xem cái nào tốt hơn.

## Dùng để làm gì

- **Rollout dần** — bật tính năng mới cho 5% user, ổn thì tăng dần.
- **Kill switch** — tính năng lỗi giữa đêm → tắt bằng 1 nút, không cần hotfix.
- **A/B test** — nửa user thấy nút A, nửa thấy nút B → đo cái nào chuyển đổi tốt hơn.
- **Beta/quyền** — tính năng chỉ cho nhóm user nhất định.

## Dịch vụ / thư viện

- **[PostHog](https://posthog.com)** — flags + A/B test + analytics chung một chỗ, free tier tốt (khuyên dùng)
- **[LaunchDarkly](https://launchdarkly.com)** — chuẩn enterprise
- **[Unleash](https://getunleash.io)** / **[Flagsmith](https://flagsmith.com)** — mã nguồn mở, self-host
- **[Vercel Flags](https://vercel.com/docs/feature-flags)** — SDK flags cho Next.js
- Tự làm mức đơn giản: bảng `feature_flags` trong DB + env var.

## Cách dùng

### HTML tĩnh

Mức đơn giản nhất — config JSON tải lúc chạy:

```js
const flags = await fetch('/flags.json').then(r => r.json());
if (flags.newBanner) showBanner();
```

Hoặc nhúng PostHog snippet → dùng `posthog.isFeatureEnabled('new-banner')`.

### React

**PostHog** SDK:

```bash
npm install posthog-js
```

```jsx
import { useFeatureFlagEnabled } from 'posthog-js/react';

function Checkout() {
  const newFlow = useFeatureFlagEnabled('new-checkout');
  return newFlow ? <NewCheckout /> : <OldCheckout />;
}
```

A/B test = flag nhiều variant + đo sự kiện:

```js
const variant = posthog.getFeatureFlag('cta-test'); // 'control' | 'variant-b'
posthog.capture('cta_clicked'); // PostHog tự so sánh theo variant
```

### Next.js

- Đánh giá flag **ở server** → không nhấp nháy UI (client-side flag hay bị "lóe" bản cũ):

```jsx
// Server Component
import { PostHog } from 'posthog-node';
const ph = new PostHog(process.env.POSTHOG_KEY);
const newFlow = await ph.isFeatureEnabled('new-checkout', userId);
return newFlow ? <NewCheckout /> : <OldCheckout />;
```

- Vercel Flags SDK tích hợp sẵn pattern này cho App Router.

## Mẹo

- Flag phải có **default an toàn** — dịch vụ flag sập thì app vẫn chạy bản cũ.
- **Dọn flag chết**: tính năng đã bật 100% lâu rồi → xoá flag + nhánh code cũ, đừng để rác `if` khắp nơi.
- A/B test cần **đủ mẫu + 1 chỉ số chính** định trước — đừng nhìn 10 chỉ số rồi chọn cái đẹp.
- Giữ **tên flag theo quy ước** (`checkout-new-flow`) và ghi chú chủ nhân/ngày hết hạn.
- Flag phân theo user → cần **định danh ổn định** (userId, không phải random mỗi lần vào — xem [auth.md](auth.md)).
