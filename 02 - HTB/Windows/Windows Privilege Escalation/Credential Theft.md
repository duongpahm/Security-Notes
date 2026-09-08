# Credential Hunting
Thông tin xác thực có thể mở ra nhiều cánh của cho chúng ta trong quá trình đánh giá. Chúng ta có thể tìm thấy nhiều thông tin xác thực trong quá trình liệt kê leo thang đặc quyền, điều này có thể dẫn trực tiếp đến quyền truy cập quản trị viên cục bộ, cung cấp cho chúng ta một điểm tựa vào môi trường AD, hoặc thậm chí được sử dụng để leo thang đặc quyền trong miền.
## Application Configuration Files
### Searching for Files
Các ứng dụng thường lưu trữ mật khẩu trong các tệp cấu hình dạng cleartext, chúng ta có thể tìm thấy thông tin xác thực cho tài khoản quản trị trong trường hợp chúng ta đã có shellsession. Chúng ta có thể sử dụng `findstr` để tìm kiếm các thông tin nhạy cảm:
```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

>**Giải thích các cờ:** 
>- `/S`: Tìm kiếm đệ quy trong tất cả các thư mục con
>- `/I`: Không phân biệt chữ hoa chữ thường
>- `/M`: Chỉ hiển thị tên tệp chứa kết quả (không hiển thị dòng cụ thể)
>- `/C:"password"`: Tìm chuỗi chính xác "password"

![[Screenshot 2026-08-17 at 06.02.43.png|center|border|650]]
Thông tin IIS nhạy cảm như thông tin xác thực có thể được lưu trữ trong tệp `web.config`. Đối với trang web IIS mặc định, tệp này có thể nằm tại `C:\inetpub\wwwroot\web.config`, nhưng có thể có nhiều phiên bản của tệp này ở các vị trí khác nhau, mà chúng ta có thể tìm kiếm một cách đệ quy.
Một trường hợp thú vị khác là các tệp từ điển. Ví dụ, thông tin nhạy cảm như mật khẩu có thể được nhập vào một ứng dụng email client hoặc một ứng dụng dựa trên trình duyệt, điều này gạch chân bất kỳ từ nào mà nó không nhận ra. Người dùng có thể thêm các từ này vào từ điển của họ để tránh gạch chân đỏ gây mất tập trung.![[Screenshot 2026-08-17 at 06.04.19.png|center|border|650]]
## Unattended Installation Files
Các tệp cài đặt không giám sát có thể xác định các cài đặt tự động đăng nhập hoặc các tài khoản bổ sung sẽ được tạo như một phần của quá trình cài đặt. Mật khẩu trong `unattend.xml` được lưu trữ ở dạng văn bản thuần (plaintext) hoặc mã hóa base64.
```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="specialize">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <AutoLogon>
                <Password>
                    <Value>local_4dmin_p@ss</Value>
                    <PlainText>true</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <LogonCount>2</LogonCount>
                <Username>Administrator</Username>
            </AutoLogon>
            <ComputerName>*</ComputerName>
        </component>
    </settings>
```
Mặc dù các tệp này nên được tự động xóa như một phần của quá trình cài đặt, các quản trị viên hệ thống có thể đã tạo các bản sao của tệp trong các thư mục khác trong quá trình phát triển hình ảnh (image) và tệp trả lời (answer file).
## PowerShell History File
Bắt đầu từ **PowerShell 5.0** trong Windows 10, PowerShell lưu trữ lịch sử lệnh vào tệp:`C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
![[Screenshot 2026-08-17 at 06.10.04.png|center|border|650]]Như đã thấy trong tài liệu Windows Commands PDF hữu ích, được Microsoft xuất bản tại đây, có nhiều lệnh có thể truyền thông tin xác thực trên dòng lệnh. Chúng ta có thể thấy trong ví dụ dưới đây rằng người dùng đã chỉ định thông tin xác thực quản trị cục bộ để truy vấn Application Event Log bằng wevutil.
Khi đã biết vị trí tệp, chúng ta có thể thử đọc nội dung của nó bằng `gc` (Get-Content).![[Screenshot 2026-08-17 at 06.11.32.png|center|border|650]]Chúng ta cũng có thể sử dụng one-liner này để truy xuất nội dung của tất cả các tệp lịch sử PowerShell mà chúng ta có thể truy cập với tư cách người dùng hiện tại. Điều này cũng có thể cực kỳ hữu ích như một bước hậu khai thác (post-exploitation). Chúng ta nên luôn kiểm tra lại các tệp này khi đã có quyền quản trị viên cục bộ, nếu quyền truy cập trước đó của chúng ta không cho phép đọc các tệp của một số người dùng. Lệnh này giả định rằng đường dẫn lưu mặc định đang được sử dụng.![[Screenshot 2026-08-17 at 06.12.50.png|center|border|650]]Thông tin xác thực PowerShell thường được sử dụng cho các tác vụ viết script và tự động hóa như một cách thuận tiện để lưu trữ thông tin xác thực được mã hóa. Thông tin xác thực được bảo vệ bằng DPAPI (Data Protection API), điều này thường có nghĩa là chúng chỉ có thể được giải mã bởi cùng một người dùng trên cùng một máy tính mà chúng được tạo ra.

## Other Files
Có nhiều loại tệp khác nhau mà chúng ta có thể tìm thấy trên hệ thống cục bộ hoặc trên các ổ đĩa chia sẻ mạng có thể chứa thông tin xác thực hoặc thông tin bổ sung có thể được sử dụng để leo thang đặc quyền. Trong AD, có thể sử dụng [Snaffler](https://github.com/SnaffCon/Snaffler) để thu thập dữ liệu các ổ đĩa chia sẻ mạng tìm kiếm các phần mở rộng như `.kdbx`, `.vmdk`, `.vdhx`, `.ppl`, v.v. Ta có thể tìm thấy một ổ đĩa cứng ảo mà chúng ta có thể gắn kết và trích xuất hash mật khẩu quản trị cục bộ từ đó, một khoá riêng SSH, có thể được sử dụng để truy cập các hệ thống khác, hoặc các trường hợp người dùng lưu trữ mật khẩu trong các tài liệu. 
### Manually Searching the File System for Credentials![[Screenshot 2026-08-17 at 08.24.27.png|center|border|650]]
## Further Credential Theft
Có nhiều kỹ thuật khác có thể được sử dụng để thu thập thông tin xác thực trên hệ thống Windows. 
## Cmdkey Saved Credentials
Lệnh `cmdkey` có thể được sử dụng để tạo, liệt kê và xoá tên người dùng cùng mật khẩu đã được lưu trữ. Người dùng có thể lưu thông tin xác thực cho một máy chủ cụ thể lưu trữ thông tin xác thực cho các kết nối Terminal Services, từ đó kết nối đến máy chủ từ xa thông qua Remote Desktop mà không cần nhập lại mật khẩu. Điều này hỗ trợ chúng ta thực hiện leo quyền ngang sang một hệ thống khác với một tài khoản người dùng khác, hoặc leo thang đặc quyền trên server bằng cách tận dụng thong

![[Screenshot 2026-09-02 at 09.16.18.png]]

![[Screenshot 2026-09-02 at 09.00.45.png]]

![[Screenshot 2026-09-02 at 09.23.53.png]]

![[Screenshot 2026-09-02 at 09.26.54.png]]