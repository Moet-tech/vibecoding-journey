# 06 — Branching workflows và commit conventions

## 1. GitHub Flow — đơn giản, hợp startup / web app

Chỉ có 1 nhánh chính: `main` luôn deployable.

```
main: ────●────●────●────●────●────● (luôn deploy được)
              ╲                ╱
               ●──●──● (feature/x, PR vào main)
```

Quy trình:

1. `git switch -c feature/x` từ `main` (đã pull mới nhất).
2. Code, commit nhiều lần.
3. `git push -u origin feature/x`.
4. Mở Pull Request.
5. CI chạy test → review code.
6. Merge PR vào `main` → deploy ngay.
7. Xóa branch.

**Phù hợp**: app web có CI/CD, deploy liên tục. Đa số team modern dùng cái này.

## 2. Git Flow — phức tạp, hợp release theo version

5 loại nhánh:

| Branch | Mục đích | Tách từ | Merge vào |
|---|---|---|---|
| `main` | Production | — | — |
| `develop` | Integration | `main` | `main` (qua release) |
| `feature/*` | Feature mới | `develop` | `develop` |
| `release/*` | Chuẩn bị release | `develop` | `main` + `develop` |
| `hotfix/*` | Sửa khẩn cấp prod | `main` | `main` + `develop` |

```
main:    ●─────────────────────●─────────────●  (v1.0)        (v1.1)
                              ╱             ╱
release:                    ●─●            │
                           ╱               │
develop: ●─●─●─●─●─●─●─●─●─●─●─●─●─●─●─●─●─●
            ╲     ╱   ╲   ╱
feature:     ●─●─●     ●─●
```

**Phù hợp**: phần mềm release theo version (mobile app phải submit store, desktop software, library có semver).

**Không hợp**: web app deploy liên tục — quá phức tạp.

## 3. Trunk-based Development — cho team CI/CD trưởng thành

Chỉ có `main` (trunk). Mọi người commit thẳng vào main, hoặc dùng nhánh sống **< 1 ngày**. Feature lớn ẩn sau **feature flag**.

```
main: ●─●─●─●─●─●─●─●─●─●─●─●  (deploy hàng giờ)
```

Yêu cầu cao: test tự động tốt, feature flag system, code review nhanh. Là cách Google, Facebook làm việc.

## 4. So sánh nhanh

| | GitHub Flow | Git Flow | Trunk-based |
|---|---|---|---|
| Số nhánh chính | 1 | 2 | 1 |
| Độ phức tạp | Thấp | Cao | Thấp |
| Deploy | Liên tục | Theo release | Liên tục (nhanh hơn) |
| Phù hợp | SaaS, web | App có version | Big tech |

→ **Khuyến nghị cho cá nhân / team nhỏ**: bắt đầu với GitHub Flow.

## 5. Conventional Commits

Quy ước commit message phổ biến nhất hiện nay, mở đường cho changelog tự động + semantic versioning tự động.

Cú pháp:

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Các type chính

| Type | Nghĩa | Tác động version |
|---|---|---|
| `feat` | Thêm tính năng | minor (1.X.0) |
| `fix` | Sửa bug | patch (1.0.X) |
| `docs` | Tài liệu | — |
| `style` | Format, semicolon… không đổi logic | — |
| `refactor` | Refactor code, không thêm tính năng | — |
| `perf` | Tối ưu hiệu năng | patch |
| `test` | Thêm/sửa test | — |
| `build` | Build system, dependencies | — |
| `ci` | CI config | — |
| `chore` | Việc lặt vặt khác | — |
| `revert` | Đảo ngược commit cũ | — |

### Breaking change

Thêm `!` sau type, hoặc dòng `BREAKING CHANGE:` ở footer → bump **major**:

```
feat(api)!: change response format

BREAKING CHANGE: /users endpoint nay trả về {data:...} thay vì array trực tiếp
```

### Ví dụ tốt

```
feat(auth): add password reset flow

Allow users to reset password via email link valid for 1 hour.

Closes #142
```

```
fix(cart): prevent negative quantity on edit

Quantity input was not validated server-side, allowing
negative values that crashed checkout.
```

```
refactor(db): extract connection pool config

Move pool sizes from hardcoded to env vars to support
per-environment tuning.
```

### Ví dụ xấu

```
update                    ← update gì???
fix bug                   ← bug nào???
WIP                       ← không nên có trong main
asdfasdf                  ← đừng :)
"Made some changes to the auth flow because the previous one was bad and..."  ← quá dài, không có cấu trúc
```

## 6. Quy tắc viết message tốt (kể cả không theo Conventional)

1. **Dòng tiêu đề ≤ 50 ký tự**, mệnh lệnh ("add", "fix", không phải "added", "fixes").
2. **Dòng trống**, rồi đến **body**.
3. Body wrap ở 72 ký tự, giải thích **WHY**, không phải WHAT.
4. Liên kết issue: `Closes #123`, `Fixes #456`, `Refs #789`.
5. 1 commit = 1 ý tưởng logic. Đừng nhét fix typo và feature lớn vào cùng commit.

## 7. Quy trình mở Pull Request "đẹp"

1. Branch của bạn được **rebase** lên `main` mới nhất (lịch sử thẳng).
2. Commits đã được **clean** (squash các "wip", "fix typo" lại).
3. PR description giải thích **WHY**, không lặp lại commit message.
4. Có **screenshot / GIF** nếu là UI change.
5. Có **test plan** rõ ràng (checklist).
6. CI xanh.
7. Tự review 1 lượt diff trước khi xin review.

PR description template:

```markdown
## Summary
- Thêm trang reset password
- Gửi mail qua SendGrid khi user request reset

## Why
User feedback hiện không thể tự reset password, phải nhờ support.

## Test plan
- [ ] Request reset từ /forgot-password
- [ ] Nhận mail < 30s
- [ ] Link reset hết hạn sau 1h
- [ ] Đặt password mới → login OK
- [ ] Password cũ không còn dùng được

Closes #142
```

## 8. Cheat sheet workflows

| Tình huống | Nên dùng |
|---|---|
| Web app, CI/CD, team < 20 | **GitHub Flow** |
| Mobile / desktop app, release có version | **Git Flow** |
| Big tech, CI/CD mạnh, feature flag | **Trunk-based** |
| Commit message | **Conventional Commits** |
| Branch naming | `<type>/<short-desc>` |

→ Quay lại: [notes.md](notes.md) hoặc làm bài tập trong [exercises/](exercises/).
