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
- [Ứng dụng Beads trong quy trình code](#ứng-dụng-beads-trong-quy-trình-code)
  - [Lập kế hoạch tính năng mới](#1-lập-kế-hoạch-tính-năng-mới)
  - [Quy trình code hàng ngày](#2-quy-trình-code-hàng-ngày)
  - [Gắn issue ID vào commit](#3-gắn-issue-id-vào-commit-message)
  - [Tích hợp git hooks](#4-tích-hợp-với-git-hooks)
  - [Làm việc nhóm](#5-làm-việc-nhóm)
  - [Tích hợp AI agent](#6-tích-hợp-với-ai-agent-claude-code-cursor-copilot)
  - [Đa agent song song](#7-đa-agent-làm-việc-song-song)
  - [Tích hợp CI/CD](#8-tích-hợp-cicd)
  - [Quy trình contributor](#9-quy-trình-contributor-đóng-góp-mã-nguồn-mở)
  - [Kết thúc session an toàn](#10-kết-thúc-session-an-toàn)

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

## Ứng dụng Beads trong quy trình code

### 1. Lập kế hoạch tính năng mới

Khi bắt đầu một tính năng lớn, dùng beads để phân rã công việc thành đồ thị phụ thuộc:

```bash
# Tạo epic cho tính năng
bd create "Hệ thống thanh toán" -t epic -p 1
# → bd-a3f8

# Phân rã thành các task con
bd create "Thiết kế schema database" -p 1 --parent bd-a3f8        # bd-a3f8.1
bd create "API xử lý thanh toán" -p 1 --parent bd-a3f8            # bd-a3f8.2
bd create "Tích hợp cổng thanh toán" -p 1 --parent bd-a3f8        # bd-a3f8.3
bd create "Giao diện checkout" -p 2 --parent bd-a3f8               # bd-a3f8.4
bd create "Test end-to-end" -p 1 --parent bd-a3f8                  # bd-a3f8.5

# Thiết lập thứ tự phụ thuộc
bd dep add bd-a3f8.2 bd-a3f8.1   # API phụ thuộc schema
bd dep add bd-a3f8.3 bd-a3f8.2   # Tích hợp phụ thuộc API
bd dep add bd-a3f8.4 bd-a3f8.2   # UI phụ thuộc API
bd dep add bd-a3f8.5 bd-a3f8.3   # Test phụ thuộc tích hợp
bd dep add bd-a3f8.5 bd-a3f8.4   # Test phụ thuộc UI

# Xem đồ thị phụ thuộc
bd dep tree bd-a3f8
```

Kết quả: Beads tự động xác định `bd-a3f8.1` (schema) là task sẵn sàng duy nhất. Khi hoàn thành schema, API và UI sẽ được mở khóa song song.

### 2. Quy trình code hàng ngày

```bash
# === BUỔI SÁNG: Tìm việc ===
bd ready                              # Xem task nào đang sẵn sàng
bd ready --priority 0                 # Chỉ xem P0 (khẩn cấp)

# === BẮT ĐẦU LÀM: Nhận task ===
bd update bd-a3f8.1 --status in_progress

# === TRONG KHI CODE: Phát hiện vấn đề mới ===
bd create "Cần migration cho bảng orders" -p 1 -t task
bd dep add <new-id> bd-a3f8.1 --type discovered-from

# === HOÀN THÀNH: Đóng task ===
bd close bd-a3f8.1 --reason "Schema đã tạo xong, đã migration"

# === KIỂM TRA: Task tiếp theo ===
bd ready                              # Xem task nào vừa được mở khóa
bd stats                              # Xem tiến độ tổng thể
```

### 3. Gắn issue ID vào commit message

Gắn ID beads vào commit message giúp truy vết và phát hiện issue mồ côi:

```bash
git commit -m "Tạo schema bảng orders và payments (bd-a3f8.1)"
git commit -m "Thêm API endpoint /checkout (bd-a3f8.2)"

# bd doctor có thể phát hiện issue đã commit nhưng chưa đóng
bd doctor --fix
```

### 4. Tích hợp với git hooks

Git hooks tự động đồng bộ beads mỗi khi commit/push/pull:

```bash
# Cài đặt hooks
bd hooks install

# Hooks sẽ tự động:
# - pre-commit:  Flush thay đổi DB → JSONL trước khi commit
# - pre-push:    Chặn push nếu JSONL có thay đổi chưa commit
# - post-merge:  Import JSONL vào DB sau khi pull
```

### 5. Làm việc nhóm

```bash
# === LẬP KẾ HOẠCH SPRINT ===
bd create "Feature A" -p 1 -t feature
bd create "Feature B" -p 2 -t feature
bd create "Fix bug đăng nhập" -p 0 -t bug

# Phân công
bd update bd-abc --assignee alice
bd update bd-def --assignee bob

# Thiết lập phụ thuộc
bd dep add bd-def bd-abc            # B phụ thuộc A

# === THEO DÕI TIẾN ĐỘ ===
bd list --status in_progress        # Ai đang làm gì?
bd ready                            # Task nào chưa ai nhận?
bd blocked                          # Task nào đang bị chặn?
bd stats                            # Thống kê tổng quan

# === ĐỒNG BỘ ===
bd sync                             # Đồng bộ với remote
```

### 6. Tích hợp với AI agent (Claude Code, Cursor, Copilot...)

Beads được thiết kế đặc biệt cho AI agent. Agent hoạt động theo vòng lặp:

```bash
# Vòng lặp agent tự động:
while true; do
    # 1. Tìm task sẵn sàng
    TASK=$(bd ready --json --limit 1)

    # 2. Nhận task
    bd update <id> --status in_progress

    # 3. Thực thi (agent viết code, chạy test...)

    # 4. Phát hiện vấn đề mới → tạo issue
    bd create "Bug phát hiện khi test" -p 1
    bd dep add <new-id> <parent-id> --type discovered-from

    # 5. Hoàn thành
    bd close <id> --reason "Đã implement và test"

    # 6. Lặp lại
done

# Kết thúc session: đồng bộ
bd sync
```

**Cài đặt cho Claude Code:**
```bash
bd setup claude                     # Tự động cấu hình hooks
```

**Cài đặt plugin (tùy chọn):**
```
/plugin install beads
```

Các lệnh plugin: `/beads:ready`, `/beads:create`, `/beads:show`, `/beads:close`

### 7. Đa agent làm việc song song

Nhiều agent có thể làm việc đồng thời nhờ hash-based ID:

```
Agent A: bd ready → nhận bd-a3f8.2 (API) → code → close
Agent B: bd ready → nhận bd-a3f8.4 (UI)  → code → close
                                                    ↓
                                    bd-a3f8.5 (Test) tự động mở khóa
                                                    ↓
Agent C: bd ready → nhận bd-a3f8.5 (Test) → test → close
```

- Hash ID tránh xung đột khi tạo issue đồng thời
- Dependency graph tự động mở khóa task khi blocker hoàn thành
- Mỗi agent kết thúc bằng `bd sync` để đẩy thay đổi lên remote

### 8. Tích hợp CI/CD

```bash
# Trong pipeline CI/CD:
bd --no-daemon list --status open --json    # Kiểm tra issue mở
bd --no-daemon ready --json                 # Kiểm tra task sẵn sàng
bd --no-daemon stats --json                 # Báo cáo tiến độ

# Tự động đóng issue khi merge PR:
bd --no-daemon close bd-abc --reason "PR #123 merged"
```

### 9. Quy trình contributor (đóng góp mã nguồn mở)

```bash
# 1. Fork repo và thêm remote upstream
git clone https://github.com/your-fork/project.git
git remote add upstream https://github.com/upstream/project.git

# 2. Khởi tạo ở chế độ contributor
bd init --contributor
# → Issue được lưu ở ~/.beads-planning (KHÔNG đi vào PR)

# 3. Lên kế hoạch
bd create "Nghiên cứu module auth" -p 2
bd create "Sửa bug #42" -p 1

# 4. Code trên nhánh
git checkout -b fix-auth-bug
# ... viết code ...
git add . && git commit -m "Fix: auth validation bug"

# 5. Tạo PR (không chứa file planning)
git push origin fix-auth-bug

# 6. Đóng issue khi PR được merge
bd close bd-abc --reason "PR merged"
```

### 10. Kết thúc session an toàn

Luôn "hạ cánh" đúng cách khi kết thúc phiên làm việc:

```bash
# 1. Tạo issue cho công việc còn lại
bd create "Thêm integration test cho checkout" -p 2

# 2. Chạy quality gates
make lint && make test

# 3. Cập nhật trạng thái issue
bd close bd-42 --reason "Hoàn thành"

# 4. Đồng bộ và đẩy code (BẮT BUỘC!)
bd sync
git push

# 5. Kiểm tra
git status                          # Phải hiện "up to date"
bd ready                            # Xem task tiếp theo cho session sau
```

---

## Tham khảo thêm

- [Quickstart (English)](QUICKSTART.md) - Hướng dẫn nhanh 2 phút
- [CLI Reference](CLI_REFERENCE.md) - Tham chiếu đầy đủ các lệnh
- [Architecture](ARCHITECTURE.md) - Kiến trúc hệ thống
- [Agent Instructions](../AGENT_INSTRUCTIONS.md) - Hướng dẫn cho AI agent
- [Git Integration](GIT_INTEGRATION.md) - Tích hợp git chi tiết
- [Molecules](MOLECULES.md) - Mô hình thực thi đồ thị công việc
- [FAQ](FAQ.md) - Câu hỏi thường gặp
- [Troubleshooting](TROUBLESHOOTING.md) - Xử lý sự cố
