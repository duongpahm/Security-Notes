## Enumerating the Password Policy - from Linux - Credentialed
Chúng ta có thể lấy domain password policy bằng nhiều phương pháp khác nhau, tùy thuộc vào cách domain được cấu hình và việc chúng ta có valid domain credentials hay không.
Khi có valid domain credentials, password policy cũng có thể được lấy từ xa bằng các tool như [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) hoặc `rpcclient`.
![[Pasted image 20260906063429.png]]

## Enumerating the Password Policy - from Linux - SMB NULL Sessions
Khi không có credentials, chúng ta vẫn có thể lấy được password policy thông qua SMB NULL sessions hoặc LDAP anonymous bind.
Phương pháp đầu tiên là sử dụng SMB NULL session. SMB NULL session cho phép một attacker chưa được xác thực truy xuất thông tin từ domain, chẳng hạn như danh sách đầy đủ về users, groups, computers, các thuộc tính của user account và domain password policy.
Những cấu hình sai liên quan đến SMB NULL session thường là kết quả của việc các Domain Controller Legacy được nâng cấp trực tiếp tại chỗ, qua đó giữ lại những cấu hình không an toàn vốn được thiết lập mặc định trong các phiên bản Windows Server cũ hơn.
Khi tạo domain trên những phiên bản Windows Server trước đây, anonymous access được cấp cho một số share nhất định, cho phép thực hiện domain enumeration. 
SMB NULL session có thể được enumeration khá dễ dàng. Để thực hiện enumeration, chúng ta có thể sử dụng các tool như `enum4linux`, `CrackMapExec`, `rpcclient`, v.v.
Chúng ta có thể sử dụng [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) để kiểm tra Domain Controller xem có cho phép SMB NULL session access hay không. Sau khi kết nối, chúng ta có thể thực thi một RPC command như `querydominfo` để lấy thông tin về domain và xác nhận NULL session access.
![[Pasted image 20260906064207.png]]
Chúng ta cũng có thể lấy password policy. Có thể thấy password policy tương đối yếu, cho phép mật khẩu có độ dài tối thiểu là 8 ký tự.
![[Pasted image 20260906064245.png]]
`enum4linux` là một tool được xây dựng dựa trên [Samba suite of tools](https://www.samba.org/samba/docs/current/man-html/samba.7.html), bao gồm `nmblookup`, `net`, `rpcclient` và `smbclient`, nhằm phục vụ việc enumeration các Windows host và domain.
Tool này được cài đặt sẵn trên nhiều penetration testing distro khác nhau, trong đó có Parrot Security Linux. Bên dưới là một ví dụ về output hiển thị những thông tin mà `enum4linux` có thể cung cấp. Dưới đây là một số enumeration tool phổ biến cùng với các port mà chúng sử dụng:

| Tool      | Ports                                               |
| --------- | --------------------------------------------------- |
| nmblookup | 137/UDP                                             |
| nbtstat   | 137/UDP                                             |
| net       | 139/TCP, 135/TCP, TCP và UDP 135 và 49152 đến 65535 |
| rpcclient | 135/TCP                                             |
| smbclient | 445/TCP                                             |
#### Using enum4linux
![[Pasted image 20260906064424.png]]
#### Using enum4linux-ng
![[Pasted image 20260906064801.png]]
`enum4linux ng` cung cấp output rõ ràng hơn và đồng thời tạo JSON तथा YAML output thông qua flag `-oA`.
![[Pasted image 20260906064828.png]]
## Enumerating Null Session - from Windows
Việc thực hiện loại null session attack này từ Windows ít phổ biến hơn. Tuy nhiên, chúng ta có thể sử dụng command `net use \\host\ipc$ "" /u:""` để thiết lập một null session từ Windows machine và xác nhận xem chúng ta có thể thực hiện thêm các hoạt động enumeration dạng này hay không.
#### Establish a null session from windows
```cmd
C:\htb> net use \\DC01\ipc$ "" /u:"" 
The command completed successfully.
```
Chúng ta cũng có thể sử dụng username/password combination để thử kết nối. Hãy xem một số lỗi phổ biến khi authentication.
#### Error: Account is Disabled
```cmd
C:\htb> net use \\DC01\ipc$ "" /u:guest
System error 1331 has occurred. This user can't sign in because this account is currently disabled.
```
#### Error: Password is Incorrect
```cmd
C:\htb> net use \\DC01\ipc$ "password" /u:guest 
System error 1326 has occurred. The user name or password is incorrect.
```
#### Error: Account is locked out (Password Policy)
```cmd
C:\htb> net use \\DC01\ipc$ "password" /u:guest 
System error 1909 has occurred. The referenced account is currently locked out and may not be logged on to.
```
## Enumerating the Password Policy - from Linux - LDAP Anonymous Bind
[LDAP anonymous bind](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled) cho phép attacker chưa xác thực truy xuất thông tin từ domain, chẳng hạn như danh sách đầy đủ users, groups, computer, user account attributes và domain passoword policy. 
Đây là một legacy configuration và kể từ Windows Server 2003, chỉ những authenticated user mới được phép khởi tạo LDAP request. Tuy nhiên chúng ta vẫn có thể bắt đầu cấu hình này trong một số trường hợp. Ví dụ, administrator có thể từng cấu hình một application cụ thể để cho phép anonymous bind và vô tình cấp nhiều quyền truy cập hơn dự kiến. Điều này có thể khiến authenticated user có quyền truy cập vào toàn bộ object trong AD.
Với LDAP anonymous bind, chúng ta có thể sử dụng các tool enumeration chuyên biệt cho LDAP như `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, v.v. để lấy password policy.
Với [ldapsearch](https://linux.die.net/man/1/ldapsearch), cú pháp có thể hơi cumbersome nhưng hoàn toàn khả thi. Một command để lấy password policy như sau:
![[Pasted image 20260906065710.png]]**Lưu ý:** Trong các phiên bản `ldapsearch` mới hơn, parameter `-h` đã được deprecated và được thay thế bằng `-H`.
Ở đây, chúng ta có thể thấy:
- `minPwdLength: 8` cho biết độ dài password tối thiểu là **8 ký tự**.
- `lockoutThreshold: 5` cho biết account sẽ bị lockout sau **5 lần authentication thất bại**.
- `pwdProperties: 1` cho biết **password complexity được enable**.

## Enumerating the Password Policy - from Windows
Nếu chúng ta có thể authenticate vào domain từ một Windows host, chúng ta có thể sử dụng các Windows binary có sẵn như `net.exe` để truy xuất password policy. Ngoài ra, chúng ta cũng có thể sử dụng nhiều tool khác như PowerView, phiên bản CrackMapExec được port sang Windows, SharpMapExec, SharpView, v.v.

Việc sử dụng các built in command rất hữu ích khi chúng ta đã foothold trên một Windows system nhưng không thể transfer tool vào đó, hoặc khi client cung cấp cho chúng ta một Windows system để thực hiện assessment nhưng không có cách nào đưa tool vào hệ thống.

Một ví dụ sử dụng binary `net.exe` có sẵn trong Windows:
```cmd
C:\htb> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              8
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.
```
Từ output trên, chúng ta có thể thu thập được các thông tin sau:
- Password không bao giờ hết hạn vì `Maximum password age` được đặt thành `Unlimited`.
- Độ dài password tối thiểu là 8 ký tự, do đó khả năng weak password được sử dụng là khá cao.
- Account lockout threshold là 5 lần nhập password sai.
- Account sẽ duy trì trạng thái locked trong 30 phút.
Password policy này rất thuận lợi cho password spraying. Yêu cầu tối thiểu 8 ký tự cho phép chúng ta thử những weak password phổ biến như `Welcome1`. Lockout threshold bằng 5 có nghĩa là chúng ta có thể thực hiện 2 đến 3 lần spray để đảm bảo an toàn trong mỗi 31 phút mà không có nguy cơ lockout account.
Nếu một account đã bị lockout, account đó sẽ tự động được unlock sau 30 phút mà không cần administrator can thiệp. Tuy nhiên, chúng ta nên tránh lockout **BẤT KỲ** account nào bằng mọi giá.

PowerView cũng rất hữu ích cho việc này:
```powershell
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy

Unicode        : @{Unicode=yes}
SystemAccess   : @{MinimumPasswordAge=1; MaximumPasswordAge=-1; MinimumPasswordLength=8; PasswordComplexity=1;
                 PasswordHistorySize=24; LockoutBadCount=5; ResetLockoutCount=30; LockoutDuration=30;
                 RequireLogonToChangePassword=0; ForceLogoffWhenHourExpire=0; ClearTextPassword=0;
                 LSAAnonymousNameLookup=0}
KerberosPolicy : @{MaxTicketAge=10; MaxRenewAge=7; MaxServiceAge=600; MaxClockSkew=5; TicketValidateClient=1}
Version        : @{signature="$CHICAGO$"; Revision=1}
RegistryValues : @{MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash=System.Object[]}
Path           : \\INLANEFREIGHT.LOCAL\sysvol\INLANEFREIGHT.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHI
                 NE\Microsoft\Windows NT\SecEdit\GptTmpl.inf
GPOName        : {31B2F340-016D-11D2-945F-00C04FB984F9}
GPODisplayName : Default Domain Policy
```
PowerView cung cấp cho chúng ta output tương tự command `net accounts`, chỉ khác về format. Ngoài ra, output còn cho thấy password complexity đang được enable với `PasswordComplexity=1`. Tương tự như trên Linux, chúng ta có nhiều tool để truy xuất password policy khi đang ở trên Windows system, bất kể đó là attack system của chúng ta hay một system do client cung cấp.
PowerView và SharpView luôn là những lựa chọn tốt, bên cạnh CrackMapExec, SharpMapExec và các tool khác. Việc lựa chọn tool phụ thuộc vào mục tiêu của assessment, các yêu cầu về stealth, sự hiện diện của antivirus hoặc EDR và những restriction khác trên target host.
## Analyzing the Password Policy
Đến đây, chúng ta đã lấy được password policy bằng nhiều phương pháp khác nhau. Hãy phân tích từng thành phần của policy đối với domain `INLANEFREIGHT.LOCAL`.
- Độ dài password tối thiểu là 8 ký tự. 8 ký tự là một giá trị rất phổ biến, tuy nhiên hiện nay ngày càng có nhiều tổ chức yêu cầu password dài từ 10 đến 14 ký tự. Điều này có thể loại bỏ một số password option đối với chúng ta, nhưng không hoàn toàn loại bỏ password spraying vector.
- Account lockout threshold là 5. Không hiếm trường hợp thấy threshold thấp hơn, chẳng hạn như 3, hoặc thậm chí không thiết lập lockout threshold.
- Lockout duration là 30 phút. Giá trị này có thể cao hơn hoặc thấp hơn tùy thuộc vào tổ chức. Nếu chúng ta vô tình lockout một account, hãy tránh điều này, account sẽ được unlock sau khi khoảng thời gian 30 phút kết thúc.
- Account tự động được unlock. Ở một số tổ chức, administrator phải manually unlock account. Chúng ta không bao giờ muốn lockout account khi thực hiện password spraying, nhưng đặc biệt phải tránh điều này trong những tổ chức mà administrator phải can thiệp để unlock hàng trăm hoặc hàng nghìn account bằng thủ công hoặc script.
- Password complexity được enable, nghĩa là user phải lựa chọn password đáp ứng 3 trong 4 yêu cầu sau: chữ hoa, chữ thường, chữ số và ký tự đặc biệt. Ví dụ `Password1` hoặc `Welcome1` đáp ứng yêu cầu về complexity trong trường hợp này, nhưng rõ ràng vẫn là những weak password.
Password policy mặc định khi tạo một domain mới như sau, và đã có rất nhiều tổ chức không bao giờ thay đổi policy mặc định này:

|Policy|Default Value|
|---|---|
|Enforce password history|24 days|
|Maximum password age|42 days|
|Minimum password age|1 day|
|Minimum password length|7|
|Password must meet complexity requirements|Enabled|
|Store passwords using reversible encryption|Disabled|
|Account lockout duration|Not set|
|Account lockout threshold|0|
|Reset account lockout counter after|Not set|
