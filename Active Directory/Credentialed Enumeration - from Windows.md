## TTPs
Công cụ đầu tiên chúng ta tìm hiểu là [Active Directory PowerShell Module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps). Khi có được quyền truy cập vào một Windows host trong domain, đặc biệt là một host được admin sử dụng, có khả năng chúng ta sẽ tìm thấy các công cụ và script có giá trị trên host đó.
## ActiveDirectory PowerShell Module
ActiveDirectory PowerShell module là tập hợp các PowerShell cmdlet dùng để quản trị môi trường AD từ command line. Trước khi có thể sử dụng module, chúng ta phải đảm bảo rằng module đã được import trước. Cmdlet `Get-Module`, thuộc [Microsoft.PowerShell.Core module](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/?view=powershell-7.2), sẽ liệt kê tất cả các module hiện có, phiên bản của chúng và các command có khả năng được sử dụng. Đây là một cách rất hữu ích để kiểm tra xem trên host có những thứ như Git hoặc các administrator script tùy chỉnh hay không. Nếu module chưa được load, hãy chạy `Import-Module ActiveDirectory` để load module và sử dụng nó.
#### Discover Modules
![[Pasted image 20260909083741.png]]
Chúng ta sẽ thấy rằng module `ActiveDirectory` hiện chưa được import. Hãy tiến hành import module này.
#### Load ActiveDirectory Module
![[Pasted image 20260909083939.png]]
Bây giờ các module đã được load, chúng ta có thể bắt đầu. Trước tiên, chúng ta sẽ enumerate một số thông tin cơ bản về domain bằng cmdlet [`Get-ADDomain`](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-addomain?view=windowsserver2022-ps)
### Get Domain Info
![[Pasted image 20260909084157.png]]
Lệnh này hiển thị những thông tin hữu ích như domain SID, domain functional level, các child domain nếu có và nhiều thông tin khác.
Tiếp theo, chúng ta sẽ sử dụng cmdlet `Get-ADUser`. Chúng ta sẽ filter các account có thuộc tính `ServicePrincipalName` được thiết lập. Điều này cung cấp cho chúng ta danh sách các account có khả năng dễ bị tấn công bằng Kerberoasting.
#### Get-ADUser
![[Pasted image 20260909085404.png]]Một kiểm tra thú vị khác mà chúng ta có thể thực hiện bằng Active Directory module là xác minh các domain trust relationships bằng cmdlet `Get-ADTrust`.
#### Checking For Trust Relationships
![[Pasted image 20260909085554.png]]Cmdlet này sẽ in ra tất cả trust relationship mà domain có. Chúng ta có thể xác định liệu đó là trust nằm trong forest hiện tại hay với các domain thuộc forest khác, loại trust, hướng của trust và tên domain mà relationship đó kết nối tới. Điều này sẽ hữu ích về sau khi tìm cách khai thác các child-to-parent trust relationship và thực hiện tấn công xuyên qua forest trust.
#### Group Enumeration
![[Pasted image 20260909090040.png]]Chúng ta có thể lấy kết quả này và đưa những tên group đáng chú ý trở lại cmdlet dể lấy thông tin chi hơn về một group cụ thể.
#### Detailed Group Info
![[Pasted image 20260909090203.png]]v
Bây giờ khi đã biết thêm thông tin về group, chúng ta hãy lấy danh sách thành viên bằng cmdlet `Get-ADGroupMember`.
### Group Membership
![[Pasted image 20260909090338.png]]Chúng ta có thể thấy một account, `backupagent` thuộc group này. Điều đáng lưu ý là nếu chúng ta có thể chiếm được service account này thông qua một cuộc tấn công nào đó, chúng ta có thể sử dụng member của nó trong group `Backup Operators` để chiếm quyền kiểm soát domain. Chúng ta có thể thực hiện quá trình này đối với các group khác để hiểu đầy đủ cách membership trong domain được thiết lập. 
## PowerView
[PowerView](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) là một công cụ được viết bằng PowerShell nhằm giúp chúng ta có được nhận thức về tình hình bên trong một môi trường AD. Tương tự như `BloodHound`, công cụ này cung cấp khả năng xác định nơi người dùng đang đăng nhập trong mạng, liệt kê thông tin domain như người dùng, máy tính, group, ACL, trust, tìm kiếm file share và password, thực hiện Kerberoasting, v.v. Đây là công cụ rất linh hoạt, có thể cung cấp cho chúng ta nhiều thông tin về tình trạng bảo mật của domain. Công cụ này yêu cầu nhiều thao tác thủ công hơn để xác định các misconfiguration và mối quan hệ bên trong domain so với `BloodHound` nhưng khi được đúng cách, nó có thể giúp chúng ta xác định những misconfiguration tinh vi.
Bảng bên dưới mô tả một số function hữu ích nhất mà PowerView cung cấp.

