Active Directory bắt buộc phải có bốn thứ sau hoạt động cùng nhau:
- LDAP (Lightweight Directory Access Protocol) để tra cứu thông tin trong directory.
- Kerberos (bản của Microsoft) để xác thực.
- DNS để xác thực và giao tiepesm định vị các thành phần trong mạng.
- MSRPC bản triển khai của Microsoft cho RPC (Remote Procedure Call), một kỹ thuật giao tiếp liên tiến trình (interprocess communication) dùng cho các ứng dụng theo mô hình client-server.
## Kerberos 
Có cụm từ cần lưu ý:
- AS: Authentication Server
- TGS: Ticket Granting Server
- KDC (= AS+TGS): Key Distribution Center
- SS: Service Server
- TGT: Ticket Granting Ticket
- ST: Service Ticket
**Kerberos** là giao thức xác thực mặc định cho tài khoản domain kể từ Windows 2000. Đây là một chuẩn mở, cho phép tương thích với các hệ thống khác cùng dùng chuẩn này. Khi người dùng đăng nhập vào PC, Kerberos xác thực họ bằng mutual authentication tức cả người dùng lẫn server đều xác minh danh tính lẫn nhau khác với NTLM vốn chỉ xác minh client.
**Kerberos** là giao thức stateless (không lưu trạng thái), dựa trên ticket thay vì truyền mật khẩu người dùng qua mạng. Trong AD Domain Services (AD DS), mỗi Domain Controller có một Kerberos Key Distribution Center (KDC) chịu trách nhiệm cấp phát ticket. 
**Cơ chế hoạt động:**
1. Khi người dùng đăng nhập, mật khẩu của họ được dùng để mã hóa một timestamp, gửi tới KDC dưới dạng yêu cầu xác thực (AS-REQ), KDC giải mã yêu cầu này để xác minh tính toàn vẹn của việc xác thực. Nếu giải mã thành công, KDC tạo ra một Ticket Granting Ticket (TGT), mã hóa bằng secret key của tài khoản rồi gửi trả về cho người dùng.
2. Dịch vụ KDC trên DC kiểm tra AS-REQ, xác minh thông tin người dùng, và tạo TGT để chuyển cho người dùng.
3. Người dùng trình TGT cho DC để yêu cầu một Ticket Granting Service (TGS) ticket cho một dịch vụ cụ thể, đây là bước `TGS_REQ`. Nếu TGT hợp lệ, dữ liệu của nó được sao chép để tạo ra TGS ticket.
4. TGS được mã hóa bằng NTLM password hash của tài khoản dịch vụ hoặc máy tính đang chạy dịch vụ đó, và được gửi trả cho người dùng trong `TGS-REP`.
5. Người dùng trình TGS cho dịch vụ cần truy cập (`AS-REQ`); nếu hợp lệ, người dùng được phép kết nối tới tài nguyên đó.
Ý nghĩa cốt lõi: Kerberos tách rời thông tin đăng nhập của người dùng khỏi các yêu cầu truy cập tài nguyên, mật khẩu không bao giờ bị truyền qua mạng khi truy cập một dịch vụ (ví dụ một trang SharePoint nội bộ). KDC không ghi nhớ các giao dịch trước đó: TGS chỉ dựa vào việc TGT còn hợp lệ, KDC mặc định rằng nếu người dùng có TGT hợp lệ, họ ắt hẳn đã chứng minh được danh tính từ trước, không cần kiểm tra lại từ đầu.
Port sử dụng: Kerberos dùng port 88 (Cả TCP lẫn UDP). Khi enumerate một môi trường AD, có thể định vị Domain Controller bằng cách quét cổng tìm port 88 đang mở, ví dụ dùng Nmap.
![[Pasted image 20260906011620.png]]
## DNS
AD DS dùng DNS để cho phép client (workstation, server, và các hệ thống khác giao tiếp với domain) định vị được Domain Controller, và để các Domain Controller lưu trữ directory service giao tiếp được với nhau. DNS dùng để phân giải hostname thành địa chỉ IP, được sử dụng rộng rãi cả trong mạng nội bộ lẫn Internet. Các mạng nội bộ riêng dùng AD DNS namespace để tạo điều kiện giao tiếp giữa server, client và các peer.
**Cơ chế hoạt động:**
- AD duy trì một cơ sở dữ liệu các dịch vụ đang chạy trên mạng dưới dạng service record (SRV). Các bản ghi này cho phép client trong môi trường AD định vị được dịch vụ họ cần, ví dụ file server, máy in, hoặc Domain Controller.
- Dynamic DNS được dùng để tự động cập nhật database DNS khi địa chỉ IP của một hệ thống thay đổi nếu phải nhập tay từng bản ghi sẽ rất tốn thời gian và để sai sot.
- Nếu database DNS không có đúng địa chỉ IP của một host, client sẽ không thể định vị và giao tiếp được với host đó trên mạng.
- Khi một client gia nhập mạng, nó định vị Domain Controller bằng cách gửi truy vấn tới dịch vụ DNS, lấy về một SRV record từ database DNS (chứa hostname của Domain Controller), rồi dùng hostname đó để lấy  địa chỉ IP tương ứng của Domain Controller.
**Cổng sử dụng:** DNS dùng cả **TCP và UDP port 53**. UDP là mặc định, nhưng sẽ chuyển sang TCP khi không thể giao tiếp qua UDP hoặc khi message DNS lớn hơn 512 byte.

## LDAP
Active Directory hỗ trợ Lightweight Directory Access Protocol (LDAP) để tra cứu directory. LDAP là giao thức mã nguồn mở, đa nền tảng, dùng để xác thực với các directory service khác nhau. 
LDAP là ngôn ngữ mà các ứng dụng để giao tiếp với các server cung cấp directory service, nói cách khác, LDAP là cách các hệ thống trong mạng nói chuyện với AD. Có thể ví mối quan hệ giữa AD và LDAP giống như Apache và HTTP. Apache là web server dùng giao thức HTTP, còn Active Directory là directory server dùng giao thức LDAP.
