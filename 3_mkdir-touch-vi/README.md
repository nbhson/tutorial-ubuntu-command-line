# mkdir, touch, vi

## Lệnh mkdir (make directory) - Tạo thư mục

> mkdir ten_thu_muc
> mkdir parent/child1/child2
Khi tạo các thư mục lồng nhau, mà đường dẫn ở giữa chưa tồn tại thư mục trung gian đó thì sẽ sinh ra lỗi. Nên thư mục trung gian phải tồn tại

> mkdir parent/child1/child2 -p
Sẽ tạo hết các thư mục con bên trong mà trung gian không tồn tại luôn

## Lệnh rmdir (remove directory) - Xoá thư mục trống

> rmdir ten_thu_muc
Xóa thư mục

> rm -r ten_thu_muc/ten_file
Xóa thư mục và các thư mục/tệp bên trong

## Lệnh touch - Tạo ra một file không có nội dung

> touch ten_file

## Lệnh vi (VIM) - Editor được cài mặc định trên những hệ điều hành Unix (Linux, Mac)

> vi ten_file
Tạo một file bằng VIM

> i: chế độ insert