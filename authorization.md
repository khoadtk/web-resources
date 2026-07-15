# Phân quyền (Authorization) — deep-dive

Authentication trả lời "**bạn là ai**" ([auth-server.md](auth-server.md)); authorization trả lời "**bạn được làm gì**". File này đi sâu 5 mô hình phân quyền phổ biến, usecase của từng loại, và cách chọn.

> Ví dụ cài đặt trên NestJS (Guard, CASL, OpenFGA): [authorization-nestjs.md](authorization-nestjs.md)

## Bài toán chung

Mọi hệ thống phân quyền đều đang trả lời một câu hỏi dạng:

```
Chủ thể (user/service)  +  Hành động (read/edit/delete)  +  Tài nguyên (document #42)
→ CHO PHÉP hay TỪ CHỐI?
```

5 mô hình dưới đây khác nhau ở chỗ **dựa vào đâu để trả lời**: danh sách gắn trên tài nguyên, vai trò của user, thuộc tính của cả hai, quan hệ giữa hai bên, hay một bộ policy tách riêng.

## 1. ACL — Access Control List

**Ý tưởng:** mỗi *tài nguyên* mang theo một danh sách "ai được làm gì" với chính nó.

```
document #42:
  - user:alice  → owner
  - user:bob    → edit
  - user:carol  → view
```

```sql
CREATE TABLE document_permissions (
  document_id BIGINT REFERENCES documents(id),
  user_id     BIGINT REFERENCES users(id),
  level       TEXT CHECK (level IN ('view','edit','owner')),
  UNIQUE (document_id, user_id)
);
-- check: tồn tại dòng (doc, user, level đủ cao) không?
```

**Usecase:** chia sẻ theo từng đối tượng cụ thể — **Google Docs share cho từng email**, file server, quyền theo từng bản ghi. Hợp khi quyền là *của riêng từng tài nguyên*, không suy ra được từ vai trò.

**Giới hạn:** triệu tài nguyên × nghìn user = bảng permission khổng lồ; đổi chính sách chung ("mọi editor giờ được xoá") phải sửa hàng loạt; không diễn đạt được "quyền theo phòng ban".

## 2. RBAC — Role-Based Access Control (phổ biến nhất)

**Ý tưởng:** gom quyền vào **vai trò**, gán vai trò cho user. User → role → permissions.

```
admin   → [user:manage, billing:manage, content:*]
editor  → [content:create, content:edit, content:publish]
viewer  → [content:read]
```

```ts
// bảng: users, roles, permissions, user_roles, role_permissions
function can(user: User, permission: string): boolean {
  return user.roles.some(r => r.permissions.includes(permission));
}
// dùng: if (!can(user, 'content:publish')) return forbidden();
```

**Usecase:** **hệ thống quản trị nội bộ, SaaS B2B, CMS** — nơi user chia thành nhóm nghề nghiệp rõ ràng (admin/manager/staff). Ví dụ có thật: **GitHub org role** (owner/member), **WordPress** (administrator/editor/author), K8s RBAC.

**Giới hạn — "role explosion":** khi cần chi tiết theo ngữ cảnh ("editor nhưng chỉ khu vực miền Nam", "manager nhưng chỉ giờ hành chính") → phải đẻ role mới liên tục (`editor_mien_nam`, `editor_mien_bac`...) — dấu hiệu bạn cần ABAC.

**Mẹo quan trọng:** code nên check **permission, đừng check role** (`can(user, 'content:publish')` thay vì `user.role === 'editor'`) — đổi cấu trúc role sau này không phải sửa code rải rác.

## 3. ABAC — Attribute-Based Access Control

**Ý tưởng:** quyết định bằng cách **so thuộc tính với thuộc tính**, thay vì so user với một danh sách role tĩnh. Thuộc tính (attribute) là dữ liệu sẵn có trên mỗi bên:

