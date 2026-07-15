# Phân quyền trong NestJS (deep-dive)

> Lý thuyết 5 mô hình ở [authorization.md](authorization.md). File này map chúng vào NestJS: Guard, Decorator, CASL, OpenFGA.

## Bộ đồ nghề của NestJS cho phân quyền

| Khối | Vai trò trong phân quyền |
| --- | --- |
| **Guard** (`CanActivate`) | Chốt chặn trước handler — trả `true/false`, nơi enforce chính |
| **Decorator + `SetMetadata`** | Khai báo yêu cầu quyền ngay trên route (`@Roles('editor')`) |
| **`Reflector`** | Guard đọc metadata mà decorator đã gắn |
| **Service** | Nơi check quyền *cần dữ liệu* (ABAC/ownership) — sau khi load resource |

Thứ tự chạy: `Middleware → Guard → Interceptor → Pipe → Handler`. Guard **authentication** (gắn `req.user`) phải đứng trước guard **authorization** — Nest chạy guard theo đúng thứ tự khai báo.

## 1. RBAC — Roles guard kinh điển

```ts
// roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

```ts
// roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(ctx: ExecutionContext): boolean {
    // đọc @Roles trên handler, không có thì fallback lên class
    const required = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      ctx.getHandler(),
      ctx.getClass(),
    ]);
    if (!required?.length) return true;            // route không khai @Roles → cho qua

    const { user } = ctx.switchToHttp().getRequest();
    return required.some(role => user?.roles?.includes(role));
  }
}
```

```ts
// posts.controller.ts
@Controller('posts')
@UseGuards(JwtAuthGuard, RolesGuard)   // auth trước, quyền sau
export class PostsController {
  @Post()
  @Roles('editor', 'admin')            // editor HOẶC admin
  create(@Body() dto: CreatePostDto) { ... }
}
```

**Nâng cấp theo bài học "check permission, đừng check role"**: thay `@Roles('editor')` bằng `@RequirePermissions('content:publish')` — guard giống hệt, chỉ đổi metadata key và so với `user.permissions` (đã resolve từ role lúc đăng nhập). Đổi cấu trúc role sau này không phải sửa controller.

## 2. ABAC — vì sao KHÔNG nằm gọn trong guard

Guard chạy **trước khi resource được load** — mà ABAC cần so `user.branch === post.branch`, tức phải có `post` trong tay. Hai cách xử lý:

### Cách a: check trong service (đơn giản, đủ cho đa số)

```ts
// posts.service.ts
async update(user: User, postId: string, dto: UpdatePostDto) {
  const post = await this.postsRepo.findOneByOrFail({ id: postId });

  if (user.branch !== post.branch)           // rule ABAC
    throw new ForbiddenException();          // Nest tự trả 403

  return this.postsRepo.save({ ...post, ...dto });
}
```

Load 1 lần, check quyền + dùng luôn — không query 2 lần. Nhược: luật nằm trong service, dễ thành "if rải rác" khi app lớn → dẫn tới cách b.

### Cách b: CASL — gom luật một chỗ (Nest docs cũng khuyên)

CASL là "PBAC thu nhỏ trong app": định nghĩa **toàn bộ luật ở một factory**, chỗ khác chỉ hỏi.

```bash
npm install @casl/ability
```

```ts
// casl-ability.factory.ts — MỌI luật quyền của app nằm ở đây
@Injectable()
export class CaslAbilityFactory {
  createForUser(user: User) {
    const { can, build } = new AbilityBuilder(createMongoAbility);

    if (user.roles.includes('admin')) {
      can('manage', 'all');                            // RBAC: admin làm tất
    }
    can('read', 'Post');                               // ai cũng đọc được
    can('update', 'Post', { branch: user.branch });    // ABAC: điều kiện trên resource
    can('delete', 'Post', { authorId: user.id });      // ownership (ACL-style)

    return build();
  }
}
```

```ts
// posts.service.ts — chỗ dùng chỉ hỏi, không biết luật
import { subject } from '@casl/ability';

