# Bài tập 05 — Git nâng cao

> Sandbox:
> ```bash
> mkdir -p ~/git-sandbox/advanced && cd ~/git-sandbox/advanced
> ```

---

## Bài 1 — Rebase đơn giản

**Đề**:
1. `main` có A, B.
2. Tạo `feature/x` từ B, commit thêm C, D.
3. Trên `main` commit thêm E.
4. Rebase `feature/x` lên `main` (mới).
5. So sánh cây trước & sau.

<details>
<summary><b>Đáp án</b></summary>

```bash
git init && echo A > a && git add . && git commit -m "A"
echo B > b && git add . && git commit -m "B"

git switch -c feature/x
echo C > c && git add . && git commit -m "C"
echo D > d && git add . && git commit -m "D"

git switch main
echo E > e && git add . && git commit -m "E"

git log --oneline --graph --all
# * E (main)
# | * D (feature/x)
# | * C
# |/
# * B
# * A

git switch feature/x
git rebase main

git log --oneline --graph --all
# * D' (feature/x)
# * C'
# * E (main)
# * B
# * A
```
</details>

**Giải thích**: Trước rebase: cây hình chữ Y. Sau rebase: thẳng. C và D có hash mới (C', D') vì base đã đổi. Lưu ý: chỉ rebase được khi feature/x chưa share.

---

## Bài 2 — Rebase conflict

**Đề**: Tạo conflict bằng cách cùng sửa 1 file:
1. `main` có `a.txt` = "1".
2. `feature/x` sửa thành "2".
3. `main` sửa thành "3".
4. Rebase `feature/x` lên main → conflict. Resolve để giữ "2".

<details>
<summary><b>Đáp án</b></summary>

```bash
git init
echo 1 > a.txt && git add . && git commit -m "init"

git switch -c feature/x
echo 2 > a.txt && git commit -am "feature: 2"

git switch main
echo 3 > a.txt && git commit -am "main: 3"

git switch feature/x
git rebase main
# CONFLICT in a.txt

# Mở a.txt:
# <<<<<<< HEAD            ← rebase: HEAD = main (đã reset feature/x về main rồi áp commit)
# 3
# =======
# 2
# >>>>>>> feature: 2

echo 2 > a.txt
git add a.txt
git rebase --continue
```
</details>

**Giải thích**: **Cảnh báo "ours"/"theirs" đảo ngược trong rebase**:
- Khi merge: ours = HEAD (nhánh bạn đứng).
- Khi rebase: ours = upstream (main); theirs = commit của bạn (feature). Vì rebase coi như "đi tới từng commit và replay".

Nếu confused, dùng tay sửa file trực tiếp, không dùng `--ours/--theirs`.

---

## Bài 3 — Interactive rebase: squash

**Đề**: Trên `feature/y` có 4 commit: `"feat: form"`, `"fix typo"`, `"fix typo 2"`, `"fix bug"`. Gộp 3 commit cuối vào commit đầu → còn 1 commit duy nhất.

<details>
<summary><b>Đáp án</b></summary>

```bash
git init && echo init > x && git add . && git commit -m "init"

git switch -c feature/y
echo form > form && git add . && git commit -m "feat: form"
echo typo > typo1 && git add . && git commit -m "fix typo"
echo typo > typo2 && git add . && git commit -m "fix typo 2"
echo bug > bug && git add . && git commit -m "fix bug"

git rebase -i HEAD~4
# Editor mở (thứ tự cũ → mới):
#   pick aaa feat: form
#   pick bbb fix typo
#   pick ccc fix typo 2
#   pick ddd fix bug

# Đổi 3 commit sau thành 'fixup' (gộp + bỏ message):
#   pick aaa feat: form
#   f    bbb fix typo
#   f    ccc fix typo 2
#   f    ddd fix bug

# Save & exit. Kết quả:
git log --oneline
# xxx feat: form    ← 1 commit chứa form + typo1 + typo2 + bug
# yyy init
```
</details>

**Giải thích**:
- `squash` (s) = gộp + gộp cả message (Git sẽ mở editor để bạn chỉnh message).
- `fixup` (f) = gộp + **bỏ message của commit này** (giữ message của commit phía trên).

Hữu ích: trong khi code feature, mỗi lần fix lặt vặt có thể commit `--fixup=<hash>` → rebase với `--autosquash` sẽ tự sắp xếp và gộp.

---

## Bài 4 — Cherry-pick 1 commit

**Đề**: Trên `main` có hotfix commit (hash `xxx`). Branch `release/1.0` cũ không có hotfix. Cherry-pick hotfix sang `release/1.0`.

<details>
<summary><b>Đáp án</b></summary>

