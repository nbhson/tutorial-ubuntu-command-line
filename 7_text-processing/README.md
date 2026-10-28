# Text Processing — Xử lý văn bản nâng cao (sed, awk, cut, sort, uniq...)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · bash shell
> **Đối tượng:** Người muốn xử lý log, file text, dữ liệu bằng command line

---

## Mục lục

1. [`cut` — Cắt cột/ký tự](#1-cut--cắt-cộtký-tự)
2. [`sort` — Sắp xếp dòng](#2-sort--sắp-xếp-dòng)
3. [`uniq` — Loại bỏ dòng trùng liền kề](#3-uniq--loại-bỏ-dòng-trùng-liền-kề)
4. [`wc` — Đếm dòng/từ/ký tự](#4-wc--đếm-dòngtừký-tự)
5. [`tr` — Thay thế/xóa ký tự](#5-tr--thay-thểxóa-ký-tự)
6. [`sed` — Stream editor theo pattern](#6-sed--stream-editor-theo-pattern)
7. [`awk` — Ngôn ngữ xử lý văn bản mạnh mẽ](#7-awk--ngôn-ngữ-xử-lý-văn-bản-mạnh-mẽ)
8. [Chuỗi pipe phức tạp thực tế](#8-chuỗi-pipe-phức-tạp-thực-tế)
9. [Mẹo và lưu ý](#9-mẹo-và-lưu-ý)

---

## 1. `cut` — Cắt cột/ký tự

`cut` trích xuất một phần mỗi dòng theo ký tự hoặc trường (field).

```bash
# Cắt theo trường (delimiter mặc định: TAB)
cut -d',' -f1 data.csv          # Trường 1, delimiter là dấu phẩy
cut -d: -f1,6 /etc/passwd       # Trường 1 và 6
cut -d: -f1-3 /etc/passwd       # Trường 1 đến 3

# Cắt theo vị trí ký tự
cut -c1-10 file.txt             # Ký tự 1 đến 10 của mỗi dòng
cut -c1 file.txt                # Chỉ ký tự đầu
cut -c5- file.txt               # Từ ký tự 5 tới cuối

# Kết hợp pipe
ps aux | tr -s ' ' | cut -d' ' -f1
```

> 💡 `cut` coi **các ký tự delimiter liền nhau là 2 trường khác nhau** — với dữ liệu có nhiều khoảng trắng liền (như `ps aux`), dùng `tr -s ' '` để nén trước, hoặc dùng `awk` sẽ tiện hơn.

### Ví dụ thực tế

```bash
# Lấy danh sách username trong hệ thống
cut -d: -f1 /etc/passwd

# Lấy IP đầu dòng log
echo "192.168.1.100 - - [27/Oct/2026] GET /api" | cut -d' ' -f1

# Lấy tên file không kèm đuôi mở rộng
echo "report.2026.pdf" | rev | cut -d. -f2- | rev
```

---

## 2. `sort` — Sắp xếp dòng

```bash
sort file.txt              # Sắp xếp alphabet
sort -r file.txt           # Đảo ngược
sort -n numbers.txt        # Sắp xếp theo số (numeric)
sort -h sizes.txt          # Theo dung lượng human (1K, 2M, 3G)
sort -u file.txt           # Sắp xếp + loại trùng (giống sort | uniq)
sort -k2 data.txt          # Sắp theo cột 2
sort -t',' -k2 -n data.csv # Delimiter phẩy, sắp cột 2 theo số
sort -R file.txt           # Sắp ngẫu nhiên (random)
```

### Ví dụ

```bash
# Sắp xếp 5 thư mục lớn nhất
du -sh */ | sort -rh | head -5

# Sắp các process theo RAM
ps aux --sort=-%mem | head -5

# Sắp user theo tên trong /etc/passwd
cut -d: -f1 /etc/passwd | sort
```

---

## 3. `uniq` — Loại bỏ dòng trùng liền kề

> ⚠️ `uniq` chỉ bỏ trùng với **dòng liền kề** — thường phải `sort` trước:

```bash
sort file.txt | uniq        # Loại bỏ trùng thực sự
sort file.txt | uniq -c     # Đếm số lần xuất hiện
sort file.txt | uniq -d     # Chỉ hiện dòng bị trùng
sort file.txt | uniq -u     # Chỉ hiện dòng KHÔNG trùng
uniq -c | sort -rn          # Top xuất hiện nhiều nhất
```

### Ví dụ kinh điển — đếm truy cập theo IP

```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn | head -10
```

---

## 4. `wc` — Đếm dòng/từ/ký tự

```bash
wc file.txt            # In: dòng  từ  byte  tên file
wc -l file.txt         # Đếm dòng
wc -w file.txt         # Đếm từ
wc -c file.txt         # Đếm byte
wc -m file.txt         # Đếm ký tự (chính xác với Unicode)
```

### Ví dụ

```bash
# Đếm số file trong thư mục
ls | wc -l

# Đếm số user hệ thống
wc -l < /etc/passwd

# Đếm số lần lỗi trong log
grep -c ERROR app.log          # grep -c tự đếm
grep ERROR app.log | wc -l     # Hoặc qua pipe

# Đếm tổng dòng code
find src -name "*.py" -exec cat {} + | wc -l
```

---

## 5. `tr` — Thay thế/xóa ký tự

`tr` làm việc với **tập ký tự**, đọc từ stdin:

```bash
# Chuyển chữ hoa → thường
echo "HELLO" | tr 'A-Z' 'a-z'

# Nén nhiều khoảng trắng thành 1
echo "a    b   c" | tr -s ' '

# Xóa ký tự
echo "a1b2c3" | tr -d '0-9'      # abc

# Thay thế
echo "hello" | tr 'l' 'L'        # heLLo
```

### Ví dụ thực tế

```bash
# Chuẩn hóa delimiter: chuyển tab thành phẩy
cat data.tsv | tr '\t' ',' > data.csv

# Xóa ký tự CR (fix lỗi file từ Windows)
cat windows.txt | tr -d '\r' > unix.txt

# Mã hóa đơn giản (ROT13)
echo "hello" | tr 'a-zA-Z' 'n-za-mN-ZA-M'
```

---

## 6. `sed` — Stream editor theo pattern

`sed` chỉnh sửa text theo từng dòng. Cú pháp phổ biến nhất là thay thế:

```bash
sed 's/cũ/mới/' file.txt          # Thay lần đầu trên mỗi dòng
sed 's/cũ/mới/g' file.txt         # Thay TẤT CẢ trên mỗi dòng
sed -i 's/cũ/mới/g' file.txt      # Ghi đè trực tiếp vào file
sed -i.bak 's/cũ/mới/g' file.txt  # Ghi đè + lưu bản backup
```

### Các lệnh sed thông dụng

```bash
# Thay thế (s = substitute)
sed 's/nginx/apache/' config.txt
sed -i 's/port 80/port 8080/' nginx.conf
sed -i 's|/old/path|/new/path|g' file   # Dùng | khi pattern có dấu /

# Xóa dòng
sed '/pattern/d' file.txt               # Xóa dòng chứa pattern
sed '/^#/d' /etc/ssh/sshd_config        # Xóa dòng comment
sed '/^$/d' file.txt                    # Xóa dòng trống
sed '5d' file.txt                       # Xóa dòng 5
sed '10,20d' file.txt                   # Xóa dòng 10-20

# In / chọn dòng
sed -n '5p' file.txt                    # In dòng 5
sed -n '10,20p' file.txt                # In dòng 10-20
sed -n '/ERROR/p' app.log               # In dòng chứa ERROR

# Chèn / thêm
sed '5i Dòng mới chèn trên dòng 5' file.txt
sed '5a Dòng mới chèn sau dòng 5' file.txt
sed '$a Dòng cuối file' file.txt

# Sửa trên nhiều file
sed -i 's/localhost/0.0.0.0/g' *.conf

# Regex
echo "2026-10-27" | sed -E 's/([0-9]{4})-([0-9]{2})-([0-9]{2})/\2\/\3\/\1/'
# Kết quả: 10/27/2026
```

> ⚠️ **Lưu ý:** `sed -i` sửa file **ngay lập tức, không hoàn tác**. Hãy luôn dùng `sed -i.bak` để tự động tạo bản backup, hoặc thử trước không có `-i` để xem kết quả sẽ như thế nào.

> 💡 macOS dùng phiên bản BSD sed — cần `sed -i ''` thay vì `sed -i`. Trên Ubuntu thì `sed -i` hoạt động bình thường.

---

## 7. `awk` — Ngôn ngữ xử lý văn bản mạnh mẽ

`awk` xử lý text theo **cột (field)** — `$1` là cột 1, `$2` cột 2, `$NF` cột cuối, `$0` cả dòng:

```bash
# Cú pháp: awk 'BEGIN{...} /pattern/ {actions} END{...}' file

# In cột
awk '{print $1}' file.txt
ps aux | awk '{print $2, $11}'

# Cột cuối
awk '{print $NF}' file.txt

# Lọc theo điều kiện
awk '$3 > 100 {print $1}' file.txt
awk '/ERROR/ {print $1}' app.log

# delimiter tùy chỉnh
awk -F: '{print $1}' /etc/passwd         # tương đương cut -d: -f1
awk -F',' '{print $1, $3}' data.csv
```

### Nhiều khả năng hơn

```bash
# In có nhãn
awk '{print "User:", $1}' /etc/passwd

# Đếm dòng
awk 'END{print NR}' file.txt

# Tính tổng cột
awk '{sum += $1} END {print sum}' numbers.txt

# Đếm số dòng có ERROR
awk '/ERROR/{count++} END{print count}' app.log

# Đổi dấu phân cách đầu ra
awk -F: -v OFS=' | ' '{print $1, $3}' /etc/passwd

# Điều kiện if
awk -F: '{if ($3 >= 1000) print $1 " là user thường"}' /etc/passwd

# In dòng với số thứ tự
awk '{print NR": "$0}' file.txt

# Bỏ dòng đầu (header CSV)
awk 'NR>1' data.csv

# Xử lý nhiều file
awk '{total[FNR] += $1} END {for (i=1; i<=FNR; i++) print total[i]}' f1.txt f2.txt
```

### sed vs awk — chọn khi nào?

| Tình huống | Công cụ |
|---|---|
| Thay thế chuỗi đơn giản | `sed` |
| Xóa/chọn dòng theo pattern | `sed` |
| Làm việc theo cột, tính toán | `awk` |
| Đếm, tổng hợp, thống kê | `awk` |
| Biến đổi phức tạp nhiều dòng | `awk` hoặc Python |

---

## 8. Chuỗi pipe phức tạp thực tế

### Phân tích access log Nginx

```bash
# Top 10 IP truy cập nhiều nhất
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Top 10 URL được truy cập
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -10

# Số request mỗi giờ
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c

# Request gây lỗi 500
awk '$9 == 500 {print $7}' access.log | sort | uniq -c | sort -rn
```

### Phân tích hệ thống

```bash
# 5 process chiếm RAM nhất
ps aux | awk 'NR>1 {print $6/1024" MB", $11}' | sort -rn | head -5

# Tổng dung lượng các file log
find /var/log -name "*.log" -exec du -ch {} + | tail -1

# Đếm tổng dung lượng theo loại file
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Liệt kê users shell /bin/bash
grep bash /etc/passwd | cut -d: -f1
```

### Biến đổi dữ liệu CSV

```bash
# Thêm cột mới vào cuối mỗi dòng
awk -F',' '{print $0 ",processed"}' data.csv > data_processed.csv

# Đổi thứ tự cột 2 và 3
awk -F',' 'BEGIN{OFS=","} {tmp=$2; $2=$3; $3=tmp; print}' data.csv

# Lọc dòng có giá trị cột 3 lớn hơn 100
awk -F',' '$3 > 100' data.csv
```

---

## 9. Mẹo và lưu ý

### Dùng `less` để kiểm tra trước khi biến đổi

```bash
less data.csv             # Xem cấu trúc
head -3 data.csv          # Xem vài dòng đầu
awk -F',' 'NR==1{print NF}' data.csv    # Số cột
```

### Kết hợp với `xargs`

```bash
# Xóa các dòng chứa pattern trong nhiều file
grep -rl "old_domain" . --include="*.conf" | xargs sed -i 's/old_domain/new_domain/g'
```

### Debug chuỗi pipe từng bước

```bash
# Chạy từng lệnh một, xem output trước khi thêm lệnh tiếp theo
cat app.log
cat app.log | grep ERROR
cat app.log | grep ERROR | cut -d' ' -f5
# ...từng bước như vậy để tìm ra lỗi ở đâu
```

### Hiệu năng với file lớn

```bash
# Với file rất lớn (GB), grep/awk đơn luồng nhanh hơn; nếu cần tốc độ tối đa:
apt install parallel
cat big.log | parallel --pipe grep ERROR
```

---

## Nguồn tham khảo

- GNU sed manual: https://www.gnu.org/software/sed/manual/sed.html
- GNU awk manual: https://www.gnu.org/software/gawk/manual/gawk.html
- GNU Coreutils (cut, sort, uniq, wc, tr): https://www.gnu.org/software/coreutils/manual/
