# 01 — Git cơ bản

## 1. Cấu hình lần đầu

Sau khi cài Git, cần khai báo danh tính (sẽ gắn vào mọi commit):

```bash
git config --global user.name "Moet"
git config --global user.email "zfighting660@gmail.com"
```

Một vài cấu hình nên bật:

```bash
git config --global init.defaultBranch main      # nhánh mặc định là main, không phải master
git config --global core.editor "code --wait"    # dùng VS Code / Cursor làm editor
git config --global pull.rebase false            # pull = merge (an toàn cho người mới)
git config --global color.ui auto                # bật màu trong terminal
```

Xem lại cấu hình:

```bash
git config --list                  # tất cả
git config user.name               # 1 key
git config --global --edit         # mở file ~/.gitconfig
```

**Phạm vi**: `--system` (cả máy) < `--global` (1 user) < `--local` (1 repo). Cấu hình ở phạm vi nhỏ hơn sẽ override.

## 2. Tạo và clone repository

```bash
git init                                      # tạo repo mới từ thư mục hiện tại
git init my-project                           # tạo thư mục my-project + repo
git clone https://github.com/user/repo.git    # clone repo từ remote
git clone <url> ten-thu-muc                   # clone vào thư mục có tên khác
```

`git init` tạo thư mục ẩn `.git/` — toàn bộ lịch sử và metadata nằm trong đó. Xóa `.git/` = xóa toàn bộ lịch sử Git.

## 3. Workflow cơ bản: add → commit

Ba lệnh dùng nhiều nhất:

```bash
git status                  # xem file đang ở vùng nào
git add <file>              # đưa file vào staging
git commit -m "message"     # tạo commit từ staging
```

Các biến thể `add`:

```bash
git add file.txt            # 1 file
git add src/ docs/          # nhiều thư mục
git add .                   # mọi thứ trong thư mục hiện tại
git add -A                  # mọi thay đổi trong cả repo (kể cả file bị xóa)
git add -p                  # interactive — chọn từng hunk (rất hữu ích!)
```

Commit:

```bash
git commit -m "fix: handle null user in login"           # commit ngắn
git commit                                                # mở editor để viết message dài
git commit -a -m "msg"                                    # add + commit file đã tracked (không thêm file mới)
git commit --amend                                        # sửa commit cuối (CẢNH BÁO: đổi hash, không amend commit đã push)
git commit --amend --no-edit                              # amend nhưng giữ message cũ
```

### Quy tắc viết commit message tốt

```
<type>(<scope>): <subject>     ← dòng tiêu đề, ≤ 50 ký tự, mệnh lệnh

<body — giải thích WHY, không phải WHAT>

<footer — issue refs, breaking changes>
```

Ví dụ:

```
fix(auth): reject expired JWT before DB lookup

Previously the token was decoded and the user was fetched
before we checked exp, causing wasted DB queries.

Closes #142
```

Xem chi tiết ở [06-workflows.md](06-workflows.md) phần Conventional Commits.

## 4. Xem trạng thái và lịch sử

```bash
git status                          # file nào đang stage / modified / untracked
git status -s                       # bản gọn (2 cột: staged | working)

git log                             # toàn bộ lịch sử
git log --oneline                   # 1 commit / 1 dòng
git log --oneline --graph --all     # cây nhánh, tất cả branches
git log -n 5                        # 5 commit gần nhất
git log --author="Moet"             # lọc theo tác giả
git log --since="2 days ago"        # lọc theo thời gian
git log -- path/to/file             # lịch sử của 1 file
git log -p                          # kèm diff
```

So sánh thay đổi:

```bash
git diff                       # working tree vs staging
git diff --staged              # staging vs last commit (= --cached)
git diff HEAD                  # working tree vs last commit
git diff main feature          # giữa 2 branch
git diff <hash1> <hash2>       # giữa 2 commit
git diff -- file.txt           # chỉ 1 file
```

Xem 1 commit cụ thể:

```bash
git show                       # commit HEAD
git show <hash>                # commit cụ thể
git show HEAD~3                # commit cách HEAD 3 bước về quá khứ
git show HEAD:path/to/file     # nội dung file tại commit đó
```

## 5. Phân biệt các "vùng" của file

| Trạng thái | Nghĩa | Cách đưa vào |
|---|---|---|
| **Untracked** | Git chưa biết file này | `git add` |
| **Modified** (unstaged) | Đã sửa, chưa add | `git add` |
| **Staged** | Đã add, đợi commit | `git commit` |
| **Committed** | Đã vào local repo | `git push` (lên remote) |
| **Ignored** | Bị `.gitignore` bỏ qua | — |

## 6. .gitignore

File `.gitignore` cho biết file/thư mục nào Git nên bỏ qua.

Ví dụ điển hình cho Node.js:

```gitignore
# Dependencies
node_modules/

# Build output
dist/
build/
*.log

# Environment
.env
.env.local

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/

# Specific file
config/secrets.json

# Except this one (negation)
!config/secrets.example.json
```

Cú pháp:

- `name` — match file hoặc folder có tên đó ở mọi cấp.
- `name/` — chỉ match folder.
- `/name` — chỉ match ở root của repo.
- `*.log` — wildcard.
- `**/temp` — bất kỳ cấp nào.
- `!pattern` — phủ định (giữ lại file đã bị pattern trước bỏ qua).

**Lưu ý quan trọng**: `.gitignore` chỉ áp dụng cho file **chưa được tracked**. Nếu file đã được commit từ trước, phải gỡ tracking thủ công:

```bash
git rm --cached file.txt           # gỡ khỏi tracking nhưng giữ file trên đĩa
git rm --cached -r node_modules/   # đệ quy
```

Sample `.gitignore` cho nhiều ngôn ngữ: https://github.com/github/gitignore

## 7. Bỏ và đổi tên file

```bash
git rm file.txt                # xóa file + stage thay đổi
git rm --cached file.txt       # gỡ tracking, giữ file
git mv old.txt new.txt         # đổi tên (= mv + git add + git rm)
```

Nếu lỡ xóa thủ công bằng `rm`, dùng `git add -A` để stage cả việc xóa.

## 8. Bảng cheat sheet 10 lệnh dùng 95% thời gian

```bash
git status                                         # tôi đang ở đâu
git add <file> | git add -p                        # chọn cái sẽ commit
git commit -m "msg"                                # ghi snapshot
git log --oneline --graph --all                    # xem lịch sử
git diff | git diff --staged                       # xem thay đổi
git switch <branch> | git switch -c <new>          # nhảy / tạo nhánh
git merge <branch>                                 # gộp nhánh khác vào nhánh hiện tại
git pull                                           # fetch + merge từ remote
git push                                           # đẩy lên remote
git restore <file> | git restore --staged <file>   # hoàn tác
```

→ Tiếp theo: [02-branching.md](02-branching.md)
