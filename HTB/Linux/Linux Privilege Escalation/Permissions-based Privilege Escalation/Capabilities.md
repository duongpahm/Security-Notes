Linux capabilities là một tính năng bảo mật trong hệ điều hành Linux, cho phép cấp các đặc quyền cụ thể cho các process, giúp chúng thực hiện những hành động vốn bị hạn chế. Điều này cho phép kiểm soát chi tiết hơn đối với những đặc quyền mà các process có quyền truy cập, từ đó bảo mật hơn so với mô hình Unix truyền thống chỉ cấp quyền cho user và group.
Tuy nhiên, giống như bất kỳ tính năng bảo mật nào, Linux capabilities không phải là không thể khai thác và có thể bị lợi dụng. Một lỗ hổng phổ biến là sử dụng capabilites để cấp đặc quyền cho các process không được sandbox hoặc cô lập đầy đủ, cho phép chúng ta khai thác đặc quyền này để leo thang quyền hạn và truy cập thông tin nhạy cảm hoặc thực hiện các hành động trái phép.
Trong Ubuntu, chẳng hạn, chúng ta có thể sử dụng command `setcap` để thiết lập capabilities cụ thể cho executable cụ thể. Command này cho phép chúng ta chỉ định capability muốn thiết lập và giá trị muốn gán.
Ví dụ, chúng ta có thể sử dụng command sau để thiết lập capability `cap_net_bind_service` cho một executable.
#### Set Capability
```shell
w3bis2bi@htb[/htb]$ sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
```
Khi capabilities được thiết lập cho một binary, điều đó có nghĩa là binary đó sẽ có khả năng thực hiện những hành động cụ thể mà nó không thể thực hiện nếu không có capabilities. Ví dụ, capability `cap_net_bind_service` được thiết lập cho một binary, binary đó sẽ có thể bind vào network port , đây là một đặc quyền thường bị hạn chế.
Một số capabilities chẳng hạn như `cap_sys_admin`, cho phép executable thực hiện các hành động với đặc quyền quản trị, có thể rất nguy hiểm nếu không được sử dụng đúng cách. Ví dụ chúng ta có thể khai thác chúng để leo thang đặc quyền, truy cập thông tin nhạy cảm hoặc thực hiện các hành động trái phép. Do đó, điều này quan trọng là phải thiết lập những loại capabilities này cho các executable được sandbox và cô lập đúng cách, đồng thời tránh cấp chúng khi không cần thiết.