```bash
# Setup
git init && echo init > a && git add . && git commit -m "init"
git switch -c release/1.0
echo "rel 1.0" > rel && git add . && git commit -m "release 1.0"

git switch main
echo "hotfix" > fix.txt && git add . && git commit -m "fix: critical bug"
HOTFIX=$(git rev-parse HEAD)

git switch release/1.0
git cherry-pick $HOTFIX

git log --oneline   # → có commit hotfix (hash mới, message giống)
ls                  # → có fix.txt trên release/1.0
```
</details>

**Giải thích**: Cherry-pick = áp diff của commit X lên nhánh hiện tại + tạo commit mới. Hash sẽ khác (vì parent khác). Dùng nhiều khi: backport bug fix sang release branch cũ, lấy commit hữu ích từ branch khác mà không muốn merge cả branch.

---

## Bài 5 — Cherry-pick 1 dải commit

**Đề**: Trên `main` có 5 commit A-B-C-D-E. Cherry-pick C và D sang `feature/z`.

<details>
<summary><b>Đáp án</b></summary>

```bash
git switch -c feature/z <commit-trước-A>

# Cách 1: từng commit
git cherry-pick <hash-C> <hash-D>

# Cách 2: dùng range — LƯU Ý cú pháp ^
git cherry-pick <hash-B>..<hash-D>     # KHÔNG bao gồm B → từ C đến D
git cherry-pick <hash-C>^..<hash-D>    # bao gồm C → từ C đến D
```
</details>

**Giải thích**: Range trong Git theo convention "exclusive start, inclusive end": `B..D` = "commit có ở D mà chưa có ở B" = C, D. Để include B, dùng `B^..D` (`^` = parent của B). Hoặc liệt kê từng hash.

---

## Bài 6 — Stash cơ bản

**Đề**:
1. Sửa file đang dở. Cần switch nhánh khác để fix gấp.
2. Stash thay đổi.
3. Switch branch khác, fix.
4. Quay về, lấy stash ra tiếp tục.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo init > a && git add . && git commit -m "init"

# Đang code dở
echo "WIP" >> a

git switch -c hotfix
# → ERROR: file đang sửa chưa commit, blocks switch

git stash push -m "WIP feature X"
git status   # → clean

git switch -c hotfix
echo "fix" > fix && git add . && git commit -m "hotfix"
git switch -

git stash list                 # → stash@{0}: ... WIP feature X
git stash pop                  # áp lại + xóa stash

cat a   # → có "WIP"
```
</details>

**Giải thích**: Stash cất working tree + staging vào kho LIFO. `pop` = `apply` + `drop`. Nếu conflict khi pop → stash không tự drop, phải xóa thủ công.

Hữu ích: `git stash -u` để bao gồm cả file untracked (mặc định chỉ stash file tracked).

---

## Bài 7 — Multiple stashes

**Đề**:
1. Sửa file → stash với message "A".
2. Sửa khác → stash với message "B".
3. Sửa khác → stash với message "C".
4. List, xem nội dung từng stash, pop "B" (giữa).

<details>
<summary><b>Đáp án</b></summary>

```bash
echo 1 > a && git add . && git commit -m "init"

echo A >> a && git stash push -m "A"
echo B >> a && git stash push -m "B"
echo C >> a && git stash push -m "C"

git stash list
# stash@{0}: On main: C
# stash@{1}: On main: B
# stash@{2}: On main: A

git stash show -p stash@{1}    # xem diff của B

git stash pop stash@{1}
git stash list
# stash@{0}: On main: C
# stash@{1}: On main: A
# (B đã được pop và xóa)
```
</details>

**Giải thích**: Stash là stack — index 0 là mới nhất. Có thể pop/apply bất kỳ. `show -p` xem diff đầy đủ.

---

## Bài 8 — Tag annotated cho release

**Đề**:
1. Tạo tag `v1.0.0` (annotated) cho commit hiện tại với message "First stable release".
2. Tạo tag `v0.9.0` cho commit cũ hơn (HEAD~2).
3. Push tags lên remote (mô phỏng bằng bare repo).
4. List tag, xem chi tiết `v1.0.0`.

<details>
<summary><b>Đáp án</b></summary>

```bash
git init
for i in 1 2 3 4 5; do echo $i > $i.txt && git add . && git commit -m "$i"; done

git tag -a v1.0.0 -m "First stable release"
git tag -a v0.9.0 HEAD~2 -m "Beta release"

git tag                    # liệt kê
git show v1.0.0            # chi tiết
git tag --list 'v*'        # filter

