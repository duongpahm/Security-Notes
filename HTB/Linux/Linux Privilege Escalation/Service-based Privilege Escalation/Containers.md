Container hoạt động ở cấp độ hệ điều hành, trong khi máy ảo hoạt động ở cấp độ phần cứng. Do đó, các container dùng chung một hệ điều hành và cô lập các tiến trình ứng dụng khỏi phần còn lại của hệ thống, trong khi cơ chế virtualization truyền thống cho phép nhiều hệ điều hành chạy đồng thời trên cùng một hệ thống.
Isolation và virtualization là những yếu tố thiết yếu vì chúng giúp quản lý tài nguyên các khía cạnh bảo mật một cách hiệu quả nhất có thể. Ví dụ, chúng hỗ trợ việc monitoring để phát hiện các lỗi trong hệ thống.
## Linux Containers
Linux Containers (`LXC`) là một kỹ thuật virtualization ở cấp độ hệ điều hành, cho phép hệ thống Linux chạy isolated với nhau, trên một host duy nhất bằng cách sở hữu các process riêng nhưng dùng chung kernel của host system.
LXC rất phổ biến nhờ tính dễ sử dụng và đã trở thành một thành phần thiết yếu trong IT security. Theo mặc định, `LXC` tiêu thụ ít tài nguyên hơn virtual machine và cung cấp một standard interface, giúp dễ dàng quản lý nhiều container đồng thời. Một platform sử dụng `LXC` thậm chí có thể được tổ chức trên nhiều cloud khác nhau, cung cấp khả năng portability và đảm bảo rằng các application hoạt động chính xác trên hệ thống của developer cũng sẽ hoạt động trên bất kỳ hệ thống nào khác. Ngoài ra, các appilication lớn có thể được start, stop hoặc thay đổi environment variables thông qua Linux container interface. Tính dễ sử dụng của `LXC` là ưu điểm lớn nhất so với kỹ thuật virtualization truyền thống. Tuy nhiên, sự phổ biến rộng rãi của `LXC`, một ecosystem gần như bao phủ toàn bộ lĩnh vực này và các công cụ mang tính đổi mới chủ yếu đến từ Docker platform, nền tảng đã góp phần thiết lập Linux containers.
Toàn bộ quá trình setup, từ việc tạo container templates và deploying chúng, cấu hình operating system và networking cho đến deploying applications, về cơ bản vẫn giữ nguyên.
#### Linux Daemon
Linux Daemon (LXD) tương tự ở một số khía cạnh nhưng được thiết kế để chứa một operating system hoàn chỉnh. Do đó, đây không phải là một application container mà là một system container.
Trước khi có thể sử dụng service này để thực hiện escalation, chúng ta phải thuộc một trong hai group là `lxc` hoặc `lxd`. Chúng ta có thể kiểm tra điều này bằng command `id`:
```shell
container-user@nix02:~$ id 
uid=1000(container-user) gid=1000(container-user) groups=1000(container-user),116(lxd)
```

![[Pasted image 20260910172607.png]]
Từ đây, có một số cách để exploit `LXC`/`LXD`. Chúng ta có thể tự tạo một container và transfer nó sang target system hoặc sử dụng một container có sẵn.
Đáng tiếc là administrators thường sử dụng những templates có rất ít hoặc thậm chí không có security. Cách tiếp cận này dẫn đến việc chúng ta có sẵn các tools có thể sử dụng để tấn công hệ thống.
![[Pasted image 20260910172659.png]]
Những templates như vậy thường không có password, đặc biệt nếu chúng sử dụng trong các test environments đơn giản. Những môi trường này cần có khả năng truy cập nhanh và sử dụng dễ dàng.
Việc tập trung quá nhiều vào security sẽ làm phức tạp toàn bộ quá trình khởi tạo, khiến nó khó khăn hơn và do đó làm chậm đáng kể quá trình triển khai. Nếu chúng ta may mắn và có một container như vậy trên system, container đó có thể bị exploit. Để thực hiện điều này, chúng ta cần import container đó dưới dạng một image.
![[Pasted image 20260910173055.png]]
Sau khi xác minh rằng image đã được import thành công, chúng ta có thể khởi tạo image và cấu hình nó bằng cách chỉ định `security.privileged` flag và root path cho container.
Flag này vô hiệu hóa các isolation features, cho phép chúng ta tương tác với host system.
![[Pasted image 20260910175135.png]]
Sau khi hoàn tất, chúng ta có thể start container và đăng nhập vào nó. Bên trong container, chúng ta có thể truy cập path đã chỉ định để truy cập vào `resource` của host system với quyền `root`.
![[Pasted image 20260910180703.png]]

![[Pasted image 20260910181041.png]]