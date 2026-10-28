# SSH & Remote — Làm việc từ xa (ssh, scp, rsync, ssh-key)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · OpenSSH 10.x
> **Đối tượng:** Người dùng cần SSH vào server, copy file qua mạng, và cấu hình key-based authentication

---

## Mục lục

1. [SSH là gì? Kiến trúc client/server](#1-ssh-là-gì-kiến-trúc-clientserver)
2. [`ssh` — Kết nối cơ bản](#2-ssh--kết-nối-cơ-bản)
3. [`~/.ssh/config` — Quản lý nhiều server](#3-sshconfig--quản-lý-nhiều-server)
4. [SSH key — Đăng nhập không mật khẩu](#4-ssh-key--đăng-nhập-không-mật-khẩu)
5. [`ssh-copy-id` và cài key thủ công](#5-ssh-copy-id-và-cài-key-thủ-công)
6. [`scp` — Copy file qua SSH](#6-scp--copy-file-qua-ssh)
7. [`rsync` — Đồng bộ file thông minh](#7-rsync--đồng-bộ-file-thông-minh)
8. [Hardening server SSH](#8-hardening-server-ssh)
9. [Port forwarding và tunnel](#9-port-forwarding-và-tunnel)
10. [Use-cases thực tế](#10-use-cases-thực-tế)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. SSH là gì? Kiến trúc client/server

**SSH (Secure Shell)** là giao thức đăng nhập và điều khiển máy tính từ xa qua kênh **mã hóa**. SSH hoạt động theo mô hình client/server:

- **Server** — chạy trên máy bị điều khiển, service là `sshd` (port mặc định `22`).
- **Client** — lệnh `ssh` trên máy bạn đang ngồi.
- Toàn bộ phiên (mật khẩu, lệnh, output, file copy) đều được mã hóa.

```
┌──────────────┐        mã hóa        ┌──────────────────┐
│  Laptop bạn  │ ────── SSH ────────▶ │  Server Ubuntu   │
│  ssh client  │        port 22       │  sshd (service)  │
└──────────────┘                      └──────────────────┘
```

Ubuntu 26.04 cài sẵn client `openssh-client`; server cần cài riêng:

```bash
sudo apt install openssh-client    # client (thường đã có)
sudo apt install openssh-server    # server — chỉ cài trên máy bị điều khiển
systemctl status ssh
```

---

## 2. `ssh` — Kết nối cơ bản

```bash
ssh user@hostname                  # Dạng đầy đủ
ssh nbhson@192.168.1.10            # Theo IP
ssh web01                          # Dùng alias trong ~/.ssh/config (mục 3)

ssh -p 2222 user@host              # Khi server chạy port khác 22
ssh -v user@host                   # Verbose — gỡ lỗi kết nối (-vvv chi tiết hơn)
ssh -i ~/.ssh/id_ed25519 user@host # Chỉ định key cụ thể
ssh -t user@host sudo systemctl restart nginx   # Ép cấp TTY cho lệnh cần tương tác
```

### Chạy lệnh trực tiếp mà không vào shell

```bash
ssh user@host "uname -a"
ssh user@host "df -h / && free -h"
ssh user@host "sudo dmesg | tail"           # sudo qua ssh cần -t nếu hỏi mật khẩu
ssh user@host 'bash -s' < script.sh         # Chạy script cục bộ trên máy xa
```

> 💡 Lần đầu kết nối, client hỏi xác nhận **host key** (vân tay của server). Kiểm tra kỹ khi nghi ngờ:
> `ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub` (chạy trên server để in vân tay chuẩn).

---

## 3. `~/.ssh/config` — Quản lý nhiều server

Thay vì gõ `ssh -p 2222 -i ~/.ssh/work_key deploy@203.0.113.10`, khai báo một lần:

```text
# ~/.ssh/config (client)
Host web01
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/work_key
    ServerAliveInterval 60

Host db01
    HostName 10.0.0.5
    User admin
    IdentityFile ~/.ssh/work_key

Host staging
    HostName staging.example.com
    User deploy
    ProxyJump web01        # Đi qua web01 (jump host) rồi mới tới staging
```

```bash
chmod 600 ~/.ssh/config    # Bắt buộc, ssh từ chối nếu file mở rộng
ssh web01                  # Giờ chỉ cần thế này
scp file.tar.gz db01:~/    # scp/rsync/git cũng tự hiểu alias
```

| Chỉ thị hay dùng | Ý nghĩa |
|---|---|
| `HostName` | IP/domain thật |
| `User`, `Port` | Thông tin đăng nhập mặc định |
| `IdentityFile` | Key riêng cho host này |
| `ProxyJump` | Đi qua một server trung gian (thay bastion cũ) |
| `ServerAliveInterval` | Gửi keepalive mỗi n giây — chống bị ngắt kết nối |
| `LocalForward` / `RemoteForward` | Port forwarding (mục 9) |

---

## 4. SSH key — Đăng nhập không mật khẩu

### Tạo cặp key

```bash
ssh-keygen -t ed25519 -C "nbhson@laptop"       # Chuẩn hiện đại, ngắn và nhanh
ssh-keygen -t rsa -b 4096 -C "nbhson@laptop"   # RSA 4096 khi cần tương thích hệ thống cũ
```

- `-t` — thuật toán; **ed25519** là lựa chọn mặc định nên dùng (khóa ngắn, nhanh, bảo mật tốt).
- `-C` — comment ghi chú nguồn gốc key.
- Nhấn Enter để chấp nhận đường dẫn mặc định, đặt **passphrase** để khóa file key.

Kết quả trong `~/.ssh/`:

| File | Vai trò |
|---|---|
| `id_ed25519` | **Private key** — giữ tuyệt đối bí mật, `chmod 600`, không bao giờ copy đi đâu |
| `id_ed25519.pub` | **Public key** — thoải mái đặt lên mọi server bạn muốn vào |

```bash
ls -l ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
```

### Các loại key

| Loại | Dùng khi |
|---|---|
| `ed25519` | Mặc định nên dùng (OpenSSH 6.5+) |
| `ed25519-sk` | Hardware security key (YubiKey, FIDO2) |
| `rsa -b 4096` | Tương thích server/công cụ rất cũ |
| ❌ DSA | Đã bị loại bỏ khỏi OpenSSH |
| ⚠️ rsa nhỏ hơn 2048 | Không dùng |

### Agent — không gõ passphrase nhiều lần

```bash
eval "$(ssh-agent -s)"          # Khởi động ssh-agent (bash)
ssh-add ~/.ssh/id_ed25519       # Thêm key vào agent (hỏi passphrase 1 lần duy nhất)
ssh-add -l                      # Liệt kê key đang được agent giữ
```

---

## 5. `ssh-copy-id` và cài key thủ công

### Cách chuẩn — `ssh-copy-id`

```bash
ssh-copy-id user@host                 # Đẩy public key lên server
ssh-copy-id -i ~/.ssh/id_ed25519.pub web01
ssh-copy-id -p 2222 user@host         # Port khác 22
```

Lệnh này nối thêm key vào `~/.ssh/authorized_keys` phía server và đặt quyền đúng.

### Thủ công (khi không có ssh-copy-id)

```bash
# Trên client — in key ra rồi copy
cat ~/.ssh/id_ed25519.pub

# Trên server — thêm key vào authorized_keys
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "ssh-ed25519 AAAA... nbhson@laptop" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Kiểm tra

```bash
ssh web01                 # Phải vào thẳng, không hỏi mật khẩu
ssh -v web01 2>&1 | grep -i "auth"    # Xem client thử phương thức nào
```

> ⚠️ Trình tự SSH thử: `publickey` → `keyboard-interactive/password` (tùy cấu hình server). Nếu key thất bại, server rơi về hỏi mật khẩu — dấu hiệu cần xem log phía server (mục 11).

---

## 6. `scp` — Copy file qua SSH

Cú pháp giống `cp` nhưng thêm `user@host:` phía máy xa:

```bash
# Đẩy file lên server
scp backup.tar.gz nbhson@192.168.1.10:/home/nbhson/

# Lấy file về máy
scp nbhson@192.168.1.10:/var/log/syslog ./syslog-server.log

# Copy cả thư mục (đệ quy)
scp -r ~/project web01:/var/www/

# Copy giữa 2 server (chạy qua máy bạn) hoặc giữ nguyên thuộc tính
scp -p -r app.conf web01:/etc/myapp/
scp -P 2222 file.txt web01:~/      # -P viết hoa: port (khác cp!)
```

> ⚠️ `scp` giao thức cũ đang bị thay dần: OpenSSH mới mở mặc định dùng **SFTP** phía sau. Với đồng bộ nhiều file, chuyển sang `rsync` (mục 7).

---

## 7. `rsync` — Đồng bộ file thông minh

`rsync` chỉ **truyền phần khác biệt** (delta), có thể tiếp tục khi bị ngắt, và xóa file dư ở đích — phù hợp backup và deploy.

```bash
# Đồng bộ thư mục dự án lên server
rsync -avz ~/project/ web01:/var/www/project/

# Lấy backup về máy
rsync -avz web01:/var/www/html/ ~/backups/html/

# Đồng bộ qua SSH port khác
rsync -avz -e "ssh -p 2222" src/ user@host:/dest/

# Xóa file ở đích đã không còn ở nguồn (mirror)
rsync -avz --delete ~/project/ web01:/var/www/project/

# Mô phỏng trước (dry-run) — bắt buộc trước khi --delete
rsync -avzn --delete ~/project/ web01:/var/www/project/

# Loại trừ
rsync -avz --exclude 'node_modules' --exclude '.git' ~/project/ web01:~/project/

# Copy giữa 2 máy xa (chỉ với rsync >= 3, hoặc qua máy trung gian)
rsync -avz user1@host1:/data/ user2@host2:/backup/
```

| Cờ | Ý nghĩa |
|---|---|
| `-a` | archive: đệ quy + giữ quyền, thời gian, symlink, owner |
| `-v` | hiện từng file |
| `-z` | nén khi truyền |
| `-P` | `--partial --progress`: tiếp tục file dở, hiện tiến trình |
| `-n` | dry-run, chỉ in ra chứ không làm gì |
| `--delete` | xóa ở đích những file không còn ở nguồn |
| `--exclude=` | bỏ qua pattern (dùng nhiều lần được) |

> ⚠️ **Dấu `/` cuối nguồn rất quan trọng:** `~/project/` copy **nội dung** thư mục; `~/project` (không có `/`) copy **chính thư mục** project vào đích.

> 💡 Xem tiến trình: `rsync -avz --info=progress2 src/ dst/`. Ở Ubuntu 26.04, rsync 3.4.x hỗ trợ daemon qua socket và xác thực tối ưu hơn với OpenSSH mới.

---

## 8. Hardening server SSH

Sửa `/etc/ssh/sshd_config` (hoặc file drop-in trong `/etc/ssh/sshd_config.d/` — cách Ubuntu 24.04+ khuyến nghị):

```bash
sudo nano /etc/ssh/sshd_config.d/99-hardening.conf
```

```text
# Chỉ cho đăng nhập bằng key
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes

# Không cho root đăng nhập trực tiếp
PermitRootLogin no

# Giới hạn user được phép SSH
AllowUsers deploy admin

# Chống ngắt phiên khi idle
ClientAliveInterval 300
ClientAliveCountMax 2
```

```bash
sudo sshd -t                      # Kiểm tra cú pháp config TRƯỚC khi restart
sudo systemctl restart ssh
```

> ⚠️ **Không đóng phiên SSH hiện tại trước khi mở một phiên mới kiểm tra thành công.** Sai cấu hình là mất quyền truy cập server (với VPS không có console, hậu quả nặng nề).

### Thay port + tường lửa

```bash
# Đổi Port 22 thành 2222 trong config (giảm scan tự động)
sudo ufw allow 2222/tcp
sudo ufw delete allow ssh        # Chỉ làm SAU khi chắc chắn port mới chạy được
```

### Fail2ban — tự chặn IP dò mật khẩu

```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd     # Xem IP đã bị ban
```

---

## 9. Port forwarding và tunnel

### Local forwarding — truy cập service chỉ nghe localhost trên server

```bash
# Mở http://localhost:8080 phía bạn → port 3000 trên server
ssh -L 8080:localhost:3000 web01

# Ví dụ thực tế: xem Grafana/DB admin chạy local trên server
ssh -L 5432:localhost:5432 db01    # Kết nối Postgres từ máy bạn như localhost
```

### Remote forwarding — mở port máy bạn ra server

```bash
ssh -R 9000:localhost:3000 web01   # Server truy cập port 9000 → app dev trên laptop bạn
```

### Dynamic forwarding (SOCKS proxy)

```bash
ssh -D 1080 -C -N web01            # SOCKS proxy qua server; đặt proxy 127.0.0.1:1080 trong trình duyệt
```

| Tùy chọn | Ý nghĩa |
|---|---|
| `-L local:target:port` | Đưa port máy xa về máy bạn (xem web admin, DB) |
| `-R` | Đẩy port máy bạn ra máy xa |
| `-D` | SOCKS proxy động |
| `-N` | Không chạy lệnh, chỉ giữ tunnel |
| `-f` | Chạy nền sau khi xác thực |

---

## 10. Use-cases thực tế

### Backup hằng ngày bằng rsync + cron

```bash
# /etc/cron.daily/backup-home (xem chi tiết bài 15)
rsync -az --delete -e ssh ~/important/ backup@192.168.1.50:/srv/backup/nbhson/
```

### Deploy ứng dụng nhanh

```bash
rsync -avzn --delete --exclude node_modules ./dist/ deploy@web01:/var/www/app/   # dry-run
rsync -avz --delete ./dist/ deploy@web01:/var/www/app/
ssh deploy@web01 'sudo systemctl reload nginx'
```

### Vào server chỉ cho phép jump host

```bash
ssh -J web01 internal-10-0-0-7        # Hoặc khai báo ProxyJump trong ~/.ssh/config
```

### Chạy nhiều lệnh kiểm tra sức khỏe server

```bash
for h in web01 web02 db01; do
  echo "== $h =="
  ssh "$h" "uptime; df -h / | tail -1"
done
```

### Chuyển máy — đưa key cũ sang máy mới

```bash
# Trên máy mới, khôi phục toàn bộ ~/.ssh từ backup rồi đặt quyền:
chmod 700 ~/.ssh && chmod 600 ~/.ssh/* && chmod 644 ~/.ssh/*.pub
```

---

## 11. Troubleshooting

### `Connection refused`

```bash
systemctl status ssh                      # Server đang chạy chưa?
sudo systemctl enable --now ssh
sudo ufw status                           # Port 22 (hoặc port tùy chỉnh) đã mở chưa?
ss -tlnp | grep ssh                       # sshd đang nghe port nào
```

### `Permission denied (publickey)` — key không được chấp nhận

```bash
ssh -v user@host 2>&1 | grep -iE "identity|offer|denied"   # Xem client đưa key nào
# Phía server xem lý do từ chối:
sudo journalctl -u ssh -n 50
sudo tail -20 /var/log/auth.log
```

Kiểm tra phía server: quyền `~/.ssh` (700), `authorized_keys` (600), chủ sở hữu đúng user.

### `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`

Server cài lại / đổi IP → client từ chối kết nối. Xóa vân tay cũ **chỉ khi bạn chắc chắn** (không phải bị MITM):

```bash
ssh-keygen -R 192.168.1.10     # Xóa dòng cũ trong ~/.ssh/known_hosts rồi SSH lại
```

### Kết nối hay bị ngắt giữa chừng

```bash
# Trong ~/.ssh/config thêm ServerAliveInterval 60 (client)
# Hoặc ClientAliveInterval phía server (mục 8)
```

### Đánh máy sai làm mất quyền truy cập sshd

```bash
# Luôn chạy sshd -t kiểm tra trước khi restart:
sudo sshd -t
# Với console VPS (provider panel) luôn sẵn sàng trước khi chỉnh firewall/port.
```

### Đã khóa chính mình ngoài server

```bash
# Qua console/VNC của provider:
sudo usermod -aG sudo deploy
sudo nano /etc/ssh/sshd_config.d/99-hardening.conf   # Sửa AllowUsers/PermitRootLogin
sudo sshd -t && sudo systemctl restart ssh
```

---

## Nguồn tham khảo

- OpenSSH manual: https://man.openbsd.org/ssh
- Ubuntu Server Guide — OpenSSH Server: https://documentation.ubuntu.com/server/
- `man ssh`, `man ssh_config`, `man sshd_config`, `man scp`, `man rsync`
- rsync documentation: https://download.samba.org/pub/rsync/rsync.1
