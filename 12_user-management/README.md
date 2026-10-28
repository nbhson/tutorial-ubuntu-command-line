# User Management — Quản lý người dùng (useradd, sudo, groups, /etc/passwd)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS
> **Đối tượng:** Admin cần quản lý user, group và quyền sudo

---

## Mục lục

1. [Cấu trúc user trên Linux](#1-cấu-trúc-user-trên-linux)
2. [`/etc/passwd`, `/etc/shadow`, `/etc/group`](#2-etcpasswd-etcshadow-etcgroup)
3. [`useradd` / `adduser` — Tạo user](#3-useradd--adduser--tạo-user)
4. [`usermod` — Sửa user](#4-usermod--sửa-user)
5. [`passwd` — Đặt/thay đổi mật khẩu](#5-passwd--đặtthay-đổi-mật-khẩu)
6. [Xóa và khóa user](#6-xóa-và-khóa-user)
7. [Groups — Nhóm người dùng](#7-groups--nhóm-người-dùng)
8. [`sudo` — Phân quyền quản trị](#8-sudo--phân-quyền-quản-trị)
9. [`who`, `w`, `last` — Xem ai đang đăng nhập](#9-who-w-last--xem-ai-đang-đăng-nhập)
10. [Use-cases thực tế](#10-use-cases-thực-tế)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Cấu trúc user trên Linux

Mỗi user được định danh bằng:

- **Username** — tên đăng nhập (vd: `nbhson`)
- **UID** — số định danh user. Quy ước:
  - `0` = root
  - `1–999` = user hệ thống (service, daemon)
  - `≥ 1000` = user thường
- **GID** — ID nhóm chính
- **Home directory** — thư mục riêng (`/home/nbhson`)
- **Login shell** — shell khi đăng nhập (`/bin/bash`, `/usr/sbin/nologin`...)

### Root vs user thường vs sudo

| | Root (UID 0) | User thường | User thuộc nhóm sudo |
|---|---|---|---|
| Quyền | Toàn quyền | Bị giới hạn | Gần như root, qua `sudo` |
| Prompt | `#` | `$` | `$` |
| Đăng nhập trực tiếp SSH | Nên tắt | ✅ | ✅ |

---

## 2. `/etc/passwd`, `/etc/shadow`, `/etc/group`

### `/etc/passwd` — Thông tin user

```bash
cat /etc/passwd
```

```text
root:x:0:0:root:/root:/bin/bash
nbhson:x:1000:1000:Nguyen Son,,,:/home/nbhson:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

Định dạng 7 trường cách nhau bằng `:`:

| # | Trường | Ví dụ |
|---|---|---|
| 1 | Username | `nbhson` |
| 2 | Mật khẩu (x = trong /etc/shadow) | `x` |
| 3 | UID | `1000` |
| 4 | GID (nhóm chính) | `1000` |
| 5 | GECOS (tên đầy đủ, thông tin) | `Nguyen Son,,,` |
| 6 | Home directory | `/home/nbhson` |
| 7 | Login shell | `/bin/bash` |

> 💡 User hệ thống có shell là `/usr/sbin/nologin` — không thể đăng nhập, chỉ dùng để chạy service.

### `/etc/shadow` — Mật khẩu đã hash (cần sudo)

```bash
sudo cat /etc/shadow | grep nbhson
# nbhson:$6$rounds=...:20283:0:99999:7:::
```

Trường 3 = ngày đổi mật khẩu gần nhất (số ngày từ 01/01/1970). Trường 5 = tuổi tối đa của mật khẩu.

### `/etc/group` — Nhóm

```bash
cat /etc/group
# sudo:x:27:nbhson,docker-admin
```

Định dạng: `group_name:password:GID:user_list`

```bash
groups                 # Nhóm của user hiện tại
groups username        # Nhóm của user khác
id                     # UID, GID và mọi nhóm
id username
```

---

## 3. `useradd` / `adduser` — Tạo user

### `adduser` — Đơn giản, tương tác (khuyến nghị)

```bash
sudo adduser tenuser
```

Sẽ hỏi mật khẩu, tên đầy đủ, thông tin GECOS... và tự tạo home directory, copy file `/etc/skel` (`.bashrc`, `.profile`...).

```bash
sudo adduser --no-create-home tenuser    # Không tạo home
sudo adduser --disabled-password tenuser # Không mật khẩu (dùng SSH key)
sudo adduser --home /opt/tenuser tenuser # Home tùy chỉnh
```

### `useradd` — Cấp thấp, không tương tác

```bash
# Cú pháp đầy đủ
sudo useradd -m -s /bin/bash -G sudo,docker -c "Nguyen Son" tenuser
```

| Cờ | Ý nghĩa |
|---|---|
| `-m` | Tạo home directory |
| `-s /bin/bash` | Login shell |
| `-G group1,group2` | Các nhóm bổ sung |
| `-c "Comment"` | Thông tin GECOS |
| `-d /custom/path` | Home tùy chỉnh |
| `-e 2026-12-31` | Ngày hết hạn tài khoản |
| `-r` | Tạo user hệ thống |

> 💡 Trên Ubuntu, `adduser` là wrapper thân thiện của `useradd` — người mới nên dùng `adduser`. `useradd` phù hợp trong script tự động hóa.

---

## 4. `usermod` — Sửa user

```bash
# Thêm user vào nhóm (cần -aG: append + groups)
sudo usermod -aG docker nbhson
sudo usermod -aG sudo nbhson

# Đổi username
sudo usermod -l tenmoi tencu

# Đổi home directory (và di chuyển nội dung)
sudo usermod -d /new/home -m nbhson

# Đổi shell
sudo usermod -s /usr/bin/zsh nbhson

# Đặt ngày hết hạn
sudo usermod -e 2026-12-31 nbhson

# Khóa / mở khóa mật khẩu
sudo usermod -L nbhson    # Khóa (Lock)
sudo usermod -U nbhson    # Mở khóa (Unlock)
```

> ⚠️ **Quên `-a` (append) là mất nhóm!** `usermod -G docker nbhson` (không có `-a`) sẽ **thay thế toàn bộ** các nhóm bổ sung bằng `docker`. Luôn dùng `-aG`.

---

## 5. `passwd` — Đặt/thay đổi mật khẩu

```bash
# Đổi mật khẩu của chính mình
passwd

# Root đặt mật khẩu cho user khác
sudo passwd tenuser

# Xóa mật khẩu (user đăng nhập không cần mật khẩu — nguy hiểm!)
sudo passwd -d tenuser

# Xem trạng thái
sudo passwd -S tenuser
# nbhson P 10/27/2026 0 99999 7 -1
# P = có mật khẩu, L = locked, NP = không mật khẩu

# Ép user đổi mật khẩu lần đăng nhập tiếp theo
sudo passwd -e tenuser
```

### Chính sách mật khẩu qua `chage`

```bash
sudo chage -l nbhson                     # Xem chính sách
sudo chage -M 90 nbhson                  # Đổi mật khẩu tối đa 90 ngày
sudo chage -d 0 nbhson                   # Ép đổi ngay lần đăng nhập kế
```

---

## 6. Xóa và khóa user

```bash
# Xóa user (giữ home directory)
sudo deluser tenuser

# Xóa user + home directory
sudo deluser --remove-home tenuser
# hoặc:
sudo userdel -r tenuser

# Khóa tạm thời (ngăn đăng nhập, không xóa dữ liệu)
sudo usermod -L tenuser
sudo passwd -l tenuser

# Mở khóa
sudo usermod -U tenuser

# Đánh dấu shell không đăng nhập (khóa khác)
sudo usermod -s /usr/sbin/nologin tenuser
```

> ⚠️ **Không xóa user khi đang có tiến trình chạy** — có thể làm hỏng dữ liệu. Kiểm tra bằng `ps -u tenuser` trước.

---

## 7. Groups — Nhóm người dùng

```bash
# Tạo / xóa nhóm
sudo groupadd developers
sudo groupdel old_group

# Thêm user vào nhóm
sudo adduser nbhson developers          # Cách 1
sudo usermod -aG developers nbhson      # Cách 2

# Xóa user khỏi nhóm
sudo deluser nbhson developers
sudo gpasswd -d nbhson developers

# Xem thành viên nhóm
getent group developers

# Đổi nhóm chính của user
sudo usermod -g developers nbhson
```

### Hiệu lực của việc thêm nhóm

Sau khi `usermod -aG`, user **phải đăng nhập lại** hoặc dùng `newgrp` để có quyền nhóm mới:

```bash
# Đăng xuất rồi đăng nhập lại
exit
ssh nbhson@server

# Hoặc ngay tại chỗ (chỉ shell hiện tại)
newgrp docker
```

### Nhóm hệ thống phổ biến trên Ubuntu

| Nhóm | Quyền |
|---|---|
| `sudo` | Được dùng sudo |
| `docker` | Quản lý Docker không cần sudo |
| `www-data` | Chạy dịch vụ web |
| `adm` | Đọc log hệ thống |
| `audio`, `video` | Thiết bị âm thanh/hình ảnh |
| `disk` | Truy cập block devices |
| `lpadmin` | Quản lý máy in |

---

## 8. `sudo` — Phân quyền quản trị

### Cấu hình sudo

```bash
# Cho user dùng sudo
sudo usermod -aG sudo nbhson

# Sửa cấu hình sudo (KHÔNG sửa /etc/sudoers trực tiếp)
sudo visudo

# Thêm user với quyền hạn chế (chỉ chạy một lệnh)
# File: /etc/sudoers.d/deployer
deployer ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

### Định dạng sudoers

```text
user  host=(run_as_user:run_as_group)  commands
nbhson ALL=(ALL:ALL) ALL               # Tất cả quyền
nbhson ALL=(root) NOPASSWD: /usr/bin/apt  # Chỉ apt, không cần mật khẩu
```

### Sử dụng sudo

```bash
sudo command                    # Chạy với quyền root
sudo -i                         # Vào shell root hoàn toàn
sudo -u otheruser command       # Chạy như user khác
sudo -l                         # Xem mình được phép chạy gì
sudo -k                         # Xóa cache mật khẩu sudo
sudo !                          # Chạy lại lệnh trước với sudo (rất tiện!)
```

> ⚠️ **Mẹo an toàn:** tránh `sudo su` hoặc `sudo -i` thường xuyên — dễ quên mình đang root. Hãy dùng `sudo <lệnh>` cụ thể.

> 💡 Từ Ubuntu 26.04, `sudo` được triển khai dưới dạng **sudo-rs** (viết bằng Rust, memory-safe) nhưng cú pháp sử dụng giữ nguyên.

---

## 9. `who`, `w`, `last` — Xem ai đang đăng nhập

```bash
# Ai đang đăng nhập
who
# nbhson  pts/0        2026-10-27 09:00 (192.168.1.50)

# Chi tiết hơn — kể cả lệnh đang chạy
w
#  USER  TTY  FROM  LOGIN@  IDLE  WHAT
#  nbhson pts/0 192.168.1.50 09:00  0:00  vim file.txt

# Người đăng nhập gần đây (lịch sử)
last
last nbhson          # Chỉ user cụ thể
last -n 10           # 10 bản ghi gần nhất
last reboot          # Lịch sử reboot

# Thông tin chính mình
whoami               # Tên user
id                   # UID, GID, groups
```

### Kiểm tra lần đăng nhập thất bại (SSH brute-force)

```bash
sudo lastb                     # Các lần đăng nhập SAI
sudo lastb | head -20
sudo grep "Failed password" /var/log/auth.log | tail -20
```

---

## 10. Use-cases thực tế

### Tạo user mới với quyền sudo trên server

```bash
# 1. Tạo user
sudo adduser nbhson

# 2. Thêm vào nhóm sudo
sudo usermod -aG sudo nbhson

# 3. Thiết lập SSH key
sudo mkdir -p /home/nbhson/.ssh
sudo cp ~/.ssh/authorized_keys /home/nbhson/.ssh/
sudo chown -R nbhson:nbhson /home/nbhson/.ssh
sudo chmod 700 /home/nbhson/.ssh
sudo chmod 600 /home/nbhson/.ssh/authorized_keys

# 4. Đăng xuất và test SSH bằng user mới
exit
ssh nbhson@server
```

### Cho phép user quản lý Docker

```bash
sudo usermod -aG docker nbhson
newgrp docker                # Hoặc đăng nhập lại
docker ps                    # Không cần sudo nữa
```

### Tạo user cho dịch vụ (không login)

```bash
sudo useradd -r -s /usr/sbin/nologin myservice
sudo usermod -aG www-data myservice
```

---

## 11. Troubleshooting

### "User is not in the sudoers file"

```bash
# Đăng nhập root (hoặc user sudo khác) rồi thêm vào nhóm sudo
su -
usermod -aG sudo nbhson
exit
```

### Sửa hỏng `/etc/sudoers` — không sudo được nữa

```bash
# Boot vào recovery mode hoặc dùng phương pháp khác (console/VNC)
# Trên recovery mode chọn "root - Drop to root shell"
mount -o remount,rw /
visudo                # Sửa lại file
```

### Nhóm mới không có tác dụng

```bash
id                    # Xem user đang thực sự thuộc nhóm nào
newgrp docker         # Kích hoạt nhóm trong session hiện tại
# Hoặc đăng xuất / đăng nhập lại
```

### Không đổi được mật khẩu của chính mình

```bash
passwd
# "passwd: Authentication token manipulation error"
# → Hệ thống đặt mật khẩu qua LDAP/AD, hoặc /etc/shadow không ghi được.
sudo passwd $USER
```

### SSH chỉ cho phép key, không cho mật khẩu

```bash
sudo nano /etc/ssh/sshd_config
# PasswordAuthentication no
sudo systemctl restart ssh
```

---

## Nguồn tham khảo

- Ubuntu Server Guide — User Management: https://documentation.ubuntu.com/server/
- Sudo documentation: https://www.sudo.ws/docs/man/sudo.man/
- `man useradd`, `man usermod`, `man passwd`, `man sudoers`
