Password Sparying có thể giúp giành được quyền truy cập vào các hệ thống vào tạo foothold trên target network. Attack này thực hiện việc thử đăng nhập vào một exposed service bằng một mật khẩu phổ biến trên một danh sách username hoặc email address có số lượng lớn hơn. Các username và email này có thể đã được thu thập trong giai đoạn OSINT hoặc trong enumeration.
#### Password Spray Visualization
Mặc dù password spraying rất hữu ích đối với penetration tester hoặc red teamer, việc sử dụng bất cẩn có thể gây ra hậu quả đáng kể, chẳng hạn như khiến hàng trăm production account bị lockout.

Một ví dụ là **brute force** nhằm xác định password của một account bằng cách thử một danh sách password rất dài. Ngược lại, password spraying là một attack có tính kiểm soát hơn, sử dụng những password rất phổ biến trong nhiều ngành nghề.

Bảng dưới đây minh họa một password spray.

|**Attack**|**Username**|**Password**|
|---|---|---|
|1|[bob.smith@inlanefreight.local](mailto:bob.smith@inlanefreight.local)|Welcome1|
|1|[john.doe@inlanefreight.local](mailto:john.doe@inlanefreight.local)|Welcome1|
|1|[jane.doe@inlanefreight.local](mailto:jane.doe@inlanefreight.local)|Welcome1|
|DELAY|||
|2|[bob.smith@inlanefreight.local](mailto:bob.smith@inlanefreight.local)|Passw0rd|
|2|[john.doe@inlanefreight.local](mailto:john.doe@inlanefreight.local)|Passw0rd|
|2|[jane.doe@inlanefreight.local](mailto:jane.doe@inlanefreight.local)|Passw0rd|
|DELAY|||
|3|[bob.smith@inlanefreight.local](mailto:bob.smith@inlanefreight.local)|Winter2022|
|3|[john.doe@inlanefreight.local](mailto:john.doe@inlanefreight.local)|Winter2022|
|3|[jane.doe@inlanefreight.local](mailto:jane.doe@inlanefreight.local)|Winter2022|
Password spraying gửi ít login request hơn trên mỗi username và do đó ít có khả năng gây account lockout hơn so với brute force attack. Tuy nhiên, password spraying vẫn tiềm ẩn nguy cơ lockout. Vì vậy, việc đưa delay vào giữa các lần login attempt là điều cần thiết. Internal password spraying có thể được sử dụng để move laterally trong network, và những vấn đề liên quan đến account lockout vẫn cần được cân nhắc. Tuy nhiên, khi đã có internal access, có thể enumeration được domain password policy, qua đó giảm đáng kể rủi ro này. Một password policy phổ biến có thể cho phép 5 lần đăng nhập sai trước khi account bị lockout, với thời gian tự động unlock là 30 phút. Một số tổ chức có thể cấu hình thời gian lockout dài hơn, thậm chí yêu cầu administrator phải unlock account thủ công. Trong trường hợp đó, chúng ta có thể lựa chọn phương án thận trọng hơn, chẳng hạn chỉ thực hiện một targeted password spraying attempt với một weak/common password như một "hail mary", nếu tất cả các phương án khác để tạo foothold hoặc mở rộng access đều đã được sử dụng hết. Tùy thuộc vào loại assessment, chúng ta luôn có thể yêu cầu client xác nhận password policy. Nếu chúng ta đã có foothold hoặc được cung cấp một user account như một phần của quá trình testing, chúng ta có thể enumeration password policy bằng nhiều phương pháp khác nhau.