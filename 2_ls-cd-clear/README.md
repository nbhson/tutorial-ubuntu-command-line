# ls, cd, clear — Điều hướng và hiển thị thư mục

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · Shell: bash/zsh
> **Đối tượng:** Người mới bắt đầu sử dụng terminal

---

## Mục lục

1. [Giải thích dấu nhắc terminal (prompt)](#1-giải-thích-dấu-nhắc-terminal-prompt)
2. [Lệnh `ls` — Liệt kê file/thư mục](#2-lệnh-ls--liệt-kê-filethư-mục)
3. [Các cờ thông dụng của `ls`](#3-các-cờ-thông-dụng-của-ls)
4. [Lệnh `cd` — Di chuyển giữa thư mục](#4-lệnh-cd--di-chuyển-giữa-thư-mục)
5. [Các shortcut thư mục đặc biệt](#5-các-shortcut-thư-mục-đặc-biệt)
6. [Lệnh `clear` và các phím tắt thay thế](#6-lệnh-clear-và-các-phím-tắt-thay-thế)
7. [Đường dẫn tuyệt đối và tương đối](#7-đường-dẫn-tuyệt-đối-và-tương-đối)
8. [Thao tác với path thông dụng](#8-thao-tác-với-path-thông-dụng)
9. [Mẹo và tricks](#9-mẹo-và-tricks)

---

## 1. Giải thích dấu nhắc terminal (prompt)

Dòng lệnh terminal thường trông như thế này:

```text
nbhson@srv-01:~/projects/tutorial$
```

Hãy phân tích từng phần:

| Phần | Ý nghĩa | Ví dụ |
|---|---|---|
| `nbhson` | Tên người dùng hiện tại | Tài khoản đang đăng nhập |
| `@` | Dấu phân cách | username@hostname |
| `srv-01` | Hostname — tên máy | Trên desktop là tên máy tính; trên cloud/server là tên server hoặc IP |
| `:` | Dấu phân cách giữa hostname và path | — |
| `~/projects/tutorial` | Thư mục hiện tại | `~` = thư mục home của user |
| `$` | Dấu kết thúc prompt | `$` = user thường; `#` = root |

### Các ký hiệu path đặc biệt

| Ký hiệu | Ý nghĩa |
|---|---|
| `~` | Thư mục home của user hiện tại (`/home/nbhson`) |
| `/` | Thư mục root — đỉnh cao nhất của cây thư mục |
| `.` | Thư mục hiện tại |
| `..` | Thư mục cha (parent) của thư mục hiện tại |
| `-` | Thư mục trước đó (cd -) |

---

## 2. Lệnh `ls` — Liệt kê file/thư mục

```bash
# Liệt kê nội dung thư mục hiện tại
ls

# Liệt kê thư mục cụ thể
ls /var/log

# Liệt kê nhiều thư mục cùng lúc
ls /home /var/log
```

### Mặc định vs chi tiết

```bash
ls           # Chỉ hiển thị tên file
ls -l        # Dạng danh sách dài, gồm: quyền, owner, size, ngày chỉnh sửa
```

Output `ls -l` ví dụ:

```text
-rw-r--r--  1 nbhson nbhson 4096 Oct 27 10:30 README.md
drwxr-xr-x  2 nbhson nbhson 4096 Oct 27 09:00 src
lrwxrwxrwx  1 nbhson nbhson   18 Oct 27 08:00 link -> /usr/share/...
```

---

## 3. Các cờ thông dụng của `ls`

| Cờ | Tên đầy đủ | Chức năng |
|---|---|---|
| `ls -l` | long | Danh sách chi tiết (permissions, owner, size, date) |
| `ls -a` | all | Bao gồm file ẩn (bắt đầu bằng `.`) |
| `ls -A` | almost-all | Giống `-a` nhưng bỏ `.` và `..` |
| `ls -la` | long + all | Kết hợp — danh sách chi tiết + file ẩn |
| `ls -lh` | human-readable | Hiển thị size dễ đọc (KB, MB, GB) |
| `ls -lS` | sort by size | Sắp xếp theo dung lượng giảm dần |
| `ls -lt` | sort by time | Sắp xếp theo thời gian chỉnh sửa mới nhất |
| `ls -ltr` | reverse + time | Mới nhất ở cuối (xem file thay đổi gần đây) |
| `ls -R` | recursive | Liệt kê đệ quy tất cả thư mục con |
| `ls -1` | numeric | Mỗi item một dòng |
| `ls -d */` | directory only | Chỉ liệt kê thư mục |
| `ls --color=auto` | color | Tô màu phân biệt file/thư mục/link |

### Ví dụ thực tế

```bash
# Xem tất cả file (kể cả ẩn) với size dễ đọc
ls -lha

# Xem 5 file/thư mục lớn nhất trong thư mục hiện tại
ls -lSh | head -6

# Xem 10 file mới chỉnh sửa gần nhất
ls -lt | head -11

# Đệ quy — xem toàn bộ cây thư mục
ls -R

# Chỉ xem thư mục con (không file)
ls -d */

# Xem thông tin thư mục mà không liệt kê nội dung bên trong
ls -ld /var/log
```

---

## 4. Lệnh `cd` — Di chuyển giữa thư mục

```bash
cd /usr/local          # Đi tới thư mục tuyệt đối
cd projects            # Đi tới thư mục con tên projects
cd ..                  # Lên thư mục cha
cd ../src              # Lên cha rồi vào src
cd ~/Documents         # Vào thư mục Documents trong home
```

### Các shortcut phổ biến

```bash
cd            # Quay về home
cd ~          # Quay về home
cd ~username  # Tới home của user khác
cd -          # Quay lại thư mục trước đó (toggle)
cd ..         # Lên một cấp
cd ../..      # Lên hai cấp
```

> 💡 **Mẹo:** `cd -` rất tiện khi bạn đang ở giữa hai thư mục và cần nhảy qua lại nhanh.

```bash
# Ví dụ thực tế:
/etc/apt/sources.list.d$ cd -
# Kết quả: /home/nbhson
~# cd -
# Quay lại: /etc/apt/sources.list.d
```

---

## 5. Các shortcut thư mục đặc biệt

### `.` — Thư mục hiện tại

```bash
# Chạy script trong thư mục hiện tại
./run.sh

# Xem thông tin file trong thư mục hiện tại
cat ./config.txt
```

### `..` — Thư mục cha

```bash
# Xem thư mục cha
ls ..

# Tạo file cạnh thư mục hiện tại
touch ../note.txt

# Lên 2 cấp
cd ../..
```

### `~` — Home directory

```bash
cd ~              # Tương đương cd không tham số
mkdir ~/backup    # Tạo thư mục backup trong home
```

### Tilde expansion trong bash

```bash
echo ~                    # In ra /home/nbhson
echo ~root                # In ra /root
echo ~nobody              # In ra /nonexistent
```

---

## 6. Lệnh `clear` và các phím tắt thay thế

```bash
clear          # Xóa toàn bộ màn hình (cuộn về trạng thái sạch)
Ctrl + L       # Phím tắt tương đương clear
```

> 💡 Về kỹ thuật, `clear` chỉ cuộn terminal xuống phần trống, lịch sử lệnh vẫn còn trong buffer và cuộn lên vẫn thấy. `Ctrl+L` làm y hệt.

```bash
# Reset hoàn toàn terminal (reset terminal về trạng thái mặc định)
reset
```

---

## 7. Đường dẫn tuyệt đối và tương đối

### Đường dẫn tuyệt đối (absolute path)

Bắt đầu bằng `/` — cho biết vị trí chính xác từ root:

```bash
# Luôn đúng bất kể thư mục hiện tại ở đâu
ls /var/log/nginx
cat /etc/apt/sources.list.d/ubuntu.sources
```

### Đường dẫn tương đối (relative path)

Không bắt đầu bằng `/` — tính từ thư mục hiện tại:

```bash
# Đang ở /home/nbhson
ls projects/tutorial    # Tương đương /home/nbhson/projects/tutorial
cd ../var               # Tương đương /home/var
cd ../../etc            # Tương đương /etc
```

---

## 8. Thao tác với path thông dụng

### Các thư mục hệ thống quan trọng

| Path | Nội dung |
|---|---|
| `/` | Root — đỉnh cây thư mục |
| `/home/username` | Thư mục home của user |
| `/etc` | File cấu hình hệ thống |
| `/var` | Dữ liệu thay đổi (log, cache, web) |
| `/usr` | Ứng dụng và thư viện (cài đặt bởi apt) |
| `/usr/local` | Ứng dụng tự cài (không qua apt) |
| `/tmp` | File tạm — xóa khi khởi động lại |
| `/opt` | Phần mềm thương mại/third-party |
| `/root` | Home của root user |
| `/proc` | Thông tin kernel/process (không phải file thật) |

```bash
# Kiểm tra home của user hiện tại
echo $HOME
echo ~

# Di chuyển vào thư mục home
cd $HOME
cd ~
```

---

## 9. Mẹo và tricks

### Dùng tab completion

```bash
cd /var/lo[TAB][TAB]   # Bash sẽ gợi ý /var/log/
cd ~/[pro[TAB]          # Gợi tên thư mục bắt đầu bằng "pro"
```

### Cờ kết hợp thông minh

```bash
# Liệt kê chi tiết + đệ quy + màu sắc
ls -lhR --color

# Xem 10 file mới nhất (thường dùng để xem log mới)
ls -lt | head -11

# Tổng dung lượng thư mục hiện tại
du -sh .
```

### Kiểm tra thư mục hiện tại

```bash
pwd        # In đường dẫn tuyệt đối (print working directory)
pwd -P     # Loại bỏ symlink (hiển thị path thật)
```

### Autojump / z (nếu cài thêm)

```bash
# Nếu cài z (apt install zsh-autosuggestions / zoxide):
z projects        # Nhảy tới thư mục thường dùng gần đây nhất
```

### Alias tiện lợi thường thêm vào `~/.bashrc`

```bash
# ~/.bashrc
alias ll='ls -lh'       # Danh sách dài, size dễ đọc
alias la='ls -lha'      # Bao gồm file ẩn
alias l.='ls -d .*'     # Chỉ file ẩn
alias cd..='cd ..'      # Gõ cd.. thay vì cd ..
alias ..='cd ..'
alias ...='cd ../..'
```

---

## Nguồn tham khảo

- GNU Coreutils ls documentation: https://www.gnu.org/software/coreutils/manual/html_node/ls-invocation.html
- Bash manual: https://www.gnu.org/software/bash/manual/
