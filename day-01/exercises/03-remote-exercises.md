# Bài tập 03 — Remote và GitHub

> Một số bài cần tài khoản GitHub. Có thể dùng repo test riêng (vd: `git-sandbox`).
> Sandbox cục bộ:
> ```bash
> mkdir -p ~/git-sandbox/remote && cd ~/git-sandbox/remote
> ```

---

## Bài 1 — Mô phỏng remote bằng repo local (bare)

**Đề**: Không cần GitHub. Mô phỏng remote bằng repo bare cục bộ:
1. Tạo "remote" bare repo: `central.git`.
2. Tạo repo `dev1/` thường, thêm `central.git` làm remote `origin`.
3. Commit 1 file, push lên `origin`.

<details>
<summary><b>Đáp án</b></summary>

```bash
mkdir bai1 && cd bai1

# Tạo "remote"
git init --bare central.git

# Tạo dev1
mkdir dev1 && cd dev1
git init
git remote add origin ../central.git
echo "hello" > a.txt
git add . && git commit -m "init"
git push -u origin main

git remote -v
# origin   ../central.git (fetch)
# origin   ../central.git (push)
```
</details>

**Giải thích**: Repo bare = repo không có working tree, chỉ có `.git/` — thường dùng làm "server". URL của remote có thể là đường dẫn cục bộ (như đây), SSH, hoặc HTTPS. Mô hình giống hệt GitHub nhưng không cần mạng.

---

## Bài 2 — Mô phỏng 2 dev chia sẻ qua remote

**Đề**: Tiếp theo bài 1:
1. Clone từ `central.git` thành `dev2/`.
2. Trong `dev2/`, sửa `a.txt`, push.
3. Trong `dev1/`, pull về — thấy thay đổi của dev2.

<details>
<summary><b>Đáp án</b></summary>

```bash
# Từ thư mục bai1/
git clone central.git dev2

cd dev2
echo "hello from dev2" > a.txt
git commit -am "edit by dev2"
git push

cd ../dev1
git pull
cat a.txt   # → "hello from dev2"
```
</details>

**Giải thích**: Workflow chuẩn của Git — central repo là điểm đồng bộ. `pull` = `fetch` + `merge` từ tracking branch (`origin/main`).

---

## Bài 3 — fetch vs pull

**Đề**: Tiếp theo:
1. Trong `dev2/`, commit + push thêm thay đổi.
2. Trong `dev1/`, dùng **`fetch` thay vì pull**. Kiểm tra: working tree có đổi không? `origin/main` có đổi không?
3. So sánh `main` vs `origin/main`.
4. Merge thủ công.

<details>
<summary><b>Đáp án</b></summary>

```bash
cd dev2
echo "another change" >> a.txt
git commit -am "more edits"
git push

cd ../dev1
git fetch
cat a.txt    # → CHƯA đổi
git log --oneline main..origin/main   # → thấy commit của dev2 ở origin/main

git merge origin/main
cat a.txt    # → đã đổi
```
</details>

**Giải thích**: `fetch` chỉ cập nhật tracking branch (`origin/main`) — local branch của bạn KHÔNG đổi. Đây là cách an toàn để "xem có gì mới trên remote" mà không bị merge tự động. `pull` = `fetch` + `merge` 1 phát.

`main..origin/main` đọc là "commit có ở origin/main mà chưa có ở main".

---

## Bài 4 — Push branch mới lần đầu (-u)

**Đề**: Trong `dev1/`:
1. Tạo branch `feature/x`, commit.
2. `git push` → thấy lỗi gì? Vì sao?
3. Push đúng cách, kèm thiết lập tracking.

<details>
<summary><b>Đáp án</b></summary>

```bash
git switch -c feature/x
echo "x" > x.txt && git add . && git commit -m "feat: x"

git push
# → fatal: The current branch feature/x has no upstream branch.
# Hoặc gợi ý dùng --set-upstream

git push -u origin feature/x

git branch -vv   # thấy feature/x track origin/feature/x
```
</details>

**Giải thích**: Branch mới chưa có "upstream" — Git không biết push vào đâu trên remote. `-u` (= `--set-upstream`) làm 2 việc: push lần này, và lưu config để các lần sau chỉ cần `git push`.

Có thể cấu hình tự động: `git config --global push.autoSetupRemote true` → từ đó `git push` cũng tạo nhánh remote tự động.

---

## Bài 5 — Xóa branch trên remote

**Đề**: Tiếp theo:
1. Trong `dev1/`, xóa `feature/x` cả local và remote.
2. Trong `dev2/`, `fetch` về — thấy branch `origin/feature/x` còn không? Cách dọn?

<details>
<summary><b>Đáp án</b></summary>

```bash
cd dev1
git switch main
git branch -d feature/x                       # xóa local
git push origin --delete feature/x            # xóa trên remote

cd ../dev2
git branch -r            # → vẫn thấy origin/feature/x (cache cũ)
git fetch --prune        # dọn nhánh remote đã bị xóa
git branch -r            # → không còn
```
</details>

**Giải thích**: Local cache nhánh remote không tự dọn — phải `fetch --prune`. Tự động hóa: `git config --global fetch.prune true`.

---

## Bài 6 — Đối phó với push bị từ chối (non-fast-forward)

**Đề**:
1. Dev2 commit `c1` lên `origin/main`.
2. Dev1 (chưa pull) cũng commit `c2` lên `main` local, rồi push → từ chối. Vì sao?
3. Xử lý đúng cách (không force).

<details>
<summary><b>Đáp án</b></summary>

