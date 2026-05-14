# 05 — Git nâng cao: rebase, cherry-pick, stash, tag, bisect, hooks

## 1. Rebase — viết lại lịch sử

`rebase` "di dời" 1 chuỗi commit sang 1 base mới. Khác với merge (tạo commit hợp nhất), rebase tạo lịch sử **thẳng** như chưa từng nhánh.

```
Trước:  A ── B ── C        ← main
              └── D ── E   ← feature

Sau git rebase main (đứng ở feature):
        A ── B ── C        ← main
                  └── D' ── E'   ← feature (D, E đã có hash mới)
```

```bash
git switch feature
git rebase main                    # áp D, E lên đầu main
```

Khi conflict:

```bash
# resolve file
git add <file>
git rebase --continue              # tiếp tục
# hoặc
git rebase --abort                 # bỏ rebase, quay lại trạng thái trước
git rebase --skip                  # bỏ qua commit hiện tại (hiếm khi dùng)
```

### Quy tắc vàng

**Đừng rebase commit đã share** — vì rebase đổi hash, đồng nghiệp đã pull sẽ bị lệch lịch sử. An toàn nhất: chỉ rebase **nhánh cá nhân chưa push** (hoặc đã push nhưng chỉ mình bạn dùng).

### merge vs rebase

| | merge | rebase |
|---|---|---|
| Lịch sử | Có hợp nhất, dễ thấy nhánh | Thẳng, gọn |
| Hash | Giữ nguyên | Tạo mới |
| Truy vết | Dễ thấy "feature này gộp khi nào" | Khó thấy |
| An toàn nhánh share | ✓ | ✗ |
| Bisect (mục 7) | Khó hơn | Dễ hơn |

## 2. Interactive rebase — biên tập lịch sử

Đây là chỗ rebase phát huy sức mạnh nhất:

```bash
git rebase -i HEAD~5               # mở editor cho 5 commit gần nhất
git rebase -i <hash>               # tất cả commit sau hash đó
```

Editor sẽ hiện:

```
pick abc123 fix typo
pick def456 add login form
pick ghi789 wip
pick jkl012 wip 2
pick mno345 fix login bug
```

Đổi `pick` thành 1 trong các action:

| Action | Tác dụng |
|---|---|
| `pick` | Giữ nguyên |
| `reword` (`r`) | Giữ commit, đổi message |
| `edit` (`e`) | Dừng để bạn sửa nội dung commit |
| `squash` (`s`) | Gộp commit này vào commit phía trên, **gộp cả 2 message** |
| `fixup` (`f`) | Như squash nhưng **bỏ message của commit này** |
| `drop` (`d`) | Bỏ commit |
| `exec` (`x`) | Chạy 1 lệnh shell sau commit |

Sau khi save, Git áp lần lượt — có thể dừng giữa chừng nếu chọn `edit` / `reword` hoặc khi gặp conflict.

Ví dụ: gộp 3 commit "wip" vào 1 commit chính:

```
pick abc123 add login form
fixup def456 wip
fixup ghi789 wip 2
```

→ Kết quả: 1 commit `add login form` chứa tất cả thay đổi.

## 3. Cherry-pick — bê 1 commit sang nhánh khác

Lấy 1 (hoặc vài) commit cụ thể từ nhánh khác áp vào nhánh hiện tại:

```bash
git cherry-pick <hash>                         # 1 commit
git cherry-pick <h1> <h2> <h3>                 # nhiều commit
git cherry-pick <h1>..<h2>                     # 1 dải (KHÔNG bao gồm h1)
git cherry-pick <h1>^..<h2>                    # 1 dải (CÓ h1)
git cherry-pick --no-commit <hash>             # áp vào staging, chưa commit
git cherry-pick -x <hash>                      # thêm dòng "(cherry picked from ...)" vào message
```

Conflict: resolve → `git add` → `git cherry-pick --continue`.

**Dùng khi**: hotfix trên main cần áp ngược lại branch release cũ, hoặc lấy 1 bug fix từ feature branch khác mà không merge cả branch.

## 4. Stash — cất tạm

Khi đang sửa dở thì cần switch nhánh khác:

```bash
git stash                          # cất working tree + staging vào kho
git stash push -m "WIP login"      # đặt message
git stash -u                       # bao gồm cả file untracked
git stash -a                       # bao gồm cả ignored (-a = all)

git stash list                     # liệt kê stash
git stash show stash@{0}           # tóm tắt 1 stash
git stash show -p stash@{0}        # xem diff

git stash pop                      # áp stash mới nhất + xóa khỏi kho
git stash apply                    # áp nhưng KHÔNG xóa
git stash pop stash@{2}            # áp stash thứ 3

git stash drop stash@{0}           # xóa 1 stash
git stash clear                    # xóa hết
git stash branch fix stash@{0}     # tạo branch mới từ stash (hữu ích khi conflict)
```

