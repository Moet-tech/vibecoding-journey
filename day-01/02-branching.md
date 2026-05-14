# 02 — Branching và merge

## 1. Branch là gì?

**Branch** = một con trỏ (pointer) trỏ tới 1 commit. Tạo branch = thêm 1 con trỏ — gần như miễn phí (vài byte).

```
        ┌── A ── B ── C   ← main
                  └── D   ← feature
```

`HEAD` là con trỏ "tôi đang ở đâu". Thông thường `HEAD` trỏ vào branch, branch trỏ vào commit.

```bash
git branch                         # liệt kê branch local (* = branch hiện tại)
git branch -a                      # bao gồm cả remote branches
git branch -v                      # kèm commit gần nhất của mỗi branch
git branch --merged                # đã merge vào branch hiện tại (an toàn để xóa)
git branch --no-merged             # chưa merge (cẩn thận trước khi xóa)
```

## 2. Tạo, đổi, xóa nhánh

Cú pháp mới (Git ≥ 2.23, khuyên dùng):

```bash
git switch <branch>                # chuyển sang nhánh có sẵn
git switch -c <new-branch>         # tạo mới + chuyển vào (-c = create)
git switch -c <new> <start-point>  # tạo từ commit / branch khác
git switch -                       # quay lại nhánh trước đó (giống cd -)
```

Cú pháp cũ (vẫn dùng được, đa năng hơn nhưng dễ nhầm):

```bash
git checkout <branch>              # = git switch <branch>
git checkout -b <new>              # = git switch -c <new>
git checkout <file>                # khôi phục file (giờ nên dùng git restore)
git checkout <commit>              # detached HEAD — sẽ giải thích bên dưới
```

Đổi tên / xóa:

```bash
git branch -m new-name             # đổi tên branch hiện tại
git branch -m old new              # đổi tên branch khác
git branch -d feature              # xóa (chỉ nếu đã merge — an toàn)
git branch -D feature              # xóa cưỡng bức (mất commit nếu chưa merge!)
```

## 3. Detached HEAD

Khi `HEAD` trỏ thẳng vào 1 commit (không qua branch), bạn đang ở **detached HEAD**. Commit mới ở đây sẽ "mồ côi" và có thể bị garbage-collect.

```bash
git checkout <hash>      # đi vào detached HEAD để xem
git switch -c rescue     # nếu muốn giữ lại — tạo branch ngay
git switch main          # nếu chỉ xem qua — quay về
```

Đôi khi hữu ích: build / test 1 commit cũ mà không tạo branch.

## 4. Merge — gộp nhánh

Workflow điển hình:

```bash
git switch main               # về nhánh đích
git merge feature             # gộp feature vào main
```

Có 3 loại merge:

### Fast-forward (FF)

Nếu `main` không có commit mới sau khi `feature` tách ra, Git chỉ "đẩy con trỏ" `main` về phía trước. Không tạo merge commit.

```
Trước:  A ── B ── C    ← main
                  └── D ── E   ← feature

Sau FF: A ── B ── C ── D ── E   ← main, feature
```

```bash
git merge feature                # tự fast-forward nếu được
git merge --no-ff feature        # luôn tạo merge commit (giữ "history" của feature)
git merge --ff-only feature      # chỉ chấp nhận FF, fail nếu không thể (an toàn)
```

### Three-way merge

Khi cả 2 nhánh đều có commit mới — Git tạo **merge commit** có 2 parent.

```
Trước:  A ── B ── C ── F   ← main
                  └── D ── E   ← feature

Sau:    A ── B ── C ── F ── M   ← main
                  └── D ── E ┘
```

### Squash merge

Gộp toàn bộ commit của feature thành **1 commit duy nhất** trên main.

```bash
git merge --squash feature
git commit -m "feat: add login"
```

Dùng khi feature branch có nhiều commit lặt vặt ("fix typo", "wip") — không muốn rác vào main.

## 5. Xử lý conflict

Conflict xảy ra khi cùng 1 đoạn code bị sửa khác nhau ở 2 nhánh. Git sẽ dừng merge và đánh dấu vào file:

```
<<<<<<< HEAD
const port = 3000;
=======
const port = 8080;
>>>>>>> feature
```

Cách xử lý:

1. Mở file, **chọn nội dung đúng**, xóa các marker `<<<`, `===`, `>>>`.
2. `git add <file>` để đánh dấu đã giải quyết.
3. `git commit` (Git tự gợi ý message cho merge commit).

Lệnh hỗ trợ:

```bash
git status                        # xem file nào còn conflict
git diff                          # xem cụ thể đoạn xung đột
git merge --abort                 # bỏ merge, về trạng thái trước
git checkout --ours <file>        # giữ phiên bản của nhánh hiện tại
git checkout --theirs <file>      # giữ phiên bản của nhánh đang merge vào
git mergetool                     # mở GUI merge tool
```

Sau khi resolve thành công: `git add` + `git commit`. Nếu lỡ tay: `git merge --abort`.

## 6. Chiến lược nhánh phổ biến

### Feature branch (đơn giản, phổ biến nhất)

```
main:     A ─── B ─────────── M ── N ── ...
                 └── C ── D ──┘
                feature/login
```

1. `git switch -c feature/login` từ `main`
2. Code, commit nhiều lần
3. `git switch main && git pull`
4. `git merge feature/login` (hoặc mở PR)
5. `git branch -d feature/login`

### Naming convention thường gặp

```
feature/<ticket-id>-<desc>     vd: feature/JIRA-123-user-login
fix/<short-desc>               vd: fix/null-pointer-on-logout
hotfix/<issue>                 vd: hotfix/payment-timeout
chore/<task>                   vd: chore/upgrade-react-19
```

## 7. Khi nào merge, khi nào rebase?

| Tình huống | Dùng |
|---|---|
| Gộp feature vào main, muốn giữ lịch sử nhánh | `merge --no-ff` |
| Feature có nhiều commit rác | `merge --squash` |
| Cập nhật feature theo main mới nhất (chưa share) | `rebase` |
| Branch đã share / đã push | **Không rebase** — dùng `merge` |

Rebase chi tiết: [05-advanced.md](05-advanced.md).

## 8. Cheat sheet branching

```bash
git switch -c feature/x           # tạo nhánh mới
git switch main                   # quay lại main
git branch -d feature/x           # xóa nhánh đã merge
git merge feature/x               # gộp feature vào nhánh hiện tại
git merge --abort                 # hủy khi đang conflict
git log --oneline --graph --all   # xem cây nhánh
```

→ Tiếp theo: [03-remote.md](03-remote.md)