- Của **user**: phòng ban, chi nhánh, level, `tenant_id`
- Của **tài nguyên**: chi nhánh sở hữu, trạng thái, độ mật, `tenant_id`
- Của **bối cảnh**: giờ, IP, thiết bị

### Thấy rõ nhất khi so với RBAC trên cùng một bài toán

Bài toán: *"Editor chỉ được sửa bài của chi nhánh mình"* (công ty có HN, HCM, ĐN).

```
RBAC thuần — role là nhãn tĩnh, phải đẻ role theo chi nhánh:
  editor_hn / editor_hcm / editor_dn     ← mở chi nhánh thứ 4? thêm role thứ 4 (role explosion)

ABAC — MỘT rule, không liệt kê chi nhánh nào:
  user.role == 'editor' AND user.branch == post.branch
```

Rule không nói "được sửa bài HN" mà nói "chi nhánh hai bên phải **khớp nhau**" — nên mở thêm 10 chi nhánh cũng không sửa gì. Chạy thử với data:

```ts
const alice = { role: 'editor', branch: 'HN' };

canEdit(alice, { branch: 'HN' });   // 'editor' ✓, 'HN' === 'HN' ✓  → CHO SỬA
canEdit(alice, { branch: 'HCM' });  // 'editor' ✓, 'HN' !== 'HCM' ✗ → CHẶN

function canEdit(user: User, post: Post): boolean {
  return user.role === 'editor' && user.branch === post.branch;
}
```

Điểm mấu chốt: cùng là editor nhưng quyền của alice **thay đổi theo từng bài viết** — RBAC thuần không diễn đạt được điều này (role của alice có đổi đâu). Rule còn cộng dồn thêm điều kiện ngữ cảnh được: `&& ctx.isBusinessHours && post.status !== 'archived'`.

**Usecase:** quy tắc phụ thuộc **dữ liệu & ngữ cảnh** — **multi-tenant** ("user chỉ thấy data tenant mình": `WHERE tenant_id = user.tenant_id` chính là ABAC một thuộc tính mà bạn có thể đã dùng hằng ngày), y tế/tài chính ("bác sĩ chỉ xem hồ sơ bệnh nhân *khoa mình*"), **AWS IAM policy** (condition theo tag/IP/giờ) là ABAC quy mô lớn nhất thực tế.

**Giới hạn:** khó trả lời ngược "user này được đụng *những* tài nguyên nào?" (phải dịch rule thành query — vd rule branch ở trên thành `WHERE branch = 'HN'`); rule rải trong code lâu ngày thành mê cung — nên gom về một chỗ (mục 5).

## 4. ReBAC — Relationship-Based Access Control

**Ý tưởng — trả lời câu hỏi "quyền lưu ở đâu?":** ACL lưu quyền *trên tài nguyên*; RBAC lưu *trên user* (role). ReBAC **không lưu quyền ở đâu cả** — hệ thống **tự suy ra** từ các mối quan hệ, kể cả quan hệ nối tiếp nhau (bắc cầu). Đây là mô hình của **Google Zanzibar** (hệ phân quyền chung cho Drive/Docs/YouTube).

### Ví dụ bạn dùng hằng ngày: share folder Google Drive

Share một **folder** cho bob → bob mở được **mọi file bên trong**, kể cả file thêm vào sau. Không ai ghi dòng quyền nào cho từng file — DB chỉ có 2 dòng *quan hệ*:

```
bob --viewer--> folder:BaoCao
folder:BaoCao --chứa--> doc:Q3

Hỏi: bob xem được doc:Q3 không?
Không có dòng nào nói "bob → doc:Q3"!
Hệ thống tự suy: bob là viewer của folder → folder chứa doc → ĐƯỢC XEM
```

Check quyền = **tìm đường đi trong đồ thị quan hệ**. Chuỗi dài mấy cũng được:

```
alice --thuộc--> team:KeToan --thuộc--> phong:TaiChinh --viewer--> folder:BaoCao
→ alice có quyền qua 3 bước nhảy (nhóm lồng nhóm)
```

