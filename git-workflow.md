# Git workflow

Quy trình làm việc với Git cho dự án web (cá nhân & nhóm).

## Lệnh hay dùng

```bash
git switch -c feature/login     # tạo + chuyển sang nhánh mới
git add -p                      # chọn từng phần để commit
git commit -m "feat: thêm form login"
git push -u origin feature/login
git switch main && git pull     # cập nhật main
git rebase main                 # (trên nhánh feature) đưa commit lên trên main mới
git log --oneline --graph       # xem lịch sử gọn
```

## Quy ước commit (Conventional Commits)

```
feat: tính năng mới
fix: sửa bug
docs: tài liệu
refactor: đổi code không đổi hành vi
chore: việc lặt vặt (deps, config)
```

→ commit rõ ràng, dễ tự sinh changelog/version.

## Nhánh (branch)

- **main** — luôn chạy được, deploy được.
- **feature/xxx** — mỗi tính năng 1 nhánh, xong mở PR merge vào main.
- Nhánh **ngắn ngày**, merge sớm → tránh conflict lớn.

## Cách dùng (áp cho dự án web)

### Dự án cá nhân (HTML tĩnh / học tập)

- 1 nhánh `main` là đủ; commit theo cột mốc.
- `.gitignore`: `node_modules/`, `.env`, `dist/`, `.next/`.

### React / Next.js theo nhóm

- Mỗi task → 1 nhánh feature → PR → review → merge.
- Nối với **CI/CD**: push mở **preview deploy**, PR chạy lint/test (xem [deploy-hosting.md](deploy-hosting.md), [testing.md](testing.md)).
- Bảo vệ nhánh `main` (require PR + pass check).

## File nên có

```
.gitignore        # bỏ node_modules, .env, build output
README.md         # cách chạy dự án
.env.example      # mẫu env (KHÔNG chứa secret thật)
```

## Mẹo

- **Không commit** `.env`, `node_modules`, file build → luôn thêm `.gitignore` từ đầu.
- Commit **nhỏ, một mục đích** → dễ review, dễ revert.
- Lỡ commit secret → coi như **đã lộ**, phải xoay key mới (git history khó xoá sạch). Xem cảnh báo secret trong [[personal-workspace-map]].
- Viết message theo Conventional Commits → nhất quán.
- `git switch` / `git restore` (mới) rõ nghĩa hơn `git checkout`.
