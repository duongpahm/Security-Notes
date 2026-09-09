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
![[Pasted image 20260908085117.png]]
Kết quả trên cho chúng ta biết tài khoản hiện tại có thể truy cập những gì và mức quyền tương ứng. Tương tự kết quả từ CME, chúng thấy user `forend` không có quyền truy cập vào Domain Controller thông qua các share `ADMIN$` hoặc `C$`. Điều này là bình thường với một tài khoản standard user. Tuy nhiên user này có quyền `READ` đối với `IPC$`, `NETLOGON`, `SYSVOL`, đây là các quyền mặc định thường có trong domain. Các share không chuẩn như `Department Shares`, `User Shares`, `ZZZ_archive` đáng chú ý hơn và là những nơi chúng ta nên kiểm tra sâu hơn.
#### Recursive List Of All Directories
![[Pasted image 20260908085653.png]]
Khi quá trình recursive listing đi sâu hơn, nó sẽ hiển thị nội dung của tất cả các thư mục con nằm bên trong những thư mục cấp cao hơn. Trong lệnh trên, tùy chọn `--dir-only` chỉ hiển thị các thư mục, không liệt kê toàn bộ file. 
### rpcclient
`rpcclient` là một công cụ được xây dựng để sử dụng với giao thức `Samba`, cung cấp thêm nhiều chức năng thông qua `MS-RPC`. Công cụ này có thể được sử dụng để enumerate các object trong AD, thêm object, thay đổi object, xóa object. Đây là một công cụ rất linh hoạt, điều quan trọng là tìm đúng command tương ứng với mục tiêu cần thực hiện. 
#### SMB NULL Session with rpcclient
![[Pasted image 20260908090635.png]]Từ đây, chúng ta có thể bắt đầu enumerate nhiều loại thông tin khác nhau.
#### rpcclient Enumeration
Khi xem thông tin user bằng `rpcclient`, ta có thể nhận thấy một trường có tên `RID` nằm bên cạnh mỗi user. `RID (Relative Identifier)` là một định danh duy nhất, được biểu diễn dưới dạng hexadecimal, được Windows sử dụng để theo dõi và xác định các object. Để hiểu `RID` hoạt động thế nào, ta xem ví dụ dưới dây:
- SID của domain `INLANEFREIGHT.LOCAL` là: `S-1-5-21-3842939050-3880317879-2865463114`. Khi một object được tạo bên trong domain, `SID` của domain sẽ được kết hợp với `RID` để tạo thành một giá trị duy nhất đại diện cho object đó. Ví dụ `user: htb-student` có `RID = 0x457`. Hexadecimal `0x457` khi chuyển sang decimal sẽ bằng: `1111`. 
- Do đó, SID đầy đủ của user `htb-student` sẽ là: `S-1-5-21-3842939050-3880317879-2865463114-1111`
- Giá trị này là duy nhất đối với object `htb-student` trong domain `INLANEFREIGHT.LOCAL`. Chúng ta sẽ không thấy cùng cặp giá trị này được gán cho một object khác trong domain này hoặc một domain khác.
#### RPCClient User Enumeration By RID
![[Pasted image 20260908091133.png]]
Khi sử dụng command `queryuser` với `RID = 0x457`, `rpcclient` sẽ trả về thông tin của user `htb-student`. Điều này không khó vì chúng ta đã biết RID của `htb-student`. Nếu muốn liệt kê tất cả người dùng để thu thập RID của nhiều người dùng hơn, chúng ta sẽ sử dụng lệnh `enumdomusers`.
![[Pasted image 20260908091349.png]]

