# Bài tập 02 — Branching và merge

> Sandbox:
> ```bash
> mkdir -p ~/git-sandbox/branching && cd ~/git-sandbox/branching
> ```

---

## Bài 1 — Tạo và chuyển nhánh

**Đề**:
1. Init repo `bai1/`, commit `README.md`.
2. Tạo nhánh `feature/login`, chuyển sang.
3. Commit thêm 1 file `login.js`.
4. Quay về `main`, kiểm tra: `login.js` có ở `main` không?
5. Xem cây nhánh.

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai1 && cd bai1
git init
echo "# Project" > README.md
git add . && git commit -m "init"

git switch -c feature/login
echo "console.log('login')" > login.js
git add . && git commit -m "feat: login"

git switch main
ls   # → chỉ thấy README.md, KHÔNG có login.js

git log --oneline --graph --all
```
</details>

**Giải thích**: File chỉ tồn tại ở branch mà commit của nó nằm. Khi switch về `main`, working tree được "reset" về snapshot của commit cuối cùng trên `main`. `--graph --all` cho thấy cây phân nhánh.

---

## Bài 2 — Fast-forward merge

**Đề**: Tiếp theo bài 1:
1. Đứng ở `main`, merge `feature/login`.
2. `login.js` có xuất hiện không?
3. Xem cây — có "merge commit" mới không? Vì sao?

<details>
<summary><b>Đáp án</b></summary>

```bash
git switch main
git merge feature/login

ls   # → có login.js
git log --oneline --graph --all
# → main và feature/login cùng trỏ vào commit "feat: login"
# → KHÔNG có merge commit
```
</details>

**Giải thích**: Vì `main` không có commit mới sau khi `feature/login` tách ra → Git chỉ cần "đẩy con trỏ main" về phía trước = **fast-forward**. Không tạo merge commit.

---

## Bài 3 — Three-way merge (cần merge commit)

**Đề**: Tạo repo mới `bai3/`:
1. Commit `README.md` trên `main`.
2. Tạo branch `feature/x`, commit thêm `x.js`.
3. Quay về `main`, commit thêm `y.js` (trực tiếp vào main).
4. Giờ merge `feature/x` vào `main`. Có gì khác bài 2?

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai3 && cd bai3
git init
echo "# Project" > README.md && git add . && git commit -m "init"

git switch -c feature/x
echo "X" > x.js && git add . && git commit -m "feat: x"

git switch main
echo "Y" > y.js && git add . && git commit -m "feat: y"

git merge feature/x
# → editor mở ra để viết merge commit message → save & exit
# (hoặc Git tự dùng default message)

git log --oneline --graph --all
# → có merge commit; cây hình chữ Y
```
</details>

**Giải thích**: Cả 2 nhánh đều có commit mới sau khi tách → Git **không thể fast-forward**. Phải tạo merge commit có 2 parent (main mới và feature/x). Đây là **three-way merge**.

---

## Bài 4 — Xử lý merge conflict

**Đề**: Trong `bai4/`:
1. Commit file `config.txt` chứa `port=3000`.
2. Tạo `feature/8080`, sửa thành `port=8080`, commit.
3. Quay về `main`, sửa thành `port=4000`, commit.
4. Merge `feature/8080` → CONFLICT.
5. Resolve: chọn `port=8080`. Hoàn tất merge.

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai4 && cd bai4
git init
echo "port=3000" > config.txt && git add . && git commit -m "init"

git switch -c feature/8080
echo "port=8080" > config.txt && git add . && git commit -m "use 8080"

git switch main
echo "port=4000" > config.txt && git add . && git commit -m "use 4000"

git merge feature/8080
# CONFLICT
# Mở config.txt — thấy:
#   <<<<<<< HEAD
#   port=4000
#   =======
#   port=8080
#   >>>>>>> feature/8080

# Cách 1: sửa tay → giữ "port=8080", xóa marker
echo "port=8080" > config.txt

# Cách 2: dùng lệnh
# git checkout --theirs config.txt    # giữ phiên bản feature/8080
# git checkout --ours config.txt      # giữ phiên bản main

git add config.txt
git commit                            # editor mở message merge → save

git log --oneline --graph --all
```
</details>

**Giải thích**: Conflict marker `<<< HEAD` = phiên bản nhánh **bạn đang đứng** (main). `>>> feature/8080` = phiên bản nhánh **đang merge vào**. `--ours` / `--theirs` là shortcut nhưng dễ nhầm: khi đang `merge`, "ours" = HEAD; khi đang `rebase`, "ours" và "theirs" **đảo ngược lại** (vì rebase đứng ở góc nhìn ngược).

---

## Bài 5 — Bỏ giữa chừng khi merge conflict

**Đề**: Lặp lại bài 4 đến bước CONFLICT. Sau đó **hủy merge** thay vì resolve. Kiểm tra `config.txt` quay lại trạng thái trước merge.

<details>
<summary><b>Đáp án</b></summary>

```bash
# (đang trong trạng thái conflict)
git merge --abort

cat config.txt    # → "port=4000" (trạng thái trên main, trước merge)
git status        # → clean
```
</details>

**Giải thích**: `--abort` đưa repo về trạng thái trước khi gõ `git merge`. Hữu ích khi bạn vừa merge xong nhận ra nhánh đang sai → bỏ luôn, suy nghĩ lại.

---

## Bài 6 — `--no-ff` (luôn tạo merge commit)

**Đề**: Lặp lại bài 2 nhưng dùng `--no-ff`. So sánh cây.

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai6 && cd bai6
git init
echo "1" > a.txt && git add . && git commit -m "init"

git switch -c feature/x
echo "2" > b.txt && git add . && git commit -m "feat: x"

git switch main
git merge --no-ff feature/x -m "Merge feature/x"

git log --oneline --graph --all
# → CÓ merge commit dù có thể fast-forward
```
</details>

