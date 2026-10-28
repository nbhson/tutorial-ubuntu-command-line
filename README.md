# Ubuntu Command Line — Từ Zero đến Admin

> **Môi trường tham chiếu:** Ubuntu 26.04 LTS "Resolute Raccoon" (phát hành 23/04/2026) · APT 3.1 · systemd 257+
> **Ngôn ngữ:** Tiếng Việt · Mỗi bài là một thư mục với `README.md` đầy đủ, có ví dụ chạy được

---

## Mục lục — Thứ tự đọc khuyến nghị

| # | Bài | Nội dung chính | Cấp độ |
|---|---|---|---|
| 1 | [`1_update-upgrade`](1_update-upgrade/README.md) | Cập nhật hệ thống, `apt update/upgrade`, kho phần mềm | ⭐ Mới bắt đầu |
| 2 | [`2_ls-cd-clear`](2_ls-cd-clear/README.md) | Điều hướng: `ls`, `cd`, `clear`, đường dẫn | ⭐ Mới bắt đầu |
| 3 | [`3_mkdir-touch-vi`](3_mkdir-touch-vi/README.md) | Tạo file/thư mục, soạn thảo với `vi`/`nano` | ⭐ Mới bắt đầu |
| 4 | [`4_cat-echo-tail-grep`](4_cat-echo-tail-grep/README.md) | Đọc file: `cat`, `echo`, `tail`, `grep` | ⭐ Mới bắt đầu |
| 5 | [`5_file-permissions`](5_file-permissions/README.md) | Quyền file, `chmod`, `chown`, sudo | ⭐⭐ Cơ bản |
| 6 | [`6_process-management`](6_process-management/README.md) | Tiến trình: `ps`, `top`, `kill`, systemd | ⭐⭐ Cơ bản |
| 7 | [`7_text-processing`](7_text-processing/README.md) | Xử lý text: `awk`, `sed`, `sort`, `cut`, pipe | ⭐⭐ Cơ bản |
| 8 | [`8_networking`](8_networking/README.md) | Mạng: `ping`, `curl`, `ip`, `ss`, `ufw` | ⭐⭐ Cơ bản |
| 9 | [`9_disk-storage`](9_disk-storage/README.md) | Ổ đĩa: `df`, `du`, `lsblk`, mount, LVM | ⭐⭐ Cơ bản |
| 10 | [`10_package-advanced`](10_package-advanced/README.md) | APT nâng cao, `dpkg`, snap, PPA, DEB822 | ⭐⭐⭐ Trung cấp |
| 11 | [`11_environment-variables`](11_environment-variables/README.md) | Biến môi trường, `export`, `.bashrc`, `.profile` | ⭐⭐⭐ Trung cấp |
| 12 | [`12_user-management`](12_user-management/README.md) | User/group, `sudo`, `/etc/passwd` | ⭐⭐⭐ Trung cấp |
| 13 | [`13_find-search`](13_find-search/README.md) | Tìm kiếm: `find`, `grep`, `locate`, `fd`, `xargs` | ⭐⭐⭐ Trung cấp |
| 14 | [`14_ssh-remote`](14_ssh-remote/README.md) | SSH từ xa: ssh, key, `~/.ssh/config`, `scp`, `rsync`, tunnel, hardening | ⭐⭐⭐ Trung cấp |
| 15 | [`15_cron-systemd-timers`](15_cron-systemd-timers/README.md) | Tự động hóa: cron, `at`, systemd timer, log job | ⭐⭐⭐⭐ Nâng cao |
| 16 | [`16_shell-scripting`](16_shell-scripting/README.md) | Viết script bash: biến, điều kiện, hàm, `set -euo pipefail`, `trap` | ⭐⭐⭐⭐ Nâng cao |

---

## Thứ tự học gợi ý

- **Mới làm quen terminal:** 1 → 2 → 3 → 4 (nắm thao tác cơ bản) → 5 → 6.
- **Cần làm việc với server:** 8 (mạng) → 14 (SSH) → 15 (cron/timer) → 16 (scripting) — hoàn thiện vòng tự động hóa: SSH vào server → chạy tác vụ định kỳ → viết script tự động hóa.
- **Đi nhanh theo nhu cầu:** tra Mục lục ở trên, mỗi bài tự chứa đầy đủ (mục lục, use-cases, troubleshooting).

## Kiến thức tích lũy theo lộ trình

```
Quản lý gói & cập nhật (1, 10)
        │
Thao tác file & text (2, 3, 4, 7, 13)
        │
Quyền & user (5, 12)
        │
Tiến trình & tài nguyên (6)
        │
Mạng & SSH (8, 14)
        │
Tự động hóa: cron/timer + shell script (15, 16)
```

## Quy ước trong tài liệu

- Mọi lệnh đều có ví dụ chạy được, kèm comment tiếng Việt.
- Lệnh cần `sudo` được ghi rõ; các thao tác nguy hiểm có cảnh báo ⚠️.
- Mỗi bài kết thúc bằng **Use-cases thực tế**, **Troubleshooting** và **Nguồn tham khảo**.
- Tài liệu tham chiếu Ubuntu 26.04 LTS; các ghi chú "từ Ubuntu 24.04/26.04" đánh dấu thay đổi theo phiên bản.

## Nguồn tham khảo chung

- Ubuntu Server Guide: https://documentation.ubuntu.com/server/
- Ubuntu Desktop & CLI docs: https://help.ubuntu.com/community/CommandLineResources
- Ubuntu 26.04 LTS release notes: https://discourse.ubuntu.com/t/ubuntu-26-04-resolute-raccoon-released/
- Bash Reference Manual: https://www.gnu.org/software/bash/manual/
