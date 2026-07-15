# Payment

Nhận thanh toán: một lần, gói subscription.

## Dịch vụ

- **[Stripe](https://stripe.com)** — chuẩn vàng quốc tế, tài liệu tốt (thẻ, subscription, invoice)
- **[Lemon Squeezy](https://lemonsqueezy.com)** / **[Paddle](https://paddle.com)** — "merchant of record", lo thuế/VAT giúp (hợp bán SaaS)
- **[PayPal](https://developer.paypal.com)** — phổ biến, nhiều người có sẵn ví
- **VN**: [VNPay](https://vnpay.vn), [MoMo](https://developers.momo.vn), [ZaloPay](https://docs.zalopay.vn) — cổng nội địa

## Nguyên tắc bảo mật (quan trọng)

- **Không bao giờ xử lý số thẻ trên server của bạn** → dùng UI của Stripe (Checkout/Elements).
- **Số tiền tính ở server**, không tin số client gửi lên.
- Xác nhận đơn hàng qua **webhook** (Stripe gọi về server báo "đã trả tiền"), không dựa vào redirect thành công.
- **Secret key** chỉ ở server; client chỉ dùng **publishable key**.

## Cách dùng

### HTML tĩnh

Dùng **Stripe Checkout** (trang thanh toán do Stripe host) — không tự làm form thẻ:

```html
<script src="https://js.stripe.com/v3/"></script>
<script>
  const stripe = Stripe('pk_test_...');   // publishable key
  // gọi API server tạo session rồi redirect:
  const { id } = await fetch('/create-checkout', { method: 'POST' }).then(r => r.json());
  stripe.redirectToCheckout({ sessionId: id });
</script>
```

(Cần 1 backend nhỏ để tạo Checkout Session bằng secret key.)

### React

Dùng `@stripe/react-stripe-js` + Elements để nhúng form thẻ an toàn:

```bash
npm install @stripe/stripe-js @stripe/react-stripe-js
```

### Next.js

- **Route Handler** tạo Checkout Session (secret key ở server):

```jsx
// app/api/checkout/route.ts
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
export async function POST() {
  const session = await stripe.checkout.sessions.create({ /* line_items, mode */ });
  return Response.json({ url: session.url });
}
```

- **Webhook** (`app/api/webhook/route.ts`) xác thực chữ ký Stripe → cập nhật DB khi trả tiền xong.

## Mẹo

- Test bằng **test mode** + thẻ `4242 4242 4242 4242`.
- **Idempotency**: webhook có thể gọi nhiều lần → xử lý sao cho lặp lại không sai (đừng cộng tiền 2 lần).
- Lưu **trạng thái đơn** trong DB; nguồn sự thật là webhook, không phải trang "cảm ơn".
- Bán ra nước ngoài mà ngại thuế → cân nhắc Lemon Squeezy/Paddle.
