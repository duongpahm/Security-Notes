## Internal Password Spraying from a Linux Host
Sau khi tạo wordlist bằng các phương pháp và công cụ, chúng ta có thể tiến hành cuộc tấn công. `rpcclient` là một lựa chọn rất tốt để thực hiện kỹ thuật này từ Linux. Một điểm cần lưu ý là với `rcpclient`, mỗi lần đăng nhập thành công không phải lức nào cũng được thể hiện một cách trực tiếp. Phản hồi chứa `Authority Name` cho biết quá trình xác thực đã thành công. Do đó, chúng ta có thể sử dụng `grep` để lọc các lần đăng nhập không hợp lệ bằng cách tìm kiếm chuỗi `Authority` trong response.
Bash one liner dưới đây, được điều chỉnh từ [nguồn này](https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/), có thể được sử dụng để thực hiện cuộc tấn công.
#### Using a Bash one-liner for the Attack
```shell
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```
![[Pasted image 20260906084054.png]]

#### Using Kerbrute for the Attack
![[Pasted image 20260906084148.png]]
Có nhiều phương pháp khác để thực hiện Password Spraying từ Linux. Một lựa chọn rất hữu ích khác là `CrackMapExec`. Công cụ đa năng này cho phép sử dụng một tệp văn bản chứa danh sách username để thử với một password duy nhất trong cuộc attack spraying.
Ở đây, chúng ta sử dụng `grep` với kí tự `+` để lọc các lần đăng nhập thất bại và tập trung vào những lần đăng nhập hợp lệ. Điều này đảm bảo chúng ta không bỏ sót kết quả khi phải xử lý một lượng lớn dòng output.
![[Pasted image 20260906112157.png]]
Sau khi thu được một hoặc nhiều kết quả từ cuộc tấn công Password Spraying, chúng ta có thể sử dụng `CrackMapExec` để nhanh chóng xác thực các thông tin đăng nhập đó với Domain Controller.
#### Xác thực thông tin đăng nhập bằng CrackMapExec
![[Pasted image 20260906112236.png]]
## Local Administrator Password Reuse
Password Spraying nội bộ không chỉ áp dụng cho các tài khoản domain. Nếu chúng ta có được quyền quản trị cùng với NTLM password hash hoặc cleartext password của tài khoản local Administrator hoặc một tài khoản có đặc quyền khác, chúng ta có thể thử thông tin xác thực đó trên nhiều host mạng.
Việc tái sử dụng mật khẩu của tài khoản Local Administrator khá phổ biến do các tổ chức thường sử dụng gold image, trong quá trình triển khai tự động, đồng thời cho rằng việc áp dụng cùng một mật khẩu trên nhiều host sẽ thuận tiên hơn trong quản trị.
`CrackMapExec` là một công cụ hữu ích kiểu tấn công này, chúng ta nên ưu tiên các host có giá trị cao như máy chủ `SQL` hoặc `Microsoft Exchange`, vì chúng có khả năng đang có người dùng đặc quyền đăng nhập hoặc lưu thông tin xác thực của họ trong bộ nhớ.
Khi làm việc với các tài khoản Local Administrator, một yếu tố cần xem xét là việc tái sử dụng password hoặc sử dụng các định dạng password tương tự giữa các tài khoản.
Ví dụ, nếu chúng ta phát hiện một desktop host có password của tài khoản Local Administrator được đặt thành `$desktop%@admin123`, có thể đáng để thử `$server%@admin123` trên các server.
Tương tự, nếu phát hiện một tài khoản Local Administrator không theo chuẩn như `bsmith`, chúng ta có thể phát hiện password được tái sử dụng cho một domain user có username tương tự.
Nguyên tắc tương tự cũng có thể áp dụng cho domain account. Nếu chúng ta lấy được password của một user có tên `ajones`, đáng để thử cùng password đó trên tài khoản admin của người dùng này, nếu tồn tại, chẳng hạn như `ajones_adm`, nhằm kiểm tra xem password có được tái sử dụng hay không.
Điều này cũng thường xuất hiện trong các tình huống **domain trust**. Chúng ta có thể thu được thông tin xác thực hợp lệ của một user trong domain A, trong khi thông tin đó cũng hợp lệ cho một user có username giống hoặc tương tự trong domain B, hoặc ngược lại.
Đôi khi chúng ta chỉ thu được **NTLM hash** của tài khoản Local Administrator từ cơ sở dữ liệu local SAM. Trong trường hợp này, chúng ta có thể sử dụng NT hash đó để thực hiện spraying trên toàn bộ subnet hoặc nhiều subnet, nhằm tìm kiếm các tài khoản Local Administrator có cùng password.

#### Local Admin Spraying with CrackMapExec
```shellsession
w3bis2bi@htb[/htb]$ sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +

SMB         172.16.5.50     445    ACADEMY-EA-MX01  [+] ACADEMY-EA-MX01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.25     445    ACADEMY-EA-MS01  [+] ACADEMY-EA-MS01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.125    445    ACADEMY-EA-WEB0  [+] ACADEMY-EA-WEB0\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
```
Output trên cho thấy thông tin xác thực hợp lệ với quyền Local Administrator trên `3` hệ thống trong subnet `172.16.5.0/23`.
Sau đó, chúng ta có thể tiếp tục enumerate từng hệ thống để tìm kiếm những thông tin có thể giúp mở rộng quyền truy cập.