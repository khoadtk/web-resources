# Email

Gửi email: xác nhận đăng ký, reset mật khẩu, thông báo, marketing.

## Dịch vụ gửi

- **[Resend](https://resend.com)** — API gửi email hiện đại, dễ dùng (khuyên cho dự án mới)
- **[SendGrid](https://sendgrid.com)** / **[Mailgun](https://mailgun.com)** — lâu đời, mạnh, nhiều volume
- **[Amazon SES](https://aws.amazon.com/ses/)** — rẻ nhất khi gửi số lượng lớn
- **[Nodemailer](https://nodemailer.com)** — thư viện gửi qua SMTP (dùng với bất kỳ SMTP nào)
- **[Postmark](https://postmarkapp.com)** — chuyên transactional, deliverability tốt

## Soạn nội dung email

- **[React Email](https://react.email)** — viết email bằng React component (khuyên dùng)
- **[MJML](https://mjml.io)** — markup email responsive, biên dịch ra HTML
- Email HTML "khó tính" (client Outlook cũ) → dùng template có sẵn, đừng tự CSS phức tạp.

## Cách dùng

### HTML tĩnh

Web tĩnh **không tự gửi email được** (cần server/khoá bí mật). Cách phổ biến:

- **[Formspree](https://formspree.io)** / **[Web3Forms](https://web3forms.com)** — nhận form → gửi email hộ, chỉ cần `action`:

```html
<form action="https://formspree.io/f/xxxx" method="POST">
  <input name="email" type="email">
  <button>Gửi</button>
</form>
```

### React (SPA)

SPA gọi tới **API backend**, backend mới gọi dịch vụ email (giữ API key ở server).

### Next.js

**Resend + React Email** trong Route Handler / Server Action:

```bash
npm install resend react-email @react-email/components
```

```jsx
// app/api/send/route.ts
import { Resend } from 'resend';
const resend = new Resend(process.env.RESEND_API_KEY);
export async function POST() {
  await resend.emails.send({
    from: 'no-reply@domain.com',
    to: 'user@mail.com',
    subject: 'Chào mừng',
    html: '<p>Xin chào 👋</p>',
  });
  return Response.json({ ok: true });
}
```

## Mẹo

- **API key chỉ ở server** — không bao giờ lộ ra client.
- Cấu hình **SPF / DKIM / DMARC** cho domain → email không rơi vào spam.
- Tách **transactional** (reset mật khẩu — phải tới) và **marketing** (có nút unsubscribe).
- Test bằng **[Mailtrap](https://mailtrap.io)** để không lỡ gửi vào mail thật khi dev.
- Luôn có **bản text** kèm bản HTML.
