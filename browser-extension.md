# Browser extension

Làm extension Chrome/Edge/Firefox: nút trên toolbar, can thiệp trang web, tự động hoá.

## Kiến trúc (Manifest V3)

| Phần | Chạy ở đâu | Làm gì |
| --- | --- | --- |
| **manifest.json** | — | Khai báo tên, quyền, các phần |
| **Popup** | Cửa sổ nhỏ khi bấm icon | UI chính của extension |
| **Content script** | Nhúng vào trang web đang mở | Đọc/sửa DOM của trang |
| **Background (service worker)** | Chạy nền | Sự kiện, gọi API, lưu trữ |
| **Options page** | Trang cài đặt | Cấu hình |

## Công cụ

- **[WXT](https://wxt.dev)** — framework extension hiện đại, HMR, TS, đa trình duyệt (khuyên dùng)
- **[Plasmo](https://plasmo.com)** — framework extension cho React
- **[CRXJS](https://crxjs.dev)** — plugin Vite cho extension
- Tài liệu chuẩn: [Chrome Extensions docs](https://developer.chrome.com/docs/extensions)

## Cách dùng

### HTML tĩnh (vanilla — hiểu bản chất)

Extension tối thiểu chỉ cần 2 file:

```json
// manifest.json
{
  "manifest_version": 3,
  "name": "My Extension", "version": "1.0",
  "action": { "default_popup": "popup.html" },
  "content_scripts": [{ "matches": ["<all_urls>"], "js": ["content.js"] }],
  "permissions": ["storage"]
}
```

```js
// content.js — chạy trong trang web
document.body.style.border = '3px solid red';
```

Nạp thử: `chrome://extensions` → bật Developer mode → **Load unpacked** → chọn thư mục.

### React

Popup/options là trang web bình thường → viết bằng React qua **WXT/Plasmo**:

```bash
npx wxt@latest init my-ext   # chọn template React
npm run dev                  # tự reload extension khi sửa code
```

```jsx
// entrypoints/popup/App.tsx — React như thường
function App() { return <button onClick={...}>Lưu trang này</button>; }
```

### Next.js

Next.js **không dùng cho extension** (extension không có server). Nhưng pattern hay gặp: extension làm client gọi về **API Next.js** của bạn (auth, lưu data — xem [api-layer.md](api-layer.md), [auth.md](auth.md)).

## Giao tiếp giữa các phần

```js
// content script → background
chrome.runtime.sendMessage({ type: 'SAVE', data });

// background nhận
chrome.runtime.onMessage.addListener((msg, sender, reply) => { ... });

// lưu trữ chung
await chrome.storage.local.set({ key: value });
```

## Mẹo

- **Xin quyền tối thiểu** trong `permissions` — xin `<all_urls>` bừa sẽ khó qua review store.
- MV3: background là **service worker**, có thể bị tắt bất cứ lúc nào → đừng giữ state trong biến, dùng `chrome.storage`.
- Content script sống trong trang người khác → namespace CSS cẩn thận, đừng phá style trang (dùng Shadow DOM nếu inject UI).
- Firefox dùng chuẩn `browser.*` (promise) — WXT tự lo khác biệt đa trình duyệt.
- Đăng store: [Chrome Web Store](https://chrome.google.com/webstore/devconsole) phí $5 một lần, review vài ngày.
