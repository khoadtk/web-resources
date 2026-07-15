# Search

Tìm kiếm trong app: full-text, gợi ý tức thời, lọc.

## Cách làm (từ nhẹ → nặng)

- **Client-side** — data nhỏ, lọc ngay trên trình duyệt (Fuse.js).
- **Database full-text** — Postgres `tsvector`, đủ cho phần lớn app vừa.
- **Search engine** — Meilisearch/Algolia: nhanh, typo-tolerant, gợi ý tức thời.

## Công cụ

- **[Fuse.js](https://fusejs.io)** — fuzzy search client, không cần server
- **[Meilisearch](https://meilisearch.com)** — search engine mã nguồn mở, dễ self-host (khuyên dùng)
- **[Algolia](https://algolia.com)** — search as-a-service, cực nhanh, có sẵn UI
- **[Typesense](https://typesense.org)** — mã nguồn mở, giống Algolia
- **[Orama](https://orama.com)** — search chạy ngay trong JS/edge

## Cách dùng

### HTML tĩnh

**Fuse.js** lọc mảng dữ liệu có sẵn:

```html
<script src="https://cdn.jsdelivr.net/npm/fuse.js@7/dist/fuse.min.js"></script>
<script>
  const fuse = new Fuse(items, { keys: ['title', 'tag'] });
  const results = fuse.search('nextjs').map(r => r.item);
</script>
```

### React

- Data nhỏ → Fuse.js + `useMemo`.
- Data lớn / gợi ý tức thời → Algolia có sẵn UI:

```bash
npm install react-instantsearch algoliasearch
```

Nhớ **debounce** ô input để không query mỗi phím gõ.

### Next.js

- Meilisearch/Algolia: index dữ liệu từ server, client query qua **search-only key** (không dùng admin key ở client).
- Postgres full-text: query thẳng trong Route Handler/Server Component:

```sql
SELECT * FROM posts WHERE to_tsvector(title) @@ plainto_tsquery('nextjs');
```

## Mẹo

- **Debounce** (~250–300ms) ô search → giảm số query.
- **Typo-tolerance** rất đáng có (Meili/Algolia lo sẵn) — user gõ sai vẫn ra.
- Chỉ đưa **field cần** vào index → nhẹ & nhanh.
- Search ngữ nghĩa (hiểu ý, không chỉ khớp chữ) → dùng embeddings, xem [ai-integration.md](ai-integration.md).
- Ô search cần **accessibility**: `role="search"`, điều hướng kết quả bằng bàn phím (xem [accessibility.md](accessibility.md)).
