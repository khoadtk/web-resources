# Maps

Nhúng bản đồ: hiển thị vị trí, marker, vẽ vùng, geocoding.

## Thư viện / dịch vụ

- **[Leaflet](https://leafletjs.com)** — mã nguồn mở, nhẹ, dùng tile miễn phí (OpenStreetMap)
- **[MapLibre](https://maplibre.org)** — bản đồ vector mã nguồn mở (fork Mapbox GL)
- **[Mapbox](https://mapbox.com)** — đẹp, mạnh, tùy biến style (có free tier)
- **[Google Maps](https://developers.google.com/maps)** — quen thuộc, dữ liệu địa điểm tốt (tính phí)
- **[react-leaflet](https://react-leaflet.js.org)** — wrapper Leaflet cho React

## Cách dùng

### HTML tĩnh

**Leaflet** + OpenStreetMap, miễn phí, không cần key:

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9/dist/leaflet.js"></script>
<div id="map" style="height:400px"></div>
<script>
  const map = L.map('map').setView([10.77, 106.70], 13); // TP.HCM
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);
  L.marker([10.77, 106.70]).addTo(map).bindPopup('Ở đây');
</script>
```

### React

**react-leaflet**:

```bash
npm install leaflet react-leaflet
```

```jsx
import { MapContainer, TileLayer, Marker } from 'react-leaflet';
<MapContainer center={[10.77, 106.70]} zoom={13} style={{ height: 400 }}>
  <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
  <Marker position={[10.77, 106.70]} />
</MapContainer>
```

### Next.js

- Bản đồ cần `window` → **lazy-load client**:

```jsx
const Map = dynamic(() => import('./Map'), { ssr: false });
```

- Nhớ import CSS Leaflet (trong `layout` hoặc component).

## Mẹo

- Map key (Mapbox/Google) hạn chế theo **domain/HTTP referrer** → lộ ra client vẫn khó bị lạm dụng.
- **Geocoding** (địa chỉ ↔ toạ độ): Nominatim (free, OSM), Mapbox, Google.
- Nhiều marker → **cluster** (marker cluster plugin) cho đỡ rối.
- Bản đồ nặng → chỉ load khi user cuộn tới (lazy).
- Tôn trọng điều khoản tile OSM (đừng lạm dụng tile miễn phí cho traffic lớn → tự host/Mapbox).
