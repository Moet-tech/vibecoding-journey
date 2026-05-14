# Bài tập 01 — Git cơ bản

> Chuẩn bị sandbox cho mọi bài:
> ```bash
> mkdir -p ~/git-sandbox/basics && cd ~/git-sandbox/basics
> ```
> Mỗi bài có thể tạo subfolder riêng để khỏi ảnh hưởng nhau.

---

## Bài 1 — Cấu hình Git và tạo repo đầu tiên

**Đề**:
1. Cấu hình `user.name = "Moet"` và `user.email = "zfighting660@gmail.com"` ở phạm vi global.
2. Tạo thư mục `bai1/`, khởi tạo Git repo với nhánh mặc định là `main`.
3. Kiểm tra cấu hình hiện hành.

<details>
<summary><b>Đáp án</b></summary>

```bash
git config --global user.name "Moet"
git config --global user.email "zfighting660@gmail.com"
git config --global init.defaultBranch main

mkdir bai1 && cd bai1
git init

git config user.name
git config user.email
git branch --show-current   # → main
```
</details>

**Giải thích**: `--global` lưu vào `~/.gitconfig`, áp dụng cho mọi repo của user. Nếu không set `init.defaultBranch`, Git cũ vẫn dùng `master`. `git branch --show-current` chỉ hiện nhánh hiện tại — đỡ đọc output dài của `git branch`.

---

## Bài 2 — Workflow add / commit lần đầu

**Đề**: Trong `bai1/`:
1. Tạo file `hello.txt` chứa dòng `Hello Git`.
2. Xem trạng thái — file đang ở vùng nào?
3. Stage file đó, kiểm tra trạng thái.
4. Commit với message `"feat: add hello.txt"`.
5. Xem log.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "Hello Git" > hello.txt

git status
# → Untracked files: hello.txt

git add hello.txt
git status
# → Changes to be committed: new file: hello.txt

git commit -m "feat: add hello.txt"

git log --oneline
# → abc1234 (HEAD -> main) feat: add hello.txt
```
</details>

**Giải thích**: File mới luôn ở trạng thái **Untracked** — Git chưa biết. `git add` đưa vào **Staging**. `git commit` snapshot toàn bộ staging vào local repo. `git log --oneline` xem gọn.

---

## Bài 3 — Sửa file, xem diff trước khi commit

**Đề**: Trong `bai1/`:
1. Sửa `hello.txt` thành `Hello Git, second line`.
2. Xem **diff giữa working tree và last commit**.
3. Stage file, xem **diff giữa staging và last commit**.
4. Xem **diff giữa working tree và staging** (kết quả?).
5. Commit `"docs: expand hello message"`.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "Hello Git, second line" > hello.txt

git diff                    # working tree vs staging (staging chưa có gì → cũng = vs HEAD)
git diff HEAD               # working tree vs last commit

git add hello.txt
git diff --staged           # staging vs last commit
git diff                    # working tree vs staging → KHÔNG có gì (đã add hết)

git commit -m "docs: expand hello message"
```
</details>

**Giải thích**:
- `git diff` (không tham số) = working tree vs staging.
- `git diff --staged` = staging vs last commit.
- `git diff HEAD` = working tree vs last commit (gộp cả 2 trên).

Sau `git add`, file ở staging đã giống working tree → `git diff` không hiện gì.

---

## Bài 4 — git add với nhiều file (interactive)

**Đề**: Trong `bai1/`:
1. Tạo 3 file: `a.txt`, `b.txt`, `c.txt`, mỗi file 1 dòng nội dung khác nhau.
2. Stage chỉ `a.txt` và `c.txt`, KHÔNG stage `b.txt`.
3. Commit `"feat: add a and c"`.
4. Sau đó stage và commit `b.txt` riêng: `"feat: add b"`.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "A" > a.txt
echo "B" > b.txt
echo "C" > c.txt

git add a.txt c.txt
git status            # → a.txt, c.txt staged; b.txt untracked

git commit -m "feat: add a and c"

git add b.txt
git commit -m "feat: add b"

git log --oneline
```
</details>

**Giải thích**: `git add` cho phép stage **chính xác cái mình muốn** — đây là nền tảng để có commit "atomic" (1 thay đổi logic / 1 commit). Tránh `git add -A` mọi lúc.

---

## Bài 5 — git add -p (interactive theo hunk)

**Đề**: Tạo file `numbers.txt` với 5 dòng (1, 2, 3, 4, 5). Commit. Sau đó sửa thành:
```
1
two
3
four
5
```
1. Dùng `git add -p` để **chỉ stage thay đổi dòng `2 → two`**, KHÔNG stage dòng 4.
2. Commit `"feat: spell out two"`.
3. Stage và commit nốt thay đổi còn lại.

<details>
<summary><b>Đáp án</b></summary>

```bash
printf "1\n2\n3\n4\n5\n" > numbers.txt
git add numbers.txt && git commit -m "feat: add numbers"

printf "1\ntwo\n3\nfour\n5\n" > numbers.txt

git add -p numbers.txt
# Git hiện hunk đầu tiên (có thể gộp cả 2 thay đổi nếu gần nhau).
# Nếu hunk gộp: gõ "s" (split) để tách. Sau đó:
#   - hunk dòng 2: gõ "y" (yes, stage)
#   - hunk dòng 4: gõ "n" (no)

git diff --staged   # chỉ thấy thay đổi 2→two
git diff            # còn lại 4→four
git commit -m "feat: spell out two"

