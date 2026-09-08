# Windows Group Privileges

## Windows Built-in Groups
Các máy chủ Windows đặc biệt là Domain Controller, sở hữu nhiều nhóm tích hợp được cài đặt sẵn cùng hệ điều hành hoặc được thêm vào khi vai trò Active Directory Domain Services được cài đặt trên một hệ thống để nâng cấp máy chủ thành Domain Controller. Nhiều nhóm trong số này được trao đặc quyền đặc biệt cho thành viên của group đó. Chúng ta sẽ tập trung vào các nhóm tích hợp sau đây, mỗi nhóm này tồn tại trên các hệ thống từ Server 2008 R2 đến nay, ngoại trừ quản trị viên Hyper-V.
Các tài khoản có thể được gán vào các nhóm này để thực thi nguyên tắc đặc quyền tối thiểu và tránh việc tạo thêm nhiều quản trị domain, và quản trị viên doanh nghiệp để thực hiện các tác vụ cụ thể, chẳng hạn như sao lưu. Đôi khi các ứng dụng bên thứ ba cũng yêu cầu một số đặc quyền nhất định, có thể được cấp bằng cách gán một tài khoản dịch vụ vào một trong các nhóm này. Các tài khoản cũng có thể được thêm vào do nhầm lẫn hoặc còn sót lại khi kiểm thử một công cụ hoặc một kịch bản cụ thể. 