#### Enumdomusers
```shellsesion
rpcclient $> enumdomusers

user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]
user:[pfalcon] rid:[0x459]
user:[fanthony] rid:[0x45a]
user:[wdillard] rid:[0x45b]
user:[lbradford] rid:[0x45c]
user:[sgage] rid:[0x45d]
user:[asanchez] rid:[0x45e]
user:[dbranch] rid:[0x45f]
user:[ccruz] rid:[0x460]
user:[njohnson] rid:[0x461]
user:[mholliday] rid:[0x462]

<SNIP>  
```
Sử dụng theo cách này sẽ in ra tất cả người dùng trong domain cùng với tên RID của họ. Việc enumeration của chúng ta có thể đi vào rất nhiều chi tiết khi sử dụng `rpcclient`. Chúng ta thậm chí có thể bắt đầu thực hiện các hành động như chỉnh sửa người dùng và nhóm hoặc thêm người dùng của riêng mình vào domain. Hiện tại, chúng ta chỉ muốn thực hiện domain enumeration để xác thực các phát hiện của mình 
## Impacket Toolkit
`Impacket` là một bộ công cụ đa năng cung cấp cho chúng ta nhiều cách khác nhau để enumration, tương tác và khai thác các giao thức Windows, cũng như tìm kiếm thông tin cần thiết bằng Python. Công cụ này được duy trì tích cực và có nhiều người đóng góp, đặc biệt là khi các kỹ thuật tấn công mới xuất hiện. Chúng ta có thể thực hiện nhiều hành động khác với `Impacket`. Trước đó, trong phần poisoning, chúng ta đã lấy được hash của người dùng `wley` bằng `Responder` và crack nó để thu được mật khẩu `transporter@4`. Trong phần tiếp theo, chúng ta sẽ thấy rằng người dùng này là local admin trên máy `ACADEMY-EA-FILE`. Chúng ta sẽ sử dụng các thông tin xác thực này cho một vài hành động tiếp theo.
#### Psexec.py
Một trong những công cụ hữu ích nhất trong bộ `Impacket` là `psexec.py`. `Psexec.py` là một bản sao của executable `psexec` thuộc `Sysinternals`, nhưng hoạt động hơi khác với bản gốc. Công cụ này tạo một remote service bằng cách upload một executable có tên được tạo ngẫu nhiên lên share `ADMIN$` trên máy đích. Khi đã được thiết lập, quá trình giao tiếp diễn ra thông qua một named pipe, cung cấp interactive remote shell với quyền `SYSTEM`.
![[psexec-action.gif]]
Khi chúng ta thực thi module `psexec`, nó sẽ đưa chúng ta vào thư mục `system32` trên máy đích. Chúng ta đã chạy lệnh `whoami` để xác minh và kết quả xác nhận rằng chúng ta đã truy cập máy với quyền `SYSTEM`. Từ đây, chúng ta có thể thực hiện gần như mọi tác vụ trên máy này; từ enumeration tiếp theo cho đến persistence và lateral movement. Hãy thử một module khác của Impacket: `wmiexec.py`.
#### wmiexec.py
`Wmiexec.py` sử dụng một **semi-interactive shell**, trong đó các lệnh được thực thi thông qua [Windows Management Instrumentation](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page). Nó không thả bất kỳ file hoặc executable nào lên máy đích và tạo ra ít log hơn so với các module khác. Sau khi kết nối, nó chạy dưới quyền của local admin mà chúng ta sử dụng để kết nối (điều này có thể ít gây chú ý hơn đối với người đang tìm kiếm dấu hiệu xâm nhập so với việc nhìn thấy `SYSTEM` thực thi nhiều lệnh). Đây là một phương pháp stealth hơn để thực thi trên các máy host so với những công cụ khác, nhưng nhiều khả năng vẫn sẽ bị phát hiện bởi hầu hết các hệ thống anti-virus và EDR hiện đại. Chúng ta sẽ sử dụng cùng tài khoản như với `psexec.py` để truy cập máy host.
Sử dụng wmiexec.py
```
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5  
```
Lưu ý rằng môi trường shell này không hoàn toàn interactive, vì vậy mỗi lệnh được đưa vào sẽ thực thi một `cmd.exe` mới thông qua WMI và thực thi lệnh của bạn. Nhược điểm của cách này là nếu một defender cảnh giác kiểm tra event log và xem event ID [4688: A new process has been created](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688), họ sẽ thấy một process mới được tạo để khởi chạy `cmd.exe` và thực thi một lệnh. Đây không phải lúc nào cũng là hoạt động độc hại vì nhiều tổ chức sử dụng WMI để quản trị máy tính, nhưng nó có thể là một dấu hiệu trong quá trình điều tra.

