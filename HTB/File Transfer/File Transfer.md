File transfer là một chức năng cốt lõi của mọi hệ điều hành và có rất nhiều công cụ có thể được sử dụng để thực hiện việc này. Tuy nhiên, nhiều công cụ trong số đó có thể bị các quản trị viên hệ thống chặn hoặc giám sát chặt chẽ. Vì vậy, việc xem xét nhiều kỹ thuật truyền file khác nhau rất hữu ích, bởi trong một môi trường cụ thể, một số phương pháp có thể hoạt động trong khi những phương pháp khác lại bị chặn.
Hiểu được nhiều phương pháp khác nhau để truyền file cũng như 
# Windows File Transfer Methods
Hệ điều hành Windows đã phát triển qua nhiều năm, và các phiên bản mới đi kèm với những tiện ích khác nhau để thực hiện các hoạt động truyền file. Attacker có thể sử dụng nhiều phương pháp truyền file khác nhau để thực hiện các hoạt động của mình và tránh bị phát hiện. Defender có thể tìm hiểu cách những phương pháp này hoạt động để giám sát và xây dựng các chính sách tương ứng, nhằm ngăn hệ thống bị xâm phạm. 
Chúng ta sử dụng bài viết về [Microsoft Astaroth Attack](https://www.microsoft.com/en-us/security/blog/2019/07/08/dismantling-a-fileless-campaign-microsoft-defender-atp-next-gen-protection-exposes-astaroth-attack/) làm ví dụ về một Advanced Persistent Threat (APT). Bài viết bắt đầu bằng việc đề cập đến [fileless threats](https://www.microsoft.com/en-us/security/blog/2018/01/24/now-you-see-me-exposing-fileless-malware/). Thuật ngữ `fileless` cho thấy một mối đe doạ không nhất thiết phải tồn tại dưới dạng một file, thay vào đó, nó sử dụng các công cụ hợp pháp được tích hợp sẵn trong hệ thống để thực hiện tấn công.
Điều này không có nghĩa là không xảy ra hoạt động truyền file. File không hiện diện trên hệ thống mà chạy trực tiếp trong bộ nhớ.
Astaroth attack nhìn chung diễn ra theo các bước sau:
Một liên kết độc hại trong email spear-phishing dẫn người dùng đến một file LNK. Khi người dùng nhấp đúp vào file LNK, file này thực hiện WMIC tool với tham số `/Format`, cho phép tải xuống và thực thi mã JavaScript độc hại.
Sau đó, mã Javascript tiếp tục tải các payload bằng cách lợi dụng công cụ Bitsadmin. Tất cả các payload đều được mã hoá bằng Base64 và sau đó được giải mã bằng công cụ Certutil, tạo ra một số file DLL.
Tiếp theo, công cụ `regsvr32` được sử dụng để nạp một trong những DLL đã được giải mã. DLL này sẽ giải mã và nạp các file khác, tiếp tục quá trình cho đến khi payload cuối cùng Astaroth được inject vào process `Userinit`.
![[File Transfer.png]]
## Download Operations
Chúng ta có quyền truy cập vào máy `MS02` và cần tải một file từ máy **`Pwnbox`** của mình. Hãy xem chúng ta có thể thực hiện việc này bằng nhiều phương pháp File Download khác nhau như thế nào.![[File Transfer-1.png|center|border|650]]
### PowerShell Base64 Encode & Decode
Tuỳ thuộc vào kích thước file mà chúng ta muốn truyền, có thể sử dụng những phương pháp khác nhau mà không yêu cầu giao tiếp qua mạng.
![[Screenshot 2026-08-25 at 08.11.24.png]]![[Screenshot 2026-08-25 at 08.13.16.png]]
Nếu có quyền truy cập vào một terminal, chúng ta có thể encode một file thành một chuỗi Base64, sao chép nội dung của chuỗi đó từ terminal, sau đó thực hiện thao tác ngược lại để decode file và khôi phục nội dung ban đầu.![[Screenshot 2026-08-25 at 08.20.33.png|center|border|650]]Một bước quan trọng khi sử dụng phương pháp này là phải đảm bảo rằng file được encode và decode là chính xác.![[Screenshot 2026-08-25 at 08.21.41.png|center|border]]
### PowerShell Web Uploads
Powershell không có hàm built-in dành riêng cho hoạt động upload, nhưng chúng ta có thể sử dụng `Invoke-WebRequest` hoặc `Invoke-RestMethod` để xây dựng chức năng upload.
Chúng ta cũng cần một web server có khả năng tiếp nhận file upload, trong khi đây không phải là tuỳ chọn mặc định của hầu hết các tiện ích web server phổ biến.
Đối với web server, chúng ta có thể sử dụng `uploadserver`, một module mở rộng của Python `HTTP.server` module, trong đó có tích hợp sẵn một trang cho phép upload file.
Cài đặt Web Server có hỗ trợ Upload![[Screenshot 2026-08-25 at 08.43.48.png|center|border|650]]Khởi chạy upload web server![[Screenshot 2026-08-25 at 08.44.43.png|center|border|650]]Bây giờ chúng ta có thể sử dụng PowerShell script `PSUpload.ps1`, script này sử dụng `Invoke-RestMethod` để thực hiện các hoạt động upload.
Script nhận hai tham số:
- `-File` được sử dụng để chỉ định đường dẫn đến file.
- `-Uri` là URL của server nơi chúng ta sẽ upload file.
Hãy thử upload file `hosts` từ Windows host của chúng ta.![[Screenshot 2026-08-25 at 08.53.17.png|center|border]]
![[Screenshot 2026-08-25 at 08.53.45.png]]

**PowerShell Base64 Web Upload**
Một cách khác để sử dụng Powershell cùng với các file được encode bằng base64 cho hoạt động upload là sử dụng `Invoke-WebRequest` hoặc `Invoke-RestMethod` kết hợp với Netcat.
Chúng ta sử dụng Netcat để lắng nghe trên một port được chỉ định, sau đó gửi file dưới dạng một HTTP `POST` request.
Cuối cùng, chúng ta sao chép phần output nhận được và sử dụng chức năng Base64 decode để chuyển chuỗi Base64 trở lại thành file.![[Screenshot 2026-08-25 at 09.00.08.png|center|border|650]]Chúng ta nhận dữ liệu Base64 bằng Netcat, sau đó sử dụng ứng dụng `base64` với tùy chọn decode để chuyển chuỗi đó thành file.![[Screenshot 2026-08-25 at 09.00.32.png|center|border|650]]
Q2 result:
![[Screenshot 2026-08-25 at 09.07.31.png]]

![[Screenshot 2026-08-25 at 09.07.00.png]]

## SMB Uploads
Thông thường, giao thức SMB (TCP/445) không được phép đi ra khỏi mạng nội bộ, bởi điều này có thể khiến hệ thống phải đối mặt với các cuộc tấn công tiềm ẩn.
Một giải pháp thay thế là chạy SMB over HTTP bằng `WebDAV`. `WebDAV (RFC 4918)` là một phần mở rộng của HTTP là giao thức internet được các trình duyệt và web server sử dụng để giao tiếp với nhau. Giao thức `WebDAV` cho phép một webserver hoạt động giống như một file server, đồng thời hỗ trợ việc cộng tác tạo và chỉnh sửa nội dung. `WebDAV` cũng có thể sử dụng HTTPS. Khi sử dụng `SMB`, hệ thống trước tiên sẽ cố gắng kết nối bằng giao thức SMB. Nếu không tìm thấy SMB share khả dụng, nó sẽ thử kết nối bằng HTTP.  
### Configuring WebDav Server
Để thiết lập WebDav server, chúng ta cần cài đặt Python module là `wsgidav` và `cheroot`. Sau khi cài đặt chúng, chúng ta chạy ứng dụng `wsgidav` trong thư mục mục tiêu.
#### Installing WebDav Python modules
![[Screenshot 2026-08-29 at 23.22.18.png]]
#### Using the WebDav Python module
![[Screenshot 2026-08-29 at 23.24.03.png]]
#### Connecting to the Webdav Share
![[Screenshot 2026-08-29 at 23.50.37.png]]

### FTP Uploads
Việc upload file bằng FTP rất giống với việc download file. Chúng ta có thể sử dụng PowerShell hoặc FTP client để thực hiện thao tác này.
Trước khi khởi động FTP Server bằng Python module `pyftpdlib`, chúng ta cần chỉ định tùy chọn `--write` để cho phép client upload file lên attack host của chúng ta.
![[Screenshot 2026-08-29 at 23.59.34.png]]
Bây giờ hãy sử dụng PowerShell upload function để upload một file lên FTP Server của chúng ta.
![[Screenshot 2026-08-30 at 00.01.44.png]]

![[Screenshot 2026-08-30 at 00.01.55.png]]


![[Screenshot 2026-08-30 at 00.04.45.png]]

![[Screenshot 2026-08-30 at 00.05.07.png]]

