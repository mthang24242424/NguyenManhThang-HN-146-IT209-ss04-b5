# Báo cáo Bài tập 5: Khôi phục trạng thái và Đảo ngược commit (Reset vs Revert)

## 1. Mục tiêu

- Ứng dụng thành thạo hai cơ chế `reset` và `revert` để sửa chữa sai sót trong lịch sử dự án.
- Phân biệt sự khác nhau giữa khôi phục lịch sử cục bộ (`reset`) và đảo ngược lịch sử an toàn khi làm việc nhóm (`revert`).

---

## 2. Bối cảnh

Học viên tạo ra một commit chứa mã nguồn bị lỗi (`buggy_feature.py`) ở local. Cần thực hành hai kỹ thuật khác nhau:

- **Trường hợp 1 (Sửa đổi cục bộ):** Dùng `git reset --mixed` để lùi HEAD lại 1 commit, giữ lại code lỗi ở Working Directory để viết lại.
- **Trường hợp 2 (Sửa đổi công cộng):** Dùng `git revert` để tạo một commit mới đảo ngược thay đổi — an toàn khi lịch sử đã được đẩy lên remote.

**Ràng buộc:**
- TC1: Không làm mất mã nguồn (không dùng `reset --hard`).
- TC2: Bắt buộc tạo commit mới ghi lại lịch sử đảo ngược.

---

## 3. Các bước thực hiện

### Bước 1: Tạo commit lỗi để thực hành

Tạo file `buggy_feature.py` chứa mã nguồn có lỗi bảo mật và logic:

```python
# Tính năng mới - phiên bản lỗi
def calculate_discount(price, percent):
    # BUG: thiếu kiểm tra percent > 100
    discount = price * percent / 100
    return price - discount

def get_user_data(user_id):
    # BUG: SQL Injection - không dùng parameterized query
    query = "SELECT * FROM users WHERE id = " + user_id
    return query
```

```bash
git add buggy_feature.py
git commit -m "feat: add discount and user data feature"
```

```text
[main 1ed5be2] feat: add discount and user data feature
 1 file changed, 10 insertions(+)
 create mode 100644 buggy_feature.py
```

Kiểm tra lịch sử trước khi thực hành:

```bash
git log --oneline -5
```

```text
1ed5be2 (HEAD -> main) feat: add discount and user data feature
cf16996 bai4: add gitignore management and amend report
57d7a70 chore: add .gitignore and remove credentials.txt from tracking
bbde8e1 (origin/main) bai1
8ef4cc0 btvn
```

---

### Trường hợp 1: `git reset --mixed` — Khôi phục lịch sử cục bộ

#### Bước 2: Thực hiện reset lùi HEAD 1 commit

```bash
git reset --mixed HEAD~1
```

```text
Unstaged changes after reset:
M       Bai1.md
```

#### Bước 3: Kiểm tra kết quả

```bash
git log --oneline -5
```

```text
cf16996 (HEAD -> main) bai4: add gitignore management and amend report
57d7a70 chore: add .gitignore and remove credentials.txt from tracking
bbde8e1 (origin/main) bai1
8ef4cc0 btvn
c5208a0 file baitap
```

```bash
git status
```

```text
On branch main
Your branch is ahead of 'origin/main' by 2 commits.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   Bai1.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        buggy_feature.py
```

**Kết quả đạt yêu cầu:**
- HEAD đã lùi về `cf16996` — commit lỗi `1ed5be2` biến mất khỏi lịch sử.
- `buggy_feature.py` vẫn còn nguyên trên đĩa cứng ở dạng **Untracked** trong Working Directory.
- Code không bị mất — học viên có thể sửa lỗi rồi `git add` và commit lại.

#### Bảng so sánh 3 chế độ của `git reset`

| Tùy chọn | HEAD | Staging Area (Index) | Working Directory | Dùng khi |
|---|---|---|---|---|
| `--soft` | Lùi lại | **Giữ nguyên** (staged) | Giữ nguyên | Muốn gộp lại nhiều commit |
| `--mixed` *(mặc định)* | Lùi lại | **Xóa** (unstaged) | Giữ nguyên | Muốn viết lại code rồi commit lại |
| `--hard` | Lùi lại | Xóa | **Xóa luôn** | Muốn hủy hoàn toàn, không giữ gì |

> ⚠️ `--hard` xóa cả file vật lý — **không dùng** khi còn cần giữ lại code.

---

### Trường hợp 2: `git revert` — Đảo ngược an toàn cho remote

#### Bước 4: Tái tạo commit lỗi (mô phỏng commit đã push lên remote)

