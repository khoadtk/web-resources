# Tables & Data grid

Bảng dữ liệu: sort, filter, phân trang, chọn dòng, dữ liệu lớn.

## Thư viện

- **[TanStack Table](https://tanstack.com/table)** — headless, mạnh nhất, tự style (khuyên dùng; shadcn/ui có sẵn recipe DataTable)
- **[AG Grid](https://ag-grid.com)** — "full option" kiểu Excel: edit inline, group, pivot (bản free đủ dùng)
- **[Grid.js](https://gridjs.io)** — nhẹ, dùng được với JS thuần
- Virtualization (bảng ngàn dòng): **[TanStack Virtual](https://tanstack.com/virtual)**

## Bảng thường vs Data grid

- **≤ vài chục dòng, chỉ hiển thị** → `<table>` HTML + CSS là đủ, đừng cài lib.
- **Sort/filter/phân trang/chọn dòng** → TanStack Table.
- **Edit như Excel, group, export** → AG Grid.

## Cách dùng

### HTML tĩnh

`<table>` chuẩn + **Grid.js** khi cần sort/search:

```html
<div id="table"></div>
<link href="https://cdn.jsdelivr.net/npm/gridjs/dist/theme/mermaid.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/gridjs/dist/gridjs.umd.js"></script>
<script>
  new gridjs.Grid({
    columns: ['Tên', 'Email'],
    data: [['Khoa', 'a@b.com']],
    sort: true, search: true, pagination: { limit: 10 },
  }).render(document.getElementById('table'));
</script>
```

### React

**TanStack Table** — headless, bạn render markup:

```bash
npm install @tanstack/react-table
```

```jsx
import { useReactTable, getCoreRowModel, getSortedRowModel, flexRender } from '@tanstack/react-table';

const table = useReactTable({
  data, columns,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
});
// render: table.getHeaderGroups() / table.getRowModel().rows + flexRender
```

Dùng Tailwind/shadcn → lấy sẵn **DataTable** của shadcn/ui (đã ráp TanStack Table).

### Next.js

- Data lớn → **phân trang / sort / filter ở server** (query DB với `LIMIT/OFFSET` hoặc cursor), bảng chỉ hiển thị — đừng kéo hết ngàn dòng về client.
- TanStack Table hỗ trợ chế độ `manualPagination`/`manualSorting` cho kiểu này.
- Component bảng tương tác → `'use client'`; fetch trang đầu ở Server Component rồi truyền xuống.

## Mẹo

- **Bảng dữ liệu ≠ layout**: chỉ dùng `<table>` cho dữ liệu dạng bảng; có `<th scope>`, `<caption>` cho a11y.
- Ngàn dòng không phân trang được → **virtualize** (chỉ render dòng trong khung nhìn).
- Cột nhiều → cho ẩn/hiện cột; mobile → đổi sang dạng card thay vì bóp bảng.
- Kết hợp TanStack Query để cache từng trang dữ liệu (xem [data-state.md](data-state.md)).
