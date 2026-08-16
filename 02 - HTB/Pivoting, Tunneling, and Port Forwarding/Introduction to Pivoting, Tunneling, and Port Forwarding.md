![[Pivoting, Tunneling and Port Forwarding.png|center|border|650]]
Trong các cuộc tấn, kiểm thử xâm nhập chúng ta thường thu thập được thông tin xác thực, SSH Key, hash hoặc access token cần thiết để tiếp tục xâm nhập sang một máy chủ khác.  Tuy nhiên, máy chủ mục tiêu lại không thể truy cập trực tiếp từ máy attack của chúng ta.
Trong những trường hợp như vậy, chúng ta cần sử dụng một pivot host mà mình đã chiếm quyền trước đó để làm bàn đạp tiến tới mục tiêu tiếp theo.
Một trong những việc quan trọng nhất khi lần đầu chiếm được quyền trên một máy chủ là kiểm tra:
- Mức đặc quyền (Privilege Level)
- Các kết nối mạng hiện có (Network Connections)
- Các phần mềm VPN hoặc giải pháp truy cập từ xa (Remote Access Software)
Nếu máy chủ có nhiều hơn một card mạng thì rất có khả năng chúng ta có thể tận dụng nó để di chuyển sang một phân đoạn mạng khác.
Về bản chất, Pivoting là kỹ thuật sử dụng một máy chủ đã bị xâm nhập để di chuyển sang những mạng khác nhằm tiếp cận mục tiêu nằm trên những phân đoạn mạng mà ban đầu không thể truy cập.
Có rất nhiều thuật ngữ được sử dụng để chỉ một máy chủ đã bị chiếm quyền và có thể dùng làm điểm trung gian để truy cập vào một phân đoạn mạng trước đây không thể tiếp cận. Những thuật ngữ phổ biến bao gồm: Pivot Host, Proxy, Foothold, Beach Head System, Jump Host. Mục đích chính của Pivoting là vượt qua cơ chế Network Segmentation, dù là phân tách vật lý hay phân tách logic, để truy cập vào các mạng bị cô lập.
Trong khi đó, Tunneling chỉ là một tâp con của Pivoting. Tunneling hoạt động bằng cách đóng gói lưu lượng mạn vào bên trong giao thức khác rồi định tuyến lưu lượng đó thông qua giao thức này.
## Lateral Movement, Pivoting, and Tunneling Compared
### Lateral Movement
Lateral Movement có thể được mô tả là một kỹ thuật được sử dụng để mở rộng quyền truy cập của chúng ta sang các host, application và services khác trong cùng một môi trường mạng. Laterak Movement cũng có thể giúp chúng ta truy cập vào các tài nguyên cụ thể trong Domain mà chúng ta cần để leo thang đặc quyền. Lateral Movement thường cho phép quá trình leo thang đặc quyền diễn ra trên nhiều host khác nhau. 
Một ví dụ thực tế về **Lateral Movement** như sau:
>Trong một cuộc đánh giá bảo mật, chúng ta đã có được quyền truy cập ban đầu vào môi trường mục tiêu và chiếm quyền tài khoản Local Administrator. Sau khi thực hiện quét mạng, chúng ta phát hiện thêm ba máy Windows khác trong hệ thống. Chúng ta sử dụng cùng thông tin xác thực của tài khoản Local Administrator trên các máy này và phát hiện một trong số đó sử dụng cùng tài khoản quản trị cục bộ. Chúng ta sử dụng những thông tin xác thực đó để di chuyển ngang sang thiết bị này, từ đó tiếp tục mở rộng mức độ xâm nhập vào Domain.
### Pivoting
Pivoting là kỹ thuật sử dụng nhiều host để vượt qua các ranh giới mạng mà thông thường chúng ta không thể truy cập. Đây là một kỹ thuật mang tính mục tiêu cao. Mục tiêu Pivoting là cho phép chúng ta tiến sâu hơn vào hệ thống mạng bằng cách chiếm quyền các host hoặc thành phần hạ tầng đã được xác định trước.
Một ví dụ thực tế về **Pivoting** như sau:
>Trong một cuộc Red Team phức tạp, mạng của khách hàng được phân tách cả về mặt vật lý lẫn logic. Sự phân tách này khiến chúng ta gặp nhiều khó khăn trong việc di chuyển bên trong hệ thống để hoàn thành mục tiêu đánh giá. Chúng ta phải tiếp tục khảo sát và cuối cùng chiếm được một máy trạm của bộ phận kỹ thuật. Máy này được sử dụng để bảo trì và giám sát các thiết bị trong môi trường vận hành, đồng thời cũng dùng để gửi báo cáo và thực hiện các tác vụ quản trị trong môi trường doanh nghiệp. Sau khi phân tích, chúng ta phát hiện máy này là **Dual Homed Host**, nghĩa là có nhiều hơn một **Network Interface Card (NIC)** được kết nối tới các mạng khác nhau. Nếu máy này không đồng thời kết nối với cả mạng doanh nghiệp và mạng vận hành thì chúng ta sẽ không thể thực hiện Pivoting để hoàn thành cuộc đánh giá.
### Tunneling
Chúng ta thường xuyên phải sử dụng nhiều giao thức khác nhau để truyền lưu lượng mạng vào hoặc ra khỏi hệ thống trong những trường hợp có khả năng lưu lượng bị phát hiện. Ví dụ, sử dụng HTTP để che giấu lưu lượng Command and Control (C2) giữa máy chủ điều khiển của chúng ta và máy nạn nhân.
Điểm cốt lõi của Tunneling là Obfuscation hoạt động nhằm tránh bị phát hiện càng lâu càng tốt. Chúng ta thường sử dụng các giao thức có cơ chế bảo mật mạnh như HTTPS chạy trên TLS hoặc SSH chạy trên các giao thức truyền tải khác. Những kỹ thuật này cũng tạo điều kiện cho các hoạt động như Exfiltration dữ liệu ra khỏi mạng mục tiêu hoặc đưa thêm Payload và lệnh điều khiển vào bên trong hệ thống.
Một ví dụ thực tế về **Tunneling** như sau:
>Một trong những cách chúng ta sử dụng Tunneling là ngụy trang lưu lượng bên trong các giao thức HTTP và HTTPS. Đây là phương pháp phổ biến để duy trì kết nối Command and Control (C2) với các host đã bị chiếm quyền trong mạng nội bộ. Chúng ta nhúng các lệnh điều khiển vào các yêu cầu HTTP GET và HTTP POST, khiến chúng trông giống như các yêu cầu hoặc phản hồi web thông thường. Đối với người quan sát không có kinh nghiệm, chúng sẽ chỉ giống lưu lượng truy cập đến một website bình thường. Nếu gói tin được tạo đúng định dạng, nó sẽ được chuyển tiếp đến C2 Server của chúng ta. Ngược lại, nếu không đúng định dạng, lưu lượng sẽ bị chuyển hướng đến một website khác, khiến người phòng thủ kiểm tra lưu lượng khó nhận ra bản chất thực sự của nó.
## The Networking Behind Pivoting
### IP Addressing & NICs
Mọi máy tính tham gia giao tiếp trên một mạng đều cần có một địa chỉ IP (IP Address). Nếu không có địa chỉ IP, máy tính đó sẽ không thể tham gia vào mạng.
Địa chỉ IP được gán bằng phần mềm và thông thường sẽ được cấp phát tự động từ DHCP Server. Tuy nhiên, trong thực tế cũng rất phổ biến việc các máy tính được cấu hình địa chỉ IP tĩnh (Static IP Address).
Việc gán địa chỉ IP tĩnh thường được áp dụng cho các thiết bị sau:
- Máy chủ (Servers)
- Bộ định tuyến (Routers)
- Giao diện mạng ảo của Switch (Switch Virtual Interfaces)
- Máy in (Printers)
- Và các thiết bị cung cấp những dịch vụ quan trọng cho hệ thống mạng.
Dù được cấp phát động hay gán tĩnh, địa chỉ IP đều được gán cho một Network Interface Controller (NIC). Trong thực tế, NIC thường được gọi là Network Interface Card hoặc Network Adapter. Một máy tính có thể sở hữu nhiều NIC (bao gồm cả vật lý và ảo), điều đó đồng nghĩa với việc nó có thể được gán nhiều địa chỉ IP khác nhau, cho phép giao tiếp đồng thời với nhiều mạng. Việc xác định các cơ hội Pivoting thường phụ thuộc vào những địa chỉ IP được gán cho các host mà chúng ta đã chiếm quyền, bởi các địa chỉ IP này có thể cho biết những mạng nào mà host đó có khả năng truy cập. Đó cũng là lý do tại sao, ngay khi chiếm quyền trên một máy tính, chúng ta luôn cần kiểm tra hệ thống xem có thêm NIC nào khác hay không bằng lện như `ifconfig` trên Windows hoặc MacOS hoặc `ipconfig` trên Linux.
#### Using ifconfig
```shell
w3bis2bi@htb[/htb]$ ifconfig

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 134.122.100.200  netmask 255.255.240.0  broadcast 134.122.111.255
        inet6 fe80::e973:b08d:7bdf:dc67  prefixlen 64  scopeid 0x20<link>
        ether 12:ed:13:35:68:f5  txqueuelen 1000  (Ethernet)
        RX packets 8844  bytes 803773 (784.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5698  bytes 9713896 (9.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.106.0.172  netmask 255.255.240.0  broadcast 10.106.15.255
        inet6 fe80::a5bf:1cd4:9bca:b3ae  prefixlen 64  scopeid 0x20<link>
        ether 4e:c7:60:b0:01:8d  txqueuelen 1000  (Ethernet)
        RX packets 15  bytes 1620 (1.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18  bytes 1858 (1.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 19787  bytes 10346966 (9.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 19787  bytes 10346966 (9.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.10.15.54  netmask 255.255.254.0  destination 10.10.15.54
        inet6 fe80::c85a:5717:5e3a:38de  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef:2::1034  prefixlen 64  scopeid 0x0<global>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7  bytes 336 (336.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
Trong kết quả trên, mỗi NIC đều có một định danh (eth0, eth1, lo, tun0), theo sau là thông tin về địa chỉ mạng và thống kê lưu lượng. Giao diện Tunnel (tun0) cho thấy hệ thống hiện đang có một kết nối VPN đang hoạt động. 
Mỗi khi chúng ta kết nối tới một máy chủ VPN của Hack The Box (HTB) thông qua Pwnbox hoặc từ máy tấn công của mình, chúng ta sẽ luôn thấy một Tunnel Interface được tạo ra và được gán một địa chỉ IP.
Kết nối VPN cho phép chúng ta truy cập vào các môi trường phòng lab do HTB cung cấp.
Cần lưu ý rằng những mạng phòng lab này không thể truy cập nếu chưa thiết lập kết nối Tunnel. VPN vừa thực hiện mã hoá lưu lượng mạng, vừa tạo một đường hầm  (Tunnel) đi qua mạng công cộng, đi qua cơ chế NAT trên thiết bị mạng hướng Internet, rồi kết nối vào mạng nội bộ.
Hãy chú ý đến các địa chỉ IP được gán cho từng **NIC**.
- Địa chỉ IP của `eth0 (134.122.100.200)` là một Public IP Address có thể được định tuyến trên Internet.
Điều này có nghĩa là các ISP (Internet Service Provider) sẽ định tuyến lưu lượng phát sinh từ địa chỉ IP này thông qua Internet. Chúng ta thường bắt gặp Public IP trên các thiết bị được kết nối trực tiếp với Internet, phổ biến nhất là các hệ thống được triển khai trong DMZ (Demilitarized Zone). Các NIC còn lại sử dụng Private IP Address. Những địa chỉ IP này chỉ có thể được định tuyến bên trong các mạng nội bộ và không thể được định tuyến trực tiếp trên Internet.
#### Using ipconfig
```powershell
PS C:\Users\htb-student> ipconfig

