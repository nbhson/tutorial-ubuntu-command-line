# Update, Upgrade & Quản lý Package trên Ubuntu

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS "Resolute Raccoon" (phát hành 04/2026) · APT 3.1
> **Đối tượng:** Người mới bắt đầu · Cần quyền sudo trên máy

---

## Mục lục

1. [Cập nhật danh sách package — `apt update`](#1-cập-nhật-danh-sách-package--apt-update)
2. [Nâng cấp package — `apt upgrade`](#2-nâng-cấp-package--apt-upgrade)
3. [Cài đặt package mới](#3-cài-đặt-package-mới)
4. [Xóa package](#4-xóa-package)
5. [Tìm kiếm & tra cứu package](#5-tìm-kiếm--tra-cứu-package)
6. [Quy trình update → upgrade chuẩn](#6-quy-trình-update--upgrade-chuẩn)
7. [Làm sạch hệ thống](#7-làm-sạch-hệ-thống)
8. [Lệnh `apt full-upgrade` và `do-release-upgrade`](#8-lệnh-apt-full-upgrade-và-do-release-upgrade)
9. [Lưu ý quan trọng](#9-lưu-ý-quan-trọng)
10. [Xử lý lỗi thường gặp](#10-xử-lý-lỗi-thường-gặp)

---

## 1. Cập nhật danh sách package — `apt update`

```bash
sudo apt update
```

Lệnh `apt update` kết nối tới các **repository** (kho phần mềm) đã cấu hình và tải về **danh sách package mới nhất** cùng thông tin phiên bản. Lệnh này **KHÔNG cài đặt gì cả**, chỉ cập nhật "danh bạ" phần mềm.

### File cấu hình repository

> ⚠️ **Thay đổi từ Ubuntu 24.04 (Noble Numbat):** Ubuntu chuyển sang mặc định dùng **định dạng DEB822** thay cho file `sources.list` một-dòng-một-repo cũ.

| Phiên bản | File cấu hình chính |
|---|---|
| Trước 24.04 | `/etc/apt/sources.list` |
| 24.04 trở lên | `/etc/apt/sources.list.d/ubuntu.sources` |

```bash
# Ubuntu 24.04+ (định dạng DEB822):
cat /etc/apt/sources.list.d/ubuntu.sources

# Ubuntu 22.04 trở xuống (định dạng cũ):
cat /etc/apt/sources.list
```

Kết quả ví dụ trên 24.04+:

```text
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

- `main`: phần mềm chính thức do Canonical hỗ trợ
- `restricted`: driver có bản quyền
- `universe`: phần mềm do cộng đồng duy trì
- `multiverse`: phần mềm không tự do về bản quyền

Repository từ bên thứ ba (PPA, Docker, NodeSource...) nằm trong các file riêng tại `/etc/apt/sources.list.d/`.

---

## 2. Nâng cấp package — `apt upgrade`

```bash
sudo apt upgrade
```

Lệnh này **tải xuống và cài đặt** các bản mới của package, dựa trên thông tin vừa lấy bằng `apt update`.

- Các gói phụ thuộc mới sẽ được cài nếu cần, nhưng **không gói nào bị xóa**.
- Muốn cho phép gỡ package khi cần (ví dụ xung đột phụ thuộc), dùng:

```bash
sudo apt full-upgrade
```

### Khác biệt `update` vs `upgrade`

| Lệnh | Việc làm |
|---|---|
| `apt update` | Chỉ cập nhật **danh sách** package (metadata) |
| `apt upgrade` | **Cài đặt** bản mới dựa trên danh sách đó |

> 💡 **Mẹo:** Luôn chạy `apt update` trước `apt upgrade`. Chạy `upgrade` một mình sau nhiều ngày có thể dùng danh sách cũ, gây cài phiên bản lỗi thời hoặc lỗi 404 khi tải.

---

## 3. Cài đặt package mới

```bash
# Cài một package
sudo apt install [ten_package]

# Cài nhiều package cùng lúc
sudo apt install nginx curl git

# Cài kèm mô phỏng trước (xem sẽ cài gì mà chưa cài thật)
sudo apt install -s nginx

# Cài phiên bản cụ thể
sudo apt install nginx=1.24.0-2ubuntu7

# Cài gói .deb đã tải về máy
sudo apt install ./ten_file.deb

# Cài không cần xác nhận y/n
sudo apt install -y nginx
```

> 💡 Từ Ubuntu 26.04 (APT 3.1) có thêm lệnh `apt why <package>` để xem vì sao một gói được cài (gói nào phụ thuộc vào nó):

```bash
apt why nginx
```

---

## 4. Xóa package

```bash
# Gỡ package, giữ lại file cấu hình
sudo apt remove nginx

# Gỡ package + file cấu hình (purge)
sudo apt purge nginx

# Gỡ kèm các phụ thuộc không còn dùng
sudo apt autoremove

# Xem list các package đã cài
apt list --installed
```

---

## 5. Tìm kiếm & tra cứu package

```bash
# Tìm package theo tên/mô tả
apt search redis

# Xem chi tiết một package (phiên bản, dung lượng, phụ thuộc)
apt show nginx

# Xem package nào cung cấp một file/lệnh cụ thể
sudo apt install apt-file
sudo apt-file update
apt-file search /usr/bin/convert

# Xem các phiên bản khả dụng của package
apt-cache policy nginx
```

Kết quả ví dụ của `apt-cache policy`:

```text
nginx:
  Installed: (none)
  Candidate: 1.24.0-2ubuntu7
  Version table:
     1.24.0-2ubuntu7 500
        500 http://vn.archive.ubuntu.com/ubuntu noble/universe amd64 Packages
```

---

## 6. Quy trình update → upgrade chuẩn

```bash
# 1. Cập nhật danh sách package
sudo apt update

# 2. Nâng cấp các package hiện có
sudo apt upgrade

# 3. Dọn dẹp phụ thuộc thừa
sudo apt autoremove

# 4. (Tùy chọn) Xóa cache các file .deb đã tải
sudo apt clean
```

Có thể gộp một dòng:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 7. Làm sạch hệ thống

```bash
sudo apt clean         # Xóa toàn bộ cache .deb trong /var/cache/apt/archives
sudo apt autoclean     # Chỉ xóa các file .deb không còn tải được từ repo
sudo apt autoremove --purge   # Gỡ phụ thuộc thừa kèm file cấu hình
```

> 💡 **Mẹo kiểm tra dung lượng cache apt:**
>
> ```bash
> du -sh /var/cache/apt/archives
> ```

---

## 8. Lệnh `apt full-upgrade` và `do-release-upgrade`

### `apt full-upgrade`

```bash
sudo apt full-upgrade
```

Khác `apt upgrade`: có thể **gỡ bỏ package** nếu cần để giải quyết xung đột phụ thuộc. Dùng khi nâng cấp lớn hoặc trong kho có thay đổi cấu trúc.

### Nâng cấp lên phiên bản Ubuntu mới (release upgrade)

```bash
# Cập nhật trước khi nâng cấp release
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade

# Nâng cấp lên bản Ubuntu kế tiếp (LTS → LTS hoặc bản mới nhất)
sudo do-release-upgrade
```

> ⚠️ **Lưu ý:** `do-release-upgrade` chỉ nhận diện bản LTS mới sau khi bản `.1` point release phát hành (thường khoảng 3 tháng sau ngày ra mắt). Với 26.04 LTS, thông báo nâng cấp tự động xuất hiện từ khoảng tháng 8/2026.

---

## 9. Lưu ý quan trọng

- **Không chạy `apt upgrade` khi đang có tiến trình apt khác chạy** — sẽ gặp lỗi lock (`Could not get lock /var/lib/dpkg/lock-frontend`).
- **Không ngắt đột ngột** quá trình upgrade (đóng terminal, mất điện). Nếu hệ thống hỏi về file cấu hình (`Configuring xxx`), đọc kỹ các lựa chọn.
- `sudo apt list --upgradable` — xem trước **những gì sẽ được nâng cấp** mà chưa chạy.
- Trên Ubuntu 26.04, một số tiện ích hệ thống đã được thay bằng phiên bản **Rust coreutils (uutils)**; đầu ra lệnh về cơ bản giống GNU coreutils nhưng có thể khác biệt nhỏ ở các trường hợp biên.

---

## 10. Xử lý lỗi thường gặp

### Lỗi lock khi chạy apt

```text
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
```

```bash
# Kiểm tra tiến trình nào đang giữ lock
sudo lsof /var/lib/dpkg/lock-frontend

# Nếu không có tiến trình apt nào chạy thật sự, có thể xóa lock
sudo rm /var/lib/dpkg/lock-frontend
sudo dpkg --configure -a
```

### Lỗi GPG key khi thêm repository

```bash
# Thêm key theo cách khuyến nghị hiện nay (dùng file keyring riêng)
curl -fsSL https://example.com/key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/example.gpg
echo "deb [signed-by=/usr/share/keyrings/example.gpg] https://example.com/repo stable main" | \
  sudo tee /etc/apt/sources.list.d/example.sources
```

### Lỗi 404 khi tải package

```bash
sudo apt update        # Làm mới danh sách
sudo apt --fix-broken install
```

### Sửa package bị cài hỏng

```bash
sudo dpkg --configure -a
sudo apt install -f
```

---

## Nguồn tham khảo

- Ubuntu release cycle: https://ubuntu.com/about/release-cycle
- Ubuntu 26.04 LTS release: https://canonical.com/blog/canonical-releases-ubuntu-26-04-lts-resolute-raccoon
- APT 3.1 trên Ubuntu 26.04: https://linuxconfig.org/what-is-new-in-apt-3-1-on-ubuntu-26-04
- DEB822 sources mặc định từ 24.04: https://discourse.ubuntu.com/t/spec-apt-deb822-sources-by-default/29333
