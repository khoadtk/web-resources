# OPA — Open Policy Agent (deep-dive)

> Vị trí trong bức tranh phân quyền: [authorization.md](authorization.md) mục 5 (policy-as-code). Tích hợp NestJS: [authorization-nestjs.md](authorization-nestjs.md).

**OPA** = policy engine đa dụng (CNCF graduated): app gửi **câu hỏi JSON**, OPA đối chiếu **luật viết bằng Rego** và trả **quyết định JSON**. App không chứa luật — chỉ hỏi. OPA không enforce — app nhận kết quả rồi tự chặn.

## Mental model: input / data / policy

```
INPUT  (câu hỏi mỗi lần)  : { user: {...}, action: "edit", resource: {...} }
DATA   (sự thật nạp sẵn)  : role→permissions, config, danh sách tenant...
POLICY (luật Rego)        : allow if { ... }
                                ↓
DECISION (JSON bất kỳ)    : { "allow": false }   (hoặc object lý do, danh sách field được xem...)
```

Điều phải nhớ: **OPA không tự đi lấy dữ liệu** — nó chỉ biết thứ bạn đưa qua `input` (mỗi query) hoặc `data` (nạp sẵn/bundle). Luật cần dữ liệu gì, bạn phải thiết kế đường đưa dữ liệu đó vào.

## Rego cơ bản — đủ đọc hiểu 80% policy thực tế

```rego
package authz
import rego.v1

default allow := false            # deny by default

allow if {                        # nhiều rule cùng tên = OR
  input.action == "read"
}

allow if {
  input.action == "edit"
  input.user.role == "editor"     # các dòng trong 1 rule = AND
  input.user.branch == input.resource.branch    # rule ABAC quen thuộc
}

allow if { input.user.role == "admin" }

# rule trả giá trị (không chỉ true/false)
visible_fields := ["id", "title"] if { input.user.role == "viewer" }
visible_fields := ["id", "title", "cost"] if { input.user.role == "manager" }
```

Đặc điểm ngôn ngữ (gốc Datalog, **khai báo** chứ không tuần tự):

- Không có if/else chuỗi — mô tả *điều kiện để một kết luận đúng*.
- Duyệt tập hợp bằng `some`/`every`: `some role in input.user.roles; role == "editor"`.
- `import rego.v1` (OPA 1.0) bắt buộc từ khoá `if`/`contains` — dùng ngay từ đầu cho khỏi lẫn cú pháp cũ.
- Thử nhanh không cần cài gì: **Rego Playground** (play.openpolicyagent.org).

## Luật test được như code — điểm ăn tiền nhất

```rego
# authz_test.rego
package authz_test
import rego.v1
import data.authz

test_editor_sua_bai_cung_branch if {
  authz.allow with input as {
    "action": "edit",
    "user": {"role": "editor", "branch": "HN"},
    "resource": {"branch": "HN"},
  }
}

test_editor_bi_chan_branch_khac if {
  not authz.allow with input as {
    "action": "edit",
    "user": {"role": "editor", "branch": "HN"},
    "resource": {"branch": "HCM"},
  }
}
```

```bash
opa test . -v        # chạy test
opa fmt -w .         # format
opa check .          # lint/type check
```

Policy nằm trong git + có unit test + chạy trong CI — luật phân quyền được đối xử như code thật, không phải if rải rác không ai dám sửa.

## Chạy ở đâu — 4 chế độ

| Chế độ | Cách chạy | Hợp với |
| --- | --- | --- |
| **Sidecar / daemon** | OPA chạy cạnh app, gọi REST qua localhost | Microservices (phổ biến nhất) |
| **Thư viện nhúng** | Import trực tiếp (Go) | App Go, cần latency thấp nhất |
| **WASM** | `opa build -t wasm` → chạy trong Node/browser | App JS/TS không muốn sidecar |
| **CLI trong CI** | `conftest test` file YAML/Terraform/JSON | Check hạ tầng trước khi merge |

### Sidecar + REST API (kiểu dùng chính)

```bash
opa run --server --addr :8181 policy/    # nạp thư mục policy
```

```
POST http://localhost:8181/v1/data/authz/allow
{ "input": { "action": "edit", "user": {...}, "resource": {...} } }
→ { "result": false }
```

`docker-compose` thêm 1 service `openpolicyagent/opa` là có ([docker-web.md](docker-web.md)); guard NestJS chỉ đổi ruột thành HTTP call này ([authorization-nestjs.md](authorization-nestjs.md) mục 5).

### Nạp data & phân phối policy ở quy mô lớn

- Data nhỏ (role→permission): `PUT /v1/data/...` hoặc file nạp cùng policy.
- Chuẩn production: **bundle** — OPA định kỳ pull gói policy+data (ký được) từ một bundle server → deploy luật mới không đụng app.
- **Decision log**: OPA gửi log mọi quyết định về endpoint bạn cấu hình — nền của audit ([logging-observability.md](logging-observability.md)).

## OPA được dùng ở đâu ngoài API authz

- **Kubernetes admission control** — **Gatekeeper** (chặn deploy thiếu label, image lạ, container chạy root...) — usecase phổ biến nhất của OPA.
- **Envoy / API gateway** — external authorization filter gọi OPA trước khi request tới app.
- **IaC/CI** — `conftest` check Terraform/K8s YAML/Dockerfile theo policy công ty.
- Cùng một ngôn ngữ luật phủ từ hạ tầng tới API — lý do các team platform chuộng nó.

## Cái giá phải trả — cân nhắc trước khi dùng

- **Dữ liệu để quyết phải đưa tới OPA**: nó không query DB của bạn. Pattern thường dùng: app load resource rồi **gửi kèm trong input** (như ví dụ branch); data nền ít đổi thì sync qua bundle. Luật cần data lớn + tươi từng giây là điểm khó nhất, thiết kế trước.
- **Rego là ngôn ngữ mới** cho team — dễ đọc, lạ khi viết; cần 1-2 người "giữ policy".
- **Thêm một tiến trình để vận hành** (sidecar, version, bundle server) — app đơn thể thì CASL/hàm `can()` một module vẫn là lựa chọn đúng.
- So sánh nhanh cùng họ: **Cedar** (AWS — ngôn ngữ policy có kiểm chứng hình thức, hợp authz thuần), **Casbin** (thư viện nhẹ trong app, không sidecar), **OpenFGA** (chuyên ReBAC/quan hệ — [authorization.md](authorization.md) mục 4). OPA là con dao đa dụng nhất trong nhóm: JSON vào → JSON ra, luật gì cũng viết được.

## Bắt đầu thử trong 10 phút

1. Mở **play.openpolicyagent.org**, dán policy authz ở trên, sửa input xem `allow` đổi.
2. Local: `brew install opa` → `opa run --server policy/` → `curl POST /v1/data/authz/allow`.
3. Viết 2 test như mẫu, chạy `opa test . -v`.
4. Khi nào cần thật trong app NestJS → guard gọi REST như [authorization-nestjs.md](authorization-nestjs.md).
