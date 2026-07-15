# Forms & Validation

Xử lý form: nhập liệu, kiểm tra hợp lệ (validation), báo lỗi.

## Thư viện

- **[React Hook Form](https://react-hook-form.com)** — quản lý form React nhẹ, nhanh, ít re-render
- **[Zod](https://zod.dev)** — định nghĩa schema + validate, sinh type TS luôn
- **[Yup](https://github.com/jquense/yup)** — schema validation (giống Zod, cũ hơn)
- **[Valibot](https://valibot.dev)** — như Zod nhưng nhẹ hơn nhiều

## Cách dùng

### HTML tĩnh

Dùng **validation sẵn của HTML** — không cần JS cho case đơn giản:

```html
<form>
  <input type="email" required placeholder="Email">
  <input type="text" minlength="3" maxlength="20" required>
  <input type="password" pattern=".{8,}" title="Tối thiểu 8 ký tự">
  <button>Gửi</button>
</form>
```

Cần logic phức tạp hơn → JS + Constraint Validation API (`input.setCustomValidity(...)`).

### React

**React Hook Form + Zod** là combo chuẩn:

```bash
npm install react-hook-form zod @hookform/resolvers
```

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Email không hợp lệ'),
  age: z.number().min(18, 'Phải từ 18 tuổi'),
});

function Form() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  });
  return (
    <form onSubmit={handleSubmit(data => console.log(data))}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
    </form>
  );
}
```

### Next.js

- Form phía client: giống React ở trên.
- **Server Actions** (App Router) — validate & xử lý ngay trên server, dùng chung schema Zod:

```jsx
// app/actions.ts
'use server';
import { z } from 'zod';
const schema = z.object({ email: z.string().email() });

export async function submit(formData: FormData) {
  const parsed = schema.safeParse({ email: formData.get('email') });
  if (!parsed.success) return { error: parsed.error.flatten() };
  // ...lưu DB
}
```

## Mẹo

- **Dùng chung 1 schema Zod** cho cả client và server → validate 2 lớp, không lặp code.
- Validate **khi blur/submit**, đừng báo lỗi ngay khi user vừa gõ ký tự đầu.
- Luôn validate lại ở **server** — validation client chỉ để UX, không phải bảo mật.
- Gắn lỗi với input bằng `aria-describedby` cho accessibility.
