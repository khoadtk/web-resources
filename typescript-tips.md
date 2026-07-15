# TypeScript tips (cho web)

Các mẹo TS hay dùng khi làm frontend React/Next.

## Nền tảng nên nhớ

- **`type` vs `interface`** — dùng `type` cho union/utility, `interface` cho object mở rộng. Chọn 1 kiểu, nhất quán.
- **Tránh `any`** — dùng `unknown` rồi thu hẹp, hoặc khai kiểu đúng.
- **`as const`** — biến literal thành readonly + kiểu hẹp:

```ts
const ROLES = ['admin', 'user'] as const;
type Role = typeof ROLES[number];   // 'admin' | 'user'
```

## Utility types hay dùng

```ts
Partial<T>      // mọi field optional
Required<T>     // mọi field bắt buộc
Pick<T, 'a'|'b'>// lấy vài field
Omit<T, 'id'>   // bỏ vài field
Record<K, V>    // object { [k]: V }
ReturnType<typeof fn>  // kiểu trả về của hàm
```

## Sinh type từ nguồn thật (đừng viết tay)

- Từ **Zod schema** → `z.infer<typeof schema>` (1 nguồn cho cả validate + type).
- Từ **JSON API** → [transform.tools/json-to-typescript](https://transform.tools/json-to-typescript).
- Từ **Prisma** → type tự sinh theo schema DB.

## Cách dùng

### HTML tĩnh

- JS thuần vẫn nhận gợi ý type qua **JSDoc** (không cần build):

```js
/** @param {string} name */
function greet(name) { return `Hi ${name}`; }
```

- Hoặc viết `.ts` rồi biên dịch bằng `tsc`.

### React

Kiểu cho props + hook:

```tsx
type Props = { title: string; onClose?: () => void };
function Modal({ title, onClose }: Props) { /* ... */ }

const [user, setUser] = useState<User | null>(null);
const ref = useRef<HTMLInputElement>(null);
```

- Event: `React.ChangeEvent<HTMLInputElement>`, `React.FormEvent`.

### Next.js

- Bật **`strict: true`** trong `tsconfig.json` (mặc định khi tạo mới).
- Route params/searchParams có kiểu sẵn trong page props.
- Dùng type Prisma/Zod xuyên suốt server↔client (đây là lợi thế fullstack TS → cân nhắc **tRPC**, xem [api-layer.md](api-layer.md)).

## Mẹo

- **Bật `strict`** ngay từ đầu — thêm sau rất mệt.
- Đừng ép kiểu bằng `as` bừa để "cho hết đỏ" → dễ giấu bug thật.
- Một nguồn sự thật cho type: Zod/Prisma sinh ra, đừng khai trùng tay.
- `satisfies` để vừa check kiểu vừa giữ literal hẹp (TS 4.9+).
