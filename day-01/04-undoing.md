# 04 — Hoàn tác: restore, reset, revert, reflog

Đây là phần dễ nhầm nhất của Git. Quy tắc vàng: **trước khi reset/revert, hãy biết file/commit đang ở "vùng" nào** (working tree / staging / repo / remote).

## 1. Bảng quyết định nhanh

| Tình huống | Lệnh |
|---|---|
| Lỡ sửa file, chưa add — muốn về như cũ | `git restore <file>` |
| Đã `git add`, muốn unstage (giữ thay đổi trên file) | `git restore --staged <file>` |
| Muốn xóa cả thay đổi đã add | `git restore --staged <file> && git restore <file>` |
| Commit cuối sai message | `git commit --amend -m "msg"` |
| Commit cuối thiếu file | `git add <file> && git commit --amend --no-edit` |
| Muốn bỏ commit cuối, giữ thay đổi staged | `git reset --soft HEAD~1` |
| Muốn bỏ commit cuối, unstage thay đổi | `git reset --mixed HEAD~1` (mặc định) |
| Muốn bỏ commit cuối, **xóa cả thay đổi** | `git reset --hard HEAD~1` ⚠️ |
| Muốn hủy 1 commit cũ đã push | `git revert <hash>` |
| Lỡ reset --hard, muốn cứu | `git reflog` + `git reset --hard <hash>` |

## 2. git restore — hoàn tác file (Git ≥ 2.23)

```bash
git restore <file>                       # bỏ thay đổi chưa stage (working tree → HEAD)
git restore --staged <file>              # unstage (staging → working tree)
git restore --source=HEAD~2 <file>       # lấy file từ commit cũ vào working tree
git restore --staged --worktree <file>   # vứt cả staged và unstaged
git restore .                            # mọi file (cẩn thận)
```

Cú pháp cũ `git checkout -- <file>` vẫn chạy nhưng không khuyến nghị (dễ nhầm với checkout branch).

## 3. git reset — di chuyển con trỏ branch

`reset` **dịch chuyển branch hiện tại** về 1 commit khác. Có 3 mode khác nhau ở chỗ có động vào staging và working tree hay không:

```
                            Repo    Staging   Working tree
git reset --soft  <ref>      ✓         ✗            ✗
git reset --mixed <ref>      ✓         ✓            ✗        ← mặc định
git reset --hard  <ref>      ✓         ✓            ✓        ← nguy hiểm
```

- `--soft`: di chuyển HEAD, **giữ nguyên** staging + working tree → giống như "gộp các commit lại để commit lại".
- `--mixed` (mặc định): di chuyển HEAD, **reset staging**, **giữ working tree** → các thay đổi chuyển về dạng "modified, chưa stage".
- `--hard`: **xóa sạch** staging và working tree. Mất file uncommitted → khó cứu (nhưng vẫn cứu được commit bằng reflog, mục 6).

Ví dụ:

```bash
git reset --soft HEAD~1          # bỏ 1 commit cuối, giữ thay đổi trên staging
git reset HEAD~3                 # bỏ 3 commit cuối, thay đổi về working tree
git reset --hard origin/main     # bỏ mọi commit local chưa push, đồng bộ với remote
git reset --hard HEAD            # xóa hết thay đổi chưa commit (cẩn thận!)
```

**Cảnh báo**: KHÔNG `reset` 1 branch đã share với người khác. Nó viết lại lịch sử local → khi push bị từ chối → buộc force push → đè lên việc của họ. Dùng `revert` thay thế.

## 4. git revert — hoàn tác an toàn

`revert` tạo **1 commit mới** đảo ngược 1 commit cũ. Không viết lại lịch sử → **an toàn cho nhánh share**.

```bash
git revert <hash>                # tạo commit đảo ngược commit đó
git revert HEAD                  # revert commit cuối
git revert HEAD~3..HEAD          # revert 3 commit gần nhất (mỗi commit 1 revert)
git revert --no-commit <hash>    # áp dụng thay đổi vào staging, chưa commit
```

Khi có conflict (commit đảo ngược động vào code đã đổi): resolve → `git add` → `git revert --continue`.

### Khi nào reset, khi nào revert?

```
Branch private (chỉ của bạn, chưa push):     reset OK
Branch đã push, chưa ai pull:                reset + force-with-lease tạm OK
Branch share (main, develop…):               LUÔN dùng revert
```

## 5. git commit --amend

Sửa **commit cuối cùng** — đổi message hoặc thêm file:

```bash
git commit --amend                          # mở editor, sửa message
git commit --amend -m "new message"         # sửa message tại chỗ
git add forgotten.txt
git commit --amend --no-edit                # thêm file vào commit cuối, giữ message
```

**Amend đổi hash của commit** → nếu commit đã push, cần `--force-with-lease`. Đừng amend commit chung.

## 6. git reflog — phao cứu sinh

Mọi thao tác làm `HEAD` di chuyển (commit, reset, checkout, merge…) đều được Git ghi vào **reflog**. Reflog lưu local, mặc định giữ ~90 ngày.

```bash
git reflog                       # xem lịch sử di chuyển HEAD
git reflog show feature/x        # reflog của 1 nhánh cụ thể
```

Output ví dụ:

```
abc1234 HEAD@{0}: reset: moving to HEAD~3
def5678 HEAD@{1}: commit: add login
9999aaa HEAD@{2}: commit: fix bug
...
```

Cứu sau khi `reset --hard` nhầm:

```bash
git reflog                       # tìm hash trước khi reset
git reset --hard def5678         # quay lại trạng thái đó
# hoặc
git switch -c rescue def5678     # tạo branch mới từ trạng thái đó
```

Cứu commit "mồ côi" sau khi xóa branch chưa merge:

```bash
git reflog | grep "feature/x"
git switch -c feature/x <hash>
```

## 7. Khôi phục file từ commit cũ

```bash
git restore --source=<hash> <file>          # ghi đè file bằng version cũ
git show <hash>:path/to/file > out.txt      # xuất ra file khác (không thay file gốc)
git checkout <hash> -- <file>               # cú pháp cũ, vẫn dùng được
```

## 8. Xóa file đã commit lỡ (chứa secret)

⚠️ Nếu repo đã public và secret đã bị lộ → coi như đã lộ. **Xoay secret trước**, rồi mới dọn lịch sử.

Công cụ hiện đại: [`git filter-repo`](https://github.com/newren/git-filter-repo).

```bash
git filter-repo --path secrets.json --invert-paths     # xóa file khỏi mọi commit
git push --force --all                                  # đè lên remote
```

`filter-branch` cũ đã bị deprecate.

## 9. Cheat sheet undo

```bash
git restore <file>                          # bỏ thay đổi chưa stage
git restore --staged <file>                 # unstage
git commit --amend --no-edit                # thêm file vào commit cuối
git reset --soft HEAD~1                     # bỏ commit cuối, giữ thay đổi staged
git reset --hard HEAD~1                     # bỏ commit cuối, xóa thay đổi (⚠️)
git revert <hash>                           # đảo ngược 1 commit (an toàn)
git reflog                                  # tìm về trạng thái cũ
```

→ Tiếp theo: [05-advanced.md](05-advanced.md)
