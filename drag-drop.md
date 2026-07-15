# Drag & Drop

Kéo-thả: sắp xếp list, kanban board, kéo file vào trang.

## Thư viện

- **[dnd-kit](https://dndkit.com)** — chuẩn hiện nay cho React: nhẹ, a11y tốt, sortable (khuyên dùng)
- **[Pragmatic drag and drop](https://atlassian.design/components/pragmatic-drag-and-drop/)** — của Atlassian (Trello/Jira), hiệu năng cao, framework-agnostic
- **[SortableJS](https://sortablejs.github.io/Sortable/)** — JS thuần, dùng được không cần React
- **[react-dropzone](https://react-dropzone.js.org)** — riêng cho kéo **file** vào trang

## Hai loại kéo-thả (đừng nhầm)

- **Kéo sắp xếp UI** (list, kanban) → dnd-kit / SortableJS.
- **Kéo file từ máy vào trang** (upload) → HTML Drag & Drop API / react-dropzone (xem [file-upload.md](file-upload.md)).

## Cách dùng

### HTML tĩnh

**SortableJS** qua CDN cho list sắp xếp được:

```html
<script src="https://cdn.jsdelivr.net/npm/sortablejs@1/Sortable.min.js"></script>
<ul id="list"><li>A</li><li>B</li><li>C</li></ul>
<script>
  Sortable.create(document.getElementById('list'), {
    animation: 150,
    onEnd: e => console.log('từ', e.oldIndex, 'tới', e.newIndex),
  });
</script>
```

Kéo file vào trang bằng API sẵn có:

```js
zone.addEventListener('dragover', e => e.preventDefault());
zone.addEventListener('drop', e => {
  e.preventDefault();
  const files = [...e.dataTransfer.files];
});
```

### React

**dnd-kit** cho sortable list:

```bash
npm install @dnd-kit/core @dnd-kit/sortable
```

```jsx
import { DndContext, closestCenter } from '@dnd-kit/core';
import { SortableContext, arrayMove, useSortable } from '@dnd-kit/sortable';

function onDragEnd({ active, over }) {
  if (active.id !== over?.id)
    setItems(items => arrayMove(items, items.indexOf(active.id), items.indexOf(over.id)));
}
<DndContext collisionDetection={closestCenter} onDragEnd={onDragEnd}>
  <SortableContext items={items}>{items.map(...)}</SortableContext>
</DndContext>
```

Kéo file → **react-dropzone** (`useDropzone`).

### Next.js

- Kéo-thả cần DOM/event → component `'use client'`.
- Sau khi thả, **lưu thứ tự mới** về server (Server Action / API) — thêm field `order` trong DB.

## Mẹo

- **Lưu thứ tự** bằng field `order`/`position`; cập nhật optimistic rồi sync server.
- **A11y**: dnd-kit hỗ trợ kéo bằng bàn phím + screen reader — giữ tính năng này, đừng tắt.
- Mobile: cần `touch-action: none` trên item kéo để không xung đột scroll.
- Kanban nhiều cột = nhiều `SortableContext` trong 1 `DndContext`.