**Lưu ý**: `git stash pop` có thể conflict — resolve như merge bình thường, nhưng stash sẽ không tự xóa khỏi kho cho đến khi bạn xóa thủ công.

## 5. Tag — đánh dấu phiên bản

Tag là pointer **không di chuyển**, thường dùng đánh dấu release.

```bash
git tag                                    # liệt kê
git tag v1.0.0                             # lightweight tag (chỉ là pointer)
git tag -a v1.0.0 -m "Release 1.0.0"       # annotated tag (kèm metadata) — KHUYẾN NGHỊ
git tag -a v1.0.0 <hash>                   # tag vào commit cụ thể

git show v1.0.0                            # xem chi tiết tag
git push origin v1.0.0                     # push 1 tag
git push --tags                            # push mọi tag
git tag -d v1.0.0                          # xóa local
git push origin --delete v1.0.0            # xóa trên remote
```

Convention: SemVer (`v<major>.<minor>.<patch>`, vd `v2.4.1`). Có thể đặt prerelease: `v1.0.0-beta.1`.

## 6. Worktree — nhiều working tree cùng repo

Cần checkout 2 branch cùng lúc (vd để build branch B trong khi vẫn code ở A) mà không clone lại?

```bash
git worktree add ../repo-hotfix hotfix/login    # tạo worktree mới
git worktree list                                # xem danh sách
git worktree remove ../repo-hotfix               # xóa
```

Mỗi worktree là 1 thư mục độc lập, share chung `.git/`.

## 7. Bisect — tìm commit gây bug bằng binary search

Bạn biết version cũ ổn, version mới bị lỗi → bisect tìm commit gây lỗi chỉ trong O(log n) bước:

```bash
git bisect start
git bisect bad                     # commit hiện tại bị lỗi
git bisect good v1.0.0             # version v1.0.0 chạy ổn

# Git checkout commit ở giữa → bạn test
git bisect good                    # nếu commit này OK
git bisect bad                     # nếu commit này bị lỗi
# Git lại nhảy đến commit giữa kế tiếp...

git bisect reset                   # khi xong, quay về branch ban đầu
```

Tự động hóa: nếu có script test trả về exit code 0 (OK) / khác 0 (bad):

```bash
git bisect start HEAD v1.0.0
git bisect run npm test
```

## 8. Hooks — chạy script tự động

Git có thể chạy script ở các thời điểm: trước commit, sau merge, trước push…

Hook nằm trong `.git/hooks/`:

```
.git/hooks/
├── pre-commit.sample
├── commit-msg.sample
├── pre-push.sample
└── ...
```

Bỏ đuôi `.sample` để kích hoạt. Hook phải có quyền execute (`chmod +x`).

Ví dụ `pre-commit` chạy lint:

```bash
#!/bin/sh
npm run lint || { echo "Lint failed"; exit 1; }
```

**Thực tế**: ít ai viết hook tay. Dùng [Husky](https://typicode.github.io/husky/) (Node) hoặc [pre-commit](https://pre-commit.com/) (Python) để quản lý hook share được qua repo (vì `.git/hooks` không được commit).

## 9. Submodule — repo trong repo

Khi cần nhúng repo khác vào project (vd: thư viện share):

```bash
git submodule add https://github.com/user/lib.git vendor/lib
git submodule init
git submodule update                     # checkout commit đã pin
git clone --recurse-submodules <url>     # clone repo cha kèm submodule
git submodule update --remote            # update submodule lên commit mới nhất
```

Mỗi submodule pin vào 1 commit cụ thể. Cha repo lưu hash đó.

**Cẩn trọng**: submodule khá phiền — cân nhắc dùng monorepo, package manager, hoặc `git subtree` thay thế.

## 10. Một số tip hữu ích

```bash
git log --grep="login"                       # tìm commit theo message
git log -S "functionName"                    # tìm commit thêm/xóa 1 chuỗi (pickaxe)
git log --follow file.txt                    # log của file kể cả khi đã đổi tên
git blame file.txt                           # ai sửa dòng nào, khi nào
git blame -L 10,20 file.txt                  # chỉ dòng 10-20
git shortlog -sn                             # số commit theo tác giả
git diff --stat                              # tóm tắt: file nào đổi, +/- bao nhiêu
git clean -fd                                # xóa file untracked + folder (cẩn thận!)
git clean -n                                 # dry run (xem sẽ xóa gì)
```

## 11. Cheat sheet nâng cao

```bash
git rebase -i HEAD~5                         # biên tập 5 commit cuối
git cherry-pick <hash>                       # bê 1 commit
git stash push -m "wip"                      # cất tạm
git stash pop                                # lấy lại
git tag -a v1.0.0 -m "Release"               # tag annotated
git reflog                                   # cứu mọi thứ
git bisect start; git bisect bad; git bisect good <hash>   # săn bug
git blame file.txt                           # ai sửa dòng nào
```

→ Tiếp theo: [06-workflows.md](06-workflows.md)
