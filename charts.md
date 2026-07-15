# Charts & Data viz

Vẽ biểu đồ: line, bar, pie, dashboard.

## Thư viện

- **[Recharts](https://recharts.org)** — biểu đồ React bằng component, dễ nhất (khuyên cho React/Next)
- **[Chart.js](https://chartjs.org)** — canvas, nhẹ, dùng được cả HTML thuần
- **[ECharts](https://echarts.apache.org)** — mạnh, nhiều loại biểu đồ, dữ liệu lớn
- **[visx](https://airbnb.io/visx/)** / **[D3](https://d3js.org)** — tùy biến tối đa (D3 khó, mạnh nhất)
- **[Tremor](https://tremor.so)** — component dashboard sẵn (trên nền Recharts)

## Chọn loại biểu đồ

- **Line** — xu hướng theo thời gian.
- **Bar** — so sánh giữa các nhóm.
- **Pie/Donut** — tỉ lệ phần (dùng ít, ≤5 phần).
- **Area** — khối lượng tích lũy.

## Cách dùng

### HTML tĩnh

**Chart.js** qua CDN — không cần build:

```html
<canvas id="c"></canvas>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  new Chart(document.getElementById('c'), {
    type: 'bar',
    data: { labels: ['T1','T2','T3'], datasets: [{ data: [10, 20, 15] }] },
  });
</script>
```

### React

**Recharts** — khai báo bằng component:

```bash
npm install recharts
```

```jsx
import { LineChart, Line, XAxis, YAxis, Tooltip } from 'recharts';
<LineChart width={500} height={300} data={data}>
  <XAxis dataKey="month" /><YAxis /><Tooltip />
  <Line dataKey="revenue" stroke="#3b82f6" />
</LineChart>
```

### Next.js

- Recharts dùng được, nhưng nhiều lib chart cần DOM → thêm `'use client'`.
- Chart nặng → lazy-load bằng `next/dynamic` với `ssr: false` (xem [performance.md](performance.md)):

```jsx
const Chart = dynamic(() => import('./Chart'), { ssr: false });
```

## Mẹo

- **Trước khi vẽ chart, đọc skill `dataviz`** để chọn màu, kiểu, layout cho nhất quán & dễ đọc (light/dark).
- Ít màu, đủ tương phản; **đừng chỉ dùng màu** phân biệt series (thêm nhãn/hoa văn) — a11y.
- Luôn có **nhãn trục + đơn vị + legend**.
- Dữ liệu lớn (nghìn điểm) → ECharts/canvas, không SVG (chậm).
- Responsive: bọc trong `ResponsiveContainer` (Recharts).
