# Bài 2: Tái cấu trúc lịch sử commit bằng Interactive Rebase

## Mục tiêu
- Dùng `git rebase -i` để chỉnh sửa lịch sử commit cục bộ trước khi đẩy lên remote.
- **Squash**: gộp nhiều commit vụn vặt thành một commit có ý nghĩa.
- **Reword**: đặt lại thông điệp commit cho rõ ràng.
- **Drop**: xoá bỏ hoàn toàn commit chứa file rác (`temp.txt`).

## Môi trường
- Windows 10, Git Bash (MINGW64)
- Thư mục thực hành: `/d/rikkei/IT209/Ss5/Ls2`
- Trình soạn thảo cho Git: **Notepad** (`git config --local core.editor notepad`)

---

## Các bước thực hiện

### Bước 1 – Khởi tạo repo và commit nền

```bash
mkdir -p /d/rikkei/IT209/Ss5/Ls2 && cd /d/rikkei/IT209/Ss5/Ls2
git init -b main
git config --local user.name "thang-260704"
git config --local user.email "phanhtran0501@gmail.com"
git config --local core.editor notepad
echo "# Rebase demo" > README.md
git add .
git commit -m "Initial commit"
```

Cần có một commit nền, để `HEAD~4` có điểm xuất phát khi chọn 4 commit tính năng phía trên.

### Bước 2 – Tạo 4 commit "vụn vặt"

```bash
echo "function login() {}" > auth.js
git add . && git commit -m "feat: khoi tao module auth"

echo "function login() { return true; }" > auth.js
git commit -am "fix typo"

echo "function hashPassword(p) { return p; }" >> auth.js
git commit -am "adds utility functions"

echo "debug" > temp.txt
git add . && git commit -m "add temp file for debug"

git log --oneline
```

```text
36582e8 (HEAD -> main) add temp file for debug
ab5459e adds utility functions
26d9b83 fix typo
01e5b2e feat: khoi tao module auth
6a73984 Initial commit
```

![Lịch sử trước khi rebase](images/01-before-rebase.png)

### Bước 3 – Mở Interactive Rebase cho 4 commit gần nhất

```bash
git rebase -i HEAD~4
```

Git mở file `git-rebase-todo` trong Notepad. Các commit được liệt kê **từ cũ đến mới**, tức là ngược với `git log`. Sửa lại như sau:

```text
pick 01e5b2e feat: khoi tao module auth
squash 26d9b83 fix typo
squash ab5459e adds utility functions
drop 36582e8 add temp file for debug

# Rebase 6a73984..36582e8 onto 6a73984 (4 commands)
```

| Lệnh | Ý nghĩa |
|---|---|
| `pick` | Giữ nguyên commit làm gốc để gộp |
| `squash` (`s`) | Gộp commit vào commit **phía trên**, giữ lại thông điệp để soạn lại |
| `drop` (`d`) | Xoá bỏ hoàn toàn commit (cả thay đổi lẫn thông điệp) |
| `reword` (`r`) | Giữ thay đổi nhưng sửa lại thông điệp |
| `fixup` (`f`) | Giống `squash` nhưng bỏ luôn thông điệp của commit bị gộp |

![Giao diện cấu hình Interactive Rebase](images/02-rebase-todo.png)

Lưu (Ctrl+S) và đóng Notepad.

### Bước 4 – Soạn lại thông điệp cho commit gộp (reword)

Git mở tiếp một file chứa thông điệp của cả 3 commit được gộp (`# This is a combination of 3 commits.`). Xoá toàn bộ nội dung và thay bằng:

```text
feat: hoan thien module authentication
```

![Soạn lại thông điệp commit](images/03-commit-message.png)

Lưu và đóng Notepad. Git báo:

```text
Successfully rebased and updated refs/heads/main.
```

### Bước 5 – Kiểm tra kết quả

```bash
git log --oneline
ls
cat auth.js
```

```text
7dfdabf (HEAD -> main) feat: hoan thien module authentication
6a73984 Initial commit

README.md  auth.js

function login() { return true; }
function hashPassword(p) { return p; }
```

![Lịch sử sau khi rebase](images/04-after-rebase.png)

---

## Phân tích kết quả

- **Squash thành công**: 3 commit `feat: khoi tao module auth`, `fix typo` và `adds utility functions` được gộp thành **một** commit duy nhất. Nội dung `auth.js` vẫn giữ đầy đủ thay đổi của cả 3 commit.
- **Reword thành công**: commit gộp có thông điệp rõ nghĩa theo chuẩn Conventional Commits là `feat: hoan thien module authentication`.
- **Drop thành công**: commit `add temp file for debug` bị xoá khỏi lịch sử, và `temp.txt` không còn trong thư mục làm việc.
- Phía trên commit nền `Initial commit`, nhánh tính năng giờ chỉ còn **một commit sạch sẽ**.
- Mã hash của commit gộp **khác hoàn toàn** các commit cũ, vì rebase không sửa commit cũ mà **tạo commit mới** thay thế.

## Lưu ý an toàn

> ⚠️ Chỉ rebase những commit **chưa push** lên remote, hoặc trên nhánh cá nhân chưa ai dùng. Rebase viết lại lịch sử, tức là thay đổi hash commit. Nếu nhánh đã được chia sẻ, đồng đội sẽ bị lệch lịch sử và buộc phải `git push --force`.

Nếu thao tác sai giữa chừng, có thể huỷ toàn bộ bằng `git rebase --abort`. Nếu đã rebase xong mà muốn quay lại, dùng `git reflog` như ở bài 1.

## Đối chiếu yêu cầu

| Yêu cầu | Kết quả |
|---|---|
| Gộp 3 commit nhỏ thành 1 commit (`squash`) | ✅ |
| Thông điệp sau khi gộp là `feat: hoan thien module authentication` | ✅ |
| Xoá commit chứa `temp.txt` (`drop`) | ✅ |
| Có ảnh giao diện Interactive Rebase (`pick` / `squash` / `drop`) | ✅ |
| `git log --oneline` chỉ còn một commit tính năng sạch sẽ | ✅ |
