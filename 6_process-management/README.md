# Process Management — Quản lý tiến trình (ps, top, kill, systemd)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS (systemd là init mặc định, cgroup v2)
> **Đối tượng:** Người dùng muốn quản lý tiến trình và dịch vụ trên Linux

---

## Mục lục

1. [Khái niệm tiến trình](#1-khái-niệm-tiến-trình)
2. [`ps` — Xem tiến trình](#2-ps--xem-tiến-trình)
3. [`top` và `htop` — Giám sát realtime](#3-top-và-htop--giám-sát-realtime)
4. [`kill`, `killall`, `pkill` — Kết thúc tiến trình](#4-kill-killall-pkill--kết-thúc-tiến-trình)
5. [`nohup`, `&`, `disown` — Chạy nền](#5-nohup---disown--chạy-nền)
6. [Tín hiệu (signals) cơ bản](#6-tín-hiệu-signals-cơ-bản)
7. [systemd và `systemctl`](#7-systemd-và-systemctl)
8. [`journalctl` — Xem log systemd](#8-journalctl--xem-log-systemd)
9. [Các lệnh liên quan khác](#9-các-lệnh-liên-quan-khác)
10. [Use-cases thực tế](#10-use-cases-thực-tế)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Khái niệm tiến trình

**Tiến trình (process)** là một chương trình đang chạy. Mỗi tiến trình có:

- **PID** (Process ID): số định danh duy nhất
- **PPID**: PID của tiến trình cha
- **TTY**: terminal gắn với tiến trình (`?` = không gắn terminal, ví dụ dịch vụ nền)
- **State**: `R` running, `S` sleeping, `D` disk sleep, `Z` zombie, `T` stopped
- **%CPU, %MEM**: mức tiêu thụ tài nguyên

Tiến trình đầu tiên của hệ thống là `systemd` (PID 1). Ubuntu 26.04 mặc định dùng cgroup v2 để nhóm và giới hạn tài nguyên của dịch vụ.

---

## 2. `ps` — Xem tiến trình

```bash
# Tiến trình của phiên terminal hiện tại
ps

# Toàn bộ tiến trình của user hiện tại
ps -u $USER

# Toàn bộ hệ thống (dạng BSD, phổ biến nhất)
ps aux

# Dạng System V, có cây tiến trình
ps -ef

# Cây tiến trình (thấy quan hệ cha-con)
ps auxf
# hoặc
pstree
```

### Đọc hiểu output `ps aux`

```text
USER   PID  %CPU %MEM    VSZ   RSS TTY  STAT START  TIME COMMAND
root     1   0.0  0.1 169300 13064 ?    Ss   09:00  0:02 /sbin/init
nbhson 1234   2.3  5.1 812345 210344 ?   Sl   09:05  5:20 firefox
```

### Lọc process

```bash
# Tìm tiến trình nginx
ps aux | grep nginx

# Chỉ hiện PID của nginx master
pgrep -a nginx

# Xem tiến trình chiếm CPU nhiều nhất
ps aux --sort=-%cpu | head -11

# Xem tiến trình chiếm RAM nhất
ps aux --sort=-%mem | head -11

# Cây tiến trình của 1 PID
ps -f --forest -p 1234
```

---

## 3. `top` và `htop` — Giám sát realtime

### `top` (có sẵn)

```bash
top
```

Phím tắt trong `top`:

| Phím | Chức năng |
|---|---|
| `q` | Thoát |
| `P` | Sắp xếp theo %CPU |
| `M` | Sắp xếp theo %MEM |
| `k` | Kill một tiến trình (nhập PID) |
| `1` | Hiện từng CPU core |
| `Shift+>` / `<` | Đổi cột sắp xếp |
| `u` | Lọc theo user |
| `h` | Trợ giúp |

### `htop` (cần cài, dễ dùng hơn)

```bash
sudo apt install htop
htop
```

`htop` có giao diện màu, hỗ trợ **phím mũi tên di chuyển**, `F9` để kill, `F5` xem dạng cây, `F4` để lọc theo tên, chuột hoạt động được.

### Công cụ khác

```bash
# iotop — giám sát disk I/O
sudo apt install iotop
sudo iotop

# nmon, glances, btop — công cụ giám sát toàn diện
sudo apt install btop
btop
```

---

## 4. `kill`, `killall`, `pkill` — Kết thúc tiến trình

```bash
# kill gửi tín hiệu tới PID
kill 1234                  # Mặc định: SIGTERM (15) — yêu cầu thoát "lịch sự"
kill -9 1234               # SIGKILL — ép chết ngay, không kịp dọn dẹp
kill -SIGTERM 1234         # Tên tín hiệu cũng được
kill -l                    # Liệt kê tất cả tín hiệu

# killall — theo TÊN tiến trình
killall firefox
killall -9 chrome

# pkill — theo tên hoặc pattern (linh hoạt nhất)
pkill firefox
pkill -f "python app.py"   # -f khớp cả dòng lệnh đầy đủ
pkill -u nbhson            # Kill mọi tiến trình của user nbhson
```

> ⚠️ **Thứ tự nên dùng:** luôn thử `kill PID` (SIGTERM) trước, chỉ dùng `kill -9` khi tiến trình "đơ" hoàn toàn. SIGKILL không cho tiến trình cơ hội lưu dữ liệu/cleanup, có thể gây hỏng file hoặc database.

### Lỗi "Operation not permitted" khi kill

```bash
# Tiến trình thuộc user khác → cần sudo
sudo kill -9 1234
```

---

## 5. `nohup`, `&`, `disown` — Chạy nền

### `&` — Chạy lệnh ở background

```bash
sleep 100 &
# [1] 5123      ← job number và PID
jobs           # Xem tiến trình nền của phiên
fg %1          # Đưa tiến trình nền số 1 ra trước
bg %1          # Tiếp tục chạy nền một job đang pause
```

> ⚠️ Lệnh chạy bằng `&` **vẫn bị ngắt khi đóng terminal** (nhận tín hiệu SIGHUP). Muốn sống sót, dùng `nohup` hoặc `disown`.

### `nohup` — Chống SIGHUP

```bash
# Chạy script nền, output ghi vào nohup.out
nohup python3 app.py &

# Chỉ định file log riêng
nohup python3 app.py > app.log 2>&1 &

# Với server chỉ cần đẩy log đi đâu đó
nohup ./server > /dev/null 2>&1 &
```

### `disown` — Tách tiến trình khỏi phiên hiện tại

```bash
long_task            # Đang chạy foreground
Ctrl+Z               # Tạm dừng (STOP)
bg                   # Đưa xuống background
disown %1            # Tách khỏi shell — đóng terminal không kill nó
```

---

## 6. Tín hiệu (signals) cơ bản

| Tín hiệu | Số | Ý nghĩa |
|---|---|---|
| `SIGHUP` (1) | HUP | Terminal đóng — thường dùng để daemon reload config |
| `SIGINT` (2) | INT | Ctrl+C — ngắt tiến trình foreground |
| `SIGKILL` (9) | KILL | Ép tiêu diệt, **không thể bắt/chặn** |
| `SIGTERM` (15) | TERM | Yêu cầu kết thúc "lịch sự" (mặc định của kill) |
| `SIGSTOP` (19) | STOP | Tạm dừng tiến trình |
| `SIGCONT` (18) | CONT | Tiếp tục tiến trình đã dừng |

```bash
# Reload config cho nginx mà không ngắt kết nối
sudo killall -HUP nginx

# Hoặc với systemd
sudo systemctl reload nginx
```

---

## 7. systemd và `systemctl`

Ubuntu dùng **systemd** làm init system (PID 1). Quản lý dịch vụ qua `systemctl`:

### Các lệnh cơ bản

```bash
# Khởi động / dừng / khởi động lại dịch vụ
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Reload config mà không ngắt kết nối (nếu dịch vụ hỗ trợ)
sudo systemctl reload nginx

# Cho phép/không tự khởi động cùng hệ thống
sudo systemctl enable nginx
sudo systemctl disable nginx

# enable + start cùng lúc
sudo systemctl enable --now nginx

# Xem trạng thái
systemctl status nginx
```

### Liệt kê dịch vụ

```bash
systemctl list-units --type=service              # Dịch vụ đang chạy
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service         # Tất cả unit đã biết
systemctl --failed                                # Dịch vụ bị lỗi
```

### Tự viết service file

```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Python App
After=network.target

[Service]
User=nbhson
WorkingDirectory=/home/nbhson/myapp
ExecStart=/usr/bin/python3 /home/nbhson/myapp/app.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
# Load cấu hình mới và bật dịch vụ
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

### Khác biệt service quản lý bằng systemd vs nohup

| | nohup & | systemd |
|---|---|---|
| Tự khởi động lại khi crash | ❌ | ✅ (`Restart=on-failure`) |
| Tự chạy khi boot | ❌ | ✅ (`enable`) |
| Log tập trung | Tự lo | `journalctl` tự thu |
| Phù hợp | Script ngắn, thử nghiệm | Dịch vụ dài hạn |

---

## 8. `journalctl` — Xem log systemd

```bash
# Log của một dịch vụ
journalctl -u nginx

# Log realtime của dịch vụ
journalctl -u nginx -f

# Log từ lần boot gần nhất
journalctl -b

# Theo thời gian
journalctl --since "2026-10-27 09:00" --until "2026-10-27 12:00"
journalctl --since "1 hour ago"

# Chỉ log lỗi trở lên
journalctl -p err
```

---

## 9. Các lệnh liên quan khác

```bash
# Xem tiến trình của user khác
ps -u www-data

# Đếm số tiến trình
ps aux | wc -l

# Tìm tiến trình đang mở port
sudo lsof -i :8080
sudo lsof -i -P -n | grep LISTEN

# Xem process tree ASCII
pstree -p

# Tiến trình chiếm CPU cao nhất (một dòng)
ps -eo pid,comm,%cpu --sort=-%cpu | head -2

# Timeout một lệnh
timeout 10s ./long_script.sh
```

---

## 10. Use-cases thực tế

### Tìm và kill process chiếm port 3000

```bash
sudo lsof -i :3000
# hoặc:
fuser -k 3000/tcp
# hoặc:
pkill -f "node server.js"
```

### Chạy script nền bền bỉ qua SSH

```bash
# Cách nhanh
nohup ./backup.sh > backup.log 2>&1 &

# Cách chuyên nghiệp: tạo systemd service rồi
sudo systemctl start backup
```

### Giải phóng RAM

```bash
free -h                                # Xem RAM
sudo sysctl vm.drop_caches=3           # Xóa page cache (cẩn trọng)
```

### Restart dịch vụ sau khi sửa cấu hình

```bash
sudo systemctl restart nginx
sudo systemctl status nginx
journalctl -u nginx -n 20 --no-pager   # Xem 20 dòng log mới nhất
```

---

## 11. Troubleshooting

### Lệnh không kill được bằng `kill -9`

Tiến trình ở trạng thái `D` (uninterruptible sleep — chờ I/O của kernel, thường do disk/NFS). Chờ hoặc reboot.

### `killall` báo "no process found" nhưng service vẫn chạy

Tên tiến trình có thể khác tên service. Kiểm tra tên thật:

```bash
systemctl status nginx       # Trong phần "Main PID" sẽ hiện lệnh thật
ps aux | grep nginx
```

### Systemd service không chạy, cần xem lỗi

```bash
systemctl status myapp       # Xem trạng thái + vài dòng log
journalctl -u myapp -n 50    # 50 dòng log gần nhất
journalctl -u myapp -f       # Theo dõi realtime
journalctl -xe               # Log hệ thống kèm giải thích lỗi
```

### Quá nhiều zombie process

Zombie (state `Z`) là tiến trình đã chết nhưng parent chưa "thu xác". Nếu parent không thu được, kill parent:

```bash
ps aux | awk '$8=="Z"'                  # Tìm zombie
kill -s SIGCHLD $(pgrep -P PPID)        # Báo parent thu lại
kill -9 PPID                            # Cách cứng: kill cha
```

> 💡 Từ Ubuntu 26.04, hỗ trợ cgroup v1 đã được loại bỏ hoàn toàn — mọi giới hạn tài nguyên qua systemd sử dụng cgroup v2. Lệnh `systemd-cgls` và `systemd-cgtop` giúp xem cây tài nguyên theo cgroup.

---

## Nguồn tham khảo

- systemd documentation: https://www.freedesktop.org/wiki/Software/systemd/
- Ubuntu Server Guide — Service management: https://documentation.ubuntu.com/server/
- `man ps`, `man top`, `man systemctl`, `man journalctl`
