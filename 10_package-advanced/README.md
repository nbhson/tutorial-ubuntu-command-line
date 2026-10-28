# Package Management Nâng Cao (snap, flatpak, dpkg, PPA, compile từ source)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · APT 3.1 · snapd hiện tại
> **Đối tượng:** Người dùng muốn vượt ra ngoài `apt install` cơ bản

---

## Mục lục

1. [Tổng quan hệ sinh thái package Ubuntu](#1-tổng-quan-hệ-sinh-thái-package-ubuntu)
2. [`snap` — Package sandbox của Canonical](#2-snap--package-sandbox)
3. [`flatpak` — Package đa distro](#3-flatpak--package-đa-distro)
4. [`apt-cache` và `apt` subcommands](#4-apt-cache-và-apt-subcommands)
5. [`dpkg` — Làm việc trực tiếp với file .deb](#5-dpkg--làm-việc-trực-tiếp-với-file-deb)
6. [PPA — Personal Package Archive](#6-ppa--personal-package-archive)
7. [Compile từ source](#7-compile-từ-source)
8. [So sánh các phương pháp](#8-so-sánh-các-phương-pháp)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Tổng quan hệ sinh thái package Ubuntu

Ubuntu hỗ trợ nhiều cách cài phần mềm, mỗi cách phù hợp một nhu cầu:

| Cách cài | Định dạng | Phân lập (sandbox) | Cập nhật tự động | Dùng khi nào |
|---|---|---|---|---|
| `apt` | .deb | ❌ | ✅ | Phần mềm hệ thống, server |
| `snap` | .snap | ✅ | ✅ (mặc định) | App desktop, CLI độc lập |
| `flatpak` | .flatpakref | ✅ | ❌ (tự quản) | App desktop từ Flathub |
| PPA | .deb | ❌ | ✅ | Phần mềm mới hơn bản Ubuntu có |
| Source compile | binary | ❌ | Tự quản | Phiên bản mới nhất, patch tùy chỉnh |

> 💡 **APT 3.1** (Ubuntu 26.04) có solver mới (solver3), lệnh `apt why` để truy nguyên nhân cài package, và khuyến nghị dùng **định dạng DEB822** cho repository mới (file `.sources` thay vì `.list`).

---

## 2. `snap` — Package sandbox

Snap là hệ thống package do Canonical phát triển, ứng dụng chạy trong môi trường cô lập (sandbox), kèm mọi dependency.

```bash
# Tìm và xem thông tin
snap find "code editor"
snap info code

# Cài
sudo snap install code --classic
sudo snap install firefox
sudo snap install vlc

# Xem đã cài
snap list
snap list --all

# Cập nhật
sudo snap refresh              # Cập nhật tất cả
sudo snap refresh code         # Cập nhật một app
sudo snap refresh --list       # Xem sẽ cập nhật gì

# Gỡ
sudo snap remove code

# Xem channel (stable, candidate, beta, edge)
snap info vlc
sudo snap install vlc --channel=candidate
```

### Kênh (channel) và cờ đặc biệt

```bash
sudo snap install app --classic       # Cho phép truy cập toàn bộ hệ thống (IDE, dev tools)
sudo snap install app --dangerous     # Cài file .snap tải về
sudo snap install app --jailmode      # Bắt buộc sandbox
```

### Kiểm soát cập nhật tự động

```bash
# Giữ phiên bản hiện tại (hold)
sudo snap refresh --hold code
sudo snap refresh --hold=30d code    # Hold 30 ngày

# Bỏ hold
sudo snap refresh --unhold code
```

### Kiểm tra và dọn dẹp

```bash
snap changes                          # Lịch sử các giao tác
sudo snap changes                     # Xem lỗi nếu có
sudo snap abort --last=123            # Hủy thay đổi đang kẹt
snap list --all | awk '/disabled/{print $1, $3}' | \
  while read name rev; do sudo snap remove "$name" --revision="$rev"; done
```

> ⚠️ **Bộ nhớ snap:** snap giữ nhiều **revision cũ** (disabled). Sau vài tháng có thể chiếm hàng GB. Dùng script trên để dọn, hoặc định kỳ `sudo snap set system refresh.retain=3` để chỉ giữ 3 revision.

---

## 3. `flatpak` — Package đa distro

Flatpak tập trung vào **app desktop** từ kho Flathub, hỗ trợ nhiều bản Linux:

```bash
# Cài và thêm kho Flathub
sudo apt install flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Tìm và cài
flatpak search krita
flatpak install flathub org.kde.krita

# Chạy
flatpak run org.kde.krita

# Cập nhật và dọn dẹp
flatpak update
flatpak uninstall --unused

# Danh sách app đã cài
flatpak list
```

---

## 4. `apt-cache` và `apt` subcommands

```bash
# Tìm package theo tên/mô tả
apt-cache search nginx
apt-cache search --names-only '^nginx$'

# Thông tin chi tiết
apt-cache show nginx
apt-cache policy nginx             # Phiên bản installed/candidate, repo nào

# Phụ thuộc
apt-cache depends nginx            # Package này phụ thuộc vào gì?
apt-cache rdepends nginx           # Package nào phụ thuộc vào nó?

# Thống kê
apt-cache stats
```

### Các subcommand apt hiện đại (thay apt-get)

```bash
apt list --installed               # Danh sách đã cài
apt list --upgradable              # Chờ nâng cấp
apt edit-sources                   # Mở editor sửa sources (an toàn)
sudo apt install ./local.deb       # Cài file .deb kèm giải quyết phụ thuộc
apt why <package>                  # ⭐ APT 3.1+: vì sao package này được cài?
```

> 💡 **Từ Ubuntu 26.04 (APT 3.1):** `apt why` là lệnh mới rất hữu ích:
>
> ```bash
> apt why libcurl4
> # → cho biết package nào kéo libcurl vào hệ thống
> ```

---

## 5. `dpkg` — Làm việc trực tiếp với file .deb

`apt` quản lý repository; `dpkg` là tầng thấp hơn — cài trực tiếp file `.deb`:

```bash
# Cài file .deb
sudo dpkg -i ten_file.deb

# Cài file .deb + tự giải quyết phụ thuộc (khuyến nghị)
sudo apt install ./ten_file.deb

# Xem đã cài những gì
dpkg -l                            # Tất cả
dpkg -l | grep nginx               # Lọc
dpkg -L nginx                      # File nào thuộc package này?
dpkg -S /usr/bin/curl              # Package nào cung cấp file này?

# Gỡ
sudo dpkg -r ten_package           # Gỡ, giữ cấu hình
sudo dpkg -P ten_package           # Gỡ + xóa cấu hình

# Sửa trạng thái cài hỏng
sudo dpkg --configure -a
sudo dpkg --audit                  # Tìm package lỗi
```

> 💡 Khi cài file `.deb` tải từ web, **ưu tiên `sudo apt install ./file.deb`** thay vì `dpkg -i` — apt tự tải thêm phụ thuộc, còn `dpkg` sẽ báo lỗi dependency và cần `sudo apt install -f` để sửa sau.

### Giữ (hold) một package không cho nâng cấp

```bash
sudo apt-mark hold nginx           # Ngăn apt nâng cấp
sudo apt-mark unhold nginx         # Bỏ giữ
apt-mark showhold                  # Xem các package đang giữ
```

---

## 6. PPA — Personal Package Archive

PPA là repository của bên thứ ba trên Launchpad, thường dùng để cài **phiên bản mới hơn**:

```bash
# Thêm PPA
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.12

# Xóa PPA
sudo add-apt-repository --remove ppa:deadsnakes/ppa
# hoặc:
sudo ppa-purge ppa:deadsnakes/ppa      # Gỡ + hoàn nguyên packages về bản gốc
```

### Thêm repository theo chuẩn mới (DEB822)

Từ Ubuntu 24.04, repository bên thứ ba nên dùng file `.sources`:

```bash
# Ví dụ thêm Docker repository (chuẩn mới)
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```ini
# File: /etc/apt/sources.list.d/docker.sources
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: noble
Components: stable
Signed-By: /etc/apt/keyrings/docker.gpg
```

```bash
sudo apt update
sudo apt install docker-ce
```

> ⚠️ **Cẩn thận với PPA:** chọn PPA có uy tín, kiểm tra kỹ. PPA không được Canonical ký số và có thể chứa phiên bản gây mất ổn định hệ thống. Cỡ Ubuntu 24.04 trở đi, PPA key được thêm tự động, nhưng vẫn nên kiểm tra bằng `apt-cache policy`.

---

## 7. Compile từ source

Khi không có package nào ở trên, compile từ mã nguồn:

```bash
# 1. Cài build-essential (gcc, make, headers...)
sudo apt install build-essential

# 2. Cài thêm các thư viện mà package đó yêu cầu
sudo apt install libssl-dev zlib1g-dev libreadline-dev

# 3. Tải mã nguồn
wget https://example.com/app-1.0.0.tar.gz
tar -xzf app-1.0.0.tar.gz
cd app-1.0.0

# 4. Cấu hình, build, cài
./configure --prefix=/usr/local
make -j$(nproc)          # Build song song theo số CPU
sudo make install

# 5. Nếu cần gỡ
sudo make uninstall
```

### Workflow với git và cmake

```bash
git clone https://github.com/user/project.git
cd project
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

> 💡 Luôn cài phần mềm tự compile vào `/usr/local` (mặc định của `--prefix=/usr/local`) — tách biệt với package của apt ở `/usr`. Giúp dọn dẹp và tránh xung đột với apt.

---

## 8. So sánh tổng hợp

| Tiêu chí | apt (.deb) | snap | flatpak | PPA | source |
|---|---|---|---|---|---|
| Tốc độ cài | Nhanh | Chậm hơn | Trung bình | Nhanh | Chậm nhất |
| Dung lượng | Nhỏ | Lớn (đóng gói lib) | Lớn | Như apt | Nhỏ |
| Sandbox | ❌ | ✅ | ✅ | ❌ | ❌ |
| Phiên bản mới nhất | Theo Ubuntu | Mới nhất | Mới nhất | Tùy PPA | Mới nhất |
| Cập nhật tự động | Không tự | Tự động | Tùy | Không | Thủ công |
| Hệ thống sạch | Tốt | OK | OK | Rủi ro phụ thuộc | Khó gỡ sạch |

---

## 9. Troubleshooting

### Lỗi "snap changes pending" hoặc không refresh được

```bash
snap changes                       # Xem thay đổi đang chờ
sudo snap abort <change-id>        # Hủy thay đổi
sudo snap refresh                  # Thử lại
```

### Snap app không truy cập được file ngoài home

```bash
sudo snap connect app:removable-media    # Cho phép truy cập USB/ổ ngoài
```

### apt bị lỗi GPG khi thêm repo mới

```bash
# Sử dụng signed-by để trỏ tới keyring riêng (chuẩn hiện tại)
curl -fsSL https://example.com/key.gpg | \
  sudo gpg --dearmor -o /usr/share/keyrings/example.gpg
echo "Types: deb
URIs: https://example.com/repo
Suites: stable
Components: main
Signed-By: /usr/share/keyrings/example.gpg" | \
  sudo tee /etc/apt/sources.list.d/example.sources
sudo apt update
```

### Xung đột sau khi compile từ source

```bash
# Gỡ bản tự compile
sudo make uninstall

# Kiểm tra version nào đang được ưu tiên
which app
app --version
```

### Lỗi "held broken packages" khi cài

```bash
sudo apt-mark showhold                     # Xem package bị giữ
sudo apt-mark unhold <pkg>                 # Bỏ giữ
sudo apt install -f                        # Sửa phụ thuộc hỏng
sudo dpkg --configure -a
```

---

## Nguồn tham khảo

- Ubuntu Snapcraft docs: https://snapcraft.io/docs
- Flathub: https://flathub.org/
- APT 3.1 trên Ubuntu 26.04: https://linuxconfig.org/what-is-new-in-apt-3-1-on-ubuntu-26-04
- Ubuntu PPA: https://launchpad.net/ubuntu/+ppas
- Debian packaging: https://www.debian.org/doc/devel-policy/