async update(user: User, postId: string, dto: UpdatePostDto) {
  const post = await this.postsRepo.findOneByOrFail({ id: postId });
  const ability = this.caslFactory.createForUser(user);

  if (ability.cannot('update', subject('Post', post)))
    throw new ForbiddenException();

  return this.postsRepo.save({ ...post, ...dto });
}
```

Để ý factory **trộn RBAC + ABAC + ownership trong một chỗ** — đúng lời khuyên "app thực tế thường lai" ở bài lý thuyết. Bonus: CASL chạy được cả ở React → FE dùng *cùng định nghĩa luật* để ẩn/hiện nút.

## 3. ACL — bảng permission per-resource

Guard/service check bảng `document_permissions` ([authorization.md](authorization.md) mục 1):

```ts
// documents.service.ts
async findOne(user: User, docId: string) {
  const perm = await this.permRepo.findOneBy({
    documentId: docId,
    userId: user.id,          // level: 'view' | 'edit' | 'owner'
  });
  if (!perm) throw new NotFoundException();   // 404, không xác nhận doc tồn tại
  return this.docsRepo.findOneByOrFail({ id: docId });
}
```

## 4. ReBAC — guard gọi OpenFGA

Luật quan hệ nằm trong OpenFGA; Nest chỉ hỏi "có đường quan hệ không":

```ts
// fga.guard.ts (rút gọn)
@Injectable()
export class FgaGuard implements CanActivate {
  constructor(private fga: OpenFgaClient, private reflector: Reflector) {}

  async canActivate(ctx: ExecutionContext): Promise<boolean> {
    const relation = this.reflector.get<string>('fga_relation', ctx.getHandler()); // vd 'viewer'
    const req = ctx.switchToHttp().getRequest();

    const { allowed } = await this.fga.check({
      user: `user:${req.user.id}`,
      relation,                                  // 'viewer' | 'editor' | 'owner'
      object: `document:${req.params.id}`,
    });
    return allowed;   // OpenFGA tự đi tìm đường: user → team → folder → doc
  }
}
```

## 5. PBAC đầy đủ — Casbin/OPA

Như CASL nhưng luật ra **ngoài app** (nhiều service dùng chung): guard gọi Casbin enforcer hoặc HTTP tới OPA sidecar — cấu trúc guard giống FgaGuard ở trên, chỉ đổi `fga.check(...)` thành `enforcer.enforce(sub, obj, act)` / `POST /v1/data/authz/allow`.

## Wiring cả app: global guard + @Public

Đăng ký guard **global** để không route nào bị bỏ quên (fail-safe, "deny by default"):

```ts
// app.module.ts
providers: [
  { provide: APP_GUARD, useClass: JwtAuthGuard },   // 1. authentication toàn app
  { provide: APP_GUARD, useClass: RolesGuard },     // 2. authorization
]
```

```ts
// public.decorator.ts — mở cửa CÓ CHỦ ĐÍCH cho route công khai (login, healthz)
export const Public = () => SetMetadata('isPublic', true);

// trong JwtAuthGuard: đọc metadata, isPublic thì cho qua
```

Ngược với cách `@UseGuards` từng controller (quên gắn = route hở), global + `@Public()` đảo mặc định: **quên gì đó thì bị chặn nhầm (an toàn), không phải hở nhầm**.

Multi-tenant: đừng dựa vào nhớ tay `where: { tenantId }` từng query — gom vào một chỗ (custom repository/scoped service đọc `tenantId` từ request) để "đưa điều kiện quyền vào query" thành mặc định.

## Bẫy hay gặp trong NestJS

- **Quên gắn guard cho route mới** → dùng global `APP_GUARD` + `@Public()` như trên; đừng dựa vào kỷ luật.
- **Nhét query DB vào guard cho mọi route ABAC** → guard hợp với luật *không cần resource* (role, permission, quan hệ qua FGA); luật cần resource thì check ở service sau khi load — đó không phải "làm sai", đó là đúng tầng.
- **Trộn 403/404 tuỳ hứng** → thống nhất: không đủ *quyền hành động* = 403; không *sở hữu/không nên biết tồn tại* = 404 ([api-security.md](api-security.md)).
- **Check role trong controller bằng if** (`if (user.role === 'admin')`) → luật rải rác; dồn về decorator+guard hoặc CASL factory.
- **GraphQL/WebSocket**: `ctx.switchToHttp()` không dùng được — GraphQL lấy user qua `GqlExecutionContext.create(ctx).getContext().req.user`; guard viết chung nên xử cả hai loại context.
- **Test guard như unit thường**: mock `ExecutionContext` + `Reflector` là test được luật không cần dựng app; kèm e2e bắn request thiếu quyền assert 403 ([automation-testing-playwright.md](automation-testing-playwright.md) cho lớp e2e).