```bash
cd dev2
echo "from dev2" > new.txt && git add . && git commit -m "from dev2"
git push

cd ../dev1
echo "from dev1" > another.txt && git add . && git commit -m "from dev1"
git push
# → ! [rejected] main -> main (fetch first)
# Lý do: origin/main đã có commit của dev2 mà dev1 chưa có → push sẽ "ghi đè" commit kia.

git pull              # fetch + merge → tạo merge commit hoặc fast-forward
# Hoặc:
git pull --rebase     # fetch + rebase → lịch sử thẳng

git push              # giờ OK
```
</details>

**Giải thích**: Git từ chối push nếu nó không phải fast-forward — đây là cơ chế bảo vệ không cho ghi đè lịch sử. Cách đúng: pull về trước (merge hoặc rebase), rồi push.

Nếu force push lúc này → commit của dev2 trên remote sẽ **biến mất**. Đừng làm.

---

## Bài 7 — Force-with-lease

**Đề**: Trên branch cá nhân (không phải main):
1. Commit, push lên origin.
2. Amend commit (đổi message).
3. Push thường → từ chối.
4. Push với `--force-with-lease`.
5. Giải thích vì sao `--force-with-lease` an toàn hơn `--force`.

<details>
<summary><b>Đáp án</b></summary>

```bash
git switch -c experimental
echo "v1" > exp.txt && git add . && git commit -m "exp v1"
git push -u origin experimental

git commit --amend -m "exp v1 (better message)"
git push
# → rejected (non-fast-forward)

git push --force-with-lease
# OK — vì remote vẫn ở state mình nghĩ
```

**Vì sao `--force-with-lease` an toàn hơn**:

`--force` ghi đè vô điều kiện. Nếu giữa lúc bạn amend và push, có người khác push thêm vào `experimental` → bạn không biết → force của bạn xóa luôn việc của họ.

`--force-with-lease` ghi nhớ "remote SHA tôi thấy lần fetch trước". Khi push, nó so SHA hiện tại trên remote. Nếu khớp → push (cũ và đè OK). Nếu khác (ai đó vừa push) → fail.

```bash
# Mô phỏng: nếu remote thay đổi trước khi mình push-with-lease
# → push sẽ fail thay vì ghi đè
```
</details>

---

## Bài 8 — Nhiều remote (fork workflow)

**Đề**: Mô phỏng workflow fork:
1. `central.git` = repo gốc (upstream).
2. Clone thành `myfork.git` (mô phỏng fork).
3. Trong `dev1/`, set `origin` = `myfork.git`, `upstream` = `central.git`.
4. Đồng bộ `main` của fork với upstream khi upstream có commit mới.

<details>
<summary><b>Đáp án</b></summary>

```bash
# Setup
mkdir bai8 && cd bai8
git init --bare central.git
git clone --bare central.git myfork.git   # fork = bare clone

git clone myfork.git dev1
cd dev1
echo "x" > x && git add . && git commit -m "init" && git push

# Thêm remote upstream
git remote add upstream ../central.git
git remote -v
# origin     ../myfork.git
# upstream   ../central.git

# Giả sử upstream có commit mới (mô phỏng bằng clone trực tiếp central và push)
cd ..
git clone central.git mainline
cd mainline
echo "upstream change" > upstream.txt && git add . && git commit -m "upstream"
git push

# Đồng bộ fork
cd ../dev1
git fetch upstream
git switch main
git merge upstream/main          # đưa upstream vào local main
git push origin main             # đẩy lên fork
```
</details>

**Giải thích**: Convention: `origin` = fork của bạn, `upstream` = repo gốc. Khi muốn cập nhật fork: fetch từ upstream, merge vào local, push lên origin. Trên GitHub có nút "Sync fork" tự làm.

---

## Bài 9 — Tracking branch tùy chỉnh

**Đề**:
1. Tạo branch local `dev` không có upstream.
2. Set upstream là `origin/develop` (giả sử có).
3. Kiểm tra tracking.

<details>
<summary><b>Đáp án</b></summary>

```bash
# Giả sử origin có branch develop
git fetch
git switch -c dev origin/develop
# → tự tạo tracking. Hoặc:

git switch -c dev
git branch -u origin/develop dev

git branch -vv
# dev   abc1234 [origin/develop] commit msg
```
</details>

**Giải thích**: Tên branch local không bắt buộc trùng branch remote — `-u` (`--set-upstream-to`) cho phép map bất kỳ. Hữu ích khi convention nội bộ khác convention remote.

---

## Bài 10 — Xử lý "have divergent branches" (Git 2.27+)

**Đề**: Khi cả local và remote có commit không liên quan:
1. `git pull` → Git hỏi: merge, rebase, hay FF-only?
2. Set config để Git không hỏi nữa, dùng `merge` mặc định.
3. (Bonus) Đổi sang `ff-only` để Git từ chối khi cần merge → buộc bạn quyết định thủ công.

<details>
<summary><b>Đáp án</b></summary>

```bash
git config --global pull.rebase false     # luôn merge (mặc định cũ, an toàn)
# hoặc
git config --global pull.rebase true      # luôn rebase
# hoặc
git config --global pull.ff only          # chỉ FF, fail nếu cần merge — KHUYẾN NGHỊ

# Sau khi set:
git pull                                  # không hỏi nữa
```

**Test với `ff only`**:

```bash
# Tạo divergent state, pull → sẽ fail:
# → fatal: Not possible to fast-forward, aborting.
# Bạn phải tự quyết: git merge / git rebase
```
</details>

**Giải thích**:
- `pull.rebase false` (mặc định): merge → an toàn, không mất commit.
- `pull.rebase true`: rebase → lịch sử thẳng, nhưng có thể conflict bất ngờ.
- `pull.ff only`: chỉ FF, dừng lại nếu phức tạp → cho bạn kiểm soát.

Khuyên dùng `ff only`: Git sẽ không bao giờ "tự làm gì đó" lén lút.
