# Bài tập 04 — Undo: restore, reset, revert, reflog

> Sandbox:
> ```bash
> mkdir -p ~/git-sandbox/undo && cd ~/git-sandbox/undo
> ```
>
> Mẹo: trước mỗi bài tập, snapshot trạng thái bằng:
> ```bash
> git log --oneline > /tmp/before.txt
> ```
> Sau khi làm xong: `diff /tmp/before.txt <(git log --oneline)` để thấy đã thay đổi gì.

---

## Bài 1 — Bỏ thay đổi chưa stage

**Đề**: Có file `notes.md` đã commit. Sửa nội dung, **chưa stage**. Hoàn tác về như cũ.

<details>
<summary><b>Đáp án</b></summary>

```bash
git init && echo "v1" > notes.md && git add . && git commit -m "init"

echo "wrong" > notes.md
git status      # modified
cat notes.md    # → "wrong"

git restore notes.md
cat notes.md    # → "v1"
```
</details>

**Giải thích**: `git restore <file>` lấy nội dung từ HEAD đè lên working tree, **chỉ với file chưa staged**. Nếu file đã staged → `restore` không động vào staging.

---

## Bài 2 — Unstage file đã add nhầm

**Đề**: Stage nhầm 1 file lớn (`big.zip`) cùng với code. Unstage chỉ `big.zip`, giữ file kia trong staging.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "code" > src.js
echo "binary" > big.zip
git add .            # cả 2 đã staged

git restore --staged big.zip
git status
# Changes to be committed: src.js
# Untracked files: big.zip (vì là file mới, sau khi unstage thì lại untracked)
```
</details>

**Giải thích**: `git restore --staged` đẩy file từ staging về working tree (vẫn giữ nội dung). Cú pháp cũ tương đương: `git reset HEAD big.zip` (vẫn dùng được, nhưng dễ nhầm với reset commit).

---

## Bài 3 — Vứt cả thay đổi staged và unstaged

**Đề**: Sửa file, add một phần, rồi sửa tiếp. Bây giờ muốn vứt **toàn bộ** thay đổi của file.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "v1" > f.txt && git add . && git commit -m "init"

echo "v2" > f.txt
git add f.txt
echo "v3 (unstaged)" > f.txt   # thêm thay đổi nữa, chưa stage

# Vứt cả 2:
git restore --staged --worktree f.txt
cat f.txt      # → "v1"
```
</details>

**Giải thích**: `--staged --worktree` = vứt cả thay đổi đã stage lẫn chưa stage. Tương đương `git checkout HEAD -- f.txt` cú pháp cũ.

---

## Bài 4 — Bỏ commit cuối, **giữ thay đổi** trên staging

**Đề**: Đã commit 1 file, nhận ra muốn commit lại với message khác và file thêm. Bỏ commit cuối nhưng giữ thay đổi như trạng thái "đã staged".

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "x" > x && git add . && git commit -m "wrong message"

git reset --soft HEAD~1
git status     # → x là staged, chưa committed

# Bây giờ commit lại:
echo "y" > y && git add y
git commit -m "feat: add x and y"
```
</details>

**Giải thích**: `reset --soft` chỉ di chuyển con trỏ branch về commit cũ, **không động vào staging và working tree**. Thay đổi của commit cũ bị "bóc" ra, vẫn nằm trên staging → bạn commit lại.

(Cách khác cho case "sửa commit cuối": `git commit --amend`.)

---

## Bài 5 — Bỏ commit cuối, đưa thay đổi về **working tree**

**Đề**: Commit 1 file, sau đó muốn bỏ commit và unstage thay đổi (vẫn còn trên file).

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "z" > z && git add . && git commit -m "wip"

git reset HEAD~1     # = git reset --mixed HEAD~1 (mặc định)

git status
# Untracked files: z
# (commit "wip" biến mất; thay đổi nằm ở working tree, untracked vì file mới)
```
</details>

**Giải thích**: `--mixed` (mặc định) = bỏ commit + reset staging. Thay đổi quay về working tree, có thể edit thêm rồi commit lại.

---

## Bài 6 — Bỏ commit cuối + **xóa cả thay đổi** (nguy hiểm)

**Đề**: Commit 1 file. Sau đó muốn xóa luôn cả commit và file đó.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "garbage" > junk && git add . && git commit -m "junk"

git reset --hard HEAD~1
ls junk    # → no such file (file đã biến mất)
git log --oneline    # commit "junk" biến mất
```
</details>

**Giải thích**: `--hard` xóa sạch cả staging lẫn working tree. **Mọi thay đổi uncommitted sẽ mất** → cẩn thận. Nhưng commit thì cứu được bằng `git reflog` (bài 10).

---

## Bài 7 — Revert: hoàn tác an toàn 1 commit cũ

**Đề**: Repo có 5 commit (A, B, C, D, E). Commit C giới thiệu bug. Hoàn tác C nhưng giữ D, E.

<details>
<summary><b>Đáp án</b></summary>

```bash
git init
for c in A B C D E; do
  echo "$c" > "$c.txt"
  git add . && git commit -m "$c"
done

git log --oneline
# e123 E
# d123 D
# c123 C   ← bug
# b123 B
# a123 A

git revert c123
# → editor mở message commit revert → save
git log --oneline
# f456 Revert "C"
# e123 E
# d123 D
# c123 C
# b123 B
# a123 A

