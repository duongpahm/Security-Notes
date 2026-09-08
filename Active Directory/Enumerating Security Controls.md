Sau khi giành được foothold, chúng ta có thể tận dụng quyền truy cập này để đánh giá tình trạng phòng thủ của các host, tiếp tục enumerate domain vì lúc này khả năng quan sát của chúng ta không còn bị hạn chế như trước, và nếu cần thiết, thực hiện theo phương pháp "living off the land" bằng cách sử dụng các công cụ vốn đã có sẵn trên host.
Một số tổ chức áp dụng các biện pháp bảo vệ nghiêm ngặt hơn những tổ chức khác, và đôi khi các security controls cũng không được áp dụng đồng đều trên toàn bộ môi trường. Có thể có những policy được áp dụng trên một số máy nhất định khiến việc enumeration trở nên khó khăn hơn, trong khi các policy đó lại không được áp dụng trên những máy khác.
## Windows Defender
Windows Defender, hay còn gọi là Microsoft Defender sau Windows 10 May 2020 Update đã được cải thiện đáng kể qua nhiều năm và theo mặc định có thể block những công cụ như `PowerView`. Có những phương pháp để bypass các cơ chế bảo vệ này. Chúng ta có thể sử dụng PowerShell cmdlet tích hợp sẵn `Get-MpComputerStatus` để kiểm tra trạng thái hiện tại của Defender. Trong ví dụ dưới đây, tham số `RealTimeProtectionEnabled` được đặt thành `True`, nghĩa là Defender đang được enable trên hệ thống.
```powershell
PS C:\htb> Get-MpComputerStatus

AMEngineVersion                 : 1.1.17400.5
AMProductVersion                : 4.10.14393.0
AMServiceEnabled                : True
AMServiceVersion                : 4.10.14393.0
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 9/2/2020 11:31:50 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
AntivirusSignatureAge           : 1
AntivirusSignatureLastUpdated   : 9/2/2020 11:31:51 AM
AntivirusSignatureVersion       : 1.323.392.0
BehaviorMonitorEnabled          : False
ComputerID                      : 07D23A51-F83F-4651-B9ED-110FF2B83A9C
ComputerState                   : 0
FullScanAge                     : 4294967295
FullScanEndTime                 :
FullScanStartTime               :
IoavProtectionEnabled           : False
LastFullScanSource              : 0
LastQuickScanSource             : 2
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
NISSignatureAge                 : 4294967295
NISSignatureLastUpdated         :
NISSignatureVersion             : 0.0.0.0
OnAccessProtectionEnabled       : False
QuickScanAge                    : 0
QuickScanEndTime                : 9/3/2020 12:50:45 AM
QuickScanStartTime              : 9/3/2020 12:49:49 AM
RealTimeProtectionEnabled       : True
RealTimeScanDirection           : 0
PSComputerName                  :
```
### AppLocker

Application whitelist là danh sách các software application hoặc executable được phép tồn tại và chạy trên một hệ thống. Mục tiêu là bảo vệ môi trường khỏi malware nguy hiểm và các software không được phê duyệt, không phù hợp với nhu cầu kinh doanh cụ thể của tổ chức.
AppLocker là giải pháp application whitelisting của Microsoft, cho phép system administrator kiểm soát những application và file mà user có thể chạy. AppLocker cung cấp quyền kiểm soát chi tiết đối với executable, script, Windows installer file, DLL, package app và packed app installed.
Các tổ chức thường block `cmd.exe`, `PowerShell.exe` và quyền ghi vào một số directory. Tuy nhiên tất cả những cơ chế này đều có thể bị bypass. Các tổ chức cũng thường tập trung block executable `PowerShell.exe` nhưng lại bỏ qua những vị trí khác của PowerShell executabl, chẳng hạn:
```cmd
%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
```
hoặc:
```cmd
PowerShell_ISE.exe
```
Trong ví dụ này, các rule của `AppLocker` cho thấy tất cả Domain Users bị cấm chạy PowerShell 64 bit tại:
```cmd
%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe
```
Do đó, chúng ta có thể gọi PowerShell từ những location khác. Đôi khi chúng ta sẽ gặp các `AppLocker` policy nghiêm ngặt hơn và cần có những phương pháp sáng tạo hơn để bypass. 
#### Using Get-AppLockerPolicy cmdlet
```powershell
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

PathConditions      : {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 3d57af4a-6cf8-4e5b-acfc-c2c2956061fa
Name                : Block PowerShell
Description         : Blocks Domain Users from using PowerShell on workstations
UserOrGroupSid      : S-1-5-21-2974783224-3764228556-2640795941-513
Action              : Deny

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 921cc481-6e17-4653-8f75-050b80acca20
Name                : (Default Rule) All files located in the Program Files folder
Description         : Allows members of the Everyone group to run applications that are located in the Program Files folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : a61c8b2c-a319-4cd0-9690-d2177cad7b51
Name                : (Default Rule) All files located in the Windows folder
Description         : Allows members of the Everyone group to run applications that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : fd686d83-a829-4351-8ff4-27c7de5755d2
Name                : (Default Rule) All files
Description         : Allows members of the local Administrators group to run all applications.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow
```

