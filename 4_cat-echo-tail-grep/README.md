# cat, echo, tail, grep — Xem nội dung và tìm kiếm file

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · bash shell
> **Đối tượng:** Người mới bắt đầu sử dụng terminal

---

## Mục lục

1. [`cat` — In nội dung file](#1-cat--in-nội-dung-file)
2. [`echo` — In text và tạo file có nội dung](#2-echo--in-text-và-tạo-file-có-nội-dung)
3. [`tail` — Xem phần cuối của file](#3-tail--xem-phần-cuối-của-file)
4. [`head` — Xem phần đầu của file](#4-head--xem-phần-đầu-của-file)
5. [`grep` — Tìm kiếm trong file](#5-grep--tìm-kiếm-trong-file)
6. [Pipe `|` — Kết hợp nhiều lệnh](#6-pipe---kết-hợp-nhiều-lệnh)
7. [Ví dụ thực tế thường gặp](#7-ví-dụ-thực-tế-thường-gặp)
8. [Lỗi thường gặp](#8-lỗi-thường-gặp)

---

## 1. `cat` — In nội dung file

### Cú pháp cơ bản

```bash
cat ten_file                  # In nội dung 1 file
cat file1 file2               # In nhiều file nối tiếp nhau
cat file1 file2 > file3       # Gộp file1 + file2 thành file3
cat file1 >> file2            # Nối file1 vào cuối file2
```

### Các cờ thông dụng

```bash
cat -n ten_file       # Đánh số dòng
cat -b ten_file       # Đánh số chỉ dòng có nội dung
cat -s ten_file       # Nén nhiều dòng trống liên tiếp thành 1 dòng
cat -A ten_file       # Hiện ký tự ẩn ($ cuối dòng, ^I cho tab)
cat -E ten_file       # Hiện dấu $ cuối mỗi dòng
```

### Ví dụ thực tế

```bash
# Xem file cấu hình
cat /etc/hostname

# Đánh số dòng để dễ tham chiếu
cat -n script.sh

# Gộp nhiều file log
cat access.log.1 access.log.2 > access_all.log

# Tạo file từ stdin (kết thúc bằng Ctrl+D)
cat > note.txt
Gõ nội dung ở đây...
[Ctrl+D]

# Dùng heredoc tạo file nhiều dòng
cat > config.yaml <<EOF
server:
  port: 8080
  host: localhost
EOF
```

### File lớn — đừng dùng `cat`

> ⚠️ Với file lớn (hàng nghìn dòng), `cat` sẽ in toàn bộ làm terminal tràn. Hãy dùng `less`, `more`, hoặc `head`/`tail`:
>
> ```bash
> less /var/log/syslog      # Xem theo trang, phím q để thoát
> head -50 /var/log/syslog  # Chỉ xem 50 dòng đầu
> ```

### Lệnh liên quan

```bash
tac ten_file      # In ngược (dòng cuối lên đầu)
rev ten_file      # Đảo ngược từng ký tự trong dòng
nl ten_file       # Đánh số dòng (giống cat -n nhưng nhiều tùy chọn hơn)
```

---

## 2. `echo` — In text và tạo file có nội dung

### Cú pháp

```bash
echo "Hello World"         # In ra màn hình
echo $HOME                 # In giá trị biến môi trường
echo "text" > file.txt     # GHI ĐÈ file bằng text
echo "text" >> file.txt    # THÊM text vào cuối file (không ghi đè)
```

### Ví dụ thực tế

```bash
# Tạo file mới có nội dung
echo "<h1>Demo 4</h1>" > demo4.html

# Thêm dòng mới vào cuối file
echo "appended line" >> demo4.html

# In nhiều dòng
echo -e "Dòng 1\nDòng 2\nDòng 3"     # -e bật ký tự escape \n

# Bỏ ký tự newline cuối
echo -n "no newline"

# In giá trị biến
name="Ubuntu"
echo "Hệ điều hành: $name"

# In lệnh mà KHÔNG thực thi biến (escape $)
echo "Biến HOME là \$HOME"
```

### Chuyển hướng output

| Cú pháp | Ý nghĩa |
|---|---|
| `> file` | Ghi đè file bằng stdout |
| `>> file` | Thêm vào cuối file |
| `2> file` | Ghi stderr vào file |
| `2>> file` | Thêm stderr vào cuối file |
| `&> file` | Ghi cả stdout và stderr |
| `< file` | Đọc input từ file |
| `2>&1` | Gộp stderr vào stdout |

```bash
# Ghi log lỗi riêng
ls /khong-ton-tai 2> error.log

# Gộp cả output và lỗi vào một file
command &> all.log

# Vô hiệu output (thường dùng trong cron)
command > /dev/null 2>&1
```

> ⚠️ **Cẩn thận với `>`:** nó sẽ **ghi đè** file đích mà không cảnh báo. Nhân đôi dấu `>` thành `>>` nếu chỉ muốn thêm nội dung.

---

## 3. `tail` — Xem phần cuối của file

```bash
tail ten_file             # 10 dòng cuối mặc định
tail -n 5 ten_file        # 5 dòng cuối
tail -n +5 ten_file       # Từ dòng 5 đến hết
tail -c 100 ten_file      # 100 ký tự cuối
```

### `tail -f` — Theo dõi file realtime

```bash
tail -f /var/log/syslog           # Theo dõi log hệ thống realtime
tail -f app.log | grep ERROR      # Chỉ hiện dòng chứa ERROR
tail -F app.log                   # Giống -f nhưng tự mở lại nếu file bị rotate
tail -n 100 -f app.log            # Bắt đầu từ 100 dòng cuối rồi follow
```

> 💡 `-f` (follow) là lệnh **"phải biết"** khi debug — xem log ứng dụng đang chạy realtime. Nhấn `Ctrl+C` để thoát.

### Ứng dụng thực tế

```bash
# Xem log Nginx realtime
sudo tail -f /var/log/nginx/access.log

# Xem log xác thực khi SSH đang hoạt động
sudo tail -f /var/log/auth.log

# Theo dõi nhiều file cùng lúc
tail -f /var/log/syslog /var/log/kern.log

# Lấy dòng cuối của output lệnh khác
ls -lt | tail -n +2 | head -5
```

---

## 4. `head` — Xem phần đầu của file

```bash
head ten_file             # 10 dòng đầu
head -n 20 ten_file       # 20 dòng đầu
head -c 500 ten_file      # 500 byte đầu

# Bỏ dòng đầu (ví dụ bỏ header CSV)
tail -n +2 data.csv

# Kết hợp head + tail để lấy đoạn giữa
head -n 100 file.txt | tail -n 20   # Dòng 81-100
```

---

## 5. `grep` — Tìm kiếm trong file

### Cú pháp

```bash
grep pattern file                  # Tìm trong 1 file
grep pattern file1 file2           # Tìm nhiều file
grep -r pattern /thu_muc           # Tìm đệ quy trong thư mục
command | grep pattern             # Lọc output của lệnh khác
```

### Các cờ thông dụng

| Cờ | Ý nghĩa |
|---|---|
| `grep -i` | Không phân biệt hoa/thường (case-insensitive) |
| `grep -v` | Đảo ngược — hiện dòng KHÔNG khớp |
| `grep -n` | Hiện số dòng |
| `grep -c` | Đếm số dòng khớp |
| `grep -r` | Tìm đệ quy trong thư mục |
| `grep -l` | Chỉ hiện tên file có khớp |
| `grep -w` | Khớp nguyên từ (word boundary) |
| `grep -E` | Extended regex (egrep) |
| `grep -A3` | Hiện 3 dòng SAU dòng khớp (after) |
| `grep -B3` | Hiện 3 dòng TRƯỚC dòng khớp (before) |
| `grep -C3` | Hiện 3 dòng trước và sau (context) |
| `grep --color` | Tô màu kết quả khớp |

### Ví dụ thực tế

```bash
# Tìm dòng chứa "68" trong file
cat url_file | grep 68
# Hoặc trực tiếp:
grep 68 url_file

# Tìm không phân biệt hoa thường
grep -i error /var/log/syslog

# Tìm trong nhiều file
grep error /var/log/*.log

# Tìm trong cả thư mục, hiện số dòng
grep -rn "TODO" ~/projects/

# Tìm nhưng loại trừ
grep -v "^#" /etc/ssh/sshd_config    # Bỏ dòng comment
ps aux | grep -v grep                # Bỏ chính dòng grep

# Đếm số dòng khớp
grep -c "Failed password" /var/log/auth.log

# Tìm với regex phức tạp
grep -E "^[0-9]{3}-[0-9]{4}$" data.txt     # Mẫu số 123-4567
grep -E "error|warning|critical" app.log   # Tìm 1 trong nhiều từ

# Tìm với ngữ cảnh
grep -B2 -A5 "Exception" app.log

# Chỉ hiện tên file chứa từ khóa
grep -rl "DatabaseDriver" ~/projects/src/
```

### Regex thường dùng với grep

| Pattern | Ý nghĩa |
|---|---|
| `^text` | Bắt đầu dòng bằng "text" |
| `text$` | Kết thúc dòng bằng "text" |
| `.` | Bất kỳ 1 ký tự nào |
| `*` | Lặp 0 lần trở lên của ký tự trước |
| `[abc]` | 1 trong các ký tự a, b, c |
| `[0-9]` | 1 chữ số |
| `\d` | Chữ số (cần `-P` hoặc `-E`) |
| `|` (với -E) | HOẶC |

---

## 6. Pipe `|` — Kết hợp nhiều lệnh

Pipe (`|`) lấy **output của lệnh trước** làm **input cho lệnh sau**. Đây là "vũ khí" mạnh nhất của command line:

```bash
# Lọc các process của nginx
ps aux | grep nginx

# Đếm số dòng chứa ERROR
cat app.log | grep ERROR | wc -l

# Xem 10 process tốn CPU nhất
ps aux --sort=-%cpu | head -11

# Tìm package đã cài liên quan đến python
apt list --installed 2>/dev/null | grep python

# Chuỗi dài: tìm file lớn nhất trong /var/log
sudo du -ah /var/log | sort -rh | head -5
```

### Các lệnh hỗ trợ pipe phổ biến

| Lệnh | Chức năng |
|---|---|
| `wc -l` | Đếm dòng |
| `sort` | Sắp xếp |
| `uniq` | Loại bỏ dòng trùng liền kề |
| `head` / `tail` | Lấy đầu/cuối |
| `tee file` | Vừa hiện ra màn hình vừa ghi file |
| `xargs` | Chuyển output thành tham số cho lệnh khác |

```bash
# tee: vừa xem vừa lưu log
ls -la | tee output.txt

# xargs: xóa tất cả file .tmp tìm được
find . -name "*.tmp" | xargs rm

# Đếm số file trong thư mục
ls | wc -l
```

---

## 7. Ví dụ thực tế thường gặp

### Debug log server

```bash
# Xem 50 dòng log lỗi gần nhất
tail -50 /var/log/syslog | grep -i error

# Theo dõi lỗi auth realtime
sudo tail -f /var/log/auth.log | grep --color "Failed\|Invalid"

# Đếm số lần SSH bị brute-force hôm nay
sudo grep "Failed password" /var/log/auth.log | wc -l

# Tìm IP nào đăng nhập thất bại nhiều nhất
sudo grep "Failed password" /var/log/auth.log | \
  grep -oE '[0-9]{1,3}(\.[0-9]{1,3}){3}' | \
  sort | uniq -c | sort -rn | head -5
```

### Làm việc với file config

```bash
# Xem config không có comment và dòng trống
grep -Ev "^#|^$" /etc/ssh/sshd_config

# Thêm cấu hình vào cuối file
echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config

# Kiểm tra cấu hình đã được thêm chưa
grep -n "PermitRootLogin" /etc/ssh/sshd_config
```

### Tìm kiếm trong source code

```bash
# Tìm function trong dự án
grep -rn "def main" ~/projects/ --include="*.py"

# Tìm bỏ qua thư mục node_modules
grep -rn "apiKey" ~/projects/ --exclude-dir=node_modules

# Đếm số dòng code trong dự án
find ~/projects/src -name "*.py" | xargs wc -l | tail -1
```

---

## 8. Lỗi thường gặp

### `grep` không tìm thấy nhưng mắt thấy rõ có

```bash
# Nguyên nhân: khoảng trắng không dấu, ký tự đặc biệt
# Giải pháp: dùng -F (fixed string) hoặc escape ký tự đặc biệt
grep -F "C++ (v1.2)" file.txt
grep "1\.2" file.txt      # Escape dấu . (trong regex, . = bất kỳ ký tự)
```

### `echo` hiện ký tự `\n` thay vì xuống dòng

```bash
# Thêm cờ -e để bật xử lý escape
echo -e "dòng 1\ndòng 2"
```

### `tail -f` không thấy gì khi file log vừa rotate

```bash
# Dùng -F (viết hoa) — tự mở lại file mới khi rotate
tail -F /var/log/syslog
```

### Binary file matches

```bash
# grep coi file là binary. Dùng -a để coi như text
grep -a pattern binary_file.log
```

---

## Nguồn tham khảo

- GNU Coreutils (cat, echo, tail, head): https://www.gnu.org/software/coreutils/manual/
- GNU Grep manual: https://www.gnu.org/software/grep/manual/grep.html
