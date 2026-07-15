# Video & Audio

Phát video/audio: player, streaming (HLS), nhúng YouTube, podcast.

## Kiến thức nền

- **File thường** (`.mp4`) — tải tuần tự, đủ cho clip ngắn.
- **HLS/DASH streaming** — video cắt thành đoạn nhỏ + nhiều chất lượng, tự đổi theo mạng (adaptive) — chuẩn cho video dài.
- **Đừng tự host video dài** trên hosting thường — dùng dịch vụ chuyên (encode + CDN).

## Dịch vụ / thư viện

- **[Mux](https://mux.com)** — video hosting + streaming cho dev, API đẹp
- **[Cloudflare Stream](https://www.cloudflare.com/products/cloudflare-stream/)** — upload là có HLS + player
- **YouTube/Vimeo embed** — miễn phí, đơn giản nhất nếu video công khai
- **[Video.js](https://videojs.com)** — player mã nguồn mở, plugin nhiều
- **[Plyr](https://plyr.io)** — player đẹp, nhẹ, hỗ trợ YouTube/Vimeo
- **[hls.js](https://github.com/video-dev/hls.js)** — phát HLS trên trình duyệt không hỗ trợ sẵn
- **[wavesurfer.js](https://wavesurfer.xyz)** — audio dạng sóng (podcast, voice note)

## Cách dùng

### HTML tĩnh

Thẻ có sẵn — đủ cho đa số nhu cầu:

```html
<video controls preload="metadata" poster="/thumb.jpg" width="640">
  <source src="/clip.mp4" type="video/mp4">
  <track kind="captions" src="/sub-vi.vtt" srclang="vi" label="Tiếng Việt">
</video>

<audio controls src="/podcast.mp3"></audio>

<!-- YouTube: lite-youtube-embed cho nhẹ (không kéo cả player khi chưa bấm) -->
<iframe src="https://www.youtube.com/embed/VIDEO_ID" allowfullscreen
        loading="lazy" title="Tên video"></iframe>
```

HLS trên trình duyệt thường (ngoài Safari) cần **hls.js**:

```js
const hls = new Hls();
hls.loadSource('https://.../playlist.m3u8');
hls.attachMedia(document.querySelector('video'));
```

### React

- `<video>`/`<audio>` dùng thẳng trong JSX; điều khiển qua `ref`:

```jsx
const ref = useRef(null);
<video ref={ref} src="/clip.mp4" controls />
<button onClick={() => ref.current.play()}>Phát</button>
```

- Player nhiều tính năng → **Plyr/Video.js** wrapper, hoặc **@mux/mux-player-react** nếu dùng Mux.

### Next.js

- Player cần DOM → `'use client'`; player nặng → `next/dynamic` với `ssr: false`.
- Video nền (hero) → file mp4 ngắn, `muted autoPlay loop playsInline`, nén kỹ.
- Video dài của user → upload lên **Mux/Cloudflare Stream** (qua luồng [file-upload.md](file-upload.md)), lưu playback URL vào DB.

## Mẹo

- `preload="metadata"` (không phải `auto`) → không kéo cả video khi chưa xem.
- **Luôn có phụ đề/captions** (`<track>`) — a11y + xem không tiếng.
- Autoplay chỉ chạy khi **muted** (chính sách trình duyệt).
- Poster/thumbnail giúp LCP đẹp hơn nhiều so với khung đen.
- Nhúng nhiều YouTube → dùng **lite-youtube-embed** để trang không ì.