Windows IP Configuration

Unknown adapter NordLynx:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . : .htb
   IPv6 Address. . . . . . . . . . . : dead:beef::1a9
   IPv6 Address. . . . . . . . . . . : dead:beef::f58b:6381:c648:1fb0
   Temporary IPv6 Address. . . . . . : dead:beef::dd0b:7cda:7118:3373
   Link-local IPv6 Address . . . . . : fe80::f58b:6381:c648:1fb0%8
   IPv4 Address. . . . . . . . . . . : 10.129.221.36
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:df81%8
                                       10.129.0.1

Ethernet adapter Ethernet:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
```
Kết quả ở trên được tạo ra khi thực thi lệnh `ipconfig` trên một hệ thống Windows. Chúng ta có thể thấy hệ thống này có nhiều Network Adapter, tuy nhiên chỉ có một adapter được gán địa chỉ IP. 
Adapter này được gán cả địa chỉ IPv6 và địa chỉ IPv4. Chúng ta cũng sẽ nhận thấy rằng một số adapter, giống như trong ví dụ trên, được gán đồng thời IPv4 và IPv6 theo mô hình Dual Stack, cho phép các tài nguyên có thể truy cập thông qua cả hai giao thức. Mỗi địa chỉ IPv4 đều đi kèm với một Subnet Mask tương ứng. Nếu coi địa chỉ IP giống như số điện thoại, thì Subnet Mask có thể được ví như mã vùng. Hãy nhớ rằng Subnet Mask xác định phần Network và phần Host của một địa chỉ IP. Khi một máy tính cần gửi lưu lượng tới một địa chỉ IP nằm trên một mạng khác, nó sẽ chuyển lưu lượng đó đến Default Gateway đã được cấu hình.  Default Gateway thường là địa chỉ IP được gán cho một NIC trên thiết bị đóng vai trò Router của mạng Lan đó. Trong bối cảnh Pivoting, điều quan trọng là phải xác định được host mà chúng ta đã chiếm quyền có thể truy cập đến những mạng nào. 
### Routing
Khi nhắc đến Router, chúng ta thường nghĩa ngay đến thiết bị mạng kết nối chúng ta với Internet. Tuy nhiên, về mặt kỹ thuật bất kỳ máy tính nào cũng có thể hoạt động như một Router và tham gia vào quá trình định tuyển (Routing).
Một trong những cách thực hiện là sử dụng AutoRoute, cho phép Attack Box của chúng ta tạo ra các tuyến đường tới những mạng mục tiêu truy cập thông qua Pivot Host.
Một đặc điểm quan trọng của Router là nó sở hữu một Routing Table được sử dụng để quyết định cách chuyển tiếp lưu lượng dựa trên địa chỉ IP đích.
#### Routing Table
```shell
w3bis2bi@htb[/htb]$ netstat -r

Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         178.62.64.1     0.0.0.0         UG        0 0          0 eth0
10.10.10.0      10.10.14.1      255.255.254.0   UG        0 0          0 tun0
10.10.14.0      0.0.0.0         255.255.254.0   U         0 0          0 tun0
10.106.0.0      0.0.0.0         255.255.240.0   U         0 0          0 eth1
10.129.0.0      10.10.14.1      255.255.0.0     UG        0 0          0 tun0
178.62.64.0     0.0.0.0         255.255.192.0   U         0 0          0 eth0
```
Chúng ta sẽ nhận thấy rằng Pwnbox, các bản phân phối Linux, Windows và nhiều hệ điều hành khác đều có Routing Table để hỗ trợ hệ thống đưa ra quyết định định tuyến.
Khi một Packet được tạo và đã có địa chỉ IP đích (Destination IP Address) trước khi rời khỏi máy tính, Routing Table sẽ được sử dụng để quyết định Packet đó cần được gửi theo tuyến đường nào.
Ví dụ, nếu chúng ta muốn kết nối đến một mục tiêu có địa chỉ **10.129.10.25**, chỉ cần quan sát Routing Table là có thể biết Packet sẽ được gửi theo đường nào để đến được địa chỉ đó.
Packet sẽ được chuyển tiếp đến một Gateway, sau đó được gửi ra ngoài thông qua Network Interface (Iface) tương ứng.
Pwnbox không sử dụng các Routing Protocol như EIGRP, OSPF, BGP hoặc các giao thức tương tự để học các tuyến đường này.
Thay vào đó, Pwnbox biết được các Route thông qua chính những Network Interface được kết nối trực tiếp như eth0, eth1 và tun0.
Đối với các thiết bị chuyên dụng đóng vai trò Router, chúng thường học các tuyến đường thông qua sự kết hợp của:
• Static Route
• Dynamic Routing Protocol
• Directly Connected Interface
Mọi lưu lượng có đích đến thuộc các mạng không tồn tại trong Routing Table sẽ được gửi đến Default Route, còn được gọi là Default Gateway hoặc Gateway of Last Resort.
Khi tìm kiếm cơ hội để thực hiện Pivoting, việc kiểm tra Routing Table của host sẽ giúp chúng ta xác định:
• Những mạng mà host hiện tại có thể truy cập.
• Những Route nào cần được bổ sung để tiếp cận các mạng khác.
### Protocols, Services & Ports
Protocol là tập hợp các quy tắc điều khiển quá trình giao tiếp trên mạng. Phần lớn Protocol và Service đều có những Port tương ứng đóng vai trò như một định danh. Logical Port không phải là thành phần vật lý mà chúng ta có thể nhìn thấy hoặc cắm cáp vào. Thay vào đó, Port là một khái niệm được triển khai trong phần mềm và được gán cho các ứng dụng. Khi nhìn thấy một địa chỉ IP, chúng ta biết rằng nó đại diện cho một máy tính có thể truy cập thông qua mạng. Khi nhìn thấy một Open Port gắn với địa chỉ IP đó, chúng ta biết rằng đại diẹn cho một ứng dụng mà chúng ta có khả năng kết nối tới.
Việc kết nối tới các Port mà thiết bị đang lắng nghe (Listening Port) thường cho phép chúng ta tận dụng những Protocol và Port đã được Firewall cho phép để giành quyền truy cập ban đầu (Foothold) vào hệ thống mạng.
Hãy lấy ví dụ về một Web Server sử dụng giao thức HTTP, thông thường sẽ lắng nghe trên Port 80.
Quản trị viên gần như không thể chặn lưu lượng đi vào Port 80, vì nếu làm như vậy thì người dùng sẽ không thể truy cập website mà họ đang vận hành.
Chính vì vậy, đây thường là một trong những con đường phổ biến để xâm nhập vào môi trường mạng, thông qua cùng một Port mà lưu lượng hợp lệ vẫn đang sử dụng.
Chúng ta cũng không được bỏ qua thực tế rằng Source Port cũng sẽ được tạo ra để theo dõi các kết nối đã được thiết lập từ phía máy khách (Client).
Do đó, chúng ta cần luôn chú ý đến những Port mình đang sử dụng nhằm đảm bảo rằng khi thực thi Payload, kết nối sẽ quay trở lại đúng Listener mà chúng ta đã thiết lập.