| Command             | Description                                                                    |
| ------------------- | ------------------------------------------------------------------------------ |
| Export-PowerViewCSV | Ghi thêm kết quả vào một file CSV                                              |
| ConvertTo-SID       | Chuyển đổi tên User hoặc Group thành giá trị SID                               |
| Get-DomainSPNTicket | Yêu cầu Kerberos ticket cho một tài khoản Service Principal Name được chỉ định |
**Domain/LDAP Functions:**

| Command                   | Description                                                                                               |
| ------------------------- | --------------------------------------------------------------------------------------------------------- |
| Get-Domain                | Trả về AD object của domain hiện tại hoặc domain được chỉ định                                            |
| Get-DomainController      | Trả về danh sách Domain Controller của domain được chỉ định                                               |
| Get-DomainUser            | Trả về tất cả user hoặc các user object cụ thể trong AD                                                   |
| Get-DomainComputer        | Trả về tất cả computer hoặc các computer object cụ thể trong AD                                           |
| Get-DomainGroup           | Trả về tất cả group hoặc các group object cụ thể trong AD                                                 |
| Get-DomainOU              | Tìm kiếm tất cả hoặc các OU object cụ thể trong AD                                                        |
| Find-InterestingDomainAcl | Tìm các object ACL trong domain có quyền modification được thiết lập trong các object không phải built-in |
| Get-DomainGroupMember     | Trả về các member của một domain group cụ thể                                                             |
| Get-DomainFileServer      | Trả về danh sách các server khả năng đang hoạt động như file server                                       |
| Get-DomainDFSShare        | Trả về danh sách tất cả distributed file system trong domain hiện tại (hoặc domain được chỉ định)         |
**GPO Functions:**

| Command          | Description                                                                    |
| ---------------- | ------------------------------------------------------------------------------ |
| Get-DomainGPO    | Trả về tất cả GPO hoặc các GPO object cụ thể trong AD                          |
| Get-DomainPolicy | Trả về default domain policy hoặc domain controller policy của domain hiện tại |
**Computer Enumeration Functions:**

| Command                 | Description                                                                                  |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| Get-NetLocalGroup       | Liệt kê các local group trên máy cục bộ hoặc máy từ xa                                       |
| Get-NetLocalGroupMember | Liệt kê các member của một local group cụ thể                                                |
| Get-NetShare            | Trả về các share đang mở trên máy cục bộ (hoặc máy từ xa)                                    |
| Get-NetSession          | Trả về thông tin session trên máy cục bộ (hoặc máy từ xa)                                    |
| Test-AdminAccess        | Kiểm tra xem user hiện tại có quyền administrator trên máy cục bộ (hoặc máy từ xa) hay không |
**Threaded 'Meta'-Functions:**

| Command                         | Description                                                                            |
| ------------------------------- | -------------------------------------------------------------------------------------- |
| Find-DomainUserLocation         | Tìm các máy mà user cụ thể đang đăng nhập                                              |
| Find-DomainShare                | Tìm các share có thể truy cập trên các máy trong domain                                |
| Find-InterestingDomainShareFile | Tìm kiếm các file phù hợp với tiêu chí cụ thể trên các share có quyền đọc trong domain |
| Find-LocalAdminAccess           | Tìm các máy trong local domain mà user hiện tại có quyền local administrator           |
**Domain Trust Functions:**

