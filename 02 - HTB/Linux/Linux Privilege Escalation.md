## Introduction to Linux Privilege Escalation
Tài khoản root trên các hệ thống Linux cung cấp toàn quyền quản trị đối với hệ điều hành. Trong quá trình thực hiện một bài đánh giá bảo mật, bạn có thể chỉ giành được một phiên shell với quyền hạn thấp trên máy Linux và cần thực hiện leo thang đặc quyền để đạt được quyền root.
Việc chiếm quyền hoàn toàn đối với máy chu cho phép chúng ta thu thập lưu lượng mạng và truy cập các tệp nhạy cảm. Những dữ liệu này có thể được sử dụng để mở rộng phạm vi truy cập trong môi trường mục tiêu. Ngoài ra, nếu máy Linux đã được tham gia miền, chúng ta có thể thu thập NTLM Hash và bắt đầu quá trình thu thập thông tin cũng như tấn công vào AD.
### Enumeration
Enumeration là yếu tố quan trọng nhất trong quá trình leo thang đặc quyền. Có nhiều tập lệnh hỗ trợ, chẳng hạn như LinEnum, giúp tự động hoá quá trình thu thập thông tin. Tuy nhiên, điều quan trọng không kèm là phải hiểu cần tìm kiếm những thông tin nào và có khả năng tự thực hiện quá trình enumeration một cách thủ công.
Khi có được quyền truy cập ban đầu thông qua một phiên shell trên máy mục tiêu, chúng ta cần kiểm tra một số thông tin quan trọng sau:
- OS Version: Việc xác định bản phân phối Linux đang sử dụng, ví dụ như Ubuntu,.. sẽ giúp chúng ta biết được những công cụ nào có thể có sẵn trong hệ thống. Đồng thời, thông tin này cũng cho biết phiên bản hệ điều hành, từ đó có thể xác định xem đã tồn tại mã khai thác công khai dành cho phiên bản đó hay chưa.
- Kernel Version: Một số mã khai thác công khai được phát triển để khai thác các lỗ hổng chỉ tồn tại trên những phiên bản kernel cụ thể. Việc khai thác lỗ hổng kernel có thể làm mất ổn định hoặc thậm chí gây treo hoàn toàn. Vì vậy, cần hết sức cẩn trọng khi sử dụng các khai thác này trên hệ thống đang vân hành và phải hiểu rõ cơ chế hoạt động cũng như những tác động có thể xảy ra trước khi thực thi.
- Running Services: Việc xác định những dịch vụ đang chạy trên máy chủ là rất quan trọng, đặc biệt là các dịch vụ đang được thực thi với quyền root. Một dịch vụ được cấu hình sai hoặc tồn tại lỗ hổng nhưng chạy dưới quyền root có thể tạo ra cơ hội rất thuận lợi để leo thang đặc quyền. Trong thực tế, nhiều lỗ hổng đã được phát hiện trên các dịch vụ phổ biến như Nagios, Exim, Samba, ProFTPd, v.v. Đối với nhiều dịch vụ này đã có sẵn Proof of Concept (PoC) được công bố công khai, chẳng hạn như CVE-2016-9566, một lỗ hổng leo thang đặc quyền cục bộ ảnh hưởng đến Nagios Core phiên bản trước 4.2.4.
#### List Current Processes
```shell
w3bis2bi@htb[/htb]$ ps aux | grep root

root         1  1.3  0.1  37656  5664 ?        Ss   23:26   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    23:26   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    23:26   0:00 [ksoftirqd/0]
root         4  0.0  0.0      0     0 ?        S    23:26   0:00 [kworker/0:0]
root         5  0.0  0.0      0     0 ?        S<   23:26   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        S    23:26   0:00 [kworker/u8:0]
root         7  0.0  0.0      0     0 ?        S    23:26   0:00 [rcu_sched]
root         8  0.0  0.0      0     0 ?        S    23:26   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        S    23:26   0:00 [migration/0]
<SNIP>
```
Installed Packages và Versions: Tương tự như các dịch vụ đang chạy, việc kiểm tra các gói phần mềm đã cài đặt và phiên bản của chúng là rất quan trọng nhằm xác định những gói đã lỗi thời hoặc tồn tại lỗ hổng có thể được khai thác để leo thang đặc quyền.
Một ví dụ điển hình là Screen, một trình quản lý nhiều phiên terminal phổ biến, có chức năng tương tự tmux. Screen cho phép người dùng khởi tạo một phiên làm việc và mở nhiều terminal ảo trong cùng một phiên thay vì phải mở nhiều cửa sổ terminal riêng biệt.
Phiên bản Screen 4.05.00 tồn tại một lỗ hổng leo thang đặc quyền có thể được khai thác tương đối dễ dàng để giành quyền cao hơn trên hệ thống.
Logged in Users: Việc xác định những người dùng hiện đang đăng nhập vào hệ thống và những tác vụ họ đang thực hiện có thể cung cấp nhiều thông tin hữu ích về các hướng di chuyển ngang trong nội bộ cũng như các con đường leo thang đặc quyền tiềm năng.
#### List Current Terminal-Attached Processes
```shell
w3bis2bi@htb[/htb]$ ps au

USER            PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root            1256  0.0  0.1  65832  3364 tty1     Ss   23:26   0:00 /bin/login --
cliff.moore     1322  0.0  0.1  22600  5160 tty1     S    23:26   0:00 -bash
shared          1367  0.0  0.1  22568  5116 pts/0    Ss   23:27   0:00 -bash
root            1384  0.0  0.1  52700  3812 tty1     S    23:29   0:00 sudo su
root            1385  0.0  0.1  52284  3448 tty1     S    23:29   0:00 su
root            1386  0.0  0.1  21224  3764 tty1     S+   23:29   0:00 bash
shared          1397  0.0  0.1  37364  3428 pts/0    R+   23:30   0:00 ps au
```
User Home Directories: Các thư mục Home của những người dùng khác có thể truy cập được hay không? Thư mục Home của người dùng có thể chứa khoá SSH (SSH keys) được sử dụng để truy cập các hệ thống khác, hoặc các script và tệp cấu hình chứa thông tin xác thực.
Trong thực tế, không hiếm trường hợp tìm thấy các tệp chứa tên người dùng, mật khẩu hoặc các thông tin xác thực khác có thể được tận dụng để truy cập các hệ thống khác, thậm chí là xâm nhập vào môi trường Active Directory.
#### Home Directory Contents
```shell
w3bis2bi@htb[/htb]$ ls /home

backupsvc  bob.jones  cliff.moore  logger  mrb3n  shared  stacey.jenkins
```
Chúng ta có thể kiểm tra từng thư mục của người dùng để xác định xem các tệp như `.bash_history` có thể đọc được hay không và có chứa các lệnh đáng chú ý hay không. Đồng thời, cần tìm kiếm các tệp cấu hình cũng như kiểm tra xem có thể thu thập được khóa SSH của người dùng hay không.
#### User's Home Directory Contents
```shell
w3bis2bi@htb[/htb]$ ls -la /home/stacey.jenkins/

total 32
drwxr-xr-x 3 stacey.jenkins stacey.jenkins 4096 Aug 30 23:37 .
drwxr-xr-x 9 root           root           4096 Aug 30 23:33 ..
-rw------- 1 stacey.jenkins stacey.jenkins   41 Aug 30 23:35 .bash_history
-rw-r--r-- 1 stacey.jenkins stacey.jenkins  220 Sep  1  2015 .bash_logout
-rw-r--r-- 1 stacey.jenkins stacey.jenkins 3771 Sep  1  2015 .bashrc
-rw-r--r-- 1 stacey.jenkins stacey.jenkins   97 Aug 30 23:37 config.json
-rw-r--r-- 1 stacey.jenkins stacey.jenkins  655 May 16  2017 .profile
drwx------ 2 stacey.jenkins stacey.jenkins 4096 Aug 30 23:35 .ssh
```
Nếu tìm thấy khoá SSH của chính người dùng hiện tại, khoá này có thể được sử dụng để mở một phiên SSH tới máy chủ, nếu dịch vụ SSH được công khai ra bên ngoài, từ đó có được một phiên làm việc ổn định và tương tác đầy đủ.
Ngoài ra, khoá SSH cũng có thể được tận dụng để truy cập các hệ thống khác trong mạng nội bộ. Ít nhất, bạn có thể kiểm tra ARP cache để xác định các máy chủ khác mà hệ thống đang giao tiếp, sau đó đối chiều với các SSH Private Key có thể sử dụng.
#### SSH Directory Contents
```shell
w3bis2bi@htb[/htb]$ ls -l ~/.ssh

total 8
-rw------- 1 mrb3n mrb3n 1679 Aug 30 23:37 id_rsa
-rw-r--r-- 1 mrb3n mrb3n  393 Aug 30 23:37 id_rsa.pub
```
Việc kiểm tra lịch sử Bash của người dùng cũng rất quan trọng, bởi họ có thể đã truyền mật khẩu trực tiếp dưới dạng tham số trên dòng lệnh, làm việc với Git repository, thiết lập Cron Job hoặc thực hiện nhiều thao tác khác.
Việc xem xét các hoạt động trước đây của người dùng có thể cung cấp nhiều thông tin về vai trò của máy chủ, cũng như gợi ý các hướng leo thang đặc quyền phù hợp.
#### Bash History
```sh
w3bis2bi@htb[/htb]$ history
    1  id
    2  cd /home/cliff.moore
    3  exit
    4  touch backup.sh
    5  tail /var/log/apache2/error.log
    6  ssh ec2-user@dmz02.inlanefreight.local
    7  history
```
Sudo Privileges: Người dùng hiện tại có thể thực thi lệnh dưới quyền của người dùng khác hoặc root hay không?
Nếu không có thông tin xác thực (credentials) của người dùng, có thể bạn sẽ không khai thác được các quyền của sudo. Tuy nhiên, trong nhiều trường hợp, tệp sudoers được cấu hình với tùy chọn NOPASSWD, nghĩa là người dùng có thể thực thi các lệnh được chỉ định mà không cần nhập mật khẩu.
Không phải mọi lệnh có thể chạy dưới quyền root đều dẫn đến leo thang đặc quyền. Tuy nhiên, trong thực tế, không hiếm trường hợp bạn truy cập được vào một tài khoản có toàn quyền sudo, nghĩa là người dùng đó có thể thực thi bất kỳ lệnh nào với quyền root.
Trong trường hợp này, chỉ cần thực thi lệnh:
```shell
sudo su
```
là có thể ngay lập tức nhận được một phiên shell với quyền root.
#### Sudo - List User's Privileges
Configuration Files: Các tệp cấu hình (configuration files) có thể chứa rất nhiều thông tin giá trị. Cần tìm kiếm trong tất cả các tệp có phần mở rộng như `.conf` và `.config` để phát hiện tên người dùng, mật khẩu và các bí mật (secrets) khác.
Readable Shadow File: Nếu tệp `/etc/shadow` có thể đọc được, bạn sẽ thu thập được hash mật khẩu của tất cả người dùng có đặt mật khẩu. Mặc dù điều này không đảm bảo sẽ có thêm quyền truy cập, nhưng các hash này có thể được đưa vào quá trình bẻ khóa ngoại tuyến (offline brute-force/password cracking) nhằm khôi phục mật khẩu gốc (cleartext password).
Password Hashes in `/etc/passwd`: Đôi khi, bạn sẽ bắt gặp hash mật khẩu được lưu trực tiếp trong tệp `/etc/passwd`. Đây là tệp mà mọi người dùng trên hệ thống đều có quyền đọc. Tương tự như các hash trong `/etc/shadow`, những hash này cũng có thể được sử dụng trong các cuộc tấn công bẻ khóa mật khẩu ngoại tuyến.
#### Passwd
```shell
w3bis2bi@htb[/htb]$ cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
<...SNIP...>
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
mrb3n:x:1000:1000:mrb3n,,,:/home/mrb3n:/bin/bash
colord:x:111:118:colord colour management daemon,,,:/var/lib/colord:/bin/false
backupsvc:x:1001:1001::/home/backupsvc:
bob.jones:x:1002:1002::/home/bob.jones:
cliff.moore:x:1003:1003::/home/cliff.moore:
logger:x:1004:1004::/home/logger:
shared:x:1005:1005::/home/shared:
stacey.jenkins:x:1006:1006::/home/stacey.jenkins:
sysadm:$6$vdH7vuQIv6anIBWg$Ysk.UZzI7WxYUBYt8WRIWF0EzWlksOElDE0HLYinee38QI1A.0HW7WZCrUhZ9wwDz13bPpkTjNuRoUGYhwFE11:1007:1007::/home/sysadm:
```
Cron Jobs: Trên các hệ thống Linux, Cron Jobs có chức năng tương tự như Scheduled Tasks trên Windows. Chúng thường được cấu hình để thực hiện các tác vụ bảo trì và sao lưu hệ thống theo lịch.
Khi kết hợp với các lỗi cấu hình khác, chẳng hạn như đường dẫn tương đối (relative paths) hoặc phân quyền không an toàn (weak permissions), Cron Jobs có thể bị khai thác để leo thang đặc quyền (privilege escalation) khi tác vụ theo lịch được thực thi.
#### Cron Jobs
```shell
w3bis2bi@htb[/htb]$ ls -la /etc/cron.daily/

total 60
drwxr-xr-x  2 root root 4096 Aug 30 23:49 .
drwxr-xr-x 93 root root 4096 Aug 30 23:47 ..
-rwxr-xr-x  1 root root  376 Mar 31  2016 apport
-rwxr-xr-x  1 root root 1474 Sep 26  2017 apt-compat
-rwx--x--x  1 root root  379 Aug 30 23:49 backup
-rwxr-xr-x  1 root root  355 May 22  2012 bsdmainutils
-rwxr-xr-x  1 root root 1597 Nov 27  2015 dpkg
-rwxr-xr-x  1 root root  372 May  6  2015 logrotate
-rwxr-xr-x  1 root root 1293 Nov  6  2015 man-db
-rwxr-xr-x  1 root root  539 Jul 16  2014 mdadm
-rwxr-xr-x  1 root root  435 Nov 18  2014 mlocate
-rwxr-xr-x  1 root root  249 Nov 12  2015 passwd
-rw-r--r--  1 root root  102 Apr  5  2016 .placeholder
-rwxr-xr-x  1 root root 3449 Feb 26  2016 popularity-contest
-rwxr-xr-x  1 root root  214 May 24  2016 update-notifier-common
```
Unmounted File Systems and Additional Drives: Nếu phát hiện thêm ổ đĩa hoặc hệ thống tệp chưa được gắn kết, và có quyền thực hiện việc gắn kết, bạn có thể tìm thấy các tệp nhạy cảm, mật khẩu, hoặc các bản sao lưu có thể được tận dụng để leo thang đặc quyền.
# File Systems và Additional Drives
```shell
w3bis2bi@htb[/htb]$ lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   30G  0 disk
├─sda1   8:1    0   29G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1  848M  0 rom
```
**SETUID và SETGID Permissions:** Một số tệp thực thi (binaries) được gán quyền **SETUID** hoặc **SETGID** nhằm cho phép người dùng thực thi một lệnh với quyền của root hoặc của một nhóm khác mà không cần cấp trực tiếp quyền root cho người dùng đó.
Nhiều chương trình có bật **SETUID** hoặc **SETGID** chứa các chức năng có thể bị khai thác để giành được phiên shell với quyền root (root shell).
Writeable Directories: Việc xác định các thư mục có quyền ghi (writeable directories) là rất quan trọng nếu bạn cần tải các công cụ lên hệ thống.
Ngoài ra, bạn có thể phát hiện một thư mục có quyền ghi mà Cron Job sử dụng để lưu trữ tệp. Điều này giúp xác định tần suất thực thi của Cron Job và có thể được tận dụng để leo thang đặc quyền, đặc biệt nếu script mà Cron Job thực thi cũng có quyền ghi.
#### Find Writable Directories
```shell
w3bis2bi@htb[/htb]$ find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null

/dmz-backups
/tmp
/tmp/VMwareDnD
/tmp/.XIM-unix
/tmp/.Test-unix
/tmp/.X11-unix
/tmp/systemd-private-8a2c51fcbad240d09578916b47b0bb17-systemd-timesyncd.service-TIecv0/tmp
/tmp/.font-unix
/tmp/.ICE-unix
/proc
/dev/mqueue
/dev/shm
/var/tmp
/var/tmp/systemd-private-8a2c51fcbad240d09578916b47b0bb17-systemd-timesyncd.service-hm6Qdl/tmp
/var/crash
/run/lock
```
Writeable Files: Có tồn tại script hoặc tệp cấu hình (configuration files) nào mà mọi người dùng đều có quyền ghi (world-writable) hay không?
Mặc dù việc chỉnh sửa các tệp cấu hình có thể gây ảnh hưởng nghiêm trọng đến hệ thống, nhưng trong một số trường hợp, chỉ cần một thay đổi nhỏ cũng có thể mở ra thêm các hướng truy cập.
Tương tự, các script được thực thi với quyền root thông qua Cron Jobs cũng có thể bị chỉnh sửa để bổ sung thêm một lệnh tùy ý, từ đó dẫn đến leo thang đặc quyền khi Cron Job được thực thi.
#### Find Writable Files
```shell
w3bis2bi@htb[/htb]$ find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

/etc/cron.daily/backup
/dmz-backups/backup.sh
/proc
/sys/fs/cgroup/memory/init.scope/cgroup.event_control

<SNIP>

/home/backupsvc/backup.sh

<SNIP>
```
## Environment Enumeration
Environment Enumeration là bước quan trọng nhất trong quá trình Privilege Escalation. Mặc dù có các công cụ như LinPEAS và LinEnum hỗ trợ tự động thu thập thông tin, người kiểm thử vẫn cần hiểu rõ cần tìm kiếm thông tin gì và có khả năng tự thực hiện Enumeration bằng tay.
## Thông tin cần thu thập ban đầu
- **OS Version**: Xác định bản phân phối Linux như Ubuntu, Debian, CentOS, Red Hat, Fedora,... Kiểm tra phiên bản hệ điều hành có còn được hỗ trợ hay đã hết vòng đời (EOL). Tra cứu xem có lỗ hổng hoặc Public Exploit dành cho phiên bản đó hay không.
- **Kernel Version**: Xác định phiên bản Kernel, tìm kiếm các kernerl exploit hoặc local privilege escalation phù hợp. Lưu ý Kernel exploit có thể làm hệ thống mất ổn định hoặc bị treo.
- **Running Services**: Liệt kê các dịch vụ đang chạy, đặc biệt là chạy dưới quyền root, kiểm tra các dịch vụ lỗi thời hoặc có CVE như Nagios, Samba, Exim...

