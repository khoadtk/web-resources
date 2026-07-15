# File upload & Storage

Cho người dùng tải ảnh/file lên và lưu trữ.

## Nơi lưu (storage)

- **[Cloudinary](https://cloudinary.com)** — lưu + tối ưu ảnh/video, có CDN + transform URL
- **[UploadThing](https://uploadthing.com)** — upload cho Next.js, dễ setup nhất
- **[AWS S3](https://aws.amazon.com/s3/)** / **[Cloudflare R2](https://developers.cloudflare.com/r2/)** — object storage rẻ, phổ biến
- **[Supabase Storage](https://supabase.com/storage)** — storage + quyền, đi kèm Supabase

## Cách upload (khái niệm)

- **Qua server** — client → API của bạn → storage. Kiểm soát tốt, tốn băng thông server.
- **Presigned URL** — server cấp URL tạm, client **upload thẳng** lên storage. Nhẹ cho server (khuyên dùng cho file lớn).

## Cách dùng

### HTML tĩnh

Form multipart hoặc `fetch` với `FormData`:

```html
<input type="file" id="f" accept="image/*">
<script>
  document.querySelector('#f').onchange = async e => {
    const fd = new FormData();
    fd.append('file', e.target.files[0]);
    await fetch('/api/upload', { method: 'POST', body: fd });
  };
</script>
```

### React

```jsx
const onUpload = async (file) => {
  const fd = new FormData();
  fd.append('file', file);
  const { url } = await fetch('/api/upload', { method: 'POST', body: fd })
    .then(r => r.json());
  setImageUrl(url);
};
<input type="file" onChange={e => onUpload(e.target.files[0])} />
```

Preview trước khi upload: `URL.createObjectURL(file)`.

### Next.js

**UploadThing** là nhanh nhất:

```bash
npm install uploadthing @uploadthing/react
```

Hoặc Route Handler + presigned URL (S3/R2) cho file lớn:

```jsx
// app/api/upload/route.ts nhận file, đẩy lên S3, trả URL
```

## Mẹo

- **Validate ở server**: loại file (MIME), kích thước tối đa — đừng tin `accept` của client.
- **Đặt tên file random** (uuid) → tránh trùng + tránh lộ tên gốc.
- File lớn → **presigned URL** (upload thẳng), không đi qua server.
- Ảnh → nén/resize trước khi lưu (Cloudinary/Sharp) để tiết kiệm.
- Không phục vụ file người dùng từ cùng domain nhạy cảm → cân nhắc CDN/subdomain riêng.