| Command                      | Description                                                                           |
| ---------------------------- | ------------------------------------------------------------------------------------- |
| Get-DomainTrust              | Trả về các domain trust của domain hiện tại hoặc domain được chỉ định                 |
| Get-ForestTrust              | Trả về tất cả forest trust của forest hiện tại hoặc forest được chỉ định              |
| Get-DomainForeignUser        | Liệt kê những user nằm trong các group bên ngoài domain của user đó                   |
| Get-DomainForeignGroupMember | Liệt kê các group có user bên ngoài domain của group đó và trả về từng foreign member |
| Get-DomainTrustMapping       | Liệt kê tất cả trust của domain hiện tại và bất kỳ trust nào khác được phát hiện.     |
#### Domain User Information
![[Pasted image 20260909092520.png]]
Chúng ta đã thấy một số thông tin cơ bản về user bằng PowerView. Bây giờ liệt kê một số thông tin về domain group. Chúng ta có thể sử dụng function `Get-DomainGroupMember` để lấy thông tin cụ thể về group. Việc thêm switch `-Recurse` cho PowerView biết rằng nếu nó tìm thấy bất kỳ group nào là một phần của group mục tiêu.
#### Recursive Group Membership
![[Pasted image 20260909092954.png]]
Kết quả trên chúng ta thực hiện việc kiểm tra đệ quy group Domain Admins để liệt kê các member của nó. Bây giờ chúng ta biết những ai cần nhắm tới để tới để có khả năng nâng cao đặc quyền. Tương tự như với AD PowerShell module, chúng ta cũng có thể liệt kê trust mapping của domain.
#### Trust Enumeration
![[Pasted image 20260909093156.png]]
Chúng ta có thể sử dụng function [Test-AdminAccess](https://powersploit.readthedocs.io/en/latest/Recon/Test-AdminAccess/) để kiểm tra quyền local admin trên máy hiện tại hoặc một máy từ xa.
#### Testing for Local Admin Access
![[Pasted image 20260909093258.png]]
Ở trên, chúng ta xác định rằng user hiện tại đang sử dụng là administrator trên host `ACADEMY-EA-MS01`. Chúng ta có thể thực hiện function tương tự với từng host để xem chúng ta có quyền administrative ở đâu. Sau này chúng ta sẽ thấy BloodHound thực hiện loại kiểm tra này hiệu quả như thế nào. Bây giờ chúng ta có thể kiểm tra những user có thuộc tính SPN được thiết lập, điều này cho biết tài khoản có thể bị thực hiện một cuộc tấn công Kerberoasting.
#### Finding Users With SPN Set
![[Pasted image 20260909093351.png]]
## SharpView
PowerView là một phần của bộ công cụ PowerShell offensive PowerSploit hiện tại không còn được phát triển. Công cụ này đang được BC-Security cập nhật như một phần của framework [Empire 4](https://github.com/BC-SECURITY/Empire/blob/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1). 
Một công cụ khác đáng để thử nghiệm là SharpView, một phiên bản .NET của PowerView. Nhiều function tương tự được PowerView hỗ trợ cũng có thể được sử dụng với SharpView. Chúng ta có thể nhập tên method cùng với `-Help` để lấy danh sách các argument.
```powershell
PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend

[Get-DomainSearcher] search base: LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainUser] filter string: (&(samAccountType=805306368)(|(samAccountName=forend)))
objectsid                      : {S-1-5-21-3842939050-3880317879-2865463114-5614}
samaccounttype                 : USER_OBJECT
objectguid                     : 53264142-082a-4cb8-8714-8158b4974f3b
useraccountcontrol             : NORMAL_ACCOUNT
accountexpires                 : 12/31/1600 4:00:00 PM
lastlogon                      : 4/18/2022 1:01:21 PM
lastlogontimestamp             : 4/9/2022 1:33:21 PM
pwdlastset                     : 2/28/2022 12:03:45 PM
lastlogoff                     : 12/31/1600 4:00:00 PM
badPasswordTime                : 4/5/2022 7:09:07 AM
name                           : forend
distinguishedname              : CN=forend,OU=IT Admins,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
whencreated                    : 2/28/2022 8:03:45 PM
whenchanged                    : 4/9/2022 8:33:21 PM
samaccountname                 : forend
memberof                       : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar Read,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Printer Access,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share H Drive,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share G Drive,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL}
cn                             : {forend}
objectclass                    : {top, person, organizationalPerson, user}
badpwdcount                    : 0
countrycode                    : 0
usnchanged                     : 3259288
logoncount                     : 26618
primarygroupid                 : 513
objectcategory                 : CN=Person,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
dscorepropagationdata          : {3/24/2022 3:58:07 PM, 3/24/2022 3:57:44 PM, 3/24/2022 3:52:58 PM, 3/24/2022 3:49:31 PM, 7/14/1601 10:36:49 PM}
usncreated                     : 3054181
instancetype                   : 4
codepage                       : 0
```
## Shares
Shares cho phép người dùng trong một domain nhanh chóng truy cập thông tin liên quan đến vai trò hằng ngày của họ và chia sẻ nội dung với tổ chức. Khi được thiết lập đúng cách, domain share sẽ yêu cầu user phải domain joined và phải authenticate khi truy cập hệ thống. Permissions cũng sẽ được thiết lập để đảm bảo user chỉ có thể truy cập và nhìn thấy những gì cần thiết cho vai trò hằng ngày của họ. Các share được cấp quyền quá rộng có khả năng gây ra việc vô tình làm lộ thông tin nhạy cảm, đặc biệt là những share chứa dữ liệu y tế, pháp lý, nhân sự, HR, v.v. Trong một cuộc tấn công, việc kiểm soát được một standard domain user có quyền truy cập vào các share như IT/infrastructure share có thể dẫn đến việc làm lộ dữ liệu nhạy cảm như configuration file hoặc authentication file, chẳng hạn như SSH key hoặc password được lưu trữ không an toàn. Chúng ta muốn xác định những vấn đề như vậy để đảm bảo khách hàng không làm lộ dữ liệu cho những user không cần quyền truy cập trong công việc hằng ngày và đảm bảo họ đáp ứng các yêu cầu pháp lý/quy định mà họ phải tuân thủ (HIPAA, PCI, v.v.).
Chúng ta có thể sử dụng PowerView để tìm kiếm các share và sau đó hỗ trợ chúng ta đào sâu vào bên trong hoặc sử dụng nhiều command thủ công khác nhau để tìm kiếm các chuỗi phổ biến, chẳng hạn như các file có `pass` trong tên. Đây có thể là một quá trình tốn nhiều thời gian và chúng ta có thể bỏ sót một số thứ, đặc biệt trong các môi trường lớn. Bây giờ, hãy dành thời gian tìm hiểu công cụ `Snaffler` và xem nó có thể hỗ trợ chúng ta xác định những vấn đề này một cách chính xác và hiệu quả hơn như thế nào.
## Snaffler
[Snaffler](https://github.com/SnaffCon/Snaffler) là một công cụ có thể giúp chúng ta thu thập thông tin xác thực hoặc dữ liệu nhạy cảm khác trong môi trường Active Directory. Snaffler hoạt động bằng cách lấy danh sách các máy chủ trong domain, sau đó enumerate các máy chủ đó để tìm các share và thư mục có thể đọc được. Sau khi hoàn tất, nó sẽ duyệt qua tất cả các thư mục mà user của chúng ta có quyền đọc và tìm kiếm file có thể giúp cải thiện vị thế của chúng ta trong quá trình assessment. Snaffler yêu cầu phải được chạy từ một host đã join domain hoặc trong context của một domain user.
![[Pasted image 20260909101616.png]]Chúng ta có thể tìm thấy password, SSH key, configuration file hoặc các dữ liệu khác có thể được sử dụng để tiếp tục mở rộng quyền truy cập của mình. Snaffler sẽ mã hóa màu cho output và cung cấp cho chúng ta thông tin tổng quan về các loại file được tìm thấy trong các share.
Bây giờ chúng ta đã có một lượng lớn dữ liệu về domain `INLANEFREIGHT.LOCAL` (và hy vọng là có các ghi chú rõ ràng cùng output từ các logfile!), chúng ta cần một phương pháp để tương quan các dữ liệu này và trực quan hóa chúng. Hãy đi sâu hơn vào `BloodHound` và xem công cụ này có thể mạnh đến mức nào trong bất kỳ security assessment tập trung vào AD nào.
### BloodHound
BloodHound là một công cụ mã nguồn mở đặc biệt hữu ích, có khả năng xác định các attack path trong môi trường AD bằng cách phân tích mối quan hệ giữa các object. Đầu tiên, chúng ta phải xác thực với tư cách là một domain user từ một Windows attack host nằm trong network hoặc chuyển tool vào một domain-joined host. Có nhiều cách để thực hiện việc này. 
Chúng ta sẽ bắt đầu bằng cách chạy collector `SharpHound.exe` từ attack host MS01.
![[Pasted image 20260909102217.png]]
Tiếp theo, chúng ta có thể exfiltrate dataset về VM của mình hoặc ingest nó vào BloodHound GUI trên MS01. Chúng ta có thể thực hiện việc này trên MS01 bằng cách nhập `bloodhound` vào CMD hoặc PowerShell console. Credentials sẽ được lưu sẵn, nhưng nếu xuất hiện credential prompt, hãy nhập `neo4j: HTB_@cademy_stdnt!`. Tiếp theo, nhấp vào nút `Upload Data` ở phía bên phải, chọn file zip vừa được tạo và nhấp `Open`. Một cửa sổ `Upload Progress` sẽ xuất hiện. Khi tất cả các file `.json` hiển thị 100% hoàn tất, nhấp vào X ở phía trên cửa sổ đó.
Chúng ta có thể bắt đầu bằng cách nhập `domain:` vào search bar ở phía trên bên trái và chọn `INLANEFREIGHT.LOCAL` từ kết quả. Hãy dành một chút thời gian để xem qua node info tab. Như chúng ta có thể thấy, đây sẽ là một công ty khá lớn với hơn 550 host để target và có trust với hai domain khác.
Bây giờ, hãy xem một vài query được xây dựng sẵn trong tab `Analysis`. Query `Find Computers with Unsupported Operating Systems` rất hữu ích để tìm các hệ điều hành đã lỗi thời và không còn được hỗ trợ đang chạy legacy software. Những hệ thống này tương đối phổ biến trong enterprise network (đặc biệt là các môi trường cũ), vì chúng thường chạy một product không thể được update hoặc thay thế tại thời điểm đó. Việc giữ lại những host này có thể giúp tiết kiệm chi phí, nhưng chúng cũng có thể tạo ra các vulnerability không cần thiết cho network. Các host cũ có thể dễ bị ảnh hưởng bởi những vulnerability remote code execution cũ như [MS08-067](https://support.microsoft.com/en-us/topic/ms08-067-vulnerability-in-server-service-could-allow-remote-code-execution-ac7878fc-be69-7143-472d-2507a179cd15). Nếu gặp các host cũ này trong quá trình assessment, chúng ta nên thận trọng trước khi tấn công chúng (hoặc thậm chí kiểm tra với client), vì chúng có thể dễ hỏng và đang chạy một critical application hoặc service. Chúng ta có thể khuyến nghị client segment các host này khỏi phần còn lại của network nhiều nhất có thể nếu họ chưa thể loại bỏ chúng, nhưng đồng thời cũng nên khuyến nghị họ bắt đầu xây dựng kế hoạch decommission và thay thế chúng.
Query này cho thấy hai host, một host chạy Windows 7 và một host chạy Windows Server 2008 (cả hai đều không "live" trong lab của chúng ta). Đôi khi chúng ta sẽ thấy các host không còn được bật nhưng vẫn xuất hiện dưới dạng record trong AD. Chúng ta luôn phải xác minh xem chúng có còn "live" hay không trước khi đưa ra recommendation trong report. Chúng ta có thể ghi nhận một high-risk finding về `Legacy Operating Systems` hoặc một best practice recommendation về việc dọn dẹp các record cũ trong AD.
#### Unsupported Operating Systems
![[Pasted image 20260909103600.png]]