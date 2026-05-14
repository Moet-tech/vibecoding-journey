# Day 1 — Tổng hợp Git: từ cơ bản đến nâng cao

Bộ tài liệu tự học Git, chia theo chủ đề. Mỗi chủ đề có phần lý thuyết riêng và một file bài tập tương ứng trong `exercises/` (có đáp án + giải thích).

## Lộ trình đề xuất

| # | Chủ đề | Lý thuyết | Bài tập |
|---|---|---|---|
| 1 | Cơ bản — config, init, add, commit, log, .gitignore | [01-basics.md](01-basics.md) | [exercises/01-basics-exercises.md](exercises/01-basics-exercises.md) |
| 2 | Branching — nhánh, merge, conflict | [02-branching.md](02-branching.md) | [exercises/02-branching-exercises.md](exercises/02-branching-exercises.md) |
| 3 | Remote — clone, fetch, pull, push, GitHub | [03-remote.md](03-remote.md) | [exercises/03-remote-exercises.md](exercises/03-remote-exercises.md) |
| 4 | Hoàn tác — restore, reset, revert, reflog | [04-undoing.md](04-undoing.md) | [exercises/04-undoing-exercises.md](exercises/04-undoing-exercises.md) |
| 5 | Nâng cao — rebase, cherry-pick, stash, tag, bisect | [05-advanced.md](05-advanced.md) | [exercises/05-advanced-exercises.md](exercises/05-advanced-exercises.md) |
| 6 | Workflows — GitHub Flow, Git Flow, Conventional Commits | [06-workflows.md](06-workflows.md) | — |

## Mô hình tinh thần về Git

Git có **4 vùng** cần phân biệt rạch ròi — hiểu chỗ này thì 80% lệnh Git sẽ tự sáng tỏ:

```
┌──────────────┐   git add    ┌──────────────┐   git commit   ┌──────────────┐   git push   ┌──────────────┐
│ Working tree │ ───────────► │ Staging area │ ─────────────► │ Local repo   │ ───────────► │ Remote repo  │
│ (file thật)  │              │  (index)     │                │  (.git/)     │              │  (GitHub…)   │
└──────────────┘              └──────────────┘                └──────────────┘              └──────────────┘
       ▲                              ▲                              │                              │
       │      git restore             │       git restore --staged   │      git fetch / pull        │
       └──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
```

- **Working tree**: file bạn đang sửa trên máy.
- **Staging area** (index): danh sách thay đổi sẽ vào commit kế tiếp.
- **Local repo**: lịch sử commit lưu trong `.git/` của bạn.
- **Remote repo**: lịch sử trên server (GitHub, GitLab…).

## Cách dùng tài liệu

1. Đọc file lý thuyết → gõ tay lại lệnh trong terminal (không copy-paste).
2. Làm bài tập cùng chủ đề mà **không nhìn đáp án** trước.
3. So sánh với đáp án, đọc phần **giải thích** để hiểu *tại sao*.
4. Nếu sai, tạo lại sandbox và làm lại từ đầu.

> Mọi bài tập đều chạy được trong sandbox cục bộ, không cần GitHub (trừ chủ đề 3).