```bash
git add buggy_feature.py
git commit -m "feat: add discount and user data feature"
```

```text
[main c2d87e7] feat: add discount and user data feature
 1 file changed, 10 insertions(+)
 create mode 100644 buggy_feature.py
```

```bash
git log --oneline -5
```

```text
c2d87e7 (HEAD -> main) feat: add discount and user data feature
cf16996 bai4: add gitignore management and amend report
57d7a70 chore: add .gitignore and remove credentials.txt from tracking
bbde8e1 (origin/main) bai1
8ef4cc0 btvn
```

#### Bước 5: Thực hiện revert

```bash
git revert HEAD --no-edit
```

```text
[main 7bd1b82] Revert "feat: add discount and user data feature"
 Date: Mon Oct 5 01:53:19 2026 +0700
 1 file changed, 10 deletions(-)
 delete mode 100644 buggy_feature.py
```

> Tùy chọn `--no-edit` bỏ qua bước mở trình soạn thảo để sửa thông điệp, dùng thông điệp mặc định `Revert "..."`.

#### Bước 6: Kiểm tra kết quả

```bash
git log --oneline -5
```

```text
7bd1b82 (HEAD -> main) Revert "feat: add discount and user data feature"
c2d87e7 feat: add discount and user data feature
cf16996 bai4: add gitignore management and amend report
57d7a70 chore: add .gitignore and remove credentials.txt from tracking
bbde8e1 (origin/main) bai1
```

```bash
git status
```

```text
On branch main
Your branch is ahead of 'origin/main' by 4 commits.

Changes not staged for commit:
        modified:   Bai1.md

no changes added to commit (use "git add" and/or "git commit -a")
```

**Kết quả đạt yêu cầu:**
- Commit `7bd1b82` có tiêu đề bắt đầu bằng `Revert "..."` xuất hiện trên đầu nhánh.
- Commit lỗi `c2d87e7` **vẫn còn trong lịch sử** — lịch sử không bị viết lại, an toàn để `git push`.
- `buggy_feature.py` đã bị xóa khỏi Working Directory bởi commit revert.

---

## 4. Lệnh kiểm tra và kết quả mong đợi

```bash
git log --oneline -n 5
```

```text
7bd1b82 (HEAD -> main) Revert "feat: add discount and user data feature"
c2d87e7 feat: add discount and user data feature
cf16996 bai4: add gitignore management and amend report
57d7a70 chore: add .gitignore and remove credentials.txt from tracking
bbde8e1 (origin/main) bai1
```

| Tiêu chí kiểm tra | Kết quả |
|---|---|
| TC1: HEAD lùi lại 1 commit sau `reset --mixed` | ✅ HEAD về `cf16996`, commit lỗi biến khỏi log |
| TC1: File `buggy_feature.py` còn trong Working Directory | ✅ Xuất hiện dạng Untracked |
| TC2: Có commit mới bắt đầu bằng `Revert "..."` | ✅ Commit `7bd1b82` trên đỉnh nhánh |
| TC2: Lịch sử cũ không bị xóa | ✅ Commit lỗi `c2d87e7` vẫn còn trong log |

---

## 5. So sánh Reset và Revert

| Tiêu chí | `git reset` | `git revert` |
|---|---|---|
| Cơ chế | Di chuyển con trỏ HEAD về commit cũ hơn | Tạo commit mới đảo ngược thay đổi |
| Lịch sử commit | **Bị viết lại** (commit bị xóa khỏi log) | **Được bảo toàn** (commit cũ vẫn còn) |
| An toàn với remote | ❌ Nguy hiểm nếu đã push — gây diverge | ✅ An toàn để `git push` bình thường |
| Phạm vi áp dụng | Chỉ dùng cho commit **chưa push** (local) | Dùng được cho commit **đã push** (public) |
| Mất code | Phụ thuộc chế độ (`--soft/--mixed/--hard`) | Không mất code (luôn tạo commit mới) |

---

## 6. Ghi chú

- `git reset --mixed HEAD~1` tương đương `git reset HEAD~1` vì `--mixed` là chế độ mặc định.
- `git revert <hash>` có thể đảo ngược bất kỳ commit nào trong lịch sử, không chỉ commit gần nhất — nhưng cần chú ý xung đột nếu các commit sau có phụ thuộc.
- Nếu chỉ muốn bỏ stage một file (không lùi commit), dùng `git restore --staged <file>`.
- Nếu muốn hủy thay đổi trong Working Directory, dùng `git restore <file>`.

---
