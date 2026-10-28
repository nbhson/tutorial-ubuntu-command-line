# File Permissions — Phân quyền trên Linux (chmod, chown)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS
> **Đối tượng:** Người dùng muốn hiểu sâu về hệ thống phân quyền Linux

---

## Mục lục

1. [Khái niệm quyền cơ bản](#1-khái-niệm-quyền-cơ-bản)
2. [Đọc hiểu `ls -l`](#2-đọc-hiểu-ls--l)
3. [Chế độ symbolic — `u g o a` và `rwx`](#3-chế-độ-symbolic--u-g-o-a-và-rwx)
4. [Chế độ numeric — 755, 644, 700...](#4-chế-độ-numeric--755-644-700)
5. [`chmod` — Thay đổi quyền](#5-chmod--thay-đổi-quyền)
6. [`chown` và `chgrp` — Thay đổi chủ sở hữu](#6-chown-và-chgrp--thay-đổi-chủ-sở-hữu)
7. [Special permissions: setuid, setgid, sticky bit](#7-special-permissions-setuid-setgid-sticky-bit)
8. [`umask` — Quyền mặc định khi tạo file](#8-umask--quyền-mặc-định-khi-tạo-file)
9. [ACL — Phân quyền nâng cao](#9-acl--phân-quyền-nâng-cao)
10. [Use-cases thực tế](#10-use-cases-thực-tế)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Khái niệm quyền cơ bản

Mỗi file/thư mục trên Linux có **3 nhóm đối tượng** và **3 loại quyền**:

### Ba nhóm đối tượng

| Ký hiệu | Nhóm | Ý nghĩa |
|---|---|---|
| `u` | user (owner) | Chủ sở hữu file |
| `g` | group | Nhóm sở hữu file |
| `o` | others | Tất cả người dùng khác |

### Ba loại quyền

| Ký hiệu | Giá trị | Trên file | Trên thư mục |
|---|---|---|---|
| `r` | 4 | Đọc nội dung file | Liệt kê nội dung (`ls`) |
| `w` | 2 | Ghi/sửa file | Tạo/xóa file bên trong |
| `x` | 1 | Thực thi file (chạy) | Vào được thư mục (`cd`) |
| `-` | 0 | Không có quyền | Không có quyền |

> ⚠️ **Lưu ý phổ biến:** Với thư mục, quyền `r` không đủ để `cd` vào bên trong — bạn cần cả `x`. Thiếu `x` bạn sẽ gặp `Permission denied` khi truy cập.

---

## 2. Đọc hiểu `ls -l`

```bash
ls -l
```

```text
-rwxr-xr--  1 nbhson developers 4096 Oct 27 10:30 deploy.sh
│└┬┘└┬┘└┬┘     └──┬──┘ └───┬───┘            └──┬───┘
│ u  g  o      owner  group                        tên file
│
└─ loại file: - = file thường, d = thư mục, l = symlink
```

Phân tích chuỗi quyền `-rwxr-xr--`:

- Vị trí 1: `-` là file thường (`d` = directory, `l` = symlink, `c`/`b` = device)
- Vị trí 2–4: `rwx` — owner đọc, ghi, thực thi
- Vị trí 5–7: `r-x` — group đọc, thực thi (không ghi)
- Vị trí 8–10: `r--` — others chỉ đọc

---

## 3. Chế độ symbolic — `u g o a` và `rwx`

Cú pháp symbolic:

```text
chmod [who][operator][permission] file
      who: u g o a   operator: + - =   permission: r w x
```

```bash
# Thêm quyền thực thi cho owner
chmod u+x script.sh

# Gỡ quyền ghi của group và others
chmod go-w file.txt

# Đặt chính xác: owner đọc+ghi, group/others chỉ đọc
chmod u=rw,go=r file.txt

# a = all (mọi người)
chmod a+r file.txt       # Ai cũng đọc được
```

---

## 4. Chế độ numeric — 755, 644, 700...

Mỗi nhóm quyền được tính bằng tổng: `r=4, w=2, x=1`.

```text
rwx r-x r--
 7   5   4   → 754
```

### Bảng quyền numeric phổ biến

| Số | Ký hiệu | Dùng cho |
|---|---|---|
| `777` | rwxrwxrwx | ⚠️ Ai cũng làm mọi thứ — gần như không bao giờ nên dùng |
| `775` | rwxrwxr-x | Thư mục chia sẻ cho nhóm |
| `755` | rwxr-xr-x | Thư mục chuẩn, script thực thi |
| `700` | rwx------ | Thư mục riêng tư (SSH keys) |
| `664` | rw-rw-r-- | File chia sẻ cho nhóm |
| `644` | rw-r--r-- | File cấu hình/HTML chuẩn |
| `600` | rw------- | File riêng tư (private key) |
| `400` | r-------- | Chỉ đọc (private key SSH khuyến nghị) |

---

## 5. `chmod` — Thay đổi quyền

### Cú pháp

```bash
chmod [options] mode file
```

### Ví dụ

```bash
# Numeric
chmod 755 deploy.sh          # owner: rwx, group/others: r-x
chmod 644 index.html         # owner: rw, group/others: r
chmod 600 ~/.ssh/id_ed25519  # chỉ owner đọc/ghi

# Symbolic
chmod +x script.sh           # thêm x cho tất cả (tương đương a+x)
chmod u+x,g-w file.txt       # owner thêm x, group gỡ w
chmod -R 755 /var/www        # đệ quy toàn bộ thư mục

# Tham chiếu (copy quyền từ nhóm khác)
chmod g=u file.txt           # group được quyền giống owner
chmod o=u file.txt           # others giống owner
```

### Cờ đệ quy

```bash
chmod -R 755 /var/www/my-site
```

> ⚠️ **Bẫy phổ biến:** `chmod -R 777` trên thư mục web là lỗ hổng bảo mật. Nếu web cần ghi vào thư mục upload, chỉ mở quyền ghi cho **thư mục đó**:
>
> ```bash
> chmod -R 755 /var/www/my-site
> chown -R www-data:www-data /var/www/my-site/uploads
> chmod -R 775 /var/www/my-site/uploads
> ```

---

## 6. `chown` và `chgrp` — Thay đổi chủ sở hữu

```bash
# Đổi owner
sudo chown nginx file.txt

# Đổi owner + group (dạng user:group)
sudo chown nginx:nginx /var/log/nginx

# Đổi đệ quy toàn bộ thư mục
sudo chown -R www-data:www-data /var/www/my-site

# Chỉ đổi group (giữ nguyên owner)
sudo chown :developers project/          # hoặc:
sudo chgrp developers project/

# Đổi group theo group của file khác
sudo chgrp --reference=ref.txt target.txt
```

> 💡 `chown` luôn cần `sudo` (chỉ root đổi được chủ sở hữu). `chgrp` cho phép user đổi group **mà họ là thành viên**.

Xem nhóm hiện tại của bạn:

```bash
id                      # uid, gid, groups của user hiện tại
groups                  # danh sách group
groups username         # group của user khác
```

---

## 7. Special permissions: setuid, setgid, sticky bit

Ngoài `rwx`, còn 3 quyền đặc biệt:

| Quyền | Ký hiệu ls | Numeric | Ý nghĩa |
|---|---|---|---|
| setuid | `s` tại chỗ `x` của owner (`-rwsr-xr-x`) | 4000 | Chạy file với quyền **owner** của file |
| setgid | `s` tại chỗ `x` của group (`-rwxr-sr-x`) | 2000 | File: chạy với quyền **group**. Thư mục: file mới bên trong **kế thừa group của thư mục** |
| sticky bit | `t` tại chỗ `x` của others (`drwxrwxrwt`) | 1000 | Trong thư mục: chỉ **owner của file** mới được xóa file đó |

### Ví dụ thực tế

```bash
# setuid — ví dụ kinh điển: passwd
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
# → user thường chạy passwd được vì nó chạy với quyền root để sửa /etc/shadow

# sticky bit — ví dụ kinh điển: /tmp
ls -ld /tmp
# drwxrwxrwt ... /tmp
# → ai cũng ghi được /tmp nhưng không xóa file của người khác

# setgid — thư mục làm việc nhóm
sudo mkdir /var/share/team
sudo chgrp developers /var/share/team
sudo chmod 2775 /var/share/team
# → mọi file tạo trong /var/share/team sẽ thuộc group "developers"

# Đặt bằng numeric (4 chữ số: setuid/setgid/sticky + 3 nhóm)
chmod 4755 program       # setuid + 755
chmod 2775 shared_dir    # setgid + 775
chmod 1777 tmp_dir       # sticky + 777

# Đặt bằng symbolic
chmod u+s program        # setuid
chmod g+s shared_dir     # setgid
chmod +t tmp_dir         # sticky bit
```

> ⚠️ **Bảo mật:** Tránh đặt setuid cho script shell — phần lớn hệ thống bỏ qua setuid trên script vì rủi ro. Chỉ dùng cho binary khi thực sự cần.

---

## 8. `umask` — Quyền mặc định khi tạo file

`umask` xác định quyền nào bị **loại bỏ** khi tạo file/thư mục mới:

```bash
umask                    # Thường in ra 0022
umask 077                # Đặt umask mới (chỉ session hiện tại)
```

Tính toán: quyền mới = quyền gốc **trừ** umask

| | File gốc | Thư mục gốc | Umask 022 → kết quả | Umask 077 → kết quả |
|---|---|---|---|---|
| File | 666 | — | 644 (rw-r--r--) | 600 (rw-------) |
| Thư mục | — | 777 | 755 (rwxr-xr-x) | 700 (rwx------) |

> 💡 Muốn umask mặc định cho mọi phiên: thêm dòng `umask 022` (hoặc `027`, `077` nếu chặt hơn) vào `~/.profile` hoặc `~/.bashrc`.

---

## 9. ACL — Phân quyền nâng cao

Khi cần cho phép **một user cụ thể** truy cập file mà không đổi owner/group:

```bash
# Cài công cụ (thường có sẵn)
sudo apt install acl

# Cho user "bob" đọc+ghi file
setfacl -m u:bob:rw document.txt

# Cho group "devs" đọc thư mục
setfacl -m g:devs:rx project/

# Đặt ACL mặc định cho thư mục (áp dụng cho file mới)
setfacl -d -m u:bob:rw project/

# Xem ACL
getfacl document.txt

# Xóa ACL của user
setfacl -x u:bob document.txt

# Xóa toàn bộ ACL
setfacl -b document.txt
```

Khi có ACL, `ls -l` sẽ hiển thị dấu `+` ở cuối chuỗi quyền: `-rw-rw-r--+`.

---

## 10. Use-cases thực tế

### Khóa private SSH phải đúng quyền

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

> ⚠️ SSH sẽ **từ chối dùng key** nếu quyền quá rộng, với lỗi: `Permissions 0644 for 'id_ed25519' are too open`.

### Web server chuẩn quyền

```bash
sudo chown -R www-data:www-data /var/www/my-site
sudo find /var/www/my-site -type d -exec chmod 755 {} \;
sudo find /var/www/my-site -type f -exec chmod 644 {} \;
```

### Script chạy được

```bash
chmod +x backup.sh
./backup.sh
```

### Thư mục chia sẻ nhóm an toàn

```bash
sudo groupadd devs
sudo usermod -aG devs alice
sudo mkdir -p /srv/shared
sudo chgrp devs /srv/shared
sudo chmod 2775 /srv/shared        # setgid để file mới kế thừa group
```

---

## 11. Troubleshooting

### `Permission denied` khi truy cập

```bash
# 1. Xem quyền hiện tại
ls -l file.txt
ls -ld thu_muc/

# 2. Xem mình là ai, thuộc nhóm nào
id

# 3. Sửa quyền hoặc dùng sudo
sudo chmod 644 file.txt
```

### `cd` vào thư mục bị ch dù `ls` được

Quyền `r` cho phép `ls`, nhưng thiếu `x` thì không `cd` được. Thêm `x`:

```bash
chmod u+x thu_muc/
```

### File không thực thi được dù đã `chmod +x`

```bash
# Kiểm tra shebang ở dòng đầu
head -1 script.sh       # Phải có: #!/bin/bash hoặc #!/usr/bin/env bash

# File từ Windows có ký tự CRLF
cat -A script.sh | head -1    # Nếu thấy ^M$ → lỗi dòng Windows
sudo apt install dos2unix
dos2unix script.sh
```

### Không xóa được file trong thư mục mình có quyền ghi

File thuộc thư mục có **sticky bit** hoặc file có thuộc tính bất biến:

```bash
lsattr file.txt        # Nếu thấy chữ i → immutable
sudo chattr -i file.txt
```

---

## Nguồn tham khảo

- Ubuntu Server Guide — File permissions: https://documentation.ubuntu.com/server/
- `man chmod`, `man chown`, `man setfacl`
- Arch Wiki (chi tiết rất tốt, áp dụng cho cả Ubuntu): https://wiki.archlinux.org/title/Permissions
