# Update, Upgrade Packages

## Lệnh apt-get update

> sudo apt update

- Lệnh `sudo apt update` được sử dụng để  kết nối với internet (cái kho package) và cập nhật **thông tin** những package mới nhất về máy
- Thông tin về những package (ĐƯỢC xác định) đó sẽ được lưu trữ tại */etc/apt/sources.list*
  + Cách xem tệp `/etc/apt/sources.list`, gõ lệnh cat: `cat /etc/apt/sources.list`
- Thông tin về những package (KHÔNG xác định) đó sẽ được lưu trữ tại */etc/apt/sources.list.d*

## Lệnh apt-get upgrade

> sudo apt upgrade

Lệnh `sudo apt upgrade` được sử dụng để  **tải xuống** và **cài đặt** những package bằng thông tin của apt-get update đã lấy về trước đó thông qua tệp sources.list

> Các gói mới sẽ được cài đặt nếu được yêu cầu để đáp ứng các phụ thuộc, nhưng các gói hiện có sẽ không bao giờ bị xóa.

## Cài một package 

> sudo apt install [tên_package]