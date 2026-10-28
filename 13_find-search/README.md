# Find & Search — Tìm kiếm file và nội dung (find, grep, locate, fd)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · GNU findutils 4.10 · grep 3.x · plocate
> **Đối tượng:** Người dùng cần tìm file, thư mục và nội dung trong hệ thống một cách hiệu quả

---

## Mục lục

1. [Tổng quan các công cụ tìm kiếm](#1-tổng-quan-các-công-cụ-tìm-kiếm)
2. [`find` — Tìm theo tên, kiểu, kích thước, thời gian](#2-find--tìm-theo-tên-kiểu-kích-thước-thời-gian)
3. [`find` — Lọc theo quyền, owner, độ sâu](#3-find--lọc-theo-quyền-owner-độ-sâu)
4. [`find -exec` — Thao tác trên kết quả tìm được](#4-find--exec--thao-tác-trên-kết-quả-tìm-được)
5. [`grep` — Tìm nội dung trong file](#5-grep--tìm-nội-dung-trong-file)
6. [`grep -r` — Tìm nội dung trong cả thư mục](#6-grep--r--tìm-nội-dung-trong-cả-thư-mục)
7. [`locate` / `plocate` — Tìm nhanh nhờ database](#7-locate--plocate--tìm-nhanh-nhờ-database)
8. [`fd` và `rg` — Công cụ hiện đại](#8-fd-và-rg--công-cụ-hiện-đại)
9. [`xargs` — Kết hợp tìm kiếm với lệnh khác](#9-xargs--kết-hợp-tìm-kiếm-với-lệnh-khác)
10. [Use-cases thực tế](#10-use-cases-thực-tế)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Tổng quan các công cụ tìm kiếm

| Công cụ | Tìm theo | Tốc độ | Cài sẵn? | Khi nào dùng |
|---|---|---|---|---|
| `find` | Tên, kích thước, thời gian, quyền (duyệt trực tiếp cây thư mục) | Trung bình | ✅ | Tiêu chí phức tạp, cần `-exec` |
| `grep` | **Nội dung** bên trong file | Tùy số file | ✅ | Tìm đoạn text, đoạn code |
| `locate` (plocate) | Tên file, dựa vào database có sẵn | Rất nhanh | ✅ (plocate từ 23.04) | Tìm nhanh tên file |
| `fd` (fd-find) | Tên file, cú pháp thân thiện | Rất nhanh (song song) | ❌ (`sudo apt install fd-find`) | Công việc hằng ngày |
| `rg` (ripgrep) | Nội dung, tôn trọng `.gitignore` | Rất nhanh | ❌ (`sudo apt install ripgrep`) | Tìm code trong dự án |

> 💡 Quy tắc chung: tìm **tên file** → `locate`/`fd`; tiêu chí phức tạp hoặc cần hành động trên kết quả → `find`; tìm **nội dung** → `grep` (hoặc `ripgrep`).

---

## 2. `find` — Tìm theo tên, kiểu, kích thước, thời gian

### Theo tên (`-name`, `-iname`)

```bash
find . -name "*.log"           # Tìm trong thư mục hiện tại, đuôi .log
find /etc -name "hosts"        # Tìm chính xác tên hosts
find . -iname "README.md"      # Không phân biệt hoa/thường
find . -name "*.tmp" -not -path "*/node_modules/*"   # Loại trừ thư mục
```

> ⚠️ Luôn đặt pattern trong ngoặc kép `"*.log"` để shell không tự expand trước khi `find` nhận được.

### Theo kiểu file (`-type`)

```bash
find . -type f                 # Chỉ file thường
find . -type d                 # Chỉ thư mục
find . -type l                 # Chỉ symlink
find /var/log -type f -name "*.gz"
```

### Theo kích thước (`-size`)

```bash
find . -size +100M             # File lớn hơn 100 MB
find . -size -1k               # File nhỏ hơn 1 KB
find . -size +50M -size -500M  # Trong khoảng 50MB–500MB
find / -type f -size +1G 2>/dev/null   # File > 1GB trên toàn hệ thống
```

### Theo thời gian

| Tùy chọn | Ý nghĩa |
|---|---|
| `-mtime n` | Sửa đổi đúng `n` ngày trước (24h một đơn vị) |
| `-mtime -7` | Sửa trong 7 ngày gần nhất |
| `-mtime +30` | Không sửa trong hơn 30 ngày |
| `-mmin -60` | Sửa trong 60 phút gần nhất |
| `-atime`, `-ctime` | Tương tự cho thời gian truy cập / thay đổi inode |

```bash
find /var/log -mtime +30 -name "*.log"     # Log cũ hơn 30 ngày
find ~ -mmin -10 -type f                    # File vừa sửa trong 10 phút
find . -newer reference.txt                 # File mới hơn một file mốc
```

---

## 3. `find` — Lọc theo quyền, owner, độ sâu

```bash
find . -perm 777                      # File có quyền 777 (nguy hiểm!)
find . -perm -u=s                     # Có setuid
find /var/www -user www-data          # Theo owner
find /var/www -group www-data         # Theo nhóm
find . -nouser -o -nogroup           # File mồ côi (owner không tồn tại)
find . -empty                         # File/thư mục rỗng
```

### Giới hạn độ sâu

```bash
find . -maxdepth 1 -type d            # Chỉ thư mục con cấp 1
find . -maxdepth 2 -name "*.conf"
find . -mindepth 2 -maxdepth 4 -type f
```

### Kết hợp điều kiện: `-and` (mặc định), `-or`, `!`

```bash
find . -type f \( -name "*.jpg" -o -name "*.png" \) -size +1M
find . -type f ! -name "*.backup" -user nbhson
```

> ⚠️ Với `-or` / `!`, bọc mỗi nhóm trong `\(...\)` để tránh nhầm lẫn thứ tự ưu tiên.

---

## 4. `find -exec` — Thao tác trên kết quả tìm được

```bash
# Xóa mọi file .tmp
find . -type f -name "*.tmp" -delete

# Chmod mọi script .sh thành 755
find . -name "*.sh" -exec chmod 755 {} \;

# In chi tiết từng kết quả (như ls -l)
find /var/log -name "*.gz" -exec ls -lh {} \;

# -exec ... {} + : gom nhiều kết quả vào 1 lần gọi lệnh (nhanh hơn \;)
find . -name "*.bak" -exec rm {} +

# Chạy lệnh hỏi xác nhận trước khi thực hiện
find . -name "*.cache" -ok rm {} \;

# grep trên các file tìm được
find . -name "*.conf" -exec grep -l "Listen" {} +
```

| Cách kết thúc | Ý nghĩa |
|---|---|
| `\;` | Chạy lệnh **mỗi file một lần**, `{}` thay bằng tên file |
| `{} +` | Gom nhiều file vào **một lần** gọi (giống `xargs`) |
| `-ok ... \;` | Như `\;` nhưng hỏi xác nhận |

> 💡 Với số lượng file rất lớn, `-exec ... {} +` an toàn hơn `xargs` khi tên file có khoảng trắng/ký tự đặc biệt.

---

## 5. `grep` — Tìm nội dung trong file

```bash
grep "error" app.log             # In các dòng chứa "error"
grep -i "error" app.log          # Không phân biệt hoa/thường
grep -n "TODO" main.py           # In kèm số dòng
grep -c "GET" access.log         # Đếm số dòng khớp
grep -v "^#" config.conf         # In dòng KHÔNG khớp (bỏ comment)
grep -w "root" /etc/passwd       # Khớp nguyên từ (word)
grep -x "bin" file.txt           # Khớp nguyên dòng
```

### Regex cơ bản

```bash
grep "^Error" app.log            # Dòng bắt đầu bằng Error
grep "timeout$" app.log          # Dòng kết thúc bằng timeout
grep -E "error|fail|critical" app.log   # regex mở rộng (egrep, nhiều mẫu)
grep -E "[0-9]{3}-[0-9]{4}" data.txt    # Mẫu dạng 123-4567
grep -o "https\?://[^ ]*" page.html     # Chỉ in phần khớp (-o)
```

### Hiển thị kết quả hữu ích

| Tùy chọn | Ý nghĩa |
|---|---|
| `-l` | Chỉ in **tên file** chứa khớp |
| `-L` | Chỉ in tên file **không** chứa khớp |
| `-h` | Không in tên file (khi grep nhiều file) |
| `-H` | Luôn in tên file |
| `-A n` | In thêm `n` dòng **sau** dòng khớp (After) |
| `-B n` | In thêm `n` dòng **trước** (Before) |
| `-C n` | In `n` dòng xung quanh (Context) |
| `-r` / `-R` | Đệ quy qua thư mục |

---

## 6. `grep -r` — Tìm nội dung trong cả thư mục

```bash
grep -r "Listen 80" /etc/apache2/
grep -rn "TODO" ~/projects/myapp/          # Kèm số dòng
grep -rl "deprecated" .                    # Chỉ tên file
grep -r --include="*.py" "import numpy" .
grep -r --exclude-dir={node_modules,.git,dist} "API_KEY" .
grep -ri "password" /etc/                  # Case-insensitive toàn /etc
```

> 💡 Với mã nguồn, `ripgrep` (`rg`) nhanh hơn nhiều và mặc định bỏ qua nội dung `.gitignore`:
> `rg "useEffect" ~/projects/myapp/src`

### Ép màu và giữ output cho script

```bash
grep --color=always "root" /etc/passwd | less -R
grep -q "ready" status.txt && echo "Đã sẵn sàng"   # -q: chỉ dùng exit code
```

`grep` trả exit code: `0` = tìm thấy, `1` = không tìm thấy, `2` = lỗi — rất tiện trong shell script (xem bài 16).

---

## 7. `locate` / `plocate` — Tìm nhanh nhờ database

Ubuntu 26.04 dùng **plocate** (bản thay thế nhanh hơn của mlocate), database được cập nhật hằng ngày bởi `plocate-updatedb.timer` (systemd timer).

```bash
locate nginx.conf          # Tìm ngay, gần như tức thì
locate -i readme           # Không phân biệt hoa/thường
locate -n 10 "*.desktop"   # Chỉ hiện 10 kết quả đầu
locate -c "*.log"          # Đếm số kết quả

sudo updatedb              # Cập nhật database thủ công (sau khi tạo file mới)
```

> ⚠️ File vừa tạo **chưa có** trong database cho đến lần `updatedb` tiếp theo — với file mới hãy dùng `find`.
> `locate` chỉ thấy được những file mà user hiện tại có quyền đi qua thư mục chứa nó (plocate tự lọc theo quyền).

---

## 8. `fd` và `rg` — Công cụ hiện đại

```bash
sudo apt install fd-find ripgrep
# Ubuntu đặt binary là fdfind để tránh trùng package khác; tạo alias:
echo "alias fd=fdfind" >> ~/.bashrc && source ~/.bashrc
```

### `fd` — thay thế `find` cho công việc hằng ngày

```bash
fd readme                  # Tìm tên chứa "readme" (regex, không hoa/thường)
fd -e py                   # Mọi file .py
fd -e log -x gzip          # Gzip mọi file .log tìm được
fd -H .git                 # -H: cả file ẩn
fd -t d node_modules -x rm -rf    # Xóa mọi thư mục node_modules
fd --changed-within 1d     # File thay đổi trong 1 ngày
```

### `rg` — thay thế `grep -r` cho mã nguồn

```bash
rg "useState" src/               # Nhanh, có màu, bỏ qua .gitignore
rg -n --no-heading "ERROR" log/
rg -t py "def main"              # -t: lọc theo loại file
rg -i -C 2 "timeout" config/
rg --files | rg "\.test\."       # Liệt kê file test
```

---

## 9. `xargs` — Kết hợp tìm kiếm với lệnh khác

```bash
find . -name "*.png" | xargs optipng          # Tối ưu mọi png
grep -rl "old-domain.com" . | xargs sed -i 's/old-domain.com/new-domain.com/g'
find /tmp -type f -mtime +7 | xargs rm -f
find . -name "*.jpg" -print0 | xargs -0 -P 4 gzip   # 4 luồng song song, an toàn tên file
cat urls.txt | xargs -n 1 curl -sO            # Tải từng URL
```

| Tùy chọn `xargs` | Ý nghĩa |
|---|---|
| `-0` | Đọc input phân tách bằng NUL (dùng với `find -print0`) |
| `-n 1` | Mỗi lệnh nhận 1 đối số |
| `-P n` | Chạy `n` tiến trình song song |
| `-I {}` | Thay `{}` bằng từng dòng input |

> ⚠️ Cặp `-print0` + `xargs -0` là cách an toàn nhất khi tên file có khoảng trắng — kết hợp `find | xargs` thường xuyên bị hỏng nếu quên.

---

## 10. Use-cases thực tế

### Dọn dẹp ổ đĩa

```bash
# Tìm 20 file/thư mục chiếm nhiều dung lượng nhất trong home
du -ah ~ | sort -rh | head -20

# Xóa log cũ hơn 30 ngày trong /var/log
sudo find /var/log -type f -name "*.log" -mtime +30 -delete

# Tìm và xóa mọi thư mục __pycache__ / node_modules trong dự án
find . -type d \( -name "__pycache__" -o -name node_modules \) -prune -exec rm -rf {} +
```

### Kiểm tra bảo mật

```bash
# Tìm file setuid (điểm khai thác phổ biến) và so sánh với danh sách chuẩn
sudo find / -perm -4000 -type f 2>/dev/null

# Tìm file quyền mở 777 trong web root
find /var/www -perm 777 -type f

# Tìm file world-writable
find / -xdev -type f -perm -0002 2>/dev/null
```

### Tìm trong mã nguồn

```bash
# Tìm mọi chỗ gọi API cũ
grep -rn "api/v1" src/ --include="*.ts"

# Đổi import hàng loạt
grep -rl "from 'lodash'" src/ | xargs sed -i "s/from 'lodash'/from 'lodash-es'/g"
```

### Tìm cấu hình của một service

```bash
# File nào trong /etc chứa từ "PermitRootLogin"
grep -rn "PermitRootLogin" /etc/ssh/

# Liệt kê file cấu hình nginx active
find /etc/nginx -name "*.conf" -newer /etc/nginx/nginx.conf
```

---

## 11. Troubleshooting

### `find` báo "Permission denied" tràn màn hình

```bash
find / -name "x" 2>/dev/null        # Chuyển stderr vào /dev/null
sudo find / -name "x"               # Hoặc chạy với sudo
```

### `locate` không thấy file mới

```bash
sudo updatedb && locate ten_file
# Kiểm tra timer cập nhật database:
systemctl list-timers plocate-updatedb.timer
```

### `grep -r` quá chậm trên repo lớn

```bash
# Loại trừ thư mục nặng hoặc chuyển sang ripgrep
grep -r --exclude-dir={node_modules,dist,.git} "pattern" .
rg "pattern"          # Nhanh gấp nhiều lần
```

### Tên file có khoảng trắng làm lệnh `xargs` hỏng

```bash
find . -name "*.mp3" -print0 | xargs -0 cp -t /mnt/music
# Hoặc dùng hẳn -exec:
find . -name "*.mp3" -exec cp -t /mnt/music {} +
```

### Regex không khớp như mong đợi

```bash
# Mặc định grep dùng BRE: \+ \? \( \) cần escape; -E thì không
grep "colou\?r" file       # BRE
grep -E "colou?r" file     # ERE, gọn hơn
grep -F "a.b(c)" file      # -F: tìm literal, không phải regex
```

---

## Nguồn tham khảo

- GNU Findutils manual: https://www.gnu.org/software/findutils/manual/
- GNU Grep manual: https://www.gnu.org/software/grep/manual/grep.html
- plocate: https://plocate.sesse.net/
- fd (sharkdp): https://github.com/sharkdp/fd
- ripgrep (BurntSushi): https://github.com/BurntSushi/ripgrep
- `man find`, `man grep`, `man xargs`, `man plocate`
