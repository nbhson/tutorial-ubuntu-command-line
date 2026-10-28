# Environment Variables — Biến môi trường (export, $PATH, .bashrc, .profile)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · bash shell
> **Đối tượng:** Người muốn hiểu biến môi trường và cách shell khởi tạo chúng

---

## Mục lục

1. [Khái niệm biến shell và biến môi trường](#1-khái-niệm-biến-shell-và-biến-môi-trường)
2. [`export` — Chuyển biến thành biến môi trường](#2-export--chuyển-biến-thành-biến-môi-trường)
3. [`$PATH` — Biến quan trọng nhất](#3-path--biến-quan-trọng-nhất)
4. [`env`, `printenv`, `set` — Xem biến](#4-env-printenv-set--xem-biến)
5. [File khởi tạo shell: `.bashrc`, `.profile`, `.bash_profile`](#5-file-khởi-tạo-shell-bashrc-profile-bash_profile)
6. [Biến môi trường hệ thống và per-user](#6-biến-môi-trường-hệ-thống-và-per-user)
7. [Biến môi trường thông dụng](#7-biến-môi-trường-thông-dụng)
8. [Use-cases thực tế](#8-use-cases-thực-tế)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Khái niệm biến shell và biến môi trường

- **Biến shell (shell variable):** chỉ tồn tại trong shell hiện tại, **không truyền** cho tiến trình con.
- **Biến môi trường (environment variable):** được **truyền cho mọi tiến trình con** mà shell khởi chạy.

```bash
# Biến shell — không truyền xuống con
MY_VAR="hello"
bash -c 'echo $MY_VAR'      # In rỗng

# Biến môi trường — truyền xuống con
export MY_VAR="hello"
bash -c 'echo $MY_VAR'      # In: hello
```

### Quy tắc đặt tên và gán

```bash
TEN="Son"                    # KHÔNG có khoảng trắng quanh dấu =
AGE=25
full_name="Nguyen Son"       # Giá trị có space → dùng nháy

echo $TEN                    # Đọc biến (dấu $)
echo "$TEN $full_name"       # Nên bọc trong nháy đôi
echo '${TEN}'                # Nháy đơn → KHÔNG mở rộng biến
```

---

## 2. `export` — Chuyển biến thành biến môi trường

```bash
# Khai báo và export cùng lúc
export DATABASE_URL="postgres://localhost/mydb"

# Hoặc gán trước, export sau
API_KEY=abc123
export API_KEY

# Xóa biến
unset API_KEY

# Liệt kê biến môi trường
printenv                     # Tất cả biến môi trường
env                          # Tương đương
set                          # Bao gồm cả biến shell + hàm
printenv PATH                # Chỉ xem một biến
echo $PATH                   # Cách ngắn gọn nhất
```

> ⚠️ **Phạm vi của `export`:** biến chỉ sống trong **phiên terminal hiện tại** (và các process con của nó). Đóng terminal → biến mất. Muốn vĩnh viễn, ghi vào `~/.bashrc` hoặc file khởi động khác.

---

## 3. `$PATH` — Biến quan trọng nhất

`$PATH` là danh sách thư mục mà shell tìm kiếm khi bạn gõ một lệnh:

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/nbhson/.local/bin
```

### Thêm thư mục vào PATH

```bash
# Tạm thời (chỉ session hiện tại)
export PATH="$PATH:/home/nbhson/bin"

# Vĩnh viễn — thêm vào ~/.bashrc
echo 'export PATH="$HOME/bin:$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Quy tắc tìm lệnh

Shell tìm **theo thứ tự từ trái sang phải** trong `$PATH`. Thư mục đầu tiên chứa file thực thi tên đó sẽ thắng:

```bash
# Xem lệnh nào sẽ được thực thi
which python3               # /usr/bin/python3
type -a python3             # Tất cả các chỗ có lệnh này

# Hash cache — shell nhớ vị trí lệnh đã gọi
hash -r                     # Xóa cache sau khi cài app mới
```

> ⚠️ **Bảo mật:** **KHÔNG BAO GIỜ** thêm dấu `.` vào `$PATH` (ví dụ `export PATH=.:$PATH`). Hacker có thể đặt script độc tên `ls` trong thư mục hiện tại để bạn vô tình chạy.

---

## 4. Xem và sử dụng biến trong lệnh

```bash
# Biến trong subshell và script
NAME="Ubuntu"
echo "Hệ điều hành: $NAME"
echo "Shell hiện tại: $SHELL"
echo "Thư mục home: $HOME"

# Thực thi lệnh và gán vào biến
TODAY=$(date +%Y-%m-%d)
echo "Hôm nay: $TODAY"

# Sử dụng trong script
#!/bin/bash
FILE="backup-$(date +%s).tar.gz"
tar -czf "$FILE" /home/nbhson/projects
echo "Đã tạo: $FILE"
```

---

## 5. File khởi tạo shell — `.bashrc`, `.profile`, `.bash_profile`

Ubuntu dùng bash làm shell mặc định. Các file được đọc khi mở terminal:

| File | Khi nào được đọc | Dành cho |
|---|---|---|
| `/etc/environment` | Mọi user khi login | Biến toàn hệ thống |
| `/etc/profile` | Login shell | Thiết lập hệ thống |
| `~/.profile` | Login shell | Thiết lập cá nhân (login) |
| `~/.bash_profile` | Login shell (nếu có) | Thay thế `~/.profile` |
| `~/.bashrc` | **Mỗi lần mở terminal** (interactive non-login) | Alias, prompt, màu sắc |
| `~/.bash_logout` | Khi thoát login shell | Cleanup |

### Ví dụ nội dung `~/.bashrc` thực tế

```bash
# ~/.bashrc

# Prompt màu
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# Alias
alias ll='ls -lh'
alias la='ls -lha'
alias gs='git status'
alias ..='cd ..'
alias update='sudo apt update && sudo apt upgrade -y'

# Biến môi trường
export EDITOR=nano
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"

# Lịch sử lệnh lớn hơn
export HISTSIZE=10000
export HISTFILESIZE=20000
export HISTCONTROL=ignoredups:erasedups
shopt -s histappend
```

### Cách áp dụng thay đổi

```bash
source ~/.bashrc        # Reload ngay (hoặc: . ~/.bashrc)
exec bash               # Khởi động lại shell hoàn toàn
```

> ⚠️ **Sai lầm phổ biến:** sửa `~/.bashrc` mà quên `source` hoặc mở terminal mới — biến "không có tác dụng" làm người dùng tưởng sai cú pháp. Luôn `source ~/.bashrc` sau khi chỉnh sửa.

---

## 6. Biến môi trường hệ thống

### `/etc/environment` — Cách hệ thống (khuyên dùng cho toàn máy)

```bash
sudo nano /etc/environment
```

```text
JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/bin"
```

### Cách qua systemd

```bash
# Cho một dịch vụ cụ thể (file service)
sudo systemctl edit myapp
# Thêm:
# [Service]
# Environment="NODE_ENV=production"

# Cho toàn hệ thống
sudo systemctl set-environment MY_VAR=value
```

---

## 7. Biến môi trường thông dụng

| Biến | Ý nghĩa | Ví dụ |
|---|---|---|
| `$HOME` | Thư mục home | `/home/nbhson` |
| `$USER` | Tên user hiện tại | `nbhson` |
| `$PWD` | Thư mục hiện tại | — |
| `$PATH` | Danh sách thư mục tìm lệnh | `/usr/bin:...` |
| `$SHELL` | Shell mặc định | `/bin/bash` |
| `$TERM` | Loại terminal | `xterm-256color` |
| `$LANG` | Ngôn ngữ hệ thống | `en_US.UTF-8` |
| `$EDITOR` | Editor mặc định | `nano`, `vim` |
| `$HOME/.local/bin` | Path thường thêm để chạy tool user | — |

```bash
# Kiểm tra biến môi trường có tồn tại không
[ -z "$MY_VAR" ] && echo "Chưa có" || echo "Có giá trị: $MY_VAR"

# Gán giá trị mặc định
echo "${NAME:-default_name}"
```

---

## 8. Use-cases thực tế

### Cấu hình environment cho một dự án

```bash
# Tạo file .env trong dự án và load khi cần
cat > .env <<EOF
DB_HOST=localhost
DB_PORT=5432
API_KEY=secret
EOF

# Load vào shell hiện tại
set -a; source .env; set +a
# Hoặc trong script:
export $(grep -v '^#' .env | xargs)
```

### Thêm tool tự cài vào PATH

```bash
# Ví dụ: cargo (Rust)
export PATH="$HOME/.cargo/bin:$PATH"

# Node version managers thường yêu cầu thêm
export PATH="$HOME/.nvm/bin:$PATH"
```

### Đổi editor mặc định

```bash
export EDITOR=vim
export VISUAL=vim
# Hoặc hệ thống:
sudo update-alternatives --config editor
```

### Biến môi trường cho một lần chạy lệnh

```bash
NODE_ENV=production node app.js          # Chỉ trong lệnh này
DEBUG=app:* node server.js
HTTP_PROXY=http://proxy.local:3128 curl https://example.com
```

---

## 9. Troubleshooting

### Biến không "sống sót" qua sudo

```bash
# sudo xóa biến môi trường theo mặc định
export MY_VAR=abc
sudo bash -c 'echo $MY_VAR'      # Trống

# Cách truyền qua
sudo MY_VAR=$MY_VAR some_command
sudo -E some_command              # -E giữ môi trường hiện tại
```

### Lệnh "command not found" sau khi cài tool mới

```bash
# 1. Kiểm tra tool nằm ở đâu
find / -name "toolname" 2>/dev/null

# 2. Thêm thư mục chứa nó vào PATH
export PATH="$PATH:/opt/tool/bin"

# 3. Nếu cần vĩnh viễn — thêm vào ~/.bashrc rồi source
echo 'export PATH="$PATH:/opt/tool/bin"' >> ~/.bashrc
source ~/.bashrc

# 4. Nếu shell cache sai vị trí
hash -r
```

### Terminal không load `.bashrc` khi SSH

```bash
# Ubuntu dùng .profile làm mặc định login shell — nó tự gọi .bashrc.
# Nếu .profile bị ghi đè hoặc mất, kiểm tra:
cat ~/.profile
# Phải có đoạn:
# if [ -n "$BASH_VERSION" ]; then
#   . ~/.bashrc
# fi
```

### Xem toàn bộ biến môi trường để debug

```bash
env | sort
printenv | grep -i java        # Tìm biến chứa từ khóa
```

---

## Nguồn tham khảo

- Bash manual — Startup files: https://www.gnu.org/software/bash/manual/bash.html#Bash-Startup-Files
- Ubuntu environment variables: https://help.ubuntu.com/community/EnvironmentVariables
- `man bash`, `man env`, `man export` (builtin)
