# Disk & Storage — Quản lý ổ đĩa (df, du, lsblk, mount, fdisk)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS
> **Đối tượng:** Người dùng cần giám sát dung lượng, mount ổ đĩa, format disk

---

## Mục lục

1. [`df` — Dung lượng ổ đĩa](#1-df--dung-lượng-ổ-đĩa)
2. [`du` — Dung lượng thư mục/file](#2-du--dung-lượng-thư-mụctệp)
3. [`lsblk` — Xem khối thiết bị (block devices)](#3-lsblk--xem-khối-thiết-bị-block-devices)
4. [`mount` / `umount` — Gắn/tháo ổ đĩa](#4-mount--umount--gắn-kết-ổ-đĩa)
5. [`fdisk` — Phân vùng đĩa](#5-fdisk--phân-vùng-đĩa)
6. [`mkfs` — Định dạng (format) phân vùng](#6-mkfs--định-dạng-format-phân-vùng)
7. [`fstab` — Tự động mount khi khởi động](#7-fstab--tự-động-mount-khi-khởi-động)
8. [`fsck` — Kiểm tra và sửa lỗi filesystem](#8-fsck--kiểm-tra-và-sửa-lỗi-filesystem)
9. [Phân tích dung lượng thực tế](#9-phân-tích-dung-lượng-thực-tế)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. `df` — Dung lượng ổ đĩa

```bash
df                    # Tất cả filesystem (theo số byte)
df -h                 # Human-readable (K, M, G, T)
df -h /               # Chỉ filesystem chứa /
df -i                 # Xem theo số inode (không phải size)
df -T                 # Hiện loại filesystem (ext4, xfs...)
df -x tmpfs -h        # Bỏ filesystem ảo (tmpfs, overlay...)
```

### Đọc hiểu output

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        98G   45G   49G  51% /
tmpfs           7.8G     0  7.8G   0% /dev/shm
```

- `Filesystem`: thiết bị (`/dev/sda2`...) hoặc loại ảo (`tmpfs`, `overlay`)
- `Use%`: phần trăm đã dùng — **cảnh báo khi > 80–90%**

> ⚠️ **Filesystem ảo:** các dòng như `tmpfs`, `udev`, `overlay` không phải ổ đĩa thật — chúng nằm trong RAM. Muốn nhìn nhanh, dùng `df -h -x tmpfs -x devtmpfs`.

---

## 2. `du` — Dung lượng thư mục/file

```bash
du -h                  # Dung lượng các thư mục con (human-readable)
du -sh .               # Tổng dung lượng thư mục hiện tại
du -h --max-depth=1    # Chỉ 1 cấp con
du -ah | sort -rh | head -10   # Top 10 file/thư mục lớn nhất

# Loại trừ
du -sh --exclude=node_modules .

# Tìm 10 file lớn nhất trong hệ thống
sudo du -ah / 2>/dev/null | sort -rh | head -10
```

### Ví dụ — tìm nguyên nhân ổ đầy

```bash
# Bước 1: filesystem nào đầy?
df -h

# Bước 2: thư mục lớn nhất ở root
sudo du -h --max-depth=1 / | sort -rh | head -10

# Bước 3: khoét sâu hơn
sudo du -h --max-depth=1 /var | sort -rh | head -10

# Thường thủ phạm: /var/log, /var/cache/apt, /var/lib/docker
sudo journalctl --vacuum-size=100M     # Thu gọn log systemd
sudo apt clean                          # Xóa cache .deb
```

### df vs du — khác biệt số liệu

`du` đếm theo **file thực tế**, `df` nhìn filesystem **bao gồm file đã xóa mà process còn giữ**. Nếu `du` nhỏ mà `df` đầy → có process giữ file đã xóa:

```bash
sudo lsof +L1 | grep deleted           # Tìm file đã xóa nhưng chưa giải phóng
sudo systemctl restart rsyslog         # Restart service để giải phóng
```

---

## 3. `lsblk` — Xem khối thiết bị (block devices)

```bash
lsblk
```

```text
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   100G  0 disk
├─sda1   8:1    0     1M  0 part
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda2   8:3    0  99.5G  0 part /
sdb      8:16   0   500G  0 disk
└─sdb1   8:17   0   500G  0 part /mnt/data
```

- `TYPE`: disk = ổ vật lý, part = phân vùng, lvm = logical volume
- `MOUNTPOINTS`: nơi gắn vào cây thư mục; trống = chưa mount

### Lệnh liên quan

```bash
lsblk -f               # Hiện filesystem + UUID
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL
blkid                  # UUID, TYPE của mọi phân vùng (cần sudo)
sudo fdisk -l          # Bảng phân vùng chi tiết
sudo parted -l         # Xem bằng parted
```

---

## 4. `mount` / `umount` — Gắn kết/gỡ ổ đĩa

```bash
# Xem những gì đang mount
mount | column -t
findmnt                        # Dạng cây, dễ đọc hơn

# Mount thủ công
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data

# Mount với filesystem cụ thể
sudo mount -t ext4 /dev/sdb1 /mnt/data
sudo mount -t ntfs /dev/sdc1 /mnt/windows-disk

# Mount ISO
sudo mount -o loop ubuntu.iso /mnt/iso

# Unmount
sudo umount /mnt/data
# Lỗi "target is busy" → xem process nào đang dùng:
sudo lsof +f -- /mnt/data
sudo umount -l /mnt/data       # Lazy unmount (chờ process nhả)
```

### Mount USB và disk tự động

Ubuntu desktop tự mount USB vào `/media/<user>/<label>/`. Trên server cần mount thủ công như trên.

### Mount vĩnh viễn — `/etc/fstab`

```bash
# 1. Xem UUID phân vùng
sudo blkid /dev/sdb1

# 2. Thêm dòng vào /etc/fstab
sudo nano /etc/fstab
```

```text
UUID=abcd-1234  /mnt/data  ext4  defaults  0  2
```

```bash
# 3. Kiểm tra và mount lại
sudo mount -a          # Nếu báo lỗi → sửa ngay trước khi reboot!
```

> ⚠️ **Sai cú pháp trong `/etc/fstab` có thể khiến hệ thống không boot được.** Luôn chạy `sudo mount -a` để kiểm tra trước khi reboot, và giữ máy có thể vào chế độ recovery.

---

## 5. `fdisk` — Phân vùng đĩa

> ⚠️ **Mọi thao tác fdisk/mkfs sẽ XÓA dữ liệu trên đĩa mục tiêu.** Kiểm tra kỹ tên thiết bị bằng `lsblk` trước khi thao tác!

```bash
sudo fdisk -l                 # Xem bảng phân vùng mọi đĩa
sudo fdisk /dev/sdb           # Vào chế độ tương tác
```

Các lệnh trong fdisk (tương tác):

| Lệnh | Chức năng |
|---|---|
| `m` | Menu trợ giúp |
| `p` | In bảng phân vùng hiện tại |
| `n` | Tạo phân vùng mới |
| `d` | Xóa phân vùng |
| `t` | Đổi loại phân vùng (vd: `8e` cho LVM, `b` FAT32, `83` Linux) |
| `w` | **Lưu** và thoát |
| `q` | Thoát **không lưu** |

### Quy trình tạo phân vùng mới hoàn chỉnh

```bash
# 1. Xác định đĩa (ví dụ /dev/sdb — còn trống, chưa partition)
lsblk

# 2. Tạo phân vùng
sudo fdisk /dev/sdb
#   g  → tạo bảng phân vùng GPT
#   n  → new partition (Enter cho mặc định)
#   w  → ghi vào đĩa

# 3. Format phân vùng
sudo mkfs.ext4 /dev/sdb1

# 4. Mount
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data

# 5. Tự động mount khi boot: thêm vào /etc/fstab
echo "UUID=$(sudo blkid -s UUID -o value /dev/sdb1) /mnt/data ext4 defaults 0 2" | sudo tee -a /etc/fstab
sudo mount -a
```

### `mkfs` — Tạo filesystem

```bash
sudo mkfs.ext4 /dev/sdb1      # ext4 — phổ biến nhất cho Linux
sudo mkfs.xfs  /dev/sdb1      # xfs — hiệu năng cao cho file lớn
sudo mkfs.vfat -F 32 /dev/sdb1  # FAT32 (tương thích đa hệ)
sudo mkfs.ntfs /dev/sdb1      # NTFS (cho ổ dùng chung với Windows)
```

---

## 6. LVM cơ bản (bổ sung)

Ubuntu server thường dùng LVM (Logical Volume Manager) cho phân vùng root:

```bash
sudo apt install lvm2
sudo pvs / sudo vgs / sudo lvs        # Xem physical/group/logical volumes
sudo lvdisplay                         # Chi tiết logical volume

# Mở rộng logical volume + filesystem (tăng dung lượng)
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv          # ext4
sudo xfs_growfs /                                 # nếu là xfs
```

> 💡 Với cloud VM (AWS/GCP/DigitalOcean), sau khi **mở rộng volume** ở provider, bạn vẫn cần `growpart` + `resize2fs` bên trong máy:
>
> ```bash
> sudo apt install cloud-guest-utils
> sudo growpart /dev/sda 1
> sudo resize2fs /dev/sda1
> ```

---

## 7. Kiểm tra và sửa lỗi filesystem — `fsck`

```bash
# Kiểm tra (UNMOUNT trước!)
sudo umount /dev/sdb1
sudo fsck /dev/sdb1

# ext4 cụ thể
sudo fsck.ext4 -f /dev/sdb1

# Với root partition — dùng qua grub hoặc chế độ recovery
sudo touch /forcefsck && sudo reboot
```

---

## 8. Phân tích dung lượng thực tế

### Kịch bản: ổ `/` đầy (100%)

```bash
# 1. Xác nhận filesystem
df -h /

# 2. Tìm thư mục lớn nhất ở root
sudo du -h --max-depth=1 / 2>/dev/null | sort -rh | head -10

# 3. Rửa các chỗ thường "ăn" dung lượng
sudo journalctl --vacuum-size=100M    # Thu gọn log systemd
sudo apt clean                         # Xóa cache .deb
sudo rm -rf /var/log/*.gz /var/log/*.1 # Xóa log đã xoay

# 4. Tìm file > 100MB
sudo find / -xdev -type f -size +100M -exec ls -lh {} \; 2>/dev/null | \
  awk '{print $5, $9}' | sort -rh | head -10

# 5. Snap old revisions
snap list --all | awk '/disabled/{print $1, $3}' | \
  while read name rev; do sudo snap remove "$name" --revision="$rev"; done
```

### Quota cho user

```bash
sudo apt install quota
sudo quotacheck -cum /
sudo edquota -u nbhson        # Đặt giới hạn
repquota /                    # Báo cáo
```

---

## 9. Troubleshooting

### `df` báo đầy nhưng không thấy file nào lớn

```bash
# File bị xóa nhưng process vẫn giữ
sudo lsof +L1 | sort -k7 -rn | head
# Restart service đang giữ file

# Hoặc dung lượng bị "reserved" cho root
sudo tune2fs -l /dev/sda1 | grep -i reserved
```

### Không mount được ổ NTFS

```bash
sudo apt install ntfs-3g
sudo mount -t ntfs-3g /dev/sdb1 /mnt/usb

# Ổ Windows ở chế độ fast startup → cần shutdown hoàn toàn trên Windows
```

### `target is busy` khi umount

```bash
sudo lsof +f -- /mnt/data     # Xem process nào đang dùng
sudo fuser -km /mnt/data      # Kill mọi process dùng nó (cẩn trọng)
sudo umount /mnt/data
```

### Disk đầy đột ngột sau khi chạy app

```bash
sudo du -h --max-depth=2 /var 2>/dev/null | sort -rh | head
sudo journalctl --disk-usage
docker system df              # Nếu dùng Docker
docker system prune -a        # Dọn Docker (CẢNH BÁO: xóa image không dùng)
```

---

## Nguồn tham khảo

- Ubuntu Server Guide — Disk & Filesystems: https://documentation.ubuntu.com/server/
- Netplan documentation: https://netplan.readthedocs.io/
- `man df`, `man du`, `man lsblk`, `man mount`, `man fstab`
