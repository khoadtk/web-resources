# Networking basics

Chuyện gì xảy ra từ lúc gõ URL tới lúc response về — đủ để debug "sao không kết nối được".

## Đường đi của 1 request

```
Trình duyệt → DNS (tên → IP) → TCP connect (IP:port) → TLS handshake (https)
→ [CDN/Load balancer → Reverse proxy] → App server → Response quay ngược lại
```

## DNS — danh bạ của internet

- `shop.com → 76.76.21.21`. Trình duyệt/OS/resolver **cache theo TTL** — vì vậy đổi DNS "chưa ăn ngay".
- Record chính: **A** (tên → IPv4), **AAAA** (IPv6), **CNAME** (tên → tên khác, vd `www` → apex), **TXT** (verify domain, SPF/DKIM cho [email.md](email.md)), **MX** (mail).
- Debug: `dig shop.com` (hoặc `nslookup`) xem tên đang trỏ đi đâu.
- Trỏ domain vào Vercel/Netlify chính là thêm A/CNAME record ([deploy-hosting.md](deploy-hosting.md)).

## IP & Port

- Một máy = 1 IP; một IP chạy nhiều dịch vụ, phân biệt bằng **port**: 80 (HTTP), 443 (HTTPS), 5432 (Postgres), 6379 (Redis), 3000 (app dev).
- `localhost`/`127.0.0.1` = chính máy đó. Trong Docker, `localhost` của container là **container đó**, không phải máy bạn — bug kinh điển (xem [docker-web.md](docker-web.md)).
- Debug port: `lsof -i :3000` (ai đang chiếm port), `curl localhost:3000` (app có sống không).

## TLS / HTTPS

- TLS mã hoá đường truyền: không ai đọc/sửa được data giữa đường (kể cả wifi quán cà phê).
- **Certificate** chứng minh "tôi đúng là shop.com" — cấp bởi CA, ngày nay **miễn phí & tự động** (Let's Encrypt; Vercel/Cloudflare tự lo).
- Dev local muốn https: `mkcert` tạo cert tự ký cho localhost.
- Lỗi hay gặp: cert hết hạn, cert không khớp domain (`www` có cert, apex không), **mixed content** (trang https gọi resource http → bị chặn).

## Reverse proxy & Load balancer

- **Reverse proxy** (nginx, Caddy, Traefik) đứng trước app: nhận request từ ngoài, chuyển vào app bên trong. Kiêm luôn: TLS termination, nén gzip, serve file tĩnh, rate limit.
- **Load balancer** = reverse proxy chia request cho **nhiều instance** app → scale ngang. Hệ quả quan trọng: app phải **stateless** — session để ở Redis/DB, không để trong RAM của 1 instance (request sau có thể rơi vào instance khác).
- `502 Bad Gateway` = proxy sống nhưng **app phía sau chết/không nghe đúng port**; `504` = app sống nhưng trả lời quá chậm.

## CORS — không phải lỗi mạng, là lỗi trình duyệt chặn

Frontend ở `app.com` gọi API `api.com` → trình duyệt chặn nếu API không **cho phép rõ ràng**:

```
Access-Control-Allow-Origin: https://app.com
Access-Control-Allow-Credentials: true
```

- CORS là cơ chế của **trình duyệt** — curl/Postman gọi vẫn được, chỉ browser chặn. Sửa ở **phía server API** (set header), không sửa được từ client.
- Request có header lạ/method khác GET/POST → trình duyệt gửi **preflight** (`OPTIONS`) hỏi trước; server phải trả lời OPTIONS đúng.

## Debug theo tầng — checklist

- [ ] Tên có phân giải không? → `dig ten-mien.com`
- [ ] Port có mở/app có nghe không? → `curl -v http://host:port`, `lsof -i :3000`
- [ ] TLS ổn không? → `curl -v https://...` đọc phần handshake, hoặc mở browser xem cert
- [ ] Proxy hay app lỗi? → 502/504 là nhìn app phía sau; xem log cả 2 tầng
- [ ] Chỉ browser fail còn curl được? → 99% là CORS, xem tab Network → request OPTIONS
