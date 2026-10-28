# Cron & Systemd Timers — Tự động hóa tác vụ theo lịch (cron, at, systemd timer)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · cron 3.0pl1 · systemd 257+ · at
> **Đối tượng:** Người dùng cần chạy tác vụ định kỳ (backup, dọn dẹp, đồng bộ) hoặc một lần vào thời điểm cụ thể

---

## Mục lục

1. [Tổng quan: 3 cách chạy việc theo lịch](#1-tổng-quan-3-cách-chạy-việc-theo-lịch)
2. [Cú pháp crontab — 5 trường thời gian](#2-cú-pháp-crontab--5-trường-thời-gian)
3. [`crontab` — Quản lý lịch của user](#3-crontab--quản-lý-lịch-của-user)
4. [Lịch của hệ thống: `/etc/cron.*` và `/etc/crontab`](#4-lịch-của-hệ-thống-etccron-và-etccrontab)
5. [`at` — Chạy một lần vào thời điểm cụ thể](#5-at--chạy-một-lần-vào-thời-điểm-cụ-thể)
6. [`systemd timer` — Thay thế cron hiện đại](#6-systemd-timer--thay-thế-cron-hiện-đại)
7. [`systemd timer` — Persistent, calendar, on-boot](#7-systemd-timer--persistent-calendar-on-boot)
8. [Ghi log và gửi email kết quả](#8-ghi-log-và-gửi-email-kết-quả)
9. [Use-cases thực tế](#9-use-cases-thực-tế)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Tổng quan: 3 cách chạy việc theo lịch

| Công cụ | Loại lịch | Ưu điểm | Khi nào dùng |
|---|---|---|---|
| **cron** | Định kỳ theo giờ/ngày/tháng | Đơn giản, quen thuộc, cài sẵn | Tác vụ user/vệ sinh hệ thống đơn giản |
| **`at`** | Một lần duy nhất | Viết nhanh | "Chạy lệnh này lúc 22h tối nay" |
| **systemd timer** | Định kỳ hoặc sự kiện | Log qua journal, phụ thuộc service, bắt bù giờ (persistent) | Tác vụ hệ thống, cần log và dependency |

> 💡 Ubuntu 26.04: phần lớn việc theo lịch của chính hệ thống đã chuyển sang **systemd timer** (`apt-daily.timer`, `fstrim.timer`, `plocate-updatedb.timer`...). Xem: `systemctl list-timers`.

---

## 2. Cú pháp crontab — 5 trường thời gian

```text
┌───────────── phút        (0–59)
│ ┌───────────── giờ       (0–23)
│ │ ┌───────────── ngày     (1–31)
│ │ │ ┌───────────── tháng   (1–12)
│ │ │ │ ┌───────────── thứ    (0–7, 0 và 7 = Chủ nhật)
│ │ │ │ │
* * * * *  lệnh-cần-chạy
```

| Ví dụ | Ý nghĩa |
|---|---|
| `* * * * *` | Mỗi phút |
| `30 2 * * *` | 02:30 hằng ngày |
| `0 4 * * 0` | 04:00 Chủ nhật |
| `0 0 1 * *` | 00:00 ngày đầu tháng |
| `*/10 * * * *` | Mỗi 10 phút |
| `0 9-17 * * 1-5` | 9h–17h, thứ 2 đến thứ 6 |
| `0 6 1,15 * *` | 6h ngày 1 và 15 hằng tháng |
| `0 3 * * mon` | 3h sáng mỗi thứ Hai |

### Giá trị đặc biệt

```text
@reboot   lệnh     # Chạy lúc khởi động máy
@daily    lệnh     # = 0 0 * * *
@hourly   lệnh     # = 0 * * * *
@weekly   lệnh     # = 0 0 * * 0
@monthly  lệnh     # = 0 0 1 * *
```

> ⚠️ **Bẫy phổ biến:** `%` trong crontab có ý nghĩa đặc biệt (kết thúc phần lệnh). Khi cần `%` (ví dụ `date +%F`), viết là `\%`.

---

## 3. `crontab` — Quản lý lịch của user

```bash
crontab -e            # Sửa crontab của user hiện tại (lần đầu hỏi chọn editor)
crontab -l            # Xem lịch hiện tại
crontab -l > backup-cron.txt     # Sao lưu trước khi sửa
crontab -r            # XÓA toàn bộ crontab (không hỏi lại!) — cẩn thận
sudo crontab -e       # Sửa crontab của root
sudo crontab -l -u deploy   # Xem crontab của user khác
```

### Ví dụ crontab user

```text
# m  h  dom mon dow   lệnh
  0  3  *   *   *   ~/scripts/backup.sh >> ~/logs/backup.log 2>&1
  */15 * * * *   curl -s https://api.example.com/health > /dev/null
  0  4  *   *   0   find ~/Downloads -mtime +30 -delete
  @reboot            ~/scripts/watchdog.sh
```

### Môi trường chạy của cron — khác terminal của bạn!

Cron chạy với môi trường tối giản: `PATH` mặc định là `/usr/bin:/bin`, không có biến từ `~/.bashrc`.

- Dùng **đường dẫn tuyệt đối** cho mọi lệnh: `/usr/bin/rsync`, `/usr/local/bin/myscript`.
- Đặt biến ngay đầu crontab nếu cần:

```text
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
SHELL=/bin/bash
MAILTO=nbhson@example.com
```

- Script cần biến môi trường riêng thì source file trong lệnh:
  `0 3 * * * . $HOME/.profile; /home/nbhson/scripts/job.sh`

---

## 4. Lịch của hệ thống: `/etc/cron.*` và `/etc/crontab`

Ubuntu chia lịch hệ thống theo tần suất thành các thư mục — chỉ cần **thả script chạy được** vào:

```bash
ls /etc/cron.hourly /etc/cron.daily /etc/cron.weekly /etc/cron.monthly
sudo nano /etc/cron.daily/backup-home
```

```bash
#!/bin/bash
rsync -az --delete /home/nbhson/important/ /srv/backup/nbhson/
```

```bash
sudo chmod 755 /etc/cron.daily/backup-home    # Phải chạy được và không có đuôi mở rộng
```

- Thời điểm chạy của `cron.daily` do file `/etc/crontab` quy định (mặc định 6h25).
- `/etc/cron.d/` chứa file crontab hệ thống theo cú pháp đầy đủ (**có thêm trường user**):

```text
# /etc/cron.d/sync-data  — chú ý trường "root" giữa lịch và lệnh
30 2 * * *  root  /usr/local/bin/sync-data.sh
```

---

## 5. `at` — Chạy một lần vào thời điểm cụ thể

```bash
sudo apt install at          # Nếu chưa có

echo "sudo apt update && sudo apt upgrade -y" | at 22:00
at 2am tomorrow              # Vào shell tương tác của at
at now + 30 minutes
at 10:15 PM Sun

# Cách dùng tương tác:
at 22:00
> /home/nbhson/scripts/big-cleanup.sh
> Ctrl+D     # Kết thúc (EOT)
```

```bash
atq                # Xem các job đang chờ (batch queue)
atrm 3             # Xóa job số 3
at -c 3            # In nội dung job số 3
```

Định dạng thời gian linh hoạt: `now + 2 hours`, `noon`, `midnight`, `teatime` (16:00), `10:15 15.09.2026`...

---

## 6. `systemd timer` — Thay thế cron hiện đại

Một timer gồm **2 file đơn vị** cặp đôi: `xxx.service` (cái việc cần làm) và `xxx.timer` (cái hẹn giờ).

```bash
sudo nano /etc/systemd/system/backup-home.service
```

```ini
[Unit]
Description=Backup thư mục important của nbhson

[Service]
Type=oneshot
User=nbhson
ExecStart=/usr/local/bin/backup-home.sh
```

```bash
sudo nano /etc/systemd/system/backup-home.timer
```

```ini
[Unit]
Description=Chạy backup-home hằng ngày lúc 3h sáng

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload          # Bắt buộc sau khi tạo/sửa file unit
sudo systemctl enable --now backup-home.timer
systemctl list-timers --all           # Xem timer đã active và lần chạy kế tiếp
systemctl list-timers backup-home.timer
```

`OnCalendar` dùng cú pháp **calendar events** của systemd, linh hoạt hơn crontab:

| `OnCalendar=` | Ý nghĩa |
|---|---|
| `*-*-* 03:00:00` | 3h sáng hằng ngày |
| `daily` | Bí danh của 00:00 hằng ngày |
| `*-*-* *:00/15:00` | Mỗi 15 phút |
| `Mon *-*-* 09:00:00` | 9h sáng thứ Hai |
| `*-*-01 06:00:00` | 6h sáng ngày 1 hằng tháng |

Kiểm tra một biểu thức calendar sẽ chạy khi nào:

```bash
systemd-analyze calendar "*-*-* 03:00:00"
systemd-analyze calendar "Mon *-*-* 09:00:00"
```

---

## 7. `systemd timer` — Persistent, calendar, on-boot

| Tùy chọn `[Timer]` | Ý nghĩa |
|---|---|
| `OnCalendar=` | Chạy theo lịch thời gian thực |
| `OnBootSec=10min` | Chạy n giây sau khi boot |
| `OnUnitActiveSec=1h` | Chạy mỗi 1h kể từ lần hoạt động trước |
| `Persistent=true` | **Bắt bù giờ:** máy tắt đúng giờ hẹn thì chạy bù ngay khi boot |
| `RandomizedDelaySec=5min` | Tránh nhiều timer dồn vào một thời điểm |
| `AccuracySec=1min` | Độ chính xác của lịch (mặc định 1 phút) |

Chạy ngay thử (không cần chờ đến giờ hẹn):

```bash
sudo systemctl start backup-home.service   # Chạy service thủ công
journalctl -u backup-home.service -n 20    # Xem kết quả
```

### Vì sao nên dùng timer thay cron cho việc hệ thống?

- **Log tự động** vào journal — không phải tự redirect ra file.
- **Persistent**: cron bỏ qua lần chạy nếu máy đang tắt; timer chạy bù.
- **Dependency**: khai báo `After=network-online.target`, `Wants=`... trong `[Unit]`.
- Giới hạn tài nguyên: `MemoryMax=`, `CPUQuota=` ngay trong service.

> ⚠️ Timer không set `Persistent=true` sẽ bỏ lỡ lần chạy khi máy tắt — tương tự nhược điểm của cron.

---

## 8. Ghi log và gửi email kết quả

### Redirect output của cron

Cron mặc định **gửi email** cho owner crontab. Trên server không có MTA, output bị bỏ — hãy tự redirect:

```text
0 3 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
0 3 * * * /usr/local/bin/backup.sh > /dev/null 2>&1        # Bỏ hoàn toàn output
```

`2>&1` gộp cả stderr vào log — không có nó bạn sẽ **mất toàn bộ thông báo lỗi**.

### Gửi email/thông báo khi cron chạy

```bash
# 1. Đặt MAILTO trong crontab (cần MTA như postfix/msmtp đã cấu hình)
MAILTO=admin@example.com

# 2. Hoặc tự gửi qua API (Telegram/Slack) trong script:
curl -s -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
     -d chat_id="$CHAT_ID" -d text="Backup xong: $(date)"
```

### Log của systemd timer

```bash
journalctl -u backup-home.service            # Toàn bộ lịch sử chạy
journalctl -u backup-home.service --since today
journalctl -u backup-home.service -f         # Theo trực tiếp
```

---

## 9. Use-cases thực tế

### Backup database hằng đêm

```text
# crontab -e (root)
0 2 * * * /usr/local/bin/db-backup.sh >> /var/log/db-backup.log 2>&1
```

```bash
#!/bin/bash
# /usr/local/bin/db-backup.sh
set -euo pipefail
STAMP=$(date +\%F_\%H%M)                       # % phải escape trong crontab
pg_dump mydb | gzip > /srv/backup/mydb_$STAMP.sql.gz
find /srv/backup -name "mydb_*.sql.gz" -mtime +14 -delete
```

### Dọn dẹp /tmp và log cũ hàng tuần

```text
0 4 * * 0  find /tmp -type f -atime +7 -delete 2>/dev/null
```

### Kiểm tra website mỗi 5 phút và cảnh báo

```text
*/5 * * * * /usr/local/bin/healthcheck.sh > /dev/null 2>&1
```

### Đồng bộ file qua rsync mỗi giờ (systemd timer)

```ini
[Timer]
OnCalendar=hourly
Persistent=true
RandomizedDelaySec=10min
```

### Ghi chú lịch chạy khi xem máy của người khác

```bash
crontab -l                       # Lịch user
sudo crontab -l                  # Lịch root
ls /etc/cron.d/ /etc/cron.daily/ # Lịch hệ thống
systemctl list-timers --all      # Timer systemd
```

---

## 10. Troubleshooting

### Cron không chạy — checklist

```bash
systemctl status cron                 # Service cron đang chạy?
grep CRON /var/log/syslog | tail      # Ubuntu có ghi lại mỗi lần cron kích hoạt job
sudo journalctl -u cron -n 20
```

1. Đường dẫn tuyệt đối chưa? (`which rsync` để lấy đường dẫn)
2. Script có quyền chạy? (`chmod +x script.sh`)
3. Output bị đưa đi đâu? Đã redirect `>> file 2>&1`?
4. Cron chạy user nào? Test lại môi trường: `crontab` dòng `* * * * * env > /tmp/cron-env.txt` rồi so sánh với `env` trong terminal.

### Cron chạy bị lỗi ký tự lạ / lệnh không tìm thấy

```bash
# PATH tối giản của cron — dùng đường dẫn đầy đủ hoặc đặt PATH đầu crontab (mục 3)
```

### Cron không chạy lúc máy tắt

Chuyển sang systemd timer với `Persistent=true` (mục 6–7) để chạy bù.

### `crontab -r` xóa nhầm lịch

Tập thói quen sao lưu: `crontab -l > ~/crontab-backup.txt` trước mỗi lần `crontab -e`. Khôi phục: `crontab ~/crontab-backup.txt`.

### Timer systemd không kích hoạt

```bash
systemctl list-timers backup-home.timer     # Có trong danh sách chưa?
systemctl status backup-home.timer          # Enabled? Next elapse?
sudo systemctl daemon-reload                # Quên reload sau khi sửa file?
journalctl -u backup-home.service -n 50     # Lỗi của service bên dưới
```

### Job chạy quá lâu chồng lên lần kế tiếp

Thêm khóa chống chạy đồng thời bằng `flock`:

```text
0 3 * * * /usr/bin/flock -n /tmp/backup.lock /usr/local/bin/backup.sh
```

---

## Nguồn tham khảo

- cron manual: `man 5 crontab`, `man 8 cron`
- at manual: `man at`, `man atq`
- systemd.timer: https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html
- systemd time/calendar syntax: https://www.freedesktop.org/software/systemd/man/latest/systemd.time.html
- Ubuntu Server Guide — Scheduling: https://documentation.ubuntu.com/server/
