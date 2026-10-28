# Shell Scripting — Viết script bash (shebang, biến, điều kiện, vòng lặp, hàm, set -euo pipefail)

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS · bash 5.2+ · POSIX sh
> **Đối tượng:** Người dùng muốn tự động hóa công việc bằng script chạy được

---

## Mục lục

1. [Script đầu tiên và shebang](#1-script-đầu-tiên-và-shebang)
2. [Biến, chuỗi, tham số dòng lệnh](#2-biến-chuỗi-tham-số-dòng-lệnh)
3. [Điều kiện: `if`, `test`, `[ ]`, `[[ ]]`](#3-điều-kiện-if-test--)
4. [`case` và vòng lặp](#4-case-và-vòng-lặp)
5. [Hàm](#5-hàm)
6. [`set -euo pipefail` — Chế độ an toàn](#6-set--euo-pipefail--chế-độ-an-toàn)
7. [Exit code và `trap` dọn dẹp](#7-exit-code-và-trap-dọn-dẹp)
8. [Đọc input, xử lý text](#8-đọc-input-xử-lý-text)
9. [Debug script](#9-debug-script)
10. [Use-cases thực tế](#10-use-cases-thực-tế)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Script đầu tiên và shebang

```bash
mkdir -p ~/scripts && nano ~/scripts/hello.sh
```

```bash
#!/usr/bin/env bash
# Script đầu tiên — in lời chào
echo "Xin chào, $(whoami)! Bây giờ là $(date '+%H:%M')."
```

```bash
chmod +x ~/scripts/hello.sh    # Cho phép thực thi — bắt buộc
~/scripts/hello.sh             # Chạy bằng đường dẫn
bash ~/scripts/hello.sh        # Hoặc chạy qua bash (không cần chmod)
```

### Shebang — dòng đầu tiên quyết định ai thông dịch script

| Shebang | Ý nghĩa |
|---|---|
| `#!/usr/bin/env bash` | ✅ Khuyến nghị: tìm `bash` qua PATH, chạy được cả macOS/BSD |
| `#!/bin/bash` | OK trên Ubuntu, nhưng hardcoded |
| `#!/bin/sh` | POSIX sh (trên Ubuntu là dash) — nhanh, ít tính năng hơn bash |
| `#!/usr/bin/env python3` | Script Python cũng cùng cơ chế |

> ⚠️ Dòng trống hoặc ký tự Windows (CRLF) trước shebang làm script báo lỗi lạ. Nếu soạn ở Windows: `dos2unix script.sh` hoặc `sed -i 's/\r$//' script.sh`.

---

## 2. Biến, chuỗi, tham số dòng lệnh

### Biến và quy tắc cơ bản

```bash
name="Son"            # KHÔNG có khoảng trắng hai bên dấu =
echo "Xin chào $name"
echo "Xin chào ${name}van"         # ${} giúp tách rõ biên của tên biến (name + "van")
readonly PI=3.14                    # Hằng số, không gán lại được
unset name                          # Xóa biến
```

### Kết quả lệnh vào biến

```bash
today=$(date +%F)          # Dùng $() — KHÔNG dùng backtick cũ
count=$(ls ~/logs | wc -l)
echo "Hôm nay $today có $count log"
```

### Chuỗi

```bash
s="Xin chào"
echo "${#s}"               # Độ dài: 8
echo "${s:0:3}"            # Cắt từ vị trí 0 lấy 3 ký tự: "Xin"
echo "${s/à/a}"            # Thay lần đầu: "Xin chao"
echo "${s,,}"              # Chữ thường toàn bộ; ${s^^} = hoa
```

**Quy tắc vàng: luôn bọc biến trong ngoặc kép `"$var"`** — tránh tách từ (word splitting) khi biến chứa khoảng trắng:

```bash
f="Bao cao nam moi.txt"
rm $f          # ❌ rm nhận 2 đối số: "Bao" và "cao..." → xóa nhầm!
rm "$f"        # ✅
```

### Tham số dòng lệnh

```bash
#!/usr/bin/env bash
echo "Script: $0"
echo "Tham số 1: $1, tham số 2: $2"
echo "Số tham số: $#"
echo "Tất cả: $@"
```

```bash
./deploy.sh web01 prod     # $1=web01, $2=prod
```

Khai báo `set -u` (mục 6) sẽ báo lỗi nếu dùng biến/tham số chưa tồn tại. Kiểm tra tham số bắt buộc:

```bash
if [ $# -lt 1 ]; then
  echo "Cách dùng: $0 <server>" >&2
  exit 1
fi
```

---

## 3. Điều kiện: `if`, `test`, `[ ]`, `[[ ]]`

```bash
if [ "$count" -gt 10 ]; then
  echo "Nhiều hơn 10"
elif [ "$count" -eq 10 ]; then
  echo "Đúng 10"
else
  echo "Ít hơn 10"
fi
```

| Toán tử số | Chuỗi | File |
|---|---|---|
| `-eq` bằng | `=` / `==` | `-e` tồn tại |
| `-ne` khác | `!=` | `-f` là file thường |
| `-gt` / `-ge` | `-z` rỗng | `-d` là thư mục |
| `-lt` / `-le` | `-n` không rỗng | `-r` / `-w` / `-x` đọc/ghi/thực thi |

```bash
[ -f /etc/passwd ] && echo "Có file passwd"
[ -d ~/logs ] || mkdir ~/logs
[ -z "$1" ] && { echo "Thiếu tham số" >&2; exit 1; }
```

> 💡 Bash có `[[ ]]` mạnh hơn `[ ]`: gộp điều kiện `&&`/`||`, khớp pattern `[[ $f == *.tar.gz ]]`, không cần quote biến bên trái. Trong script thuần POSIX (`/bin/sh`) chỉ có `[ ]`.

So sánh số thực/đại số phức tạp → dùng `bc`: `[ "$(echo "3.5 > 3" | bc)" -eq 1 ]`.

---

## 4. `case` và vòng lặp

### `case` — rẽ nhánh theo giá trị

```bash
#!/usr/bin/env bash
case "$1" in
  start)   echo "Khởi động dịch vụ..."; systemctl start myapp ;;
  stop)    echo "Dừng dịch vụ...";     systemctl stop myapp ;;
  restart) systemctl restart myapp ;;
  status)  systemctl status myapp ;;
  *)       echo "Cách dùng: $0 {start|stop|restart}" >&2; exit 1 ;;
esac
```

### `for`

```bash
for f in *.log; do
  echo "Xử lý $f"
  gzip "$f"
done

for i in {1..5}; do echo "Lần $i"; done
for ip in 192.168.1.{1..254}; do ping -c1 -W1 "$ip" &> /dev/null && echo "$ip sống"; done

# Duyệt kết quả lệnh — an toàn tên file có khoảng trắng:
while IFS= read -r line; do
  echo "Dòng: $line"
done < danhsach.txt
```

> ⚠️ Không dùng `for f in $(ls)` — sẽ vỡ với tên file có khoảng trắng. Dùng glob `*` hoặc `while read`.

### `while` và `until`

```bash
count=1
while [ "$count" -le 3 ]; do
  echo "Lần $count"
  count=$((count + 1))
done

# Chờ tới khi một service sẵn sàng (timeout 30s):
for i in $(seq 30); do
  curl -sf http://localhost:8080/health && break
  sleep 1
done
```

---

## 5. Hàm

```bash
#!/usr/bin/env bash
log() {                       # Định nghĩa hàm
  echo "[$(date '+%F %T')] $*" >> /var/tmp/myapp.log
}

greet() {
  local name="$1"             # local: biến chỉ tồn tại trong hàm
  echo "Xin chào, $name!"
}

check_port() {                # Hàm trả exit code
  local host="$1" port="$2"
  nc -z "$host" "$port"       # exit code 0 nếu port mở
}

greet "Son"                   # Gọi hàm, tham số như script
log "Đã chạy xong backup"
if check_port localhost 22; then echo "SSH đang mở"; fi
```

- `local` — tránh làm bẩn biến toàn cục.
- Hàm "trả về giá trị" bằng cách `echo` ra stdout rồi gọi: `result=$(greet "Son")`.
- Exit code của hàm là exit code của lệnh cuối, hoặc `return n`.

---

## 6. `set -euo pipefail` — Chế độ an toàn

Đặt ngay sau shebang trong **mọi** script nghiêm túc:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

| Cờ | Ý nghĩa | Không có nó thì sao? |
|---|---|---|
| `-e` | Thoát ngay khi lệnh nào trả exit code khác 0 | Script tiếp tục chạy với dữ liệu sai |
| `-u` | Báo lỗi khi dùng biến chưa khai báo | `rm -rf "$TYPo/data"` thành `rm -rf /data` |
| `-o pipefail` | Pipe trả exit code của lệnh **đầu tiên** thất bại | Lỗi giữa pipeline bị `| tail` che mất |

Ví dụ từ chính bài 15:

```bash
#!/bin/bash
set -euo pipefail
STAMP=$(date +%F_%H%M)
pg_dump mydb | gzip > "/srv/backup/mydb_$STAMP.sql.gz"   # pg_dump lỗi → thoát ngay
find /srv/backup -name "mydb_*.sql.gz" -mtime +14 -delete
```

### Lưu ý với `set -e`

```bash
# Lệnh được phép thất bại thì viết tường minh:
grep -q "ready" status.txt || echo "Chưa sẵn sàng"

if grep -q "error" app.log; then ...; fi      # if "ăn" exit code, không thoát

# Lệnh nào là "không quan trọng" có thể:
set +e
risky_command
set -e
```

---

## 7. Exit code và `trap` dọn dẹp

- **`exit 0`** — thành công; **`exit 1..255`** — thất bại.
- `$?` — exit code của lệnh vừa chạy.
- Quy ước: `0` thành công, `1` lỗi chung, `2` dùng sai cú pháp, `127` không tìm thấy lệnh.

```bash
#!/usr/bin/env bash
set -euo pipefail

TMP_DIR=$(mktemp -d)
cleanup() {
  rm -rf "$TMP_DIR"
  echo "Đã dọn dẹp"
}
trap cleanup EXIT            # Chạy cleanup dù thành công, lỗi hay Ctrl+C

rsync -az /data/ "$TMP_DIR/"    # Nếu lỗi, trap vẫn chạy và thoát với code lỗi
```

`trap` các tín hiệu hay dùng: `EXIT` (luôn), `INT` (Ctrl+C), `TERM` (kill mặc định).

---

## 8. Đọc input, xử lý text

```bash
# Đọc input từ người dùng
read -rp "Tên server: " server
read -rsp "Mật khẩu: " pass; echo    # -s: không hiện ký tự

# Đọc từng dòng file
while IFS= read -r line; do
  echo "→ $line"
done < danh-sach.txt

# Đọc tham số có cờ kiểu --flag
while [[ $# -gt 0 ]]; do
  case "$1" in
    --host) host="$2"; shift 2 ;;
    --dry)  dry=1; shift ;;
    *)      echo "Tham số lạ: $1" >&2; exit 1 ;;
  esac
done
```

Kết hợp với `grep`/`awk`/`sed` (đã học ở bài 7):

```bash
IP=$(hostname -I | awk '{print $1}')
USERS=$(grep -c "login" /var/log/auth.log || true)
```

---

## 9. Debug script

```bash
bash -n script.sh        # Chỉ kiểm tra cú pháp, không chạy
bash -x script.sh        # In từng lệnh sau khi mở biến — vết đầy đủ
bash -v script.sh        # In từng dòng khi đọc
```

Debug trong code:

```bash
set -x          # Bật trace
phần nghi vấn
set +x          # Tắt trace
```

```bash
# Log ra stderr thay vì stdout để không làm bẩn kết quả pipe:
echo "Debug: giá trị = $val" >&2
```

Công cụ kiểm tra tĩnh (khuyến nghị cho script lớn):

```bash
sudo apt install shellcheck
shellcheck ~/scripts/*.sh       # Chỉ ra lỗi và thói quen xấu rất chi tiết
```

---

## 10. Use-cases thực tế

### Script backup có log, khóa chống chồng, dọn dẹp

```bash
#!/usr/bin/env bash
set -euo pipefail
exec 9>/tmp/backup.lock
flock -n 9 || { echo "Backup đang chạy" >&2; exit 0; }

DEST="/srv/backup/$(date +%F)"
mkdir -p "$DEST"
rsync -az --delete /home/nbhson/important/ "$DEST/"
find /srv/backup -maxdepth 1 -mtime +14 -exec rm -rf {} +
curl -s -o /dev/null -X POST "$WEBHOOK_URL" -d "backup ok $(date)"
```

### Giám sát đơn giản: cảnh báo khi ổ đầy

```bash
#!/usr/bin/env bash
set -euo pipefail
usage=$(df --output=pcent / | tail -1 | tr -dc '0-9')
if [ "$usage" -ge 90 ]; then
  echo "Ổ / đã dùng ${usage}%!" | tee /dev/stderr
  curl -s -X POST "https://hooks.slack.com/..." -d "text=Disk ${usage}%"
fi
```

### Vòng lặp SSH kiểm tra nhiều server

```bash
#!/usr/bin/env bash
servers=(web01 web02 db01)
for s in "${servers[@]}"; do
  echo "== $s =="
  ssh -o ConnectTimeout=5 "$s" 'uptime; df -h / | tail -1' || echo "⚠ Không SSH được $s" >&2
done
```

### Cài đặt phần mềm cho máy mới (provision)

```bash
#!/usr/bin/env bash
set -euo pipefail
sudo apt update
for pkg in curl git htop ripgrep fd-find fail2ban; do
  dpkg -s "$pkg" &>/dev/null || sudo apt install -y "$pkg"
done
```

---

## 11. Troubleshooting

### `syntax error near unexpected token` / ký tự lạ

```bash
file script.sh          # Có CRLF line terminators không?
sed -i 's/\r$//' script.sh   # Hoặc dos2unix
bash -n script.sh       # Tìm vị trí lỗi cú pháp
```

### `command not found` khi chạy bằng cron (liên hệ bài 15)

Dùng đường dẫn tuyệt đối hoặc set `PATH` đầu script. Cron không đọc `.bashrc` của bạn.

### `unbound variable` sau khi bật `set -u`

Biến chưa khai báo. Khai báo mặc định an toàn: `PORT="${PORT:-8080}"`.

### Script in ra màn hình nhưng pipe mất dữ liệu

```bash
# Thiếu dòng shebang hoặc CRLF; kiểm tra:
head -1 script.sh | od -c | head -2
```

### `[: too many arguments` — biến rỗng/chứa khoảng trắng chưa quote

```bash
if [ "$f" = "a b" ]; then     # Luôn quote biến trong [ ]
```

### Pipe âm thầm nuốt lỗi

```bash
set -o pipefail   # Không có cờ này, `false | true` trả về 0
```

### Vòng lặp `for f in $(ls)` vỡ với tên file có khoảng trắng

Dùng glob `for f in *.txt` hoặc `while IFS= read -r f; do ... done < <(find ...)`.

---

## Nguồn tham khảo

- Bash Reference Manual: https://www.gnu.org/software/bash/manual/
- Google Shell Style Guide: https://google.github.io/styleguide/shellguide.html
- ShellCheck — static analysis: https://www.shellcheck.net/
- `man bash`, `man bash-builtins`, `man flock`
- Ubuntu Server Guide: https://documentation.ubuntu.com/server/