# Push (cần remote)
git init --bare ../central.git
git remote add origin ../central.git
git push origin main
git push origin v1.0.0 v0.9.0       # push specific tags
# hoặc
git push --tags                       # push tất cả tags
```
</details>

**Giải thích**: Lightweight tag (`git tag v1.0.0`) chỉ là pointer; annotated tag (`-a`) là object riêng có author, date, message. Annotated dùng cho release; lightweight cho bookmark tạm. Tags không tự push — phải push tường minh.

---

## Bài 9 — Bisect tìm commit gây bug

**Đề**: Repo có 20 commit. Bug xuất hiện đâu đó trong khoảng đó. v1.0.0 (commit thứ 5) chắc chắn OK. HEAD bị bug. Tìm commit gây bug bằng bisect.

<details>
<summary><b>Đáp án</b></summary>

```bash
# Tạo setup giả
git init
for i in $(seq 1 20); do
  if [ $i -eq 13 ]; then
    echo "BUG INTRODUCED" > broken.txt
  fi
  echo $i > "$i.txt"
  git add . && git commit -m "commit $i"
  [ $i -eq 5 ] && git tag v1.0.0
done

git bisect start
git bisect bad                    # HEAD = bug
git bisect good v1.0.0            # v1.0.0 = OK

# Git checkout commit ở giữa (commit ~12)
# Bạn kiểm tra: có file broken.txt không?
ls broken.txt 2>/dev/null && git bisect bad || git bisect good

# Lặp lại cho đến khi:
# → "abcd1234 is the first bad commit"

git bisect reset    # về HEAD ban đầu
```

**Tự động hóa**:

```bash
git bisect start HEAD v1.0.0
git bisect run sh -c '! test -f broken.txt'
# Git chạy lệnh trên mỗi commit; exit 0 = good, khác 0 = bad
```
</details>

**Giải thích**: Bisect = binary search trên git log. Mỗi step cắt đôi không gian → tìm bug trong 20 commit chỉ ~5 step thay vì 20. Auto-mode (`bisect run`) cực mạnh khi có test script.

---

## Bài 10 — Worktree: 2 branch song song

**Đề**: Bạn đang code feature trong `feature/x` (trong thư mục `dev/`). Cần build/test branch `main` để so sánh, không muốn switch (sẽ mất state đang code). Tạo worktree thứ hai cho `main`.

<details>
<summary><b>Đáp án</b></summary>

```bash
cd dev
git switch feature/x

# Có code đang dở, không muốn switch
git worktree add ../dev-main main

ls ../dev-main      # → working tree của main
cd ../dev-main
# build / test tại đây, không ảnh hưởng dev/

cd ../dev
git worktree list
# /path/dev          abc123  [feature/x]
# /path/dev-main     def456  [main]

# Xóa khi xong
git worktree remove ../dev-main
```
</details>

**Giải thích**: Worktree = nhiều thư mục working tree, chia sẻ chung `.git/`. Mỗi worktree checkout 1 branch độc lập. Không cần clone lại repo lớn. Lưu ý: 2 worktree không thể cùng checkout 1 branch.

---

## Bài 11 — Pre-commit hook với Husky-style (manual)

**Đề**: Viết hook `pre-commit` chặn commit nếu file có chữ "TODO".

<details>
<summary><b>Đáp án</b></summary>

```bash
cat > .git/hooks/pre-commit <<'EOF'
#!/bin/sh
if git diff --cached | grep -q "TODO"; then
  echo "❌ Found TODO in staged changes. Resolve before committing."
  exit 1
fi
exit 0
EOF
chmod +x .git/hooks/pre-commit

# Test
echo "TODO: fix this" > x && git add x
git commit -m "test"
# → ❌ Found TODO ...   (commit bị chặn)

echo "done" > x && git add x
git commit -m "test"
# → OK
```
</details>

**Giải thích**: Hook là file thực thi trong `.git/hooks/`. `pre-commit` chạy trước khi tạo commit; exit 0 = OK, khác 0 = chặn. Vấn đề: `.git/` không được commit → hook không share giữa team. Husky/pre-commit framework giải quyết bằng cách lưu config trong repo + tự copy vào `.git/hooks/` lúc install.

---

## Bài 12 — git blame: ai sửa dòng này?

**Đề**: File `code.py` có dòng 42 lỗi. Tìm:
1. Ai sửa dòng đó lần cuối, commit nào.
2. Lịch sử của riêng dòng 40-50.

<details>
<summary><b>Đáp án</b></summary>

```bash
git blame code.py | sed -n '42p'
# abc12345 (Moet 2026-05-14 10:23:45 +0700  42) buggy_line

git blame -L 40,50 code.py
# → hiện 11 dòng cùng commit / author của mỗi dòng

# Xem chi tiết commit
git show abc12345
```
</details>

**Giải thích**: `git blame` cho biết commit cuối cùng chạm vào mỗi dòng. `-L start,end` giới hạn range. Lưu ý: blame chỉ hiện commit cuối; nếu cần truy ngược nhiều bước (vd "dòng này có bao giờ bị refactor không?"), dùng `git log -L 40,50:code.py`.