**Giải thích**: `--no-ff` ép tạo merge commit → giữ "dấu vết" của nhánh, dù FF được. Nhiều team thích cái này vì dễ thấy "feature này được merge khi nào" trong git log.

---

## Bài 7 — Squash merge

**Đề**: Trong repo có nhánh `feature/messy` với 5 commit lặt vặt ("wip", "fix typo"…):
1. Squash merge vào `main` thành **1 commit** với message `"feat: complete X"`.
2. Xem log — feature có hiện 5 commit không?

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai7 && cd bai7
git init
echo "0" > a && git add . && git commit -m "init"

git switch -c feature/messy
for i in 1 2 3 4 5; do
  echo "$i" >> a
  git commit -am "wip $i"
done

git switch main
git merge --squash feature/messy
git status   # → mọi thay đổi của feature trong staging
git commit -m "feat: complete X"

git log --oneline
# → chỉ thấy "feat: complete X" + "init"
# → KHÔNG còn 5 commit wip
```
</details>

**Giải thích**: `--squash` áp dụng diff cuối cùng của feature vào staging, KHÔNG tạo merge commit, KHÔNG dời con trỏ nhánh. Bạn tự commit ra 1 snapshot mới. Hữu ích để giữ `main` sạch.

---

## Bài 8 — Xóa branch sau khi merge

**Đề**: Sau khi merge `feature/login` vào `main`:
1. Liệt kê branch đã merged.
2. Xóa `feature/login` (an toàn).
3. Tạo `feature/draft` rồi xóa **mặc dù chưa merge** (cưỡng bức).

<details>
<summary><b>Đáp án</b></summary>

```bash
git branch --merged       # → liệt kê branch đã merge (an toàn xóa)

git branch -d feature/login

git switch -c feature/draft
echo "wip" > draft.txt && git add . && git commit -m "wip"
git switch main

git branch -d feature/draft
# → ERROR: not fully merged. Use -D to force.

git branch -D feature/draft   # cưỡng bức
```
</details>

**Giải thích**: `-d` chỉ xóa branch đã merge — bảo vệ bạn khỏi mất commit nhầm. `-D` (chữ hoa) cưỡng bức. Nếu lỡ tay `-D` mất commit chưa merge → cứu bằng `git reflog` (xem [04-undoing.md](../04-undoing.md)).

---

## Bài 9 — Detached HEAD

**Đề**: Trong repo có ít nhất 3 commit:
1. Checkout vào commit thứ 2 (không qua branch) — bạn đang ở detached HEAD.
2. Tạo file mới, commit. Commit này có thuộc branch nào không?
3. Switch về main. Commit kia có còn dễ thấy không?
4. (Bonus) Cứu commit kia bằng cách tạo branch từ nó.

<details>
<summary><b>Đáp án</b></summary>

```bash
git log --oneline   # giả sử có: aaa, bbb, ccc (HEAD)

git checkout bbb
# → "You are in 'detached HEAD' state..."

echo "orphan" > orphan.txt
git add . && git commit -m "orphan commit"
# Commit này KHÔNG thuộc branch nào — chỉ "treo" sau bbb.

git log --oneline    # thấy commit orphan

git switch main
git log --oneline    # KHÔNG thấy commit orphan nữa

# Cứu — tìm hash bằng reflog
git reflog
# 1234abc HEAD@{1}: commit: orphan commit

git switch -c rescue 1234abc
# Hoặc trước khi switch: git switch -c rescue (lúc đang detached HEAD)
```
</details>

**Giải thích**: Detached HEAD = HEAD trỏ thẳng vào 1 commit, không qua branch. Commit ở đây "mồ côi" — sau khi switch đi, không có pointer nào trỏ vào → có thể bị garbage collect sau ~30 ngày. Dùng `git reflog` cứu được trong khoảng thời gian đó.

---

## Bài 10 — Workflow feature branch hoàn chỉnh

**Đề**: Mô phỏng quy trình thực tế:
1. Repo có `main`.
2. Tạo `feature/cart`, commit 3 thay đổi.
3. Trong khi đó, có người khác commit vào `main` (mô phỏng bằng cách checkout `main` commit tay).
4. Quay về `feature/cart`, merge `main` vào (cập nhật).
5. Quay về `main`, merge `feature/cart` (squash).
6. Xóa `feature/cart`.

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai10 && cd bai10 && git init
echo "init" > a.txt && git add . && git commit -m "init"

# Tạo feature
git switch -c feature/cart
echo "cart 1" > cart.js && git add . && git commit -m "feat(cart): add item"
echo "cart 2" >> cart.js && git commit -am "feat(cart): remove item"
echo "cart 3" >> cart.js && git commit -am "feat(cart): clear"

# Trong lúc đó, main có commit mới
git switch main
echo "other change" > b.txt && git add . && git commit -m "chore: add b"

# Cập nhật feature
git switch feature/cart
git merge main   # hoặc rebase (xem 05-advanced)

# Merge vào main
git switch main
git merge --squash feature/cart
git commit -m "feat(cart): complete cart feature"

# Dọn
git branch -D feature/cart   # phải -D vì squash → branch coi như "chưa merge"

git log --oneline --graph --all
```
</details>

**Giải thích**: Đây là vòng đời feature branch điển hình. Lưu ý: sau **squash merge**, Git coi `feature/cart` là "chưa merged" về mặt lý thuyết (vì hash khác) → phải `-D`. Đó là lý do nhiều team thích PR merge tự động xóa branch trên GitHub.