Trong hình ảnh phía trên, cũng có thể thấy rõ rằng process đang chạy dưới context của người dùng `wley` trên máy host, chứ không phải `SYSTEM`. **Impacket** là một công cụ vô cùng giá trị với rất nhiều trường hợp sử dụng. Chúng ta sẽ thấy nhiều công cụ khác trong bộ công cụ Impacket trong suốt phần còn lại của module này. Với tư cách là một pentester làm việc với các máy Windows, công cụ này luôn nên nằm trong arsenal của chúng ta. Hãy chuyển sang công cụ tiếp theo, `Windapsearch`.
![[wmiexec-action.gif]]

### Windapsearch
[Windapsearch](https://github.com/ropnop/windapsearch) là một Python script hữu ích khác mà chúng ta có thể sử dụng để enumeration users, groups và computers từ một Windows domain bằng cách sử dụng các truy vấn LDAP. Nó có sẵn trong thư mục `/opt/windapsearch/` trên attack host của chúng ta.
Chúng ta có một số tùy chọn với Windapsearch để thực hiện enumeration cơ bản (dump users, computers và groups) cũng như enumeration chi tiết hơn. Tùy chọn `--da` (enumerate domain admins group members) và tùy chọn `-PU` (find privileged users). Tùy chọn `-PU` rất đáng chú ý vì nó sẽ thực hiện tìm kiếm đệ quy đối với những người dùng có nested group membership.
#### Windapsearch - Domain Adminsư
```
w3bis2bi@htb[/htb]$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+] Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+] ...success! Binded as: 
[+]  u:INLANEFREIGHT\forend
[+] Attempting to enumerate all Domain Admins
[+] Using DN: CN=Domain Admins,CN=Users.CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+] Found 28 Domain Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Matthew Morgan
userPrincipalName: mmorgan@inlanefreight.local

<SNIP>
```
Từ kết quả trong shell phía trên, chúng ta có thể thấy rằng nó đã enumeration được 28 users từ nhóm `Domain Admins`. Hãy ghi chú lại một vài người dùng mà chúng ta đã thấy trước đó và thậm chí có thể đã có hash hoặc cleartext password, chẳng hạn như `wley`, `svc_qualys` và `lab_adm`.
Để xác định thêm các user tiềm năng, chúng ta có thể chạy công cụ với flag `-PU` và kiểm tra những user có elevated privileges mà có thể đã bị bỏ sót. Đây là một bước kiểm tra rất hữu ích cho việc reporting vì nó nhiều khả năng sẽ giúp thông báo cho khách hàng về những user có quá nhiều quyền do nested group membership.
#### Windapsearch - Privileged Users
```
w3bis2bi@htb[/htb]$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU

[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]     Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]     ...success! Binded as:
[+]      u:INLANEFREIGHT\forend
[+] Attempting to enumerate all AD privileged users
[+] Using DN: CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]     Found 28 nested users for group Domain Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Angela Dunn
userPrincipalName: adunn@inlanefreight.local

cn: Matthew Morgan
userPrincipalName: mmorgan@inlanefreight.local

cn: Dorothy Click
userPrincipalName: dclick@inlanefreight.local

<SNIP>

[+] Using DN: CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]     Found 3 nested users for group Enterprise Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Sharepoint Admin
userPrincipalName: sp-admin@INLANEFREIGHT.LOCAL

<SNIP>
```

### Bloodhound.py
Khi chúng ta đã có domain credentials, chúng ta có thể chạy [BloodHound.py](https://github.com/fox-it/BloodHound.py), một BloodHound ingestor, từ Linux attack host của mình. BloodHound là một trong, nếu không muốn nói là một trong những công cụ có ảnh hưởng lớn nhất từng được phát hành để kiểm tra bảo mật Active Directory, và nó mang lại lợi ích rất lớn cho chúng ta với tư cách là penetration tester. Chúng ta có thể lấy một lượng lớn dữ liệu mà nếu phân tích thủ công sẽ mất rất nhiều thời gian, sau đó tạo ra các biểu diễn đồ họa hoặc "attack paths" cho thấy quyền truy cập với một user cụ thể có thể dẫn đến đâu.
Chúng ta thường sẽ tìm thấy những điểm yếu tinh vi trong môi trường AD mà có thể đã bị bỏ sót nếu không có khả năng chạy các truy vấn bằng công cụ BloodHound GUI và trực quan hóa các vấn đề. Công cụ sử dụng [graph theory](https://en.wikipedia.org/wiki/Graph_theory) để biểu diễn trực quan các mối quan hệ và phát hiện các attack path mà sẽ rất khó, thậm chí không thể phát hiện bằng những công cụ khác.
Công cụ bao gồm hai phần: [SharpHound collector](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) được viết bằng C# để sử dụng trên Windows systems, hoặc trong phần này là **BloodHound.py collector** (còn được gọi là `ingestor`), và công cụ GUI [BloodHound](https://github.com/BloodHoundAD/BloodHound/releases), cho phép chúng ta upload dữ liệu đã thu thập dưới dạng các file JSON.
Sau khi upload, chúng ta có thể chạy nhiều truy vấn được xây dựng sẵn hoặc viết các truy vấn tùy chỉnh bằng [Cypher language](https://specterops.io/blog/2017/09/18/bloodhound-intro-to-cypher/). Công cụ thu thập dữ liệu từ AD như users, groups, computers, group membership, GPOs, ACLs, domain trusts, local admin access, user sessions, computer và user properties, RDP access, WinRM access, v.v.
Ban đầu, công cụ chỉ được phát hành với một PowerShell collector, vì vậy nó phải được chạy từ một Windows host. Sau đó, một bản port sang Python (yêu cầu Impacket, `ldap3` và `dnspython`) đã được một community member phát hành. Điều này hỗ trợ rất nhiều trong các penetration test khi chúng ta có valid domain credentials nhưng không có quyền truy cập vào một domain-joined Windows host hoặc không có Windows attack host để chạy SharpHound collector.
Điều này cũng giúp chúng ta không phải chạy collector từ một domain host, việc có thể bị chặn hoặc kích hoạt các cảnh báo (mặc dù ngay cả khi chạy từ attack host của chúng ta thì nhiều khả năng cũng sẽ kích hoạt cảnh báo trong những môi trường được bảo vệ tốt).
```
w3bis2bi@htb[/htb]$ bloodhound-python -h

usage: bloodhound-python [-h] [-c COLLECTIONMETHOD] [-u USERNAME]
                         [-p PASSWORD] [-k] [--hashes HASHES] [-ns NAMESERVER]
                         [--dns-tcp] [--dns-timeout DNS_TIMEOUT] [-d DOMAIN]
                         [-dc HOST] [-gc HOST] [-w WORKERS] [-v]
                         [--disable-pooling] [--disable-autogc] [--zip]

Python based ingestor for BloodHound
For help or reporting issues, visit https://github.com/Fox-IT/BloodHound.py

optional arguments:
  -h, --help            show this help message and exit
  -c COLLECTIONMETHOD, --collectionmethod COLLECTIONMETHOD
                        Which information to collect. Supported: Group,
                        LocalAdmin, Session, Trusts, Default (all previous),
                        DCOnly (no computer connections), DCOM, RDP,PSRemote,
                        LoggedOn, ObjectProps, ACL, All (all except LoggedOn).
                        You can specify more than one by separating them with
                        a comma. (default: Default)
  -u USERNAME, --username USERNAME
                        Username. Format: username[@domain]; If the domain is
                        unspecified, the current domain is used.
  -p PASSWORD, --password PASSWORD
                        Password

  <SNIP>
```
Như chúng ta có thể thấy, công cụ chấp nhận nhiều **collection methods** khác nhau với flag `-c` hoặc `--collectionmethod`. Chúng ta có thể thu thập dữ liệu cụ thể như user sessions, users và groups, object properties, ACLs hoặc chọn `all` để thu thập nhiều dữ liệu nhất có thể. Hãy chạy công cụ theo cách này.
#### Executing BloodHound.py
```
w3bis2bi@htb[/htb]$ sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all 

INFO: Found AD domain: inlanefreight.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 1 domains
INFO: Found 2 domains in the forest
INFO: Found 564 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 2951 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 183 groups
INFO: Found 2 trusts
INFO: Starting computer enumeration with 10 workers

<SNIP>
```
Lệnh trên đã thực thi BloodHound.py với user `forend`. Chúng ta chỉ định nameserver của mình là Domain Controller bằng flag `-ns` và domain là `INLANEFREIGHt.LOCAL` bằng flag `-d`. Flag `-c all` yêu cầu công cụ thực hiện tất cả các kiểm tra.

Sau khi script hoàn tất, chúng ta sẽ thấy các file output trong thư mục làm việc hiện tại với định dạng `<date_object.json>`.

#### Viewing the Results
```
w3bis2bi@htb[/htb]$ ls 
20220307163102_computers.json 20220307163102_domains.json 20220307163102_groups.json 20220307163102_users.json
```
#### Upload the Zip File into the BloodHound GUI
Sau đó, chúng ta có thể nhập `sudo neo4j start` để khởi động service [neo4j](https://neo4j.com/), khởi động database mà chúng ta sẽ load dữ liệu vào và cũng chạy các Cypher query.
Tiếp theo, chúng ta có thể nhập `bloodhound` từ Linux attack host khi đã đăng nhập bằng `freerdp` để khởi động ứng dụng BloodHound GUI và upload dữ liệu. Credentials đã được điền sẵn trên Linux attack host, nhưng nếu vì lý do nào đó xuất hiện credential prompt, hãy sử dụng:  `user == neo4j` / `pass == HTB_@cademy_stdnt!`. Sau khi hoàn thành tất cả những bước trên, chúng ta sẽ có công cụ BloodHound GUI được load với một trạng thái trống. Bây giờ chúng ta cần upload dữ liệu. Chúng ta có thể upload từng file JSON hoặc zip chúng trước bằng một lệnh như `zip -r ilfreight_bh.zip *.json` rồi upload Zip file.
Chúng ta thực hiện việc này bằng cách nhấn nút `Upload Data` ở phía bên phải cửa sổ (mũi tên màu xanh lá). Khi cửa sổ file browser xuất hiện để chọn file, chọn zip file (hoặc từng JSON file) (mũi tên màu đỏ) và nhấn `Open`.
#### Uploading the Zip File
Bây giờ dữ liệu đã được load, chúng ta có thể sử dụng tab **Analysis** để chạy các query trên database. Những query này có thể được tùy chỉnh và cụ thể theo những gì bạn muốn bằng cách sử dụng [custom Cypher queries](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/). Có rất nhiều cheat sheet hữu ích để hỗ trợ chúng ta ở đây.

Chúng ta sẽ thảo luận về custom Cypher queries chi tiết hơn trong một phần sau. Như hình bên dưới, chúng ta có thể sử dụng các query **Path Finding** được tích hợp sẵn trong `Analysis tab` ở phía `Left` của cửa sổ.
![[Pasted image 20260908095928.png]]

#### Searching for Relationships
![[Pasted image 20260908100036.png]]
Query được chọn để tạo ra map phía trên là `Find Shortest Paths To Domain Admins`. Nó sẽ cung cấp cho chúng ta bất kỳ logical path nào mà nó tìm thấy thông qua các mối quan hệ users/groups/hosts/ACLs/GPOs, v.v., có khả năng cho phép chúng ta escalate lên quyền **Domain Administrator** hoặc tương đương.
Điều này sẽ cực kỳ hữu ích khi lập kế hoạch cho các bước tiếp theo để lateral movement qua network. Hãy dành thời gian thử nghiệm với các tính năng khác nhau: xem tab `Database Info` sau khi upload dữ liệu, tìm kiếm một node như `Domain Users` và cuộn qua tất cả các tùy chọn trong tab `Node Info`, kiểm tra các query được xây dựng sẵn trong tab `Analysis`, trong đó có nhiều query rất mạnh và có thể nhanh chóng tìm ra nhiều cách khác nhau để domain takeover.
Cuối cùng, hãy thử nghiệm một số custom Cypher query bằng cách chọn những query thú vị từ Cypher cheatsheet được liên kết ở trên, dán chúng vào ô `Raw Query` ở phía dưới và nhấn enter.
Bạn cũng có thể thử menu `Settings` bằng cách nhấn biểu tượng bánh răng ở phía bên phải màn hình và điều chỉnh cách các node và edge được hiển thị, bật query debug mode và bật dark mode.
Trong suốt phần còn lại của module này, chúng ta sẽ sử dụng BloodHound theo nhiều cách khác nhau, nhưng nếu muốn nghiên cứu chuyên sâu về công cụ BloodHound, hãy tham khảo module [Active Directory BloodHound](https://academy.hackthebox.com/course/preview/active-directory-bloodhound).