### Vì sao không dùng ACL cho việc này?

Share folder 1000 file bằng ACL = insert **1000 dòng quyền**, thêm file mới phải nhớ thêm quyền. ReBAC = **1 dòng quan hệ**, file mới bỏ vào folder là tự được share. Đổi lại, tìm đường trên đồ thị tỷ-cạnh là bài toán khó — nên thực tế dùng hệ chuyên dụng: **OpenFGA** (mã nguồn mở, chuẩn Zanzibar), SpiceDB, Ory Keto.

**Usecase:** quyền **kế thừa theo cây và nhóm** — Google Drive (share folder → cả cây con), GitHub (quyền repo qua org → team → member), mạng xã hội ("chỉ bạn-của-bạn xem được"), B2B nhiều tầng (agency quản lý nhiều client). Nghe yêu cầu "share cho nhóm, nhóm nằm trong nhóm khác, folder kế thừa từ cha" — đó là ReBAC.

**Giới hạn:** thêm một hệ thống ngoài để vận hành; overkill cho app có cấu trúc quyền phẳng (không cây, không nhóm lồng nhóm).

## 5. PBAC / Policy-as-code — tách luật ra khỏi app

**Ý tưởng — trả lời một câu hỏi KHÁC hẳn 4 mục trên.** ACL/RBAC/ABAC/ReBAC trả lời "*quyết định dựa vào gì*". PBAC trả lời "**luật viết ở chỗ nào trong hệ thống?**" — giống việc tách config ra `.env` thay vì hard-code URL ([env-config.md](env-config.md)), ở đây là tách **luật phân quyền** ra khỏi business code.

### Trước — luật rải trong code (cách hầu hết app đang làm)

```ts
// service A
if (user.role === 'editor' && user.branch === post.branch) { ... }

// service B — 6 tháng sau, người khác viết
if (user.role === 'editor') { ... }   // ← quên check branch: LỖ HỔNG
```

Cùng một luật bị copy ra nhiều service, mỗi nơi một dị bản; đổi luật phải sửa mọi nơi, sót một chỗ là thủng.

### Sau — luật nằm một chỗ, app chỉ hỏi

```ts
// MỌI nơi trong MỌI service chỉ còn đúng một câu hỏi:
const ok = await authz.check({ user: 'alice', action: 'edit', resource: 'post:42' });
```

```rego
# còn luật viết MỘT chỗ — ví dụ OPA (ngôn ngữ Rego); nội dung vẫn là rule kiểu ABAC
allow if {
  input.action == "edit"
  input.user.role == "editor"
  input.user.branch == data.posts[input.resource_id].branch
}
```

Sửa 1 file policy → mọi service theo ngay; policy nằm trong git → diff được "luật đổi khi nào, ai đổi" (audit). Lưu ý điểm hay nhầm: **bên trong policy bạn vẫn viết luật kiểu RBAC/ABAC như thường** — PBAC chỉ là *chỗ đặt luật*, không phải cách quyết mới; vì vậy bảng so sánh ghi nó "bọc các mô hình trên".

Công cụ: **OPA** (chuẩn CNCF, dùng nhiều cho hạ tầng/K8s), **Cedar** (AWS), **Casbin** (thư viện đa ngôn ngữ, nhẹ), **CASL** (JS/TS — dùng chung định nghĩa quyền cho cả server enforce lẫn FE ẩn/hiện nút).

**Usecase:** **≥2 service cần chung một bộ luật** (microservices), yêu cầu **audit/compliance** ("vì sao request này được phép?" phải trả lời được bằng văn bản), team security muốn quản luật mà không đụng code app.

**Giới hạn:** thêm hạ tầng vận hành + một ngôn ngữ mới cho team. App đơn thể thì **gom hết if vào một hàm `can(user, action, resource)` trong một module** là đủ — đó chính là tinh thần PBAC thu nhỏ (và là lý do common mistakes bên dưới khuyên đừng rải if khắp controller).

