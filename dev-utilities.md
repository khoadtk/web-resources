# Dev utilities

Các công cụ web nhỏ nhưng hay dùng khi code frontend.

## Chuyển đổi & sinh code

- **[transform.tools](https://transform.tools)** — chuyển đổi mọi thứ: JSON→TS type, HTML→JSX, CSS→JS, SVG→JSX...
- **[SVGOMG](https://jakearchibald.github.io/svgomg/)** — tối ưu/nén SVG
- **[SVG to JSX](https://svg2jsx.com)** — SVG → React component
- **[JSON to TS](https://transform.tools/json-to-typescript)** — sinh type từ JSON API

## Regex & text

- **[Regex101](https://regex101.com)** — viết + test regex, có giải thích từng phần
- **[RegExr](https://regexr.com)** — tương tự, trực quan

## Tra cứu & tương thích

- **[Can I use](https://caniuse.com)** — feature CSS/JS trình duyệt nào hỗ trợ
- **[MDN](https://developer.mozilla.org)** — tài liệu chuẩn cho HTML/CSS/JS
- **[Bundlephobia](https://bundlephobia.com)** — cân nặng package npm

## CSS hỗ trợ

- **[CSS Gradient](https://cssgradient.io)** / **[Shadows Brumm](https://shadows.brumm.af)** — sinh gradient, shadow (xem [shadows-gradients.md](shadows-gradients.md))
- **[Flexbox Froggy](https://flexboxfroggy.com)** / **[Grid Garden](https://cssgridgarden.com)** — game học Flexbox/Grid
- **[Cubic-bezier](https://cubic-bezier.com)** — chỉnh đường cong easing cho animation

## Khác

- **[Squoosh](https://squoosh.app)** — nén ảnh
- **[Carbon](https://carbon.now.sh)** / **[ray.so](https://ray.so)** — ảnh đẹp của code (để share)
- **[JSON Crack](https://jsoncrack.com)** — xem JSON dạng sơ đồ

## Cách dùng

Nhóm này chủ yếu là **công cụ online** — mở web, dán input, copy output.
Không phụ thuộc HTML tĩnh / React / Next.js. Ví dụ luồng hay dùng:

1. Có 1 SVG icon → **SVGOMG** nén → **SVG to JSX** → dán vào React component.
2. Có response API JSON → **JSON to TS** → có type dùng ngay trong Next.js.
3. Cần validate email/slug → viết regex ở **Regex101** → copy vào code.