| Backup Operators       | Event Log Readers | DnsAdmins        |
| ---------------------- | ----------------- | ---------------- |
| Hyper-V Administrators | Print Operators   | Server Operators |
### Backup Operators
Sau khi truy cập được vào một máy, chúng ta có thể sử dụng lệnh `whoami \groups` để xem các nhóm mà users đó đang là thành viên. Bây giờ chúng ta xét trường hợp user là thành viên của nhóm Backup Operators, việc là thành viên của nhóm này cấp cho chúng ta các đặc quyền `SeBackup` và `SeRestore`. Đặc quyền `SeBackupPrivilege` cho phép ta duyệt bất kỳ thư mục nao và liệt kê nội dung thư mục, ngay cả khi không có mục kiểm soát truy cập (ACE) dành cho chúng ta trong danh sách kiểm soát truy cập (ACL) của thư mục đó. Tuy nhiên, chúng ta không thể thực hiện việc này bằng lệnh sao chép tiêu chuẩn. Thay vào đó, chúng ta cần sao chép dữ liệu một cách lập trình, đảm bảo chỉ định cờ`FILE_FLAG_BACKUP_SEMANTICS`.
![[Windows Privilege Escalation.png|center|border|650]]Kết quả như có thể thấy chúng ta đã xác định được user đang là thành viên của Backup Operators, Users và Remote Desktops Users. Tiếp theo chúng ta tiến hành kiểm tra quyền của user bằng lệnh `whoami /priv`
![[Screenshot 2026-08-13 at 08.17.09.png|center|border|650]]
Output cho thấy user có các quyền liên quan đến Backup/Restore nhưng hiện tại hầu hết đều ở trạng thái Disabled. Tiếp theo chúng ta thực hiện nghiên cứu các leo thang đặc quyền, mình sẽ sử dụng [PoC](https://github.com/giuliano108/SeBackupPrivilege) để tiến hành bypass.
Đầu tiên mình import các module `SeBackupPrivilegeUtils.dll` và `SeBackupPrivilegeCmdLets.dll` để mở rộng khả năng của PowerShell do mặc định của PS không có lệnh `cmdlet` đẻ quản lý đặc quyền `SeBackupPrivilege`. Các thư viện này cung cấp thêm các lệnh tuỳ chỉnh như `Get-SeBackupPrivilege`, `Set-SeBackupPrivilege` và các hàm hỗ trợ sao chép tệp với cờ `FILE_FLAG_BACKUP_SEMANTICS`, thêm vào đó các module trên giúp khai thác đặc quyền Backup vì đặc quyền này cho phép đọc bất kỳ tệp nào trên hệ thống, kể cả khi không có quyền ACL.
![[Screenshot 2026-08-13 at 08.27.16.png|center|border|650]]
Sau khi import, mình thực hiện xác minh `SeBackupPrivilege` đã được bật bằng lệnh `whoami /priv` hoặc `Get-SeBackupPrivilege`.

>Lưu ý: Dựa trên cấu hình của máy chủ, có thể cần khởi tạo dấu nhắc cmd có quyền nâng cao để vượt qua UAC và sở hữu quyền này.

![[Screenshot 2026-08-13 at 08.30.18.png|center|border|650]]
Output cho thấy đặc quyền `SeBackupPrivilege` đang bị tắt, mình sẽ bật nó bằng `Set-SeBackupPrivilege`.
![[Screenshot 2026-08-13 at 08.32.34.png|center|border|650]]Như kết quả trên, đặc quyền đã được bật thành công. Đặc quyền này hiện có thể khai thác để sao chép bất kỳ tệp được bảo vệ nào.
![[Screenshot 2026-08-13 at 08.35.48.png|center|border|650]]![[Screenshot 2026-08-13 at 08.38.51.png|center|border]]
Các lệnh trên minh hoạ cách thông tin nhạy cảm được truy cập mà không cần sở hữu các quyền yêu cầu, đến đây ta thu thập flag để submit cho lab:![[Screenshot 2026-08-13 at 08.41.22.png|center|border|650]]
### Attacking a Domain Controller - Copying NTDS.dit
Nhóm này cũng cho phép đăng nhập cục bộ vào Domain Controller. Cơ sở dữ liệu AD `NTDS.dit` là một mục tiêu quan trọng vì nó chứa các giá trị băm NTLM của tất cả đối tượng người dùng và máy tính trong miền và tệp này cũng bị khoá và không thể truy cập bởi người dùng không có đặc quyền.
Với tệp `NTDS.dit` bị khoá mặc định, ta có thể sử dụng tiện ích `diskshadow` của Windows để tạo một bản sao của ổ `C` và xuất nó dưới dạng ổ `E`. Tệp `NTDS.dit` trong bản sao này. sẽ không bị hệ thống sử dụng.
```powershell
PS C:\htb> diskshadow.exe

Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC,  10/14/2020 12:57:52 AM

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
```

![[Screenshot 2026-08-13 at 08.50.29.png|Center|border|650]]

Tiếp theo mình sử dụng lệnh `Copy-FileSeBackupPrivilege` để vượt qua ACL và sao chép tệp NTDS.dit về máy cục bộ.
![[Screenshot 2026-08-13 at 08.52.29.png|center|border]]
Với đặc quyền chúng ta cũng cho phép chúng ta sao lưu các registry SAM và SYSTEM, từ đó có thể trích xuất thông tin xác thực tài khoản cục bộ ngoại tuyến bằng côn gục `secretsdump.py` của Impacket.![[Screenshot 2026-08-13 at 08.54.40.png|center|border|650]]Đáng lưu ý rằng nếu một thư mục hoặc tệp có mục từ chối rõ ràng (explicit deny) đối với người dùng hiện tại hoặc một nhóm mà họ thuộc về, điều này sẽ ngăn chặn việc truy cập, ngay cả khi cờ `FILE_FLAG_BACKUP_SEMANTICS` được chỉ định.
Sau khi đã trích xuất tệp NTDS.dit, chúng ta có thể sử dụng một công cụ như `secretsdump.py` hoặc module PowerShell `DSInternals` để trích xuất tất cả thông tin xác thực tài khoản Active Directory. Hãy lấy giá trị băm NTLM của tài khoản quản trị viên miền bằng DSInternals.
```powershell
PS C:\htb> Import-Module .\DSInternals.psd1
PS C:\htb> $key = Get-BootKey -SystemHivePath .\SYSTEM
PS C:\htb> Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key

DistinguishedName: CN=Administrator,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
Sid: S-1-5-21-669053619-2741956077-1013132368-500
Guid: f28ab72b-9b16-4b52-9f63-ef4ea96de215
SamAccountName: Administrator
SamAccountType: User
UserPrincipalName:
PrimaryGroupId: 513
SidHistory:
Enabled: True
UserAccountControl: NormalAccount, PasswordNeverExpires
AdminCount: True
Deleted: False
LastLogonDate: 5/6/2021 5:40:30 PM
DisplayName:
GivenName:
Surname:
Description: Built-in account for administering the computer/domain
ServicePrincipalName:
SecurityDescriptor: DiscretionaryAclPresent, SystemAclPresent, DiscretionaryAclAutoInherited, SystemAclAutoInherited,
DiscretionaryAclProtected, SelfRelative
Owner: S-1-5-21-669053619-2741956077-1013132368-512
Secrets
  NTHash: cf3a5525ee9414229e66279623ed5c58
  LMHash:
  NTHashHistory:
  LMHashHistory:
  SupplementalCredentials:
    ClearText:
    NTLMStrongHash: 7790d8406b55c380f98b92bb2fdc63a7
    Kerberos:
      Credentials:
        DES_CBC_MD5
          Key: d60dfbbf20548938
      OldCredentials:
      Salt: WIN-NB4NGP3TKNKAdministrator
      Flags: 0
    KerberosNew:
      Credentials:
        AES256_CTS_HMAC_SHA1_96
          Key: 5db9c9ada113804443a8aeb64f500cd3e9670348719ce1436bcc95d1d93dad43
          Iterations: 4096
        AES128_CTS_HMAC_SHA1_96
          Key: 94c300d0e47775b407f2496a5cca1a0a
          Iterations: 4096
        DES_CBC_MD5
          Key: d60dfbbf20548938
          Iterations: 4096
      OldCredentials:
      OlderCredentials:
      ServiceCredentials:
      Salt: WIN-NB4NGP3TKNKAdministrator
      DefaultIterationCount: 4096
      Flags: 0
    WDigest:
Key Credentials:
Credential Roaming
  Created:
  Modified:
  Credentials:
```

Tiện ích tích hợp `robocopy` cũng có thể được sử dụng để sao chép tệp ở chế độ sao lưu. Robocopy là một công cụ nhân bản thư mục dòng lệnh. Nó có thể được sử dụng để tạo các tác vụ sao lưu và bao gồm các tính năng như sao chép đa luồng, tự động thử lại, khả năng tiếp tục sao chép, v.v. Robocopy khác với lệnh `copy` ở chỗ thay vì chỉ sao chép tất cả các tệp, nó có thể kiểm tra thư mục đích và xóa các tệp không còn trong thư mục nguồn. Nó cũng có thể so sánh các tệp trước khi sao chép để tiết kiệm thời gian bằng cách không sao chép các tệp chưa được thay đổi kể từ lần chạy tác vụ sao chép/sao lưu trước đó.
![[Screenshot 2026-08-13 at 09.05.56.png|center|border|650]]
## Event Log Readers
Trong trường hợp tính năng auditing of process creation và các giá trị dòng lệnh tương đương được bật, thì thông tin này được lưu vào nhật ký bảo mật Windows dưới dạng sự kiện có ID `4688: A new process has been created`. Các tổ chức có thể bật ghi nhật ký dòng lệnh của tiến trình để giúp các chuyên gia phòng thủ giám sát và xác định các hành vi có khả năng độc hại, đồng thời nhận diện các tệp thực thi không nên có trên hệ thống. Dữ liệu này có thể được chuyển đến công cụ SIEM hoặc đưa vào công cụ tìm kiếm như ElasticSearch, nhằm giúp các chuyên gia phòng thủ có khả năng hiển thị các tệp thực thi đang chạy trên các hệ thống trong mạng.

Các quản trị viên hoặc thành viên của nhóm Event Log Readers có quyền truy cập nhật ký này. Có thể hình dung rằng quản trị viên hệ thống muốn thêm người dùng quyền lực hoặc nhà phát triển vào nhóm để thực hiện các tác vụ nhất định mà không cần cấp cho họ quyền truy cập quản trị.

Mình thực hiện các lệnh như dưới đây để kiểm tra quyền và thuộc thành viên của groups nào:![[Screenshot 2026-08-13 at 09.24.14.png|center|border|650]]![[Screenshot 2026-08-13 at 09.24.51.png|center|border|650]]Kết quả trên cho thấy hiện tại user đang là thành viên của nhóm Event Log Readers. Chúng ta có thể truy vấn các sự kiện Windows từ dòng lệnh bằng tiện ích `wevtutil` và lệnh ghép ngắn PowerShell `Get-WinEvent`.

Tiếp theo mình sử dụng cmdline dưới đây để thực hiện tìm kiếm các dòng lệnh chứa thông tin xác thực trong nhật ký bảo mật Windows, trong đó `wevtutil qe Security` có ý nghĩa truy vấn nhật ký bảo mật, `/rd:true` giúp đọc ngược sự kiện mới nhất lên đầu, `/f:text` định dạng xuất văn bản thuẩn:![[Screenshot 2026-08-13 at 09.30.02.png|center|border|650]]
Ngoài ra chúng ta có thể sử dụng `Get-WinEvent` để kiếm nhật ký bảo mật, như kết quả dưới đây![[Screenshot 2026-08-13 at 09.38.23.png|center|border|650]]Các nhật ký khác bao gồm nhật ký PowerShell Operational, cũng có thể chứa thông tin nhạy cảm hoặc thông tin xác thực nếu tính năng ghi nhật ký khối lệnh (script block logging) hoặc ghi nhật ký module được bật. Nhật ký này có thể truy cập được bởi người dùng không có đặc quyền.

## DnsAdmins
Thành viên của nhóm DnsAdmins có quyền truy cập thông tin DNS trên mạng, dịch vụ DNS của Windows hỗ trợ các plugin tuỳ chỉnh và có thể gọi các hàm từ chúng để phân giải truy vấn tên không nằm trong phạm vi của bất kỳ vùng DNS được lưu trữ cục bộ nào. Dịch vụ DNS chạy với quyền `NT AUTHORITY\SYSTEM`, nên việc là thành viên của nhóm này có thể được khai thác để leo thang đặc quyền trên Domain Contoller hoặc trong trường hợp một máy chủ riêng biệt đang đóng vai trò là máy chủ DNS cho Domain. Ta có thể sử dụng tiện ích `dnscmd` tích hợp để chỉ định đường dẫn của plugin DLL. Các kịch bản tấn công có thể được thực hiện khi DNS chạy trên Domain Controller:
- Quản lý DNS được thực hiện qua RPC
- `ServerLevelPluginDll` cho phép chúng ta tải một DLL tuỳ chỉnh mà không cần xác minh đường dẫn DLL. Việc này có thể thực hiện bằng công cụ `dnscmd` từ dòng lệnh.
- Khi một thành viên của nhóm DnsAdmins chạy lệnh `dnscmd` bên dưới, khoá registry `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` được điền giá trị.
- Khi dịch vụ DNS được khởi động lại, DLL tại đường dẫn này sẽ được tải (tức là một chia sẻ mạng mà tài khoản máy của Domain Controller có thể truy cập).
- Các attacker có thể tải một DLL tuỳ chỉnh để thu được reverse shell hoặc thậm chí tải một công cụ như Mimikatz dưới dạng DLL để trích xuất thông tin xác thực.
### Leveraging DnsAdmins Access
Mình bắt đầu tạo một file DLL độc hại để thêm người dùng vào nhóm domain admins bằng `msfvenom`, trước đó thì sẽ kiểm tra quyền và groups của user:
![[Screenshot 2026-08-13 at 10.33.05.png|center|border|650]]![[Screenshot 2026-08-13 at 10.35.28.png|center|border|650]]
Mình thực hiện bật http server và download về máy victim, kết quả đã thành công như dưới đây:![[Screenshot 2026-08-13 at 10.38.44.png|center|border|650]]![[Screenshot 2026-08-13 at 10.39.13.png|center|border|650]]
Tiếp theo mình thực hiện đặt lại giá trị của thuộc tính registry `serverlevelplugindll`, kết quả dưới đây cho thấy ta đã thực hiện thành công và xác minh được rằng `netadm` có quyền thực thi `dnscmd` thuộc đặc quyền nhóm DnsAdmins.
![[Screenshot 2026-08-13 at 10.43.59.png|center|border|650]]
Như kết quả trên xác nhận được user `netadm` là thành viên duy nhất của nhóm `DnsAdmins`. Với thiết lập registry chứa đường dẫn của plugin độc hại được cấu hình và payload đã được tạo, DLL sẽ được tải vào lần khởi động tiếp theo của dịch vụ DNS. Việc là thành viên của nhóm `DnsAdmins` không cho phép khởi động lại dịch vụ DNS, nhưng điều này có thể là điều mà quản trị viên hệ thống cho phép các quản trị viên DNS thực hiện.

Sau khi cos SID của người dùng, chúng ta có thể sử dụng lệnh sc để kiểm tra quyền trên dịch vụ, ta có thể thấy quyền của người dùng của chúng ta có quyền `RPWP` tương ứng với `SERVICE_START` và `SERVICE_STOP`:
![[Screenshot 2026-08-13 at 10.54.34.png|center|border|650]]
Sau khi xác nhận các quyền này, mình có thể thực hiện các lệnh sau để dừng và khởi động dịch vụ:
![[Screenshot 2026-08-13 at 11.00.55.png|center|border|650]]
Bây giờ tắt `xfreerdp` sau đó truy cập lại rồi kiểm tra quyền![[Screenshot 2026-08-13 at 16.22.48.png|center|border|650]]Tiếp theo mình dùng lệnh `run /user:netadm cmd` để chạy Administrator:![[Screenshot 2026-08-13 at 16.27.54.png|center|border|650]]
## Hyper-V Administrators
Nhóm Hyper-V Administrators có toàn quyền truy cập tất cả các tính năng Hyper-V. Nếu các Domain Controller đã được ảo hoá, thì các quản trị viên ảo hoá nên được xem là quản trị viên domain. Các quản trị viên domain có thể dễ dàng tạo một bản sao của Domain Controller đang hoạt động và gắn ổ đĩa ảo ngoại tuyến để thu thập được tệp `NTDS.dit` và trích xuất các giá trị băm mật khẩu NTLM của tất cả người dùng trong domain.
Khi xoá một máy ảo, `vmms.exe` cố gắng khôi phục quyền tệp gốc trên tệp `.vhdx` tương ứng và thực hiện việc này với tư các `NT AUTHORITY\SYSTEM` mà không mạo danh người dùng. Chúng ta có thể xoá `.vhdx` và tạo một hard link gốc để trọ tệp này đến một SYSTEM được bảo vệ, mà chúng sẽ có toàn quyền kiểm soát.
Nếu hệ điều hành dễ bị tổn thương trước CVE-2018-0952 hoặc CVE-2019-0841, chúng ta có thể khai thác điều này để giành đặc quyền SYSTEM. Nếu không, chúng ta có thể cố gắng tận dụng một ứng dụng trên máy chủ đã cài đặt một dịch vụ chạy trong ngữ cảnh SYSTEM, mà người dùng không có đặc quyền có thể khởi động.
#### Tệp mục tiêu
Một ví dụ về điều này là Firefox, cài đặt Dịch vụ Bảo trì Mozilla. Chúng ta có thể cập nhật khai thác này (một bằng chứng khái niệm cho hard link NT) để cấp cho người dùng hiện tại toàn quyền kiểm soát trên tệp sau:
```shellsession
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### Chiếm quyền sở hữu tệp
Sau khi chạy kịch bản PS, chúng ta sẽ có toàn quyền kiểm soát tệp này và có thể chiếm quyền sở hữu nó:
```shellsession
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### Khởi động Dịch vụ Bảo trì Mozilla
Tiếp theo chúng ta có thể thay thế tệp này bằng một tệp `maintenanceservice.exe` độc hại, khởi động dịch vụ bảo trì và thu được quyền thực thi lệnh với tư cách SYSTEM.
```shellsession
C:\htb> sc.exe start MozillaMaintenance
```
>**Lưu ý:** Vector này đã được giảm thiểu bởi các bản cập nhật bảo mật Windows tháng 3 năm 2020, thay đổi hành vi liên quan đến hard link.

## Print Operators
Print Operator là một nhóm có đặc quyền cao khác, cấp cho các thành viên đặc quyền `SeLoadDriverPrivilege`, quyền quản lý, tạo chia sẻ và xoá các máy in được kết nối với Domain Controller, cũng như khả năng đăng nhập cục bộ vào Domain Controller và tắt máy. 

Đầu tiên mình kiểm tra quyền bằng lệnh `whoami /priv`  và không thấy `SeLoadDriverPrivilege`, vì vậy mình cần vượt qua UAC.![[Screenshot 2026-08-13 at 19.03.33.png|center|border|650]]Kho lữu trữ UACMe có một danh sách toàn diện các cách vượt qua UAC, có thể được sử dụng từ dòng lệnh, ngoài ra từ giao diện đồ hoá chúng ta có thể mở một shell lệnh quản trị và nhập thông tin xác thực của tài khoản là thành viên nhóm Print Operators. Nếu chúng ta kiểm tra đặc quyền lại, SeLoadDriverPrivilege sẽ hiển thị nhưng bị tắt.
![[Screenshot 2026-08-13 at 19.13.23.png|center|border|650]]
Trình điều khiển `Capcom.sys` chứa chức năng cho phép bất kỳ người dùng nào thực thi shellcode với đặc quyền SYSTEM. Ta có thể sử dụng đặc quyền của mình để tải trình điều khiển này để leo thang đặc quyền. 
Mình sử dụng notepad để copy code từ [code]((https://raw.githubusercontent.com/3gstudent/Homework-of-C-Language/master/EnableSeLoadDriverPrivilege.cpp) và edit chúng để thực hiện bật quyền `SeLoadDriverPrivilege` ![[Screenshot 2026-08-13 at 19.19.43.png|center|border|650]]Tiếp theo mình sử dụng `cl.exe` để thực hiện compile file cpp sang file thực thi, hoặc có thể sử dụng build trên máy kali rồi thực hiện tải về windows:
![[Screenshot 2026-08-13 at 19.29.05.png|center|border|650]]Trước khi chạy file enable, ta cần thêm tham chiếu đến trình điều khiển, có thể tham khảo trong link trước như các lệnh có trong ảnh dưới đây:![[Screenshot 2026-08-13 at 19.32.30.png|center|border|650]]Mình thực hiện tải trình điều khiển `Capcom.sys` từ đây và lưu nó vào `C:\Temp`. Thực hiện các lệnh bên dưới đây để một tham chiếu đến trình điều khiển này trong cây `HKEY_CURRENT_USER` của chúng ta.![[Screenshot 2026-08-13 at 20.21.26.png|center|border|650]]Trong câu lệnh trên có thể thấy `\??\` sử dụng để tham chiếu đếm `ImagePath` của trình điều khiển độc hại của mình là một đường dẫn đối tượng `NT.` Win32 API sẽ phân tích cú pháp và giải quyết đường dẫn này để định vị và tải đúng trình điều khiển độc hại.
Tiếp theo mình thực hiện xác minh trình điều khiển được tải chưa bằng cách sử dụng `DriverView.exe` của Nirsoft:![[Screenshot 2026-08-13 at 20.34.03.png|center|border|650]]Sau đó mình thực hiện bật quyền `SeLoadDriverPrivilege` bằng tệp thực thi bên trên:![[Screenshot 2026-08-13 at 20.36.36.png|center|border|650]]Mình tiếp tục sử dụng công cụ exploitcapcom để thực hiện leo thang đặc quyền, ![[Screenshot 2026-08-13 at 20.38.24.png|center|border|650]]Thu thập flag để submit![[Screenshot 2026-08-13 at 20.42.19.png|center|border|650]]
#### Alternate Exploitation - No GUI
Chúng ta có thể sử dụng một công cụ như `EoPLoadDriver` để tự động hóa quá trình bật đặc quyền, tạo khóa registry và thực thi NTLoadDriver để tải trình điều khiển. Để làm điều này, chúng ta sẽ chạy lệnh sau:
Sau đó, chúng ta sẽ chạy `ExploitCapcom.exe` để mở shell SYSTEM hoặc chạy tệp thực thi tùy chỉnh của chúng ta:
#### Clean up
Chúng ta có thể che dấu dấu vết một chút bằng cách xóa khóa registry đã thêm trước đó.![[Screenshot 2026-08-13 at 20.55.32.png|center|border|650]]
>**Lưu ý:** Kể từ Windows 10 Phiên bản 1803, "SeLoadDriverPrivilege" không thể bị khai thác, vì không còn có thể bao gồm các tham chiếu đến các khóa registry dưới "HKEY_CURRENT_USER".
## Server Operators
Đối với nhóm Server Operators cho phép các thành viên quản trị máy chủ Windows mà không cần được gán đặc quyền quản trị viên domain. Đây là một nhóm có đặc quyền rất cao, có thể đăng nhập cục bộ vào các server, bao gồm cả Domain Controller. Thành viên nhóm này cấp các đặc quyền mạnh mẽ như `SeBackupPrivilege` và `SeRestorePrivilege` và khả năng kiểm soát các dịch vụ cục bộ
![[Screenshot 2026-08-13 at 21.15.31.png|center|border|650]]
Đầu tiên mình xem xét dịch vụ AppReadiness, mình có thể xác nhận rằng dịch vụ này khởi động với tư cách SYSTEM bằng tiện ích `sc.exe`
![[Screenshot 2026-08-13 at 21.16.59.png|center|border|650]]Tiếp theo ta kiểm tra quyền dịch vụ bằng `PSService` là một phần của bộ Sysinternals. `PSService` hoạt động tương tự như tiện ích `sc` và có thể hiển thị trạng thái và cấu hình dịch vụ đồng thời cho phép chúng ta khởi động, tạm dừng tiếp tục và khởi dộng lại các dịch vụ cả cục bộ và trên máy từ xa.![[Screenshot 2026-08-13 at 21.19.59.png|center|border|650]]Điều này xác nhận rằng nhóm Server Operators có quyền truy cập `SERVICE_ALL_ACCESS`, cấp cho chúng ta toàn quyền kiểm soát dịch vụ này.
Chúng ta tiếp tục kiểm tra tư cách thành viên nhóm quản trị viên cục bộ và xác minh rằng mục của chúng ta có trong danh sách không:![[Screenshot 2026-08-13 at 21.22.34.png|center|border|650]]Để leo thang đặc quyền mình tiến hành thay đổi đường dẫn nhị phân để thực thi một lệnh thêm người dùng hiện tại vào nhóm quản trị viên cục bộ mặc định.![[Screenshot 2026-08-13 at 21.26.31.png|center|border|650]]Mình kiểm tra lại với kết quả trên để xem tư cách thành viên của nhóm quản trị viên, chúng ta thấy rằng lệnh đã được thực thi thành công.
![[Screenshot 2026-08-13 at 21.29.01.png|center|border|650]]Từ đây, chúng ta có toàn quyền kiểm soát Bộ điều khiển Miền và có thể trích xuất tất cả thông tin xác thực từ cơ sở dữ liệu NTDS, truy cập các hệ thống khác và thực hiện các tác vụ khai thác sau xâm nhập.![[Screenshot 2026-08-13 at 21.32.12.png|center|border|650]]mình trích xuất giá trị băm mật khẩu NTLM từ Bộ điều khiển Miền![[Screenshot 2026-08-13 at 21.35.26.png|center|border|650]]Mình thu được flag:![[Screenshot 2026-08-13 at 21.43.15.png|center|border|650]]
> Lưu ý: Vì khi leo thành công mình vẫn ở session của shell cũ nên cần session mới để reload quyền nên sử dụng "runas /user:server_adm cmd" để chạy session thì các đặc quyền mới được áp dụng.