### PowerShell Constrained Language Mode
PowerShell Constrained Language Mode giới hạn nhiều tính năng cần thiết để sử dụng PowerShell một cách đầy đủ, chẳng hạn như block COM objects, chỉ cho phép sử dụng các `.NET` type được approve, XAML based workflow, PowerShell classes và nhiều tính năng khác.
Chúng ta có thể nhanh chóng kiểm tra xem PowerShell hiện đang ở **Full Language Mode** hay **Constrained Language Mode**.
```powershell
PS C:\htb> $ExecutionContext.SessionState.LanguageMode 

ConstrainedLanguage
```
## LAPS
Microsoft Local Administrator Password Solution (LAPS) được sử dụng để randomize và rotate local administrator password trên các Windows host nhằm ngăn chặn lateral movement. Chúng ta có thể enumerate những domain user nào có quyền đọc LAPS password được thiết lập cho các máy đã cài LAPS, đồng thời xác định những máy nào chưa cài LAPS.
**LAPSToolkit** giúp thực hiện việc này dễ dàng hơn thông qua một số function. Một trong số đó là phân tích `ExtendedRights` của tất cả computer đã enable LAPS.
Điều này cho chúng ta biết những group nào được delegate quyền đọc LAPS password, thường là các user thuộc những protected group. 
Một account đã join computer vào domain sẽ nhận `All Extended Rights` trên host đó. Quyền này cho phép account đọc password. Việc enumeration có thể cho thấy một user account có khả năng đọc LAPS password trên một host. Điều này giúp chúng ta xác định những AD user cụ thể có quyền đọc LAPS password.
#### Using Find-LAPSDelegatedGroups
```powershell
PS C:\htb> Find-LAPSDelegatedGroups

OrgUnit                                             Delegated Groups
-------                                             ----------------
OU=Servers,DC=INLANEFREIGHT,DC=LOCAL                INLANEFREIGHT\Domain Admins
OU=Servers,DC=INLANEFREIGHT,DC=LOCAL                INLANEFREIGHT\LAPS Admins
OU=Workstations,DC=INLANEFREIGHT,DC=LOCAL           INLANEFREIGHT\Domain Admins
OU=Workstations,DC=INLANEFREIGHT,DC=LOCAL           INLANEFREIGHT\LAPS Admins
OU=Web Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\Domain Admins
OU=Web Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\LAPS Admins
OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\Domain Admins
OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\LAPS Admins
OU=File Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\Domain Admins
OU=File Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\LAPS Admins
OU=Contractor Laptops,OU=Workstations,DC=INLANEF... INLANEFREIGHT\Domain Admins
OU=Contractor Laptops,OU=Workstations,DC=INLANEF... INLANEFREIGHT\LAPS Admins
OU=Staff Workstations,OU=Workstations,DC=INLANEF... INLANEFREIGHT\Domain Admins
OU=Staff Workstations,OU=Workstations,DC=INLANEF... INLANEFREIGHT\LAPS Admins
OU=Executive Workstations,OU=Workstations,DC=INL... INLANEFREIGHT\Domain Admins
OU=Executive Workstations,OU=Workstations,DC=INL... INLANEFREIGHT\LAPS Admins
OU=Mail Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\Domain Admins
OU=Mail Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\LAPS Admins
```
`Find-AdmPwdExtendedRights` kiểm tra quyền trên từng computer đã enable LAPS để tìm các group có quyền đọc và những user có `"All Extended Rights"`.
Các user có `"All Extended Rights"` có thể đọc LAPS password và có thể được bảo vệ kém hơn so với những user nằm trong các delegated group, vì vậy đây là một quyền đáng kiểm tra.
#### Using Find-AdmPwdExtendedRights
```powershell
PS C:\htb> Get-LAPSComputers

ComputerName                Password       Expiration
------------                --------       ----------
DC01.INLANEFREIGHT.LOCAL    6DZ[+A/[]19d$F 08/26/2020 23:29:45
EXCHG01.INLANEFREIGHT.LOCAL oj+2A+[hHMMtj, 09/26/2020 00:51:30
SQL01.INLANEFREIGHT.LOCAL   9G#f;p41dcAe,s 09/26/2020 00:30:09
WS01.INLANEFREIGHT.LOCAL    TCaG-F)3No;l8C 09/26/2020 00:46:04
```