git add numbers.txt
git commit -m "feat: spell out four"
```
</details>

**Giải thích**: `git add -p` mở interactive — cho phép stage từng "hunk" (đoạn thay đổi). `s` để split hunk thành nhiều phần nhỏ hơn. Cực hữu ích khi 1 lần sửa nhiều thay đổi không liên quan và muốn tách thành nhiều commit.

---

## Bài 6 — .gitignore

**Đề**: Tạo thư mục `bai6/`:
1. Init repo.
2. Tạo `app.js`, `secrets.env`, thư mục `node_modules/lib/index.js`, `debug.log`.
3. Tạo `.gitignore` để bỏ qua `*.env`, `*.log`, `node_modules/`.
4. `git status` chỉ nên hiện `.gitignore` và `app.js`.

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai6 && cd bai6
git init

echo "console.log('hi');" > app.js
echo "API_KEY=secret" > secrets.env
mkdir -p node_modules/lib
echo "export default 1" > node_modules/lib/index.js
echo "[info] started" > debug.log

cat > .gitignore <<'EOF'
*.env
*.log
node_modules/
EOF

git status
# → .gitignore, app.js (untracked)
# → KHÔNG hiện secrets.env, debug.log, node_modules/
```
</details>

**Giải thích**: `.gitignore` chặn file untracked. Pattern `*.env` match mọi file đuôi `.env`. `node_modules/` (có `/`) chỉ match folder. Tự `.gitignore` cũng được commit để cả team dùng chung.

---

## Bài 7 — File đã commit rồi mới ignore

**Đề**: Trong `bai6/`:
1. Tạo `config.json` chứa `{"debug": true}`, commit.
2. Bây giờ muốn ignore file này → thêm `config.json` vào `.gitignore`.
3. Sửa nội dung file. `git status` vẫn báo file modified — vì sao?
4. Sửa để Git ngừng tracking file này (nhưng giữ file trên đĩa).

<details>
<summary><b>Đáp án</b></summary>

```bash
echo '{"debug": true}' > config.json
git add config.json && git commit -m "feat: add config"

echo "config.json" >> .gitignore
git add .gitignore && git commit -m "chore: ignore config"

echo '{"debug": false}' > config.json
git status
# → modified: config.json   ← VẪN HIỆN vì file đã được tracked

git rm --cached config.json     # gỡ khỏi tracking, GIỮ file
git status
# → deleted: config.json (staged)
# → config.json đã trong .gitignore nên KHÔNG còn untracked

git commit -m "chore: untrack config.json"

ls config.json    # → vẫn còn trên đĩa
```
</details>

**Giải thích**: `.gitignore` chỉ chặn file **chưa được Git biết tới**. File đã tracked không bị ảnh hưởng — phải dùng `git rm --cached` để gỡ tracking. Đây là sai lầm phổ biến với `.env`, `node_modules/` lỡ commit từ đầu.

---

## Bài 8 — Sửa commit cuối (amend)

**Đề**:
1. Trong repo bất kỳ, commit 1 file nhưng lỡ đánh sai message.
2. Sửa message bằng amend.
3. Sau đó nhớ ra quên thêm 1 file → thêm file vào **chính commit đó** (không tạo commit mới), giữ nguyên message.

<details>
<summary><b>Đáp án</b></summary>

```bash
echo "foo" > foo.txt
git add foo.txt
git commit -m "fxi: typo here"          # ← sai chính tả

git commit --amend -m "fix: typo here"  # sửa message

echo "bar" > bar.txt
git add bar.txt
git commit --amend --no-edit            # thêm file, GIỮ message
```
</details>

**Giải thích**: `--amend` tạo commit mới thay thế commit cuối, đổi hash. Vì đổi hash → **không amend commit đã push** trừ khi force-with-lease.

---

## Bài 9 — git log nâng cao

**Đề**: Trong repo bất kỳ có ít nhất 5 commit:
1. Xem 3 commit gần nhất, 1 dòng / 1 commit, có cây nhánh.
2. Tìm tất cả commit có chữ "fix" trong message.
3. Xem commit nào đụng vào file `app.js`.
4. Xem chi tiết (diff) của commit gần nhất.

<details>
<summary><b>Đáp án</b></summary>

```bash
git log -3 --oneline --graph --all

git log --grep="fix"
git log --oneline --grep="fix"

git log --oneline -- app.js
git log -p -- app.js                    # kèm diff

git show                                # = git show HEAD
git log -1 -p                           # cách khác
```
</details>

**Giải thích**: `git log` cực kỳ linh hoạt — flags ghép được. `--grep` lọc message; `-- <path>` lọc file (dấu `--` để Git biết đây là path, không phải branch).

---

## Bài 10 — Xem & khôi phục file ở commit cũ

**Đề**: Trong repo có file `report.md` đã commit nhiều version:
1. Xem nội dung `report.md` ở commit cách HEAD 2 bước **mà không thay đổi file hiện tại**.
2. Xuất nội dung đó ra file `report-old.md` để so sánh.
3. (Bonus) Khôi phục `report.md` về version đó.

<details>
<summary><b>Đáp án</b></summary>

```bash
git show HEAD~2:report.md                   # in ra terminal

git show HEAD~2:report.md > report-old.md   # xuất ra file mới

# Bonus — ghi đè file hiện tại bằng version cũ
git restore --source=HEAD~2 report.md
git status                                   # → modified
# nếu thực sự muốn → git add + commit
```
</details>

**Giải thích**: Cú pháp `<ref>:<path>` truy cập file ở commit cụ thể mà không cần checkout. `git restore --source=<ref>` là cách hiện đại để "lấy file cũ về".
