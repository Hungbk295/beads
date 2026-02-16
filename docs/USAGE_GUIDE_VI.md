# Hướng dẫn sử dụng Beads (bd)

**Beads** là một hệ thống quản lý issue phân tán, dựa trên git, được thiết kế đặc biệt cho AI agent. Nó cung cấp bộ nhớ có cấu trúc và bền vững cho các coding agent, giúp chúng xử lý các tác vụ phức tạp mà không mất ngữ cảnh.

## Mục lục

- [Cài đặt](#cài-đặt)
- [Khởi tạo](#khởi-tạo)
- [Tạo issue](#tạo-issue)
- [Quản lý issue](#quản-lý-issue)
- [Phụ thuộc (Dependencies)](#phụ-thuộc-dependencies)
- [Nhãn (Labels)](#nhãn-labels)
- [Quy trình làm việc](#quy-trình-làm-việc)
- [Epic và issue phân cấp](#epic-và-issue-phân-cấp)
- [Đồng bộ và daemon](#đồng-bộ-và-daemon)
- [Cấu hình](#cấu-hình)
- [Các lệnh hữu ích khác](#các-lệnh-hữu-ích-khác)

---

## Cài đặt

Chọn một trong các cách sau:

```bash
# Script cài đặt nhanh
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash

# npm
npm install -g @beads/bd

# Homebrew
brew install beads

# Go
go install github.com/steveyegge/beads/cmd/bd@latest
```

Kiểm tra cài đặt thành công:

```bash
bd --help
```

**Yêu cầu hệ thống:** Linux, macOS, Windows hoặc FreeBSD.

---

## Khởi tạo

Vào thư mục dự án của bạn và chạy:

```bash
cd your-project
bd init
```

Trình khởi tạo sẽ:
- Tạo thư mục `.beads/` và cơ sở dữ liệu
- Hỏi vai trò của bạn (maintainer hoặc contributor)
- Import các issue hiện có từ git (nếu có)
- Đề xuất cài đặt git hooks
- Tự động khởi chạy daemon cho đồng bộ

### Các chế độ khởi tạo

```bash
# Contributor (workflow fork - lưu issue ở repo riêng)
bd init --contributor

# Thành viên team (workflow nhánh)
bd init --team

# Nhánh main được bảo vệ (dùng nhánh đồng bộ riêng)
bd init --branch beads-sync

# Chế độ ẩn (dùng cá nhân, không commit vào repo chính)
bd init --stealth

# Backend Dolt (cơ sở dữ liệu SQL có version control)
bd init --backend dolt
```

---

## Tạo issue

### Cơ bản

```bash
bd create "Tiêu đề issue"
```

### Với các tùy chọn

```bash
# Đầy đủ tùy chọn
bd create "Thiết lập cơ sở dữ liệu" -p 1 -t task -d "Mô tả chi tiết"

# Tạo với nhãn
bd create "Sửa lỗi đăng nhập" -p 0 -t bug -l "backend,urgent"

# Tạo từ file markdown
bd create -f feature-plan.md

# Tạo epic
bd create "Hệ thống xác thực" -t epic -p 1
```

### Các thông số

| Tùy chọn | Mô tả | Giá trị |
|-----------|--------|---------|
| `-p` | Mức ưu tiên | 0 (cao nhất) đến 4 (thấp nhất) |
| `-t` | Loại issue | `bug`, `feature`, `task`, `epic`, `chore` |
| `-d` | Mô tả | Chuỗi văn bản |
| `-l` | Nhãn | Danh sách phân cách bằng dấu phẩy |
| `-f` | Tạo từ file | Đường dẫn file markdown |
| `--parent` | Issue cha | ID của epic |

---

## Quản lý issue

### Xem danh sách

```bash
# Liệt kê tất cả issue đang mở
bd list

# Lọc theo trạng thái
bd list --status open
bd list --status in_progress
bd list --status closed

# Lọc theo nhãn
bd list --label backend

# Xem chi tiết một issue
bd show <id>

# Xem dưới dạng JSON (hữu ích cho AI agent)
bd show <id> --json
```

### Cập nhật issue

```bash
# Bắt đầu làm việc
bd update <id> --status in_progress

# Thay đổi mức ưu tiên
bd update <id> --priority 1

# Đánh dấu bị chặn
bd update <id> --status blocked
```

### Đóng và mở lại issue

```bash
# Đóng issue
bd close <id> --reason "Hoàn thành"

# Mở lại issue
bd reopen <id>
```

### Thêm bình luận

```bash
bd comment <id> "Nội dung bình luận"
```

---

## Phụ thuộc (Dependencies)

Hệ thống phụ thuộc giúp theo dõi thứ tự công việc.

### Thêm phụ thuộc

```bash
# B phụ thuộc vào A (A phải hoàn thành trước B)
bd dep add <id-B> <id-A>
```

### Xem cây phụ thuộc

```bash
bd dep tree <id>
```

Kết quả:
```
🌲 Dependency tree for bd-f14c:

→ bd-f14c: Tạo API [P2] (open)
  → bd-a1b2: Thiết lập database [P1] (open)
```

### Phát hiện vòng lặp

```bash
bd dep cycles
```

### Các loại phụ thuộc

| Loại | Mô tả |
|------|--------|
| `blocks` | A chặn B (B không thể bắt đầu khi A chưa xong) |
| `related` | Liên quan nhưng không chặn |
| `parent-child` | Quan hệ cha-con |
| `discovered-from` | Phát hiện trong quá trình làm việc |

---

## Nhãn (Labels)

### Quản lý nhãn

```bash
# Thêm nhãn
bd label add <id> backend,urgent

# Xóa nhãn
bd label remove <id> urgent

# Xem nhãn của issue
bd label list <id>

# Lọc issue theo nhãn
bd list --label backend,auth
```

---

## Quy trình làm việc

### Tìm việc sẵn sàng

```bash
# Liệt kê các task không bị chặn bởi task nào khác
bd ready

# Lọc theo mức ưu tiên
bd ready --priority 1
```

### Xem issue bị chặn

```bash
bd blocked
```

### Tìm issue cũ

```bash
bd stale --days 30
```

### Xem thống kê

```bash
bd stats
```

### Quy trình điển hình

```bash
# 1. Tìm việc sẵn sàng
bd ready

# 2. Bắt đầu làm (ví dụ issue bd-a1b2)
bd update bd-a1b2 --status in_progress

# 3. Hoàn thành
bd close bd-a1b2 --reason "Database setup xong"

# 4. Kiểm tra task tiếp theo đã sẵn sàng chưa
bd ready
```

---

## Epic và issue phân cấp

Beads hỗ trợ cấu trúc phân cấp cho các tính năng lớn:

```bash
# Tạo epic
bd create "Hệ thống Auth" -t epic -p 1
# Trả về: bd-a3f8

# Tạo task con (tự động có hậu tố .1, .2, .3)
bd create "Thiết kế giao diện đăng nhập" -p 1 --parent bd-a3f8    # bd-a3f8.1
bd create "Xác thực backend" -p 1 --parent bd-a3f8                 # bd-a3f8.2
bd create "Test tích hợp" -p 1 --parent bd-a3f8                    # bd-a3f8.3

# Xem cấu trúc phân cấp
bd dep tree bd-a3f8
```

Kết quả:
```
🌲 Dependency tree for bd-a3f8:

→ bd-a3f8: Hệ thống Auth [epic] [P1] (open)
  → bd-a3f8.1: Thiết kế giao diện đăng nhập [P1] (open)
  → bd-a3f8.2: Xác thực backend [P1] (open)
  → bd-a3f8.3: Test tích hợp [P1] (open)
```

---

## Đồng bộ và daemon

### Kiến trúc 3 lớp

```
CLI (bd create, bd list, ...)
    ↓
SQLite (.beads/beads.db) - cục bộ, không commit vào git
    ↓ tự động đồng bộ (mỗi 5 giây)
JSONL (.beads/issues.jsonl) - được git theo dõi
    ↓ git push/pull
Remote Repository - chia sẻ giữa các máy
```

### Quản lý daemon

```bash
# Kiểm tra trạng thái daemon
bd info | grep daemon

# Liệt kê tất cả daemon đang chạy
bd daemons list

# Chạy trực tiếp (bỏ qua daemon)
bd --no-daemon ready
```

### Khi nào nên tắt daemon

- Khi dùng git worktrees
- Trong CI/CD pipeline
- Trên môi trường hạn chế tài nguyên

---

## Cấu hình

### File cấu hình

Beads sử dụng file cấu hình tại `~/.config/bd/config.yaml` hoặc `.beads/config.yaml`:

```yaml
json: false              # Xuất JSON mặc định
no-daemon: false         # Tắt daemon
no-auto-flush: false     # Tắt tự động flush
```

### Cấu hình vai trò

```bash
# Đặt vai trò contributor
git config beads.role contributor

# Đặt vai trò maintainer
git config beads.role maintainer

# Kiểm tra vai trò hiện tại
git config --get beads.role
```

---

## Các lệnh hữu ích khác

### Kiểm tra sức khỏe hệ thống

```bash
bd doctor
```

### Xem thông tin cơ sở dữ liệu

```bash
bd info --json
```

### Di chuyển cơ sở dữ liệu

```bash
# Xem trước thay đổi
bd migrate --dry-run

# Thực hiện di chuyển
bd migrate

# Di chuyển và dọn dẹp file cũ
bd migrate --cleanup --yes
```

### Nén dữ liệu (Compaction)

```bash
# Xem thống kê nén
bd admin compact --stats

# Phân tích ứng viên nén (issue đã đóng > 30 ngày)
bd admin compact --analyze --json --no-daemon
```

### Phát hiện issue trùng lặp

```bash
bd duplicates
```

---

## Tham khảo thêm

- [Quickstart (English)](QUICKSTART.md) - Hướng dẫn nhanh 2 phút
- [CLI Reference](CLI_REFERENCE.md) - Tham chiếu đầy đủ các lệnh
- [Architecture](ARCHITECTURE.md) - Kiến trúc hệ thống
- [FAQ](FAQ.md) - Câu hỏi thường gặp
- [Troubleshooting](TROUBLESHOOTING.md) - Xử lý sự cố
