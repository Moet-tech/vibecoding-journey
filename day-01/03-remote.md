# 03 — Remote: làm việc với GitHub/GitLab

## 1. Remote là gì?

**Remote** = bản sao của repo trên 1 server khác (GitHub, GitLab, máy đồng nghiệp…). Git có thể có **nhiều remote** cùng lúc, mỗi cái có 1 tên — mặc định là `origin`.

```bash
git remote                       # liệt kê tên remote
git remote -v                    # kèm URL
git remote show origin           # chi tiết: URL, branches, tracking
```

Thêm / sửa / xóa remote:

```bash
git remote add origin <url>                   # thêm remote tên origin
git remote add upstream <url>                 # thêm thêm 1 remote nữa (vd: repo gốc khi fork)
git remote rename origin gh                   # đổi tên
git remote remove origin                      # xóa
git remote set-url origin <new-url>           # đổi URL
```

**HTTPS vs SSH** — 2 kiểu URL:

```
HTTPS: https://github.com/user/repo.git       (cần Personal Access Token để push)
SSH:   git@github.com:user/repo.git           (cần SSH key — bảo mật và tiện hơn)
```

Đổi sau khi clone:

```bash
git remote set-url origin git@github.com:user/repo.git
```

## 2. Clone

```bash
git clone <url>                                  # clone về thư mục cùng tên repo
git clone <url> my-folder                        # clone vào thư mục tự đặt
git clone --depth 1 <url>                        # shallow clone (chỉ 1 commit gần nhất, nhanh)
git clone --branch dev <url>                     # clone và checkout sẵn nhánh dev
git clone --recurse-submodules <url>             # clone kèm submodules
```

Khi clone, Git tự động:
- Tạo remote tên `origin` trỏ về URL đó.
- Tạo branch local tracking `origin/HEAD` (thường là `main`).

## 3. Fetch vs Pull vs Push

### fetch — đồng bộ về local, KHÔNG merge

```bash
git fetch                        # cập nhật mọi nhánh từ origin
git fetch origin                 # tường minh remote
git fetch origin main            # chỉ 1 nhánh
git fetch --all                  # tất cả remotes
git fetch --prune                # xóa nhánh local đang track nhánh remote đã bị xóa
```

Sau fetch, các nhánh remote (`origin/main`, `origin/dev`…) được cập nhật, **nhưng nhánh local của bạn KHÔNG đổi**.

### pull — fetch + merge

```bash
git pull                         # = git fetch + git merge origin/<branch>
git pull --rebase                # = git fetch + git rebase origin/<branch>
git pull --ff-only               # chỉ nhận FF; fail nếu cần merge — an toàn
```

**Khuyến nghị**: cấu hình `pull.rebase` rõ ràng để khỏi nhầm:

```bash
git config --global pull.rebase false   # luôn merge (mặc định, an toàn)
git config --global pull.rebase true    # luôn rebase (history thẳng, nhưng dễ rebase nhầm)
git config --global pull.ff only        # chỉ FF, không tự merge/rebase (an toàn nhất)
```

### push — đẩy lên remote

```bash
git push                                 # đẩy nhánh hiện tại lên upstream của nó
git push origin main                     # tường minh
git push -u origin feature/x             # push lần đầu + thiết lập tracking (-u = --set-upstream)
git push --all                           # đẩy mọi nhánh
git push --tags                          # đẩy tags
git push --delete origin feature/x       # xóa nhánh trên remote
git push --force-with-lease              # force push AN TOÀN (xem mục 6)
```

## 4. Tracking branch

Một nhánh local có thể **track** 1 nhánh remote — Git biết nó "thuộc về" đâu khi pull / push.

```bash
git branch -vv                          # xem nhánh nào track nhánh nào
git branch -u origin/main main          # thiết lập tracking thủ công
git push -u origin feature/x            # cách dễ nhất: push lần đầu kèm -u
```

Khi đã có tracking:

```bash
git status
# On branch feature/x
# Your branch is ahead of 'origin/feature/x' by 2 commits.
```

## 5. Quy trình hợp tác qua GitHub (Pull Request)

```
1. fork repo (nếu không phải maintainer)
2. clone fork về máy
3. git switch -c feature/x
4. code, commit
5. git push -u origin feature/x
6. Mở PR trên GitHub: feature/x → main
7. Code review, sửa theo comment, push thêm
8. Maintainer merge PR
9. Xóa nhánh: git push origin --delete feature/x
              git branch -d feature/x
```

Nếu là fork của repo khác:

```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git merge upstream/main                  # đồng bộ main của fork với upstream
git push origin main
```

## 6. Force push — vũ khí 2 lưỡi

`git push --force` (hoặc `-f`) **ghi đè** lịch sử trên remote. Nếu đồng nghiệp đã pull, lịch sử của họ và bạn sẽ phân kỳ → drama.

**Quy tắc**:

- Không force push lên `main` / `master` / `develop` (nhánh chung).
- OK force push nhánh **cá nhân** của bạn (sau khi rebase / amend).
- Luôn dùng `--force-with-lease` thay vì `--force`:

```bash
git push --force-with-lease              # chỉ force nếu remote vẫn ở state mà mình nghĩ
git push --force                         # force vô điều kiện (nguy hiểm)
```

`--force-with-lease` sẽ fail nếu có ai đó vừa push thêm — tránh ghi đè nhầm việc của họ.

## 7. SSH key cho GitHub (nếu chưa có)

```bash
ssh-keygen -t ed25519 -C "zfighting660@gmail.com"     # tạo key mới
cat ~/.ssh/id_ed25519.pub                              # copy phần này lên GitHub
ssh -T git@github.com                                  # test kết nối
```

Vào GitHub → Settings → SSH and GPG keys → New SSH key → paste key.

## 8. Authentication: HTTPS với PAT (Personal Access Token)

Nếu dùng HTTPS, password account **không còn dùng được** từ 2021. Phải tạo PAT:

GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate.

Khi `git push` lần đầu, dùng PAT làm password. Thường được lưu vào keychain:

```bash
git config --global credential.helper osxkeychain      # macOS
git config --global credential.helper store            # Linux (lưu plaintext, ít an toàn)
git config --global credential.helper manager          # Windows
```

## 9. Cheat sheet remote

```bash
git clone <url>                       # tải repo về
git remote -v                         # xem remote
git fetch                             # đồng bộ về local, không merge
git pull                              # fetch + merge
git push                              # đẩy commit hiện tại
git push -u origin <branch>           # push lần đầu + set tracking
git push --force-with-lease           # force push an toàn
git push origin --delete <branch>     # xóa nhánh trên remote
```

→ Tiếp theo: [04-undoing.md](04-undoing.md)
