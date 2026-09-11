Cron job cũng có thể được cấu hình để chạy một lần, chẳng hạn như khi hệ thống khởi động. Chúng thường được sử dụng cho các tác vụ quản trị như thực hiện sao lưu, dọn dẹp thư mục, v.v. Lệnh `crontab` có thể một tệp cron, tệp này sẽ được cron daemon thực thi theo lịch trình được chỉ định.
Khi được tạo, tệp cron sẽ lưu được lưu trong `/var/spool/cron`dành riêng cho người dùng tạo tệp đó. Mỗi entry trong tệp `crontab` yêu cầu 6 thành phần theo thứ tự sau:
```
minutes, hours, days, months, weeks, commands
```
Ví dụ, entry:
```bash
0 */12 * * * /home/admin/backup.sh
```
sẽ thực thi `/home/admin/backup.sh` 12 giờ một lần.
Crontab của `root` gần như luôn chỉ có thể được chỉnh sửa bởi người dùng `root` hoặc người dùng có sudo privileges đầy đủ. Tuy nhiên, nó vẫn có thể bị khai thác. Bạn có thể thấy script có quyền `world-writable` được Cron chạy với quyền root. Ngay cả khi bạn không thể đọc crontab để biết chính xác lịch thực thi, bạn vẫn có thể xác định tần suất chạy của script. Ví dụ, một backup script tạo ra tệp `.tar.gz` mỗi 12 giờ có thể thấy cho script được thực thi theo chu kỳ đó. Trong trường hợp này, chúng ta có thể append một command vào cuối script, chẳng hạn như một reverse shell one-liner, và command đó sẽ thực thi với quyền root vào lần Cron job tiếp theo chạy.
Một số ứng dụng tạo các tệp cron trong thư mục `/etc/cron.d`. Các tệp này cũng có thể được cấu hình sai, cho phép người dùng không có quyền root chỉnh sửa chúng.
Đầu tiên, hãy kiểm tra hệ thống để tìm các tệp hoặc thư mục có quyền **ghi**. Tệp `backup.sh` trong thư mục `/dmz-backups` đáng chú ý và có vẻ như nó có thể đang được thực thi bởi một cron job.
![[Pasted image 20260910150423.png]]
Kiểm tra nhanh thư mục `/dmz-backups` cho thấy có các file được tạo dường như ba phút một lần. Đây có vẻ là một lỗi cấu hình nghiêm trọng. Có thể system administrator dự định cấu hình cron chạy ba giờ một lần bằng `0 */3 * * *`, nhưng thay vào đó lại ghi `*/3 * * * *`, khiến cron job chạy ba phút một lần. Vấn đề thứ hai là shell script `backup.sh` có quyền world-writable và được thực thi với quyền `root`.
![[Pasted image 20260910154522.png]]

![[Pasted image 20260910155531.png]]

![[Pasted image 20260910161142.png]]

![[Pasted image 20260910161626.png]]

![[Pasted image 20260910161909.png]]