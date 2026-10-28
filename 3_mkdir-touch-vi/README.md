# mkdir, touch, vi (vim) — Tạo thư mục, file và chỉnh sửa

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · bash shell
> **Đối tượng:** Người mới bắt đầu làm quen với terminal

---

## Mục lục

1. [`mkdir` — Tạo thư mục](#1-mkdir--tạo-thư-mục)
2. [`rmdir` / `rm` — Xóa thư mục, file](#2-rmdir--rm--xóa-thư-mục-file)
3. [`touch` — Tạo file rỗng / cập nhật timestamp](#3-touch--tạo-file-rỗng--cập-nhật-timestamp)
4. [`vi` / `vim` — Trình soạn thảo văn bản](#4-vi--vim--trình-soạn-thảo-văn-bản)
5. [`nano` — Trình soạn thảo thay thế dễ dùng hơn](#5-nano--trình-soạn-thảo-thay-thế-dễ-dùng-hơn)
6. [So sánh vi vs nano vs graphical editors](#6-so-sánh-vi-vs-nano-vs-graphical-editors)
7. [Mẹo thực tế](#7-mẹo-thực-tế)

---

## 1. `mkdir` — Tạo thư mục

### Cú pháp cơ bản

```bash
mkdir ten_thu_muc
mkdir /home/nbhson/projects
```

### Tạo thư mục lồng nhau — `-p`

```bash
# LỖI: parent hoặc child1 chưa tồn tại
mkdir parent/child1/child2

# ĐÚNG: tạo toàn bộ cây thư mục trung gian
mkdir -p parent/child1/child2
```

Cờ `-p` (parents) tự tạo mọi thư mục trung gian nếu chưa tồn tại. Đây là cờ được dùng rất thường xuyên.

### Ví dụ thực tế

```bash
# Tạo cấu trúc dự án nhanh
mkdir -p ~/projects/my-app/{src,docs,tests,scripts}

# Kết quả:
# ~/projects/my-app/
# ├── src/
# ├── docs/
# ├── tests/
# └── scripts/

# Tạo thư mục nhiều cấp cùng lúc
mkdir -p /var/www/my-site/public_html /var/log/my-site

# Báo lỗi nếu thư mục đã tồn tại (thay vì warning)
mkdir -p existing-dir
```

### Cờ verbose

```bash
mkdir -v backup      # In ra: mkdir: created directory 'backup'
mkdir -vp a/b/c      # In từng bước tạo
```

### Đặt quyền ngay khi tạo — `-m`

```bash
# Tạo thư mục private chỉ owner được vào
mkdir -m 700 ~/secrets

# Tạo thư mục chia sẻ cho nhóm
sudo mkdir -m 775 /var/share/team
```

---

## 2. `rmdir` / `rm` — Xóa thư mục, file

### `rmdir` — Chỉ xóa thư mục TRỐNG

```bash
rmdir empty_dir

# Lỗi nếu thư mục có nội dung:
# rmdir: failed to remove 'docs': Directory not empty
```

```bash
# Xóa chuỗi thư mục trống lồng nhau
rmdir -p a/b/c    # Xóa c, rồi b, rồi a (nếu tất cả đều trống)
```

### `rm` — Xóa file/thư mục (CÓ THỂ XÓA VĨNH VIỄN)

```bash
rm ten_file.txt               # Xóa file (hỏi xác nhận nếu alias đã đặt)
rm -i ten_file.txt            # Luôn hỏi trước khi xóa
rm -f ten_file.txt            # Xóa không hỏi (force)
rm -r ten_thu_muc/            # Xóa thư mục và toàn bộ nội dung (recursive)
rm -rf ten_thu_muc/           # Xóa thư mục + toàn bộ nội dung, không hỏi
```

> ⚠️ **CẢNH BÁO NGHIÊM TRỌNG:** `rm -rf` không đưa vào thùng rác — file **mất vĩnh viễn**. Cẩn thận với các lệnh như:
>
> ```bash
> rm -rf /          # XÓA TOÀN BỘ HỆ THỐNG — TUYỆT ĐỐI KHÔNG
> rm -rf ~         # Xóa toàn bộ home — TUYỆT ĐỐI KHÔNG
> rm -rf $VAR/     # Nếu $VAR rỗng → thành rm -rf / → THẢM HỌA
> ```
>
> **Mẹo an toàn:** Dùng `rm -ri` khi chưa chắc, hoặc đặt alias `rm='rm -i'` trong `~/.bashrc` để luôn được hỏi xác nhận.

```bash
# Xóa nhiều file theo pattern
rm *.log
rm -rf node_modules/          # Xóa dependencies của Node.js
rm -rf /tmp/my-temp-dir

# Xóa file trống/thư mục trống
find . -name "*.tmp" -delete
```

---

## 3. `touch` — Tạo file rỗng / cập nhật timestamp

### Cú pháp

```bash
touch ten_file.txt           # Tạo file rỗng (nếu chưa tồn tại)
```

Nếu file **đã tồn tại**, `touch` sẽ **cập nhật timestamp** (thời gian chỉnh sửa) lên thời điểm hiện tại — không thay đổi nội dung.

### Ví dụ thực tế

```bash
# Tạo nhiều file cùng lúc
touch index.html style.css script.js

# Tạo file có khoảng trắng trong tên
touch "my document.txt"

# Tạo file trong thư mục khác
touch /tmp/demo.txt

# Tạo nhiều file đánh số
touch file_{1..10}.txt

# Chỉ đổi timestamp mà không tạo file mới
touch -c existing.txt

# Đặt timestamp cụ thể
touch -t 202604121430.30 note.txt   # 12/04/2026 14:30:30
touch -r reference.txt target.txt   # Copy timestamp từ file khác
```

### Lưu ý

> ⚠️ `touch` **không thể tạo file bên trong thư mục chưa tồn tại**:
>
> ```bash
> touch missing_dir/file.txt
> # touch: cannot touch 'missing_dir/file.txt': No such file or directory
> ```

---

## 4. `vi` / `vim` — Trình soạn thảo văn bản

`vi` (hoặc bản mở rộng `vim`) là editor được cài sẵn trên hầu hết hệ Unix/Linux. Đây là editor mặc định bạn sẽ gặp khi `sudo visudo`, `git commit`, hoặc chỉnh sửa file hệ thống.

### Ba chế độ cơ bản của vi

```text
┌─────────────────────┐
│  NORMAL (mặc định)  │  ← Nhấn ESC để quay về đây
│         ↓ i,a,o     │
│  INSERT             │  ← Soạn thảo văn bản
│         ↑ ESC       │
│  NORMAL             │
│         ↓ :         │
│  COMMAND            │  ← :wq, :q!, :w ...
└─────────────────────┘
```

### Mở file bằng vi

```bash
vi ten_file.txt        # Mở file (tạo mới nếu chưa có)
vim ten_file.txt       # Tương đương, vim có nhiều tính năng hơn
sudo vi /etc/fstab     # Chỉnh sửa file hệ thống
```

### Các lệnh NORMAL mode — Di chuyển

| Phím | Chức năng |
|---|---|
| `h` | Sang trái |
| `j` | Xuống |
| `k` | Lên |
| `l` | Sang phải |
| `w` | Nhảy tới đầu từ tiếp theo |
| `b` | Nhảy về đầu từ trước |
| `0` | Đầu dòng |
| `$` | Cuối dòng |
| `gg` | Đầu file |
| `G` | Cuối file |
| `:5` | Nhảy tới dòng 5 |

### Các lệnh NORMAL mode — Chỉnh sửa

| Phím | Chức năng |
|---|---|
| `i` | Chèn TRƯỚC con trỏ (insert mode) |
| `a` | Chèn SAU con trỏ (append) |
| `o` | Tạo dòng mới bên dưới và vào insert |
| `O` | Tạo dòng mới bên trên và vào insert |
| `x` | Xóa 1 ký tự |
| `dd` | Xóa (cut) nguyên dòng |
| `5dd` | Xóa 5 dòng |
| `yy` | Copy nguyên dòng |
| `5yy` | Copy 5 dòng |
| `p` | Paste sau con trỏ |
| `P` | Paste trước con trỏ |
| `u` | Undo |
| `Ctrl+r` | Redo |
| `dw` | Xóa 1 từ |
| `d$` | Xóa từ con trỏ đến cuối dòng |
| `r` | Thay 1 ký tự |
| `cw` | Thay 1 từ |

### Lệnh COMMAND mode (bắt đầu bằng `:`)

| Lệnh | Chức năng |
|---|---|
| `:w` | Lưu file |
| `:q` | Thoát (nếu không có thay đổi) |
| `:wq` hoặc `ZZ` | Lưu và thoát |
| `:q!` | Thoát không lưu |
| `:w filename` | Lưu thành file khác |
| `:set number` | Hiện số dòng |
| `:set nonumber` | Ẩn số dòng |
| `/keyword` | Tìm kiếm xuống dưới (nhấn `n` để nhảy tiếp) |
| `?keyword` | Tìm kiếm lên trên |
| `:%s/old/new/g` | Thay thế `old` → `new` toàn bộ file |
| `:10,20s/old/new/g` | Thay thế từ dòng 10 đến 20 |

### Ví dụ workflow soạn file

```bash
vi hello.txt
# 1. Nhấn i         → vào insert mode
# 2. Gõ: "Hello Ubuntu"
# 3. Nhấn ESC       → về normal mode
# 4. Gõ :wq         → lưu và thoát
```

> ⚠️ **Lỗi "E45: 'readonly' option is set" hoặc không lưu được file:** Bạn đang mở file không có quyền ghi. Thoát bằng `:q!` rồi mở lại bằng `sudo vi file`.

> 💡 **Nếu bị kẹt trong vim** (người mới thường gặp): nhấn `ESC` vài lần rồi gõ `:q!` rồi Enter — sẽ thoát an toàn mà không lưu gì.

---

## 5. `nano` — Trình soạn thảo thay thế dễ dùng hơn

Nếu vi quá khó, `nano` là lựa chọn thân thiện hơn:

```bash
nano ten_file.txt
```

Các phím tắt nano (hiện ngay ở cuối màn hình):

| Phím | Chức năng |
|---|---|
| `Ctrl+O` rồi Enter | Lưu file |
| `Ctrl+X` | Thoát |
| `Ctrl+K` | Cắt dòng hiện tại |
| `Ctrl+U` | Dán |
| `Ctrl+W` | Tìm kiếm |
| `Ctrl+G` | Trợ giúp |
| `Ctrl+\` | Tìm và thay thế |

```bash
# Cài nano nếu chưa có
sudo apt install nano
```

---

## 6. So sánh vi vs nano vs graphical editors

| Tiêu chí | vi/vim | nano | VS Code (GUI) |
|---|---|---|---|
| Cài sẵn trên Ubuntu | ✅ | ✅ | ❌ |
| Có mặt trên mọi server | ✅ | Phần lớn | ❌ |
| Học nhanh | Khó | Rất dễ | Dễ (chuột) |
| Chỉnh sửa file hệ thống | ✅ | ✅ | Qua SSH Remote |
| Phù hợp | Sysadmin, dev quen dùng | Người mới | Lập trình viên |

### Cấu hình vim cơ bản — `~/.vimrc`

```vim
set number          " Hiện số dòng
syntax on           " Bật tô màu cú pháp
set tabstop=2       " Tab = 2 space
set expandtab       " Thay tab bằng space
set autoindent      " Tự thụt lề dòng mới
```

---

## 7. Mẹo thực tế

### Tạo nhanh cấu trúc dự án

```bash
mkdir -p my-project/{src,tests,docs} && \
touch my-project/{README.md,.gitignore} && \
cd my-project
```

### Tạo file với nội dung ngay bằng cat

```bash
# Tạo file và nhập nội dung (Ctrl+D để kết thúc)
cat > note.txt <<EOF
Đây là nội dung
viết nhiều dòng
EOF
```

### Kiểm tra file/thư mục vừa tạo

```bash
ls -lha            # Xem danh sách + file ẩn
ls -lta            # Sắp xếp theo thời gian mới nhất
stat ten_file.txt  # Xem chi tiết metadata của file
```

### Backup file trước khi sửa

```bash
# Backup một file trước khi chỉnh sửa
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Hoặc dùng vi để lưu bản backup cùng lúc
vi +w /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
```

---

## Nguồn tham khảo

- Vim official: https://www.vim.org/docs.php
- GNU nano: https://www.nano-editor.org/docs.php
- GNU Coreutils mkdir: https://www.gnu.org/software/coreutils/manual/html_node/mkdir-invocation.html