| Capability             | Description                                                                                                                                                |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_sys_admin`        | Cho phép thực hiện cách hành động đối với đặc quyền quản trị, chẳng hạn như sửa đổi system file hoặc thay đổi system setting.                              |
| `cap_sys_chroot`       | Cho phép thực hiện thay đổi root directory cho process hiện tại, cho phép process truy cập các file và directory vốn không thể truy cập.                   |
| `cap_sys_ptrace`       | Cho phép attach và debug các process khác, có khả năng giúp lấy thông tin nhạy cảm hoặc thay đổi hành vi của các process đó.                               |
| `cap_sys_nice`         | Cho phép tăng hoặc giảm priority của các process, có khả năng giúp truy cập các resource vốn bị hạn chế.                                                   |
| `cap_sys_time`         | Cho phép thay đổi system clock, có khả năng thao túng timestamp hoặc khiến các process khác hoạt động theo cách không mong muốn.                           |
| `cap_sys_resource`     | Cho phép thay đổi các giới hạn resource của system, chẳng hạn như số lượng file descriptor tối đa có thể mở hoặc lượng memory tối đa có thể được cấp phát. |
| `cap_sys_module`       | Cho phép load và unload kernel module, có khả năng thay đổi cách hệ điều hành hoạt động hoặc truy cập thông tin nhạy cảm.                                  |
| `cap_net_bind_service` | Cho phép bind vào các network port, có khả năng giúp truy cập thông tin nhạy cảm hoặc thực hiện các hành động trái phép.                                   |
Khi một binary được thực thi với capabilities, nó có thể thực hiện những hành động mà các capabilites đó cho phép. Tuy nhiên, nó sẽ không thể thực hiện các hành động không được capabilities cho phép. Điều này cho phép kiểm soát chi tiết đối với đặc quyền của binary và có thể giúp ngăn chặn các lỗ hổng bảo mật cũng như việc truy cập trái phép thông tin nhạy cảm.
Khi sử dụng command `setcap` để thiết lập capabilities cho một executable trong Linux, chúng ta cần chỉ định capability muốn thiết lập và giá trị muốn gán. Các giá trị được sử dụng sẽ phụ thuộc vào capability cụ thể đang được thiết lập và các đặc quyền mà chúng ta muốn cấp cho executable.
Dưới đây là một số ví dụ về các giá trị có thể sử dụng với command `setcap`, cùng với mô tả ngắn gọn về chức năng của chúng:

| **Capability Values** | **Description**                                                                                                                                                                                                                                                                                                                                     |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `=`                   | Giá trị này thiết lập capability được chỉ định cho executable nhưng không cấp bất kỳ đặc quyền nào. Điều này có thể hữu ích nếu chúng ta muốn xóa một capability đã được thiết lập trước đó cho executable.                                                                                                                                         |
| `+ep`                 | Giá trị này cấp đặc quyền effective và permitted cho capability được chỉ định đối với executbale. Điều này cho phép executable thực hiện các hành động mà các capability đó cho phép nhưng không cho phép thực hiện bất kỳ hành động nào ngoài capability đó.                                                                                       |
| `+ei`                 | Giá trị này cấp đặc quyền effective và inheritable cho capability được chỉ định đối với executable. Điều này cho phép nó kế thừa capability hoặc cho phép các process con kế thừa capability. Điều này có thể hữu ích khi muốn cấp capability cho các executable nhưng ngăn nó kế thừa capability hoặc cho phép các process con kế thừa capability. |
Một số Linux capabilities có thể được sử dụng để leo thang quyền hạn của user lên `root`, bao gồm:

| Capability         | Description                                                                                                                                                                                              |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_setuid`       | Cho phép process thiết lập effective user ID của nó, có thể được sử dụng để đạt được đặc quyền của một user khác bao gồm cả user `root`.                                                                 |
| `cap_setgid`       | Cho phép thiết lập effective group ID, có thể được sử dụng để đạt được đặc quyền của group khác, bao gồm cả group `root`.                                                                                |
| `cap_sys_admin`    | Capability này cung cấp một loạt đặc quyền quản trị, bao gồm khả năng thực hiện nhiều hành động vốn dành riêng cho user `root`, chẳng hạn như thay đổi system setting và mount hoặc unmount file system. |
| `cap_dac_override` | Cho phép bypass các kiểm tra permission đối với việc, đọc, ghi và thực thi file.                                                                                                                         |
## Enumerating Capabilities
Điều quan trọng cần lưu ý là các capabilities này phải được sử dụng một cách thận trọng và chỉ được cấp cho các process đáng tin cậy, vì chúng có thể bị sử dụng sai để đạt được quyền truy cập trái phép vào system. Để enumerate tất cả capabilities hiện có của tất cả binary executable hiện có trên một Linux system, chúng ta có thể sử dụng command sau:
```shell
w3bis2bi@htb[/htb]$ find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \; 
/usr/bin/vim.basic cap_dac_override=eip 
/usr/bin/ping cap_net_raw=ep 
/usr/bin/mtr-packet cap_net_raw=ep
```
## Exploitation
Thực hiện enumerate trên lab được cung cấp với lệnh `find`:
![[Pasted image 20260910081918.png]]
Kết quả cho thấy chúng ta có capability `cap_dac_override`.
![[Pasted image 20260910082332.png]]

![[Pasted image 20260910082229.png]]

![[Pasted image 20260910082735.png]]

![[Pasted image 20260910082834.png]]

