# File & Streaming (BE)

Xử lý file lớn mà không ngốn RAM: stream, backpressure, range request.

> Upload từ phía client + chọn storage ở [file-upload.md](file-upload.md). File này là kỹ thuật phía server.

## Vấn đề: RAM không phải vô hạn

```ts
// ❌ đọc cả file vào RAM — file 2GB × 10 user tải cùng lúc = server chết
const data = await fs.promises.readFile('video.mp4');
res.send(data);

// ✅ stream — chảy từng khúc (chunk ~64KB), RAM dùng cố định
import { createReadStream } from 'node:fs';
createReadStream('video.mp4').pipe(res);
```

Nguyên tắc: với file/payload lớn, dữ liệu nên **chảy qua** server (hoặc né server luôn — presigned URL), không bao giờ **đọng lại** trong RAM.

## Backpressure — vì sao pipe lại quan trọng

Đọc từ disk nhanh (trăm MB/s), client mạng chậm (vài Mb/s) → nếu cứ đọc thả ga, chênh lệch **dồn vào RAM**. `pipe`/`pipeline` tự xử lý: đích nghẽn thì **tạm dừng đọc nguồn**, đích rảnh thì đọc tiếp.

```ts
import { pipeline } from 'node:stream/promises';
await pipeline(
  createReadStream('input.csv'),
  csvParser(),            // transform stream: parse từng dòng
  createGzip(),           // nén trên đường đi
  res                     // đẩy ra client
);  // pipeline: tự backpressure + tự cleanup + propagate lỗi (hơn .pipe() trần)
```

Dùng `pipeline()` thay `.pipe()` — `.pipe()` không tự huỷ stream nguồn khi đích lỗi (leak file descriptor).

## Download tử tế: headers + range

```ts
res.setHeader('Content-Type', 'application/pdf');
res.setHeader('Content-Length', stat.size);          // client hiện % tải
res.setHeader('Content-Disposition', 'attachment; filename="report.pdf"'); // ép tải thay vì mở
```

### Range request — tua video & resume download

Client gửi `Range: bytes=1000000-` (tua đến giữa video, tải tiếp file dở). Server hỗ trợ:

```ts
const range = req.headers.range;           // "bytes=start-end"
if (range) {
  const [start, end] = parseRange(range, stat.size);
  res.writeHead(206, {                     // 206 Partial Content
    'Content-Range': `bytes ${start}-${end}/${stat.size}`,
    'Accept-Ranges': 'bytes',
    'Content-Length': end - start + 1,
  });
  createReadStream(path, { start, end }).pipe(res);
}
```

Không có range support = video không tua được, download đứt là tải lại từ đầu. (Dùng S3/CDN thì họ lo sẵn — thêm một lý do né tự serve file, xem [video-audio.md](video-audio.md).)

## Upload lớn phía server

- **Multipart** qua server: dùng lib stream-based (`busboy`) — file chảy thẳng lên storage, **không đợi nhận đủ rồi mới đẩy**:

```ts
busboy.on('file', (name, fileStream) => {
  // pipe thẳng sang S3, không ghi tạm ra disk/RAM
  s3.upload({ Key, Body: fileStream });
});
```

- Nhớ đặt **giới hạn kích thước** (reject sớm bằng `Content-Length` + cắt stream khi vượt) — không giới hạn là mời DoS.
- File thật lớn (video GB) → đừng đi qua server: **presigned URL / multipart upload thẳng lên S3** ([file-upload.md](file-upload.md)); server chỉ cấp phép.

## Streaming không chỉ cho file

- **Export CSV/report lớn**: query DB theo cursor, ghi từng dòng ra response — export triệu dòng với RAM phẳng:

```ts
for await (const row of db.cursor('SELECT ...')) res.write(toCsvLine(row));
res.end();
```

- **Response AI/LLM**: stream từng token về client ([ai-integration.md](ai-integration.md)) — cùng cơ chế.
- **SSE** ([realtime.md](realtime.md)) — cũng là một response mở giữ stream.

## Mẹo

- Test bằng file **thật sự lớn** (GB) + theo dõi RSS memory — bug stream chỉ lộ với file lớn.
- Timeout & cleanup: client ngắt giữa chừng phải huỷ đọc nguồn (`pipeline` + `AbortSignal`), không thì rò tài nguyên mỗi lần user đóng tab.
- Nén (`gzip/brotli`) cho text (CSV/JSON), **đừng nén** thứ đã nén sẵn (mp4, jpg, zip) — tốn CPU vô ích.
- Đừng ghi file tạm ra disk nếu chỉ để chuyển tiếp — pipe thẳng; cần file tạm thì dọn trong `finally`.
- Serverless (Vercel) có giới hạn size/duration — file lớn gần như luôn là việc của presigned URL + storage, không phải của function.
