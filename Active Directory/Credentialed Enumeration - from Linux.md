Sau khi đã có được một foothold trong domain, bước tiếp theo là đào sâu hơn bằng các domain user credentials có quyền hạn thấp mà chúng ta đang sở hữu. Vì đã có được thông tin tổng quan về người dùng và các máy trong domain, lúc này chúng ta sẽ tiến hành enumerate domain một cách chuyên sâu.
Domain user attributes, computer attributes, group membership, Group Policy Objects (GPOs), permissions, ACLs, trusts và nhiều thông tin khác.
Có nhiều công cụ khác nhau có thể được sử dụng cho mục đích này. Tuy nhiên, điều quan trọng nhất cần nhớ là phần lớn các công cụ này sẽ không hoạt động nếu không có domain user credentials hợp lệ, bất kỳ tài khoản nào đó có mức quyền cao hay thấp. Do đó, tối thiểu chúng ta cần có một trong những thứ sau: Cleartext password của một user, NTLM password hash của một user hoặc một host đã join vào domain. 
## CrackMapExec
`CrackMapExec (CME, hiện nay là NetExec)` là một bộ công cụ mạnh được sử dụng để đánh giá các môi trường AD. CME sử dụng các package đến từ bộ công cụ `Impacket` và `PowerSploit` để thực hiện các chức năng của mình.
![[Pasted image 20260908072453.png]]Chúng ta có thể rằng công cụ hỗ trợ làm việc với các giao thức: `MSSQL`, `SMB`, `SSH`, `WinRM`. Điều này có nghĩa là chúng ta có thể sử dụng credentials với các giao thức tương ứng để thực hiện enumeration hoặc các hoạt động đánh giá khác.
#### CME - Domain User Enumeration
Chúng ta bắt đầu bằng cách trỏ CME vào Domain Controller và sử dụng credentials của user `forend` để lấy danh sách tất cả domain user. Khi CME cung cấp thông tin về user, chúng ta có thể thấy thêm các thuộc tính như `badPwdCount`. Thông tin này rất hữu ích khi thực hiện các hoạt động như targeted password spraying.
Chúng ta có thể xây dựng một danh sách target users bằng cách loại bỏ những user có `badPwdCount` lớn hơn `0` để giảm nguy cơ khóa tài khoản.
![[Pasted image 20260908072835.png]]
#### CME - Domain Group Enumeration
Chúng ta có thể lấy danh sách đầy đủ các domain groups. Nên lưu toàn bộ output vào file để có thể dễ dàng sử dụng lại cho việc **reporting** hoặc làm input cho các công cụ khác. 
![[Pasted image 20260908073140.png]]Output trên liệt kê các **group trong domain** cùng với số lượng thành viên của từng group. Output cũng hiển thị các built-in groups trên Domain Controller, chẳng hạn như `Backup Operators`. Chúng ta có thể bắt đầu ghi chú lại những group đáng quan tâm. Đặc biệt chú ý đến các group quan trọng như: `Administrator`, `Domain Admins`, `Executives`. Các group có thể chứa privileged IT administrators. Những group này có khả năng chứa các user với elevated privileges, do đó có thể là những user đáng quan tâm trong quá trình assessment.
#### CME - Logged On Users
Chúng ta cũng có thể sử dụng CME để target các host khác.
Ví dụ, hãy kiểm tra một máy có vẻ là file server để xem hiện tại những user nào đang đăng nhập trên đó.
```shellsession
w3bis2bi@htb[/htb]$ sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

SMB         172.16.5.130    445    ACADEMY-EA-FILE  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-FILE) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.5.130    445    ACADEMY-EA-FILE  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2 (Pwn3d!)
SMB         172.16.5.130    445    ACADEMY-EA-FILE  [+] Enumerated loggedon users
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\clusteragent              logon_server: ACADEMY-EA-DC01
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\lab_adm                   logon_server: ACADEMY-EA-DC01
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\svc_qualys                logon_server: ACADEMY-EA-DC01
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\wley                      logon_server: ACADEMY-EA-DC01

<SNIP>
```
Chúng ta thấy có khá nhiều user đang đăng nhập vào server này, điều này rất đáng chú ý. Ngoài ra, chúng ta có thể thấy user `forend` là local administrator, bởi vì `(Pwn3d!)` xuất hiện sau khi công cụ authenticate thành công với target host. 
Một host như vậy có thể được sử dụng làm jump host hoặc một hệ thống tương tự bởi các administrative users.
Đáng chú ý, user `svc_qualys` đang đăng nhập trên máy. Trước đó chúng ta đã xác định `svc_qualys` là một domain admin.
Nếu có khả năng lấy credentials của user này từ memory hoặc impersonate user này, đây có thể là một easy win.
Như chúng ta sẽ thấy ở các phần sau, BloodHound và các công cụ khác như PowerView có thể được sử dụng để tìm kiếm user sessions. BloodHound đặc biệt mạnh vì cho phép chúng ta xem các Domain User sessions dưới dạng đồ thị và thực hiện việc tìm kiếm theo nhiều cách một cách nhanh chóng. Tuy nhiên, các công cụ như CME vẫn rất hữu ích cho việc targeted enumeration và user hunting.
#### CME Share Searching
![[Pasted image 20260908080536.png]]
Chúng ta thấy có một số share mà user hiện tại có quyền `READ`, đáng chú ý nhất là `Department Shares`, `User Shares`, `ZZZ_archive`, những share này đáng để kiểm tra saua hơn vì chúng có thể chứa dữ liệu nhạy cảm như password, PII và các thông tin nội bộ khác. Tiếp theo chúng ta có thể truy cập vào các share này bà spider từng directory để tìm kiếm files. Module `spider_plus` có thể tự động duyệt qua từng share mà chúng ta có quyền đọc trên host và liệt kê tất cả các file có thể đọc được.
#### Spider_plus
![[Pasted image 20260908080830.png]]
Trên lệnh trên, chúng ta sử dụng các chức năng `spider` để quét share `Department Shares`. Khi hoàn tất, CME sẽ ghi kết quả vào file json tại `/tmp/cme_spider_plus/<IP của máy đích>`
![[Pasted image 20260908081004.png]]
CME là một công cụ rất mạnh, và đây mới chỉ là một phần rất nhỏ trong khả năng của nó.
### SMBMap
`SMBMap` là một công cụ rất hữu ích để enumerate các SMB share từ máy linux. Công cụ này có thể được sử dụng để thu thập danh sách các share, quyền truy cập và nội dung của các share nếu chúng ta có quyền truy cập.
Sau khi có quyền truy cập, `SMBMap` còn có thể được sử dụng để download và upload file cũng như thực thi các lệnh từ xa. Tương tự CME chúng ta có thể sử dụng `SMBmap` cùng với một bộ domain user credentials để kiểm tra các share mà tài khoản có thể truy cập trên các hệ thống từ xa.
#### SMBMap To Check Access