## Bảng chọn nhanh

| Mô hình | Quyết định dựa trên | Hợp nhất với | Ví dụ có thật |
| --- | --- | --- | --- |
| **ACL** | Danh sách gắn trên từng tài nguyên | Share từng item cho từng người | Google Docs share, file server |
| **RBAC** | Vai trò của user | Admin panel, SaaS B2B, CMS | GitHub org, WordPress |
| **ABAC** | Thuộc tính user + tài nguyên + ngữ cảnh | Multi-tenant, luật theo dữ liệu/giờ/vùng | AWS IAM |
| **ReBAC** | Quan hệ (kể cả bắc cầu, nhóm, cây) | Folder kế thừa, nhóm lồng nhóm | Google Drive, GitHub teams |
| **PBAC** | Policy tách riêng (bọc các mô hình trên) | Microservices, audit, compliance | OPA trong K8s, Cedar |

Thực tế app vừa và lớn thường **lai**: RBAC làm khung (admin/staff) + vài rule ABAC (đúng tenant, đúng phòng ban) + ownership check kiểu ACL cho tài nguyên user tạo — bắt đầu đơn giản, chỉ "lên đời" phần nào đau thật.

## Nguyên tắc thi hành (mọi mô hình đều phải theo)

- **Enforce ở server, mọi endpoint** — ẩn nút trên FE chỉ là UX; kẻ tấn công gọi thẳng API ([api-security.md](api-security.md) — BOLA/IDOR chính là lỗi *quên* check quyền trên resource).
- **Deny by default** — không match luật nào = từ chối; đừng "cho qua nếu quên khai".
- **Least privilege** — cấp tối thiểu đủ dùng; quyền rộng dễ cho nhưng khó đòi lại.
- **Check sát dữ liệu**: đưa điều kiện quyền vào **query** (`WHERE tenant_id = $1`) thay vì lấy hết rồi lọc trong code — vừa an toàn vừa không dính phân trang sai.
- **Log quyết định từ chối** ([logging-observability.md](logging-observability.md)) — 403 dồn dập là tín hiệu dò quét.
- Quyền đổi thì **thu hồi phiên/cache quyền** — user bị hạ quyền mà JWT cũ còn sống 24h là lỗ hổng ([auth-server.md](auth-server.md)).

## Common mistakes — tin sai → sửa lại

- **"Check role ở FE là đủ vì user thường không thấy nút."** → Sai: API vẫn public; mọi check phải lặp lại ở server.
- **"RBAC là đủ cho mọi thứ."** → Đẻ role kiểu `editor_hn`, `editor_hcm` là role explosion — phần "theo ngữ cảnh" nên chuyển thành thuộc tính (ABAC).
- **"Nhét permissions vào JWT là xong."** → Quyền trong token **đóng băng tới khi hết hạn**; quyền hay đổi thì token phải ngắn hạn hoặc tra thêm ở server.
- **"Đã đăng nhập + đúng role = xong."** → Vẫn thiếu **ownership**: editor sửa được *bài của tenant khác* nếu quên `WHERE tenant_id = ...` — lỗi thực chiến nhiều nhất.
- **"Viết if rải khắp controller cho nhanh."** → 6 tháng sau không ai biết luật nằm đâu; gom về một module/hàm `can(user, action, resource)` ngay từ đầu — sau này muốn chuyển sang engine cũng chỉ đổi ruột hàm đó.

## Học tiếp

- Bài **Google Zanzibar paper** (2019) — nền của ReBAC hiện đại, đọc phần thiết kế là đủ.
- **OpenFGA docs** — nghịch playground mô hình hoá quyền kiểu quan hệ.
- **CASL** nếu stack JS/TS — dùng chung định nghĩa quyền cho cả server (enforce) lẫn FE (ẩn/hiện UI).
- Trong kho: [auth.md](auth.md) (tổng quan), [auth-server.md](auth-server.md) (authentication), [api-security.md](api-security.md) (BOLA, validate).