ls    # → C.txt biến mất, D.txt và E.txt vẫn còn
```
</details>

**Giải thích**: `revert` không xóa commit cũ — nó tạo **commit mới đảo ngược** thay đổi. An toàn 100% cho nhánh share vì không viết lại lịch sử. C, D, E vẫn còn trong log; chỉ có thay đổi của C bị "undo" bởi commit mới.

---

## Bài 8 — Revert commit có conflict

**Đề**: Trong bài 7, sau khi commit C tạo `C.txt`, commit D **sửa C.txt**. Bây giờ revert C → conflict với D.

<details>
<summary><b>Đáp án</b></summary>

```bash
# Tạo state có conflict
git init
echo "A" > A.txt && git add . && git commit -m "A"
echo "C content" > C.txt && git add . && git commit -m "C"
echo "C modified" > C.txt && git commit -am "D"   # D sửa C.txt

git log --oneline
# d... D
# c... C
# a... A

git revert <hash-of-C>
# CONFLICT — vì revert C muốn xóa C.txt, nhưng D đã sửa nó

# Resolve: quyết định giữ hay xóa
rm C.txt           # nếu muốn xóa hẳn (theo ý revert)
git add C.txt      # đánh dấu resolve (kể cả khi xóa)
git revert --continue
```
</details>

**Giải thích**: Revert vẫn có thể conflict — vì nó áp diff ngược của commit cũ lên trạng thái hiện tại. Resolve giống merge conflict.

---

## Bài 9 — Phục hồi file từ commit cũ (không reset)

**Đề**: File `app.js` đã được sửa qua nhiều commit. Lấy lại version ở commit cách HEAD 3 bước, **chỉ file đó**, không reset cả branch.

<details>
<summary><b>Đáp án</b></summary>

```bash
git restore --source=HEAD~3 app.js
git status      # modified: app.js
git diff        # thấy file đã đổi về version cũ

# Nếu muốn commit:
git add app.js
git commit -m "revert app.js to old version"

# Nếu không muốn: git restore app.js (về HEAD)
```
</details>

**Giải thích**: `--source` cho phép lấy file từ bất kỳ commit/branch/tag nào. `git checkout <ref> -- <file>` là cú pháp cũ, tương đương.

---

## Bài 10 — Cứu sau `reset --hard` nhầm

**Đề**:
1. Commit `important.txt`.
2. `git reset --hard HEAD~1` → mất commit.
3. Cứu nó về.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "v1" > x && git add . && git commit -m "init"
echo "VERY IMPORTANT" > important.txt && git add . && git commit -m "feat: important"

git log --oneline
# abc123 (HEAD -> main) feat: important
# def456 init

git reset --hard HEAD~1
git log --oneline
# def456 (HEAD -> main) init
# → commit "feat: important" biến mất!

# CỨU:
git reflog
# def456 HEAD@{0}: reset: moving to HEAD~1
# abc123 HEAD@{1}: commit: feat: important     ← đây
# def456 HEAD@{2}: commit: init

git reset --hard abc123
# Hoặc:
git switch -c rescue abc123   # tạo branch riêng cho an toàn

cat important.txt    # → "VERY IMPORTANT"
```
</details>

**Giải thích**: `git reflog` ghi lại MỌI di chuyển của HEAD trong ~90 ngày. Bất kể bạn reset, rebase, branch -D… commit vẫn nằm trong `.git/objects/` cho đến khi GC. Reflog là phao cứu sinh số 1 của Git.

---

## Bài 11 — Sửa commit cũ (không phải cuối) bằng interactive rebase

**Đề**: Có 5 commit, commit cách HEAD 3 bước có message sai. Sửa message của nó.

<details>
<summary><b>Đáp án</b></summary>

```bash
git log --oneline
# e... 5
# d... 4
# c... 3 (sai message)
# b... 2
# a... 1

git rebase -i HEAD~4
# Editor mở:
#   pick a... 1
#   pick b... 2
#   pick c... 3 (sai message)
#   pick d... 4
#   pick e... 5
# (lưu ý: rebase -i hiện commit theo thứ tự CŨ → MỚI, ngược log)

# Đổi "pick" của commit "3" thành "reword" (hoặc "r"):
#   pick a... 1
#   pick b... 2
#   r    c... 3 (sai message)
#   pick d... 4
#   pick e... 5

# Save & exit. Git dừng ở commit 3, mở editor mới để sửa message.
# Sửa "3 (sai message)" thành "3 (correct)", save & exit.

git log --oneline   # message commit 3 đã đổi (hash cũng đổi)
```
</details>

**Giải thích**: `rebase -i HEAD~N` cho bạn biên tập N commit cuối. `reword` chỉ đổi message; `edit` cho dừng để sửa nội dung; `squash` / `fixup` để gộp. Nhớ: **đổi message = đổi hash** → nhánh đã push thì phải force-with-lease.

---

## Bài 12 — Cứu branch lỡ xóa

**Đề**:
1. Tạo `feature/important` từ `main`, commit 1 file.
2. Quay về `main`, xóa cưỡng bức `git branch -D feature/important`.
3. Cứu lại.

<details>
<summary><b>Đáp án</b></summary>

```bash
git switch -c feature/important
echo "data" > data && git add . && git commit -m "feat: data"
git switch main
git branch -D feature/important

git reflog
# 1234abc HEAD@{1}: commit: feat: data       ← hash của commit ở branch đã xóa

git switch -c feature/important 1234abc
git log --oneline    # → thấy commit "feat: data"
```
</details>

**Giải thích**: Xóa branch chỉ xóa **con trỏ** — commit vẫn còn trong `.git/objects/` cho đến khi GC. Reflog ghi lại commit của branch đã xóa. Có thể tái tạo branch từ hash.

(Nếu reflog cũng đã bị xóa do `git gc --aggressive`, có thể thử `git fsck --lost-found` — quét commit "mồ côi".)
