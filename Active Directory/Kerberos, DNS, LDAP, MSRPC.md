Windows dùng nhiều giao thức để giao tiếp, nhưng riêng Active Directory bắt buộc phải có bốn thứ sau hoạt động cùng nhau:
- LDAP (Lightweight Directory Access Protocol) để tra cứu thông tin trong directory.
- Kerberos (bản của Microsoft) để xác thực.
- DNS để xác thực và giao tiepesm định vị các thành phần trong mạng.
- MSRPC bản triển khai của Microsoft cho RPC (Remote Procedure Call), một kỹ thuật giao tiếp liên tiến trình (interprocess communication) dùng cho các ứng dụng theo mô hình client-server.
## Kerberos 
**Kerberos** là giao thức xác thực mặc định cho tài khoản domain kể từ Windows 2000. Đây là một chuẩn mở, cho phép tương thích với các hệ thống khác cùng dùng chuẩn này. Khi người dùng đăng nhập vào PC, Kerberos xác thực họ bằng mutual authentication tức cả người dùng lẫn server đều xác minh danh tính lẫn nhau khác với NTLM vốn chỉ xác minh client.
**Kerberos** là giao thức stateless (không lưu trạng thái), dựa trên ticket thay vì truyền mật khẩu người dùng qua mạng. Trong AD Domain Services (AD DS), mỗi Domain Controller có một Kerberos Key Distribution Center (KDC) chịu trách nhiệm cấp phát ticket.


![[Pasted image 20260906011620.png]]