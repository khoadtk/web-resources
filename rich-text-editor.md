# Rich text editor

Ô soạn thảo có định dạng: bold, heading, list, ảnh — như Notion/Google Docs thu nhỏ.

## Thư viện

- **[Tiptap](https://tiptap.dev)** — phổ biến nhất cho React/Vue, headless, nhiều extension (khuyên dùng)
- **[Lexical](https://lexical.dev)** — của Meta, hiệu năng tốt, kiến trúc mới
- **[Plate](https://platejs.org)** — editor trên nền Slate, tích hợp shadcn/ui đẹp
- **[Quill](https://quilljs.com)** — lâu đời, dùng được với JS thuần
- **[EasyMDE](https://github.com/Ionaru/easy-markdown-editor)** — editor Markdown đơn giản

## Chọn định dạng lưu

- **HTML** — dễ render lại, nhớ sanitize khi hiển thị (xem [security.md](security.md)).
- **JSON** (Tiptap/Lexical) — cấu trúc rõ, dễ xử lý, cần editor để render đẹp.
- **Markdown** — gọn, đọc thô được, hợp nội dung kỹ thuật.

## Cách dùng

### HTML tĩnh

**Quill** qua CDN — không cần build:

```html
<link href="https://cdn.jsdelivr.net/npm/quill@2/dist/quill.snow.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/quill@2/dist/quill.js"></script>
<div id="editor"></div>
<script>
  const quill = new Quill('#editor', { theme: 'snow' });
  // lấy nội dung: quill.getSemanticHTML() hoặc quill.getContents() (JSON Delta)
</script>
```

### React

**Tiptap** — headless, tự style toolbar:

```bash
npm install @tiptap/react @tiptap/starter-kit
```

```jsx
import { useEditor, EditorContent } from '@tiptap/react';
import StarterKit from '@tiptap/starter-kit';

const editor = useEditor({ extensions: [StarterKit], content: '<p>Xin chào</p>' });
<EditorContent editor={editor} />
// editor.getHTML() hoặc editor.getJSON() để lưu
```

Toolbar: gọi `editor.chain().focus().toggleBold().run()` từ nút của bạn.

### Next.js

- Editor cần DOM → **`'use client'`**, và tránh SSR mismatch:

```jsx
const editor = useEditor({ extensions: [StarterKit], immediatelyRender: false });
```

- Nội dung render cho người đọc (trang blog) → render HTML đã **sanitize** ở server, không cần load editor.

## Mẹo

- **Sanitize trước khi hiển thị** HTML người dùng nhập (DOMPurify) — editor là cửa XSS kinh điển.
- Chỉ bật **extension cần dùng** — editor "full option" nặng và rối.
- Upload ảnh trong editor → đi qua luồng [file-upload.md](file-upload.md) (đừng lưu base64 vào DB).
- Cần cộng tác realtime (nhiều người cùng gõ) → Tiptap + Yjs (xem [realtime.md](realtime.md)).
- Autosave bằng debounce (~1s sau khi ngừng gõ), đừng save mỗi phím.
