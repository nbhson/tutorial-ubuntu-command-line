# Networking — Mạng trên Ubuntu (ping, curl, ssh, ufw...)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS (Netplan + systemd-networkd/cloud-init)
> **Đối tượng:** Người dùng cần kết nối, chẩn đoán mạng, SSH và bảo mật

---

## Mục lục

1. [`ping` — Kiểm tra kết nối](#1-ping--kiểm-tra-kết-nối)
2. [`curl` và `wget` — Tải dữ liệu từ web](#2-curl-và-wget--tải-dữ-liệu-từ-web)
3. [`ip` — Xem và cấu hình interface (thay thế ifconfig)](#3-ip--xem-và-cấu-hình-interface-thay-thế-ifconfig)
4. [`ss` / `netstat` — Xem kết nối và port đang mở](#4-ss--netstat--xem-kết-nối-và-port-đang-mở)
5. [`ssh` — Kết nối từ xa](#5-ssh--kết-nối-từ-xa)
6. [`scp` / `rsync` — Copy file qua mạng](#6-scp--rsync--copy-file-qua-mạng)
7. [`ufw` — Tường lửa](#7-ufw--tường-lửa)
8. [Chẩn đoán mạng khác](#8-chẩn-đoán-mạng-khác)
9. [Use-cases thực tế](#9-use-cases-thực-tế)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. `ping` — Kiểm tra kết nối

```bash
ping google.com              # Ping liên tục (Ctrl+C để dừng)
ping -c 4 google.com         # Chỉ ping 4 gói rồi dừng
ping -i 0.5 8.8.8.8          # Ping mỗi 0.5 giây
ping -c 3 -W 2 10.0.0.1      # Timeout 2 giây mỗi gói
ping -6 google.com           # Ping IPv6
```

### Đọc hiểu output

```text
64 bytes from 142.250.4.139: icmp_seq=1 ttl=118 time=10.5 ms
```

- `icmp_seq`: số thứ tự gói — nếu bị gián đoạn hoặc sai thứ tự là mạng không ổn định
- `ttl`: Time To Live — giảm 1 mỗi router đi qua (thường bắt đầu 64 hoặc 128)
- `time`: độ trễ (latency). Với VN: <30ms tốt, >100ms chậm

> ⚠️ Từ Ubuntu 24.04, `ping` không cần `sudo` nhờ cấu hình `net.ipv4.ping_group_range` trong sysctl.

---

## 2. `curl` và `wget` — Tải dữ liệu từ web

### `curl` — Vạn năng cho HTTP

```bash
# Lấy nội dung URL in ra màn hình
curl https://example.com

# Lưu ra file
curl -o ten_file.html https://example.com

# Lưu với tên từ URL
curl -O https://example.com/file.tar.gz

# Chỉ xem HTTP headers
curl -I https://example.com

# Gọi API POST với JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Son", "email": "son@example.com"}'

# Gọi API cần token
curl -H "Authorization: Bearer TOKEN" https://api.example.com/me

# Tải kèm theo tiến trình, tiếp tục khi bị ngắt
curl -C - -O https://example.com/big-file.iso

# Đo thời gian response
curl -o /dev/null -s -w "time_total: %{time_total}s\n" https://example.com
```

### `wget` — Tải file chuyên dụng

```bash
wget https://example.com/file.zip                # Tải về thư mục hiện tại
wget -O ten_moi.zip https://example.com/file.zip # Đặt tên file đích
wget -c https://example.com/big.iso              # Resume download
wget -q https://example.com/script.sh            # Chế độ im lặng
wget -r -np https://example.com/docs/            # Tải đệ quy 1 thư mục web
```

### curl vs wget — chọn khi nào?

| Tình huống | Công cụ |
|---|---|
| Gọi API, test endpoint | `curl` |
| Tải file lớn, resume | `wget` (hoặc `curl -C -`) |
| Tải cả website đệ quy | `wget` |
| Script, automation | `curl` linh hoạt hơn |

---

## 3. `ip` — Xem và cấu hình interface (thay thế ifconfig)

> ⚠️ `ifconfig` không còn cài mặc định từ Ubuntu 20.04. Dùng lệnh `ip` mới (thuộc package `iproute2`).

```bash
# Xem tất cả interface + địa chỉ IP
ip addr
ip a                     # Viết tắt

# Chỉ IPv4
ip -4 addr

# Xem một interface cụ thể
ip addr show eth0

# Xem bảng định tuyến (default gateway)
ip route

# Bật/tắt interface
sudo ip link set eth0 up
sudo ip link set eth0 down

# Gán IP tạm thời (mất khi reboot)
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0
```

### Cấu hình IP tĩnh với Netplan (Ubuntu hiện đại)

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

```bash
sudo netplan try          # Áp dụng, tự rollback nếu lỗi sau 120s
# hoặc:
sudo netplan apply
```

---

## 4. `ss` / `netstat` — Xem kết nối và port đang mở

> ⚠️ `netstat` không còn cài mặc định từ Ubuntu 22.04. Dùng `ss` (socket statistics) thay thế.

```bash
# Tất cả kết nối đang lắng nghe
ss -tlnp
# t = TCP, l = listening, n = số thay vì tên service, p = process

# Tất cả kết nối TCP (kể cả đang thiết lập)
ss -tnp

# UDP đang lắng nghe
ss -ulnp

# Tìm process nào chiếm port 80
sudo ss -tlnp | grep :80
sudo lsof -i :80

# Thống kê tổng quan
ss -s
```

Đọc hiểu output:

```text
State    Recv-Q  Send-Q  Local Address:Port   Peer Address:Port  Process
LISTEN   0       511     0.0.0.0:80           0.0.0.0:*          users:(("nginx",pid=1234))
```

- `0.0.0.0:80` — lắng nghe trên **mọi** interface
- `127.0.0.1:80` — chỉ lắng nghe nội bộ
- `ESTAB` — kết nối đã thiết lập; nhiều `ESTAB` bất thường có thể dấu hiệu bị tấn công

---

## 5. `ssh` — Kết nối từ xa

### Kết nối cơ bản

```bash
ssh user@192.168.1.10          # Đăng nhập bằng mật khẩu
ssh -p 2222 user@host          # Port tùy chỉnh
ssh -v user@host               # Verbose — debug khi không kết nối được
```

### Đăng nhập bằng SSH key (khuyến nghị)

```bash
# 1. Tạo cặp key trên máy cá nhân
ssh-keygen -t ed25519 -C "nbhson@laptop"
# Mặc định lưu ở ~/.ssh/id_ed25519 (private) và .pub (public)

# 2. Copy public key lên server
ssh-copy-id user@192.168.1.10

# 3. Từ giờ đăng nhập không cần mật khẩu
ssh user@192.168.1.10
```

### Cấu hình `~/.ssh/config` — alias nhanh

```text
Host myserver
    HostName 203.0.113.10
    User nbhson
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

```bash
ssh myserver          # Ngắn gọn hơn ssh -p 2222 nbhson@203.0.113.10
```

### SSH tunnel (port forwarding)

```bash
# Truy cập DB nội bộ qua server jump
ssh -L 5433:db-internal:5432 user@jump-host
# → localhost:5433 giờ trỏ tới db-internal:5432

# Tunnel ngược — cho phép server truy cập dịch vụ trên máy bạn
ssh -R 8080:localhost:3000 user@myserver
```

### Bảo mật SSH server

```bash
sudo nano /etc/ssh/sshd_config
```

```text
PermitRootLogin no                    # Không cho root SSH
PasswordAuthentication no             # Chỉ cho dùng key
Port 2222                             # Đổi port mặc định
MaxAuthTries 3
```

```bash
sudo systemctl restart ssh
```

---

## 6. `scp` / `rsync` — Copy file qua mạng

```bash
# scp — copy đơn giản
scp file.txt user@server:/home/user/              # Đẩy file lên
scp user@server:/var/log/app.log .                # Kéo file về
scp -r folder/ user@server:/home/user/            # Copy cả thư mục
scp -P 2222 file.txt user@server:~/               # Port tùy chỉnh

# rsync — thông minh hơn (chỉ copy phần thay đổi, resume được)
rsync -avz folder/ user@server:/home/user/folder/
rsync -avz --delete src/ server:/backup/src/      # Đồng bộ, xóa file đích thừa
rsync -avzP big.iso server:~/                     # -P = tiến trình + resume
```

> ⚠️ Dấu `/` ở cuối đường dẫn `rsync` rất quan trọng: `src/` copy **nội dung** src, còn `src` copy **chính thư mục** src vào đích.

---

## 7. `ufw` — Tường lửa

UFW (Uncomplicated Firewall) là giao diện đơn giản của iptables/nftables:

```bash
# Xem trạng thái
sudo ufw status verbose

# Mặc định: chặn vào, cho phép ra
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Cho phép SSH (quan trọng trước khi enable!)
sudo ufw allow ssh            # hoặc: sudo ufw allow 22/tcp

# Kích hoạt
sudo ufw enable

# Cho phép web
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8080/tcp

# Cho phép từ IP cụ thể
sudo ufw allow from 192.168.1.0/24 to any port 22

# Xóa rule
sudo ufw delete allow 8080/tcp
sudo ufw status numbered
sudo ufw delete 2             # Xóa theo số thứ tự

# Tắt hoàn toàn
sudo ufw disable
```

> ⚠️ **Quy tắc vàng khi làm việc qua SSH:** luôn `sudo ufw allow ssh` **trước** khi `sudo ufw enable`, nếu không bạn sẽ tự khóa mình khỏi server.

---

## 8. Chẩn đoán mạng khác

```bash
# Truy vết đường đi của gói tin
traceroute google.com          # sudo apt install traceroute
mtr google.com                 # traceroute + ping realtime

# Truy vấn DNS
nslookup google.com
dig google.com
dig +short google.com
dig MX gmail.com               # Bản ghi mail

# Xem hostname và domain
hostname
hostname -I                    # Chỉ IP

# Kiểm tra gateway và route
ip route
ping $(ip route | awk '/default/ {print $3}')
```

---

## 9. Use-cases thực tế

### Kiểm tra web server có hoạt động

```bash
curl -I http://localhost                    # Trả 200 OK?
sudo ss -tlnp | grep :80                    # Nginx đang lắng nghe?
sudo ufw status | grep 80                   # Firewall mở?
```

### Chẩn đoán "không vào được internet"

```bash
ping 8.8.8.8 -c 3           # Có ra được internet? (IP)
ping google.com -c 3        # DNS có hoạt động? (tên miền)
ip route                    # Có default gateway?
cat /etc/resolv.conf        # DNS server nào?
```

### Cho phép một port cho dịch vụ mới

```bash
sudo ufw allow 3000/tcp comment "Node app"
curl -I http://server-ip:3000
```

---

## 10. Troubleshooting

### `Connection refused`

Service chưa chạy, hoặc chạy nhưng không lắng nghe port đó:

```bash
sudo ss -tlnp | grep :80
sudo systemctl status nginx
```

### `Host unreachable` / timeout

```bash
ping -c 3 <target>          # Có gói trả về không?
traceroute <target>         # Bị kẹt ở hop nào?
sudo ufw status             # Firewall chặn?
```

### Không SSH được sau khi enable ufw

```bash
# Qua console/VNC của provider:
sudo ufw allow ssh
sudo ufw status verbose
```

### DNS không phân giải được

```bash
nslookup google.com 8.8.8.8     # Thử DNS ngoài
# Nếu được → vấn đề DNS cục bộ; nếu không → vấn đề mạng
cat /etc/resolv.conf
resolvectl status               # Trên systemd-resolved
```

### Chỉnh DNS thủ công

```bash
sudo nano /etc/systemd/resolved.conf
# DNS=8.8.8.8 1.1.1.1
sudo systemctl restart systemd-resolved
```

---

## Nguồn tham khảo

- Ubuntu Server Guide — Networking: https://documentation.ubuntu.com/server/
- Ubuntu Netplan docs: https://netplan.readthedocs.io/
- OpenSSH manual: https://man.openbsd.org/ssh
- UFW documentation: https://help.ubuntu.com/community/UFW
