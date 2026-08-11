## The Windows Operating System
Microsoft chính thức giới thiệu hệ điều hành Windows vào ngày 20 tháng 11 năm 1985. Phiên bản đầu tiên của Windows là một lớp giao diện đồ hoạ hoạt động trên MS-DOS. Những phiên bản Windows Desktop sau đó lần lượt giới thiệu các chương trình như Windows File Manager, Program Manager và Print Manager.
Windows 95 là phiên bản đầu tiên tích hợp hoàn toàn giữ Windows và DOS, đồng thời lần đầu tiên cung cấp khả năng hỗ trợ Internet được tích hợp sẵn. Phiên bản này cũng đánh dấu sự ra mắt của trình duyệt web Internet Explorer.
Kể từ phiên bản đầu tiên, Microsoft đã phát hành hơn một chục phiên bản Windows, bao gồm Windows XP, Windows Vista, Windows 8 và cho đến phiên bản hiện nay là Windows 11. Theo thời gian, Microsoft cũng cung cấp nhiều phiên bản (Edition) khác nhau cho mỗi bản phát hành Windows Desktop nhằm đáp ứng nhu cầu của nhiều đối tượng người dùng, từ người dùng cá nhân đến khách hàng doanh nghiệp.
Windows Server được phát hành lần đầu vào năm 1993 cùng với Windows NT 3.1 Advanced Server. Qua nhiều lần nâng cấp, Windows NT đã bổ sung các công nghệ như Internet Information Services (IIS), nhiều giao thức mạng khác nhau, cùng các Administrative Wizards nhằm hỗ trợ quản trị viên thực hiện các tác vụ quản trị dễ dàng hơn. Với sự ra đời của Windows 2000, Microsoft giới thiệu Active Directory, ban đầu được thiết kế để hỗ trợ quản trị viên triển khai chia sẻ tệp, mã hóa dữ liệu, VPN và nhiều dịch vụ doanh nghiệp khác. Windows Server 2000 cũng tích hợp Microsoft Management Console (MMC) và hỗ trợ Dynamic Disk Volumes.
Tiếp theo là Windows Server 2003, bổ sung các khái niệm Server Roles, tường lửa tích hợp (Built-in Firewall), Volume Shadow Copy Service (VSS) và nhiều tính năng khác. Windows Server 2008 giới thiệu Failover Clustering, nền tảng ảo hóa Hyper-V, Server Core, Event Viewer và nhiều cải tiến đáng kể đối với Active Directory. Trong các năm tiếp theo, Microsoft tiếp tục phát hành Windows Server 2012, Windows Server 2016 và gần đây là Windows Server 2019. Phiên bản Server 2019 bổ sung khả năng hỗ trợ Kubernetes, Linux Containers cùng nhiều cơ chế bảo mật tiên tiến hơn.
Khi các phiên bản Windows mới được phát hành, những phiên bản cũ sẽ dần bị ngừng hỗ trợ và không còn nhận được các bản cập nhật từ Microsoft (trừ một số trường hợp tổ chức mua gói hỗ trợ dài hạn). Windows Server 2008 và Windows Server 2012 đã chính thức kết thúc vòng đời hỗ trợ bảo mật vào ngày 14 tháng 01 năm 2020. Hiện nay, chỉ Windows Server 2012 R2 và các phiên bản mới hơn vẫn còn được Microsoft hỗ trợ. Tuy nhiên, trong những năm gần đây Microsoft vẫn từng phát hành các bản vá ngoài kế hoạch (Out-of-Band Patch) cho các phiên bản Windows cũ nhằm khắc phục lỗ hổng nghiêm trọng SMBv1 (EternalBlue).
## Windows Versions
Danh sách dưới đây thể hiện các phiên bản Windows chính cùng số hiệu phiên bản tương ứng:

| Operating System Names               | Version Number |
|:------------------------------------ | -------------- |
| Windows NT 4                         | 4.0            |
| Windows 2000                         | 5.0            |
| Windows XP                           | 5.1            |
| Windows Server 2003, 2003 R2         | 5.2            |
| Windows Vista, Server 2008           | 6.0            |
| Windows 7, Server 2008 R2            | 6.1            |
| Windows 8, Server 2012               | 6.2            |
| Windows 8.1, Server 2012 R2          | 6.3            |
| Windows 10, Server 2016, Server 2019 | 10.0           |
Chúng ta có thể sử dụng cmdlet Get-WmiObject để thu thập thông tin về hệ điều hành. Cmdlet này được dùng để lấy các đối tượng thuộc lớp Windows Management Instrumentation (WMI) hoặc truy vấn thông tin về các lớp VMI có sẵn. Có nhiều cách khác nhau để xác định phiên bản và số hiệu bản dụng (Build Number) của hệ thống.  Một trong những cách đơn giản nhất là sử dụng lớp `Win32_OperatingSystem`, ví dụ dưới đây cho thấy hệ thống đang chạy Windows 10 với Build Number 19041.
```powershell
PS C:\htb> Get-WmiObject -Class win32_OperatingSystem | select Version,BuildNumber

Version    BuildNumber
-------    -----------
10.0.19041 19041
```

Một số lớp VMI hữu ích khác có thể sử dụng cùng `Get-WmiObject` bao gồm `Win32_Process` để liệt kê các tiến trình đang chạy, `Win32_Service` để liệt kê các dịch vụ của hệ thông và `Win32_BIOS` để truy xuất thông tin về BIOS (Basic Input/Output System). BIOS là firmware được cài đặt trên bo mạch chủ, chịu trách nhiệm điều khiển các chức năng cơ bản của máy tính như quản lý nguồn điện, các giao diện vào/ ra và cấu hình hệ thống. Ngoài ra, tham số `ComputerName` có thể được sử dụng để truy vấn thông tin từ các máy tính từ xa. `Get-WmiObject` cũng hỗ trợ khởi động hoặc dùng các dịch vụ trên máy cục bộ và máy từ xa cùng nhiều tác vụ quản trị khác.
## Accessing Windows
### Local Access Concepts
Truy cập cục bộ là phương thức phổ biến nhất để sử dụng bất kỳ máy tính nào, bao gồm cả các hệ thống chạy Windows. Việc nhập liệu thường được thực hiện thông qua bàn phím, chuột hoặc Trackpad, trong khi kết quả được hiển thị trên màn hình.
Trong các tổ chức có môi trường làm việc văn phòng, các chính sách và biện pháp kiểm soát an ninh thường được xây dựng dựa trên giả định rằng nhân viên làm việc trực tiếp tại nơi làm việc bằng các máy tính do tổ chức sở hữu. Tuy nhiên, cùng với sự phát triển của mô hình làm việc từ xa (Remote Work), ngày càng nhiều tổ chức mở rộng khả năng hỗ trợ nhân viên truy cập hệ thống từ xa. Đối với các chuyên gia trong lĩnh vực CNTT, phát triển phần mềm và an ninh mạng, việc đồng thời truy cập nhiều máy tính theo cả hình thức cục bộ và từ xa đã trở thành một hoạt động thường xuyên. Do đó, trước khi tìm hiểu các kỹ thuật quản trị từ xa, chúng ta cần nắm rõ khái niệm Remote Access.
### Remote Access Concepts
Remote Access là khả năng truy cập và điều khiển một máy tính thông qua kết nói mạng. Để có thể truy cập từ xa đến một máy tính khác, trước tiên người dùng phải có quyền truy cập cục bộ vào thiết bị của chính mình. Hiện nay có rất nhiều phương pháp truy cập từ xa khác nhau. 
Sự phát triển của công nghệ mạng và Internet đã tạo ra nhiều lĩnh vực kinh doanh phụ thuộc hoàn toàn vào khả năng quản trị hệ thống từ xa. Có thể kể đến **Managed Service Providers (MSPs)** và **Managed Security Service Providers (MSSPs)**, hai mô hình dịch vụ hoạt động chủ yếu dựa trên việc quản lý hệ thống CNTT của khách hàng từ xa. Khả năng này cho phép họ tập trung hóa công tác quản trị, chuẩn hóa công nghệ được sử dụng, tự động hóa nhiều tác vụ, hỗ trợ mô hình làm việc từ xa cũng như phản ứng nhanh trước các sự cố hoặc mối đe dọa an ninh.
Không chỉ MSP hay MSSP, hầu hết các tổ chức có đội ngũ CNTT, phát triển phần mềm hoặc an ninh mạng đều sử dụng các phương thức truy cập từ xa hằng ngày để phát triển ứng dụng, quản trị máy chủ và hỗ trợ người dùng cuối. Một số công nghệ truy cập từ xa phổ biến bao gồm Virtual Private Network (VPN), Secure Shell (SSH), File Transfer Protocol (FTP), Virtual Network Computing (VNC), Windows Remote Management (WinRM hay PowerShell Remoting) và Remote Desktop Protocol (RDP). Trong học phần này, chúng ta sẽ tập trung chủ yếu vào việc sử dụng RDP (Remote Desktop Protocol).
Chúng ta có thể sử dụng **RDP (Remote Desktop Protocol)** để kết nối tới một máy đích chạy Windows từ máy tấn công sử dụng hệ điều hành Linux hoặc Windows. Nếu kết nối từ một máy Windows đến một máy đích Windows, chúng ta có thể sử dụng ứng dụng khách RDP được tích hợp sẵn có tên **Remote Desktop Connection (mstsc.exe)**. 
## Operating System Structure
Trong hệ điều hành Windows, Root Directory có dạng `<ký_tự_ổ_đĩa>:\`, phổ biến nhất là ổ `C:\`. Thư mục gốc (còn gọi là Boot Partition) là nơi hệ điều hành được cài đặt. Các ổ đĩa vật lý hoặc ổ đĩa ảo khác sẽ được gán ký tự riêng, ví dụ như Data (`E:\`). Cấu trúc Boot Partition được mô tả như sau:

| Thư mục                    | Chức năng                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Perflogs                   | Có thể chứa các nhật ký hiệu năng (Logs) của Windows, tuy nhiên theo mặc định thư mục này sẽ trống.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Program Files              | Trên hệ thống 32-bit, đây là nơi cài đặt tất cả các chương trình 16-bit và 32-bit. Trên hệ thống 64-bit, thư mục này chỉ chứ các chương trình 64-bit.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Program Files (x86)        | Trên các phiên bản Windows 64-bit, đây là nơi cài đặt các chương trình 32-bit và 16-bit nhằm đảm bảo khả năng tương thích ngược                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ProgramData                | Đây là một thư mục ẩn (Hidden Folder) chứa dữ liệu cần thiết để một số chương trình cài đặt có thể hoạt động. Dữ liệu trong thư mục này có thể được chương trình truy cập bất kể người dùng nào đang đăng nhập vào hệ thống.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Users                      | Chứa User Profile của tất cả người dùng từng đăng nhập vào hệ thống. Bên trong bao gồm hai thư mục quan trọng là Public và Default.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Public                     | Thư mục này được thiết kế để người dùng trên máy tính có thể chia sẻ tệp với nhau và mặc định mọi người dùng đều có quyền truy cập. Theo mặc định, thư mục này cũng được chia sẻ qua mạng (Network Share), tuy nhiên người dùng vẫn phải có tài khoản mạng hợp lệ để truy cập.                                                                                                                                                                                                                                                                                                                                                                                                              |
| AppData                    | Chứa dữ liệu và thiết lập của ứng dụng theo từng người (Per-user Application Data and Settings) trong một thư mục ẩn, ví dụ: `C:\Users\cliff.moore\AppData`. Bên trong gồm ba thư mục con là Roaming, Local, và LocalLow. Thư mục Roaming lưu trữ dư liệu không phụ thuộc vào máy tính, có thể đồng bộ theo hồ sơ người dùng giữa nhiều thiết bị, chẳng hạn như Custom Dictionaries. Thư mục Local chứa dữ liệu chỉ áp dụng cho máy tính hiện đại và sẽ không đồng bộ được qua mạng. LocalLow tương tự Local nhưng có mức toàn vẹn dữ liệu thấp hơn vì thường được sử dụng bởi các ứng chạy trong môi trường hạn chế, chẳng hạn như trình duyệt web ở chế độ Protected Mode hoặc Safe Mode. |
| Windows                    | Chứa phàn lớn các tệp cần thiết để hiệu điều hành Windows hoạt động.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| System, System32, SysWOW64 | Chứ các DLL cần thiết cho các chức năng cốt lõi của Windows và Windows API. Mỗi khi chương trình yêu cầu nạp một DLL mà không cần chỉ định đường dẫn tuyệt đối, hệ điều hành sẽ tìm kiếm DLL trong các thư mục này.                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| WinSxS                     | Windows Component Store chứa bản sao tất cả các thành phần Windows, các bản cập nhập và Service Pack, phụ vụ cho việc cài đặt, cập nhật và khôi phục hệ thống.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
### Exploring Directories Using Command Line
Chúng ta có thể khám phá hệ thống tệp (**File System**) bằng cách sử dụng lệnh **`dir`**.
![[Screenshot 2026-07-22 at 01.04.40.png|center|border|650]]
Lệnh **`dir`** được sử dụng để liệt kê nội dung của một thư mục. Trong ví dụ trên, tham số **`/a`** yêu cầu hiển thị **tất cả** các tệp và thư mục, bao gồm cả những mục bị ẩn (**Hidden**) hoặc được hệ thống bảo vệ (**System Files**). Đây là một lệnh thường xuyên được sử dụng trong quá trình **Enumeration** nhằm xác định cấu trúc thư mục, các tệp quan trọng cũng như những dữ liệu có thể phục vụ cho việc đánh giá an ninh.
Tiện ích **`tree`** rất hữu ích khi cần hiển thị **cấu trúc phân cấp (Hierarchy)** của các thư mục dưới dạng cây, giúp người dùng dễ dàng quan sát mối quan hệ giữa các thư mục và thư mục con.
![[Screenshot 2026-07-22 at 01.08.08.png|center|border|650]]
Trong đó, tham số **`/f`** yêu cầu hiển thị **cả tệp và thư mục**, thay vì chỉ hiển thị thư mục. Ký tự **`|`** là **Pipeline**, dùng để chuyển kết quả của lệnh **`tree`** sang lệnh **`more`**, giúp hiển thị nội dung theo từng trang thay vì in toàn bộ kết quả ra màn hình cùng một lúc. Đây là một kỹ thuật rất hữu ích khi làm việc với các thư mục có số lượng lớn tệp và thư mục con.
![[Screenshot 2026-07-22 at 01.08.56.png|center|border|650]]

## File System
Windows hỗ trợ 5 loại hệ thống tệp gồm FAT12, FAT16, FAT32, NTFS và  exFAT. Trong đó, FAT12 và FAT16 hiện không còn được sử dụng trên các phiên bản windows hiện tại.
FAT32 (File Allocation Table) được sử dụng rộng rãi trên nhiều loại thiết bị lưu trữ như USB Flash Drive, SD Card và cũng có thể được dùng để định dạng như ổ cứng. Số 32 trong tên biểu thị việc FAT32 sử dụng 32 bit để định danh các cụm dữ liệu trên thiết bị lưu trữ.
Ưu điểm của FAT32:
- Khả năng tưởng thích thiết bị cao: có thể sử dụng trên nhiều loại thiết bị như máy tính, máy ảnh kỹ thuật số, máy chơi game, điện thoại thông minh, máy tính bảng và nhiều thiết bị khác.
- Khả năng tương thích đa hệ điều hành: hoạt động trên tất cả các phiên bản Windows từ Windows 95, đồng thời được macOS và Linux hỗ trợ.
Nhược điểm của FAT32:
- Chỉ hỗ trợ các tệp có kích thước nhỏ hơn 4GB.
- Không tích hợp sẵn các cơ chế bảo vệ dữ liệu hoặc nén tệp.
- Muốn mã hoá tệp phải sử dụng các công cụ bên thứ 3.
NTFS (New Technology File System) là hệ thống tệp mặc định của Windows kể từ Windows NT 3.1. Bên cạnh việc khắc phục những hạn chế của FAT32, NTFS còn hỗ trợ metadata tốt hơn và mang lại hiệu năng cao hơn nhờ cơ chế tổ chức dữ liệu được cải tiến.
Ưu điểm của NTFS:
- NTFS có độ tin cậy cao và có khả năng khôi phục tính nhất quán của hệ thống tệp khi xảy ra lỗi hệ thống hoặc mất điện đột ngột.
- Cung cấp cơ chế bảo mật bằng cách cho phép thiết lập quyền truy cập chi tiết đối với cả tệp và thư mục.
- Tích hợp Journaling, nghĩa là mọi thay đổi đối với tệp như Create, Modify hoặc Delete đều được ghi lại trong Journal giúp tăng khả năng khôi phục dữ liệu khi xảy ra sự cố.
Nhược điểm của NTFS:
- Phần lớn các thiết bị di động không hỗ trợ NTFS một cách mặc định.
- Nhiều thiết bị đa phương tiện đời cũ như TV hoặc máy ảnh kĩ thuật số không hỗ trợ các thiết bị lưu trữ được định dạng bằng NTFS.
### Permissions
Hệ thống tệp NTFS hỗ trợ nhiều loại quyền truy cập từ cơ bản đến nâng cao. Dưới đây là một số loại quyền quan trọng:

| Permission           | Description                                                                                                                                                                                                                                                                                                                                                                  |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Full Control         | Cho phép Read, Write, Modify và Delete tệp hoặc thư mục                                                                                                                                                                                                                                                                                                                      |
| Modify               | Cho phép Read, Write, và Delete tệp hoặc thư mục                                                                                                                                                                                                                                                                                                                             |
| List Folder Contents | Cho phép xem và liệt kê các thư mục, thư mục con cũng như Execute các tệp. Quyền này chỉ được thừa kế bởi thư mục                                                                                                                                                                                                                                                            |
| Read and Execute     | Cho phép xem danh sách tệp và thư mục con, đồng thời thực thi các tệp. Quyền này được kế thừa bởi cả tệp và thư mục.                                                                                                                                                                                                                                                         |
| Write                | Cho phép tạo thêm tệp trong thư mục hoặc thư mục con và ghi dữ liệu vào tệp.                                                                                                                                                                                                                                                                                                 |
| Read                 | Cho phép xem danh sách thư mục, thư mục con và đọc nội dung của tệp.                                                                                                                                                                                                                                                                                                         |
| Traverse Folder      | Cho phép hoặc từ chối khả năng đi xuyên qua (**Traverse**) các thư mục để truy cập đến tệp hoặc thư mục khác. Ví dụ, người dùng có thể không có quyền liệt kê nội dung hoặc xem các tệp trong đường dẫn `C:\Users\bsmith\Documents\webapps\backups\backup_02042020.zip`, nhưng nếu được cấp quyền **Traverse Folder**, họ vẫn có thể truy cập trực tiếp đến tệp sao lưu này. |
Theo mặc định, các tệp, và thư mục sẽ kế thừa các quyền NTFS từ thư mục cha. Cơ chế này giúp đơn giản hoá việc quản trị vì quản trị viên không cần thiết lập quyền riêng cho từng tệp hoặc thư mục, điều vốn sẽ rất tốn thời gian. Trong trường hợp cần cấu hình quyền riêng biệt, quản trị viên có thể tắt cơ chế kế thừa quyền, đối với tệp hoặc thư mục cần thiết, sau đó gán quyền trực tiếp cho từng đối tượng.
### Integrity Control Access Control List (icacls)
Các quyền NTFS trên tệp và thư mục trong Windows có thể quản lý thông qua File Explorer tại tag Security. Bên cạnh giao diện đồ hoạ (GUI), Windows còn cung cấp tiện ích dòng lệnh `icacls`, cho phép quản trị viên quản lý quyền NTFS với mực độ chi tiết rất cao.
Chúng ta có thể xem các quyền NTFS của một thư mục cụ thể bằng cách thực thi **`icacls`** ngay trong thư mục hiện tại hoặc chỉ định trực tiếp đường dẫn, ví dụ:
![[Screenshot 2026-07-22 at 02.06.28.png|center|border|650]]
Trong kết quả trên, mức quyền truy cập (Access Level) được hiển thị ngay sau tên của từng tài khoản hoặc nhóm người dùng. Các ký hiệu kế thừa (Inheritance Flags) bao gồm:
- (CI) Container Inherit: quyền được kế thừa bởi các thư mục con.
- (OI) Object Inherit: quyền được kế thừa bởi các tệp.
- (IO) Inherit Only: quyền chỉ được áp dụng cho các đối tượng kế thừa, không áp dụng trực tiếp lên các đối tượng hiện tại.
- (NP) No Propagate Inherit: không tiếp tục truyền quyền kế thừa xuống các cấp thấp hơn.
- (I): quyền được kế thừa từ thư mục cha.
Trong ví dụ trên, tài khoản `NT AUTHORITY\SYSTEM` được gán các thuộc tính Object Inherit, Container Inherit, Inherit Only cùng quyền Full Control. Điều này có nghĩa là tài khoản này có toàn quyền đối với tất cả các đối tượng của hệ thống tệp nằm trong thư mục hiện tại và các thư mục con.
Các quyền truy cập cơ bản được ký hiệu như sau:

| Ký hiệu | Ý nghĩa                 |
| ------- | ----------------------- |
| F       | Full Access             |
| D       | Delete Access           |
| N       | No Access               |
| M       | Modify Access           |
| RX      | Read and Execute Access |
| R       | Read-Only               |
| W       | Write-Only              |
Chúng ta cũng có thể thêm hoặc xóa quyền thông qua **`icacls`**. Trong ví dụ dưới đây, lệnh được thực thi dưới ngữ cảnh của một **Local Administrator**, cho thấy người dùng **joe** hiện chưa có quyền ghi đối với thư mục **`C:\Users`**.
![[Screenshot 2026-07-22 at 02.19.30.png|center|border|650]]
Để cấp **Full Control** cho người dùng **mrb3n**, có thể sử dụng lệnh sau:
![[Screenshot 2026-07-22 at 02.27.56.png|center|border|650]]
Tuy nhiên, do trong lệnh không chỉ định các cờ `(OI)` và `(CI)`, người dùng mrb3n chỉ có Full Control đối với chính thư mục `C:\Users`, mà không được kế thừa quyền xuống các thư mục con hoặc các tệp bên trong.
Nếu muốn thu hồi quyền của người dùng **mrb3n**, có thể sử dụng lệnh:
```cmd
icacls c:\users /remove mrb3n
```
![[Screenshot 2026-07-22 at 02.30.29.png|center|border|650]]
Tiện ích `icacls` là một công cụ rất mạnh, đặc biệt trong môi trường Windows Domain. Công cụ này cho phép quản trị viên cấp quyền cụ thể cho từng người dùng hoặc nhóm (User/Group), từ chối truy cập (Explicit Deny), bật hoặc tắt cơ chế kế thừa quyền (Permission Inheritance) cũng như thay đổi chủ sở hữu của tệp hoặc thư mục.
## NTFS vs. Share Permissions
Microsoft hiện chiếm hơn 70% thị phần hệ điều hành máy tính để bàn trên toàn cầu với Windows. Đây cũng là lý do vì sao phần lớn tác giả mã độc lựa chọn phát triển mã độc cho nền tảng Windows và khiến nhiều người cho rằng Windows kém an toàn hơn các hệ điều hành khác. Tuy nhiên, xét dưới góc độ kinh doanh, việc tập trung nguồn lực để phát triển mã độc cho Windows là hoàn toàn hợp lý vì đây là mục tiêu có giá trị cao . Quan điểm cho rằng bất kỳ hệ điều hành nào miễn nhiễm với mã độc đều là một ngộ nhận về mặt kỹ thuật . Nếu có thể phát triển phần mềm cho một hệ điều hành thì cũng có thể phát triển virus hoặc mã độc cho hệ điều hành đó. Theo định nghĩa, virus chỉ đơn giản là một phần mềm được viết với mục đích độc hại và có thể tồn tại trên bất kỳ hệ điều hành nào.
Nhiều biến thể mã độc trên Windows có khả năng lây lan qua mạng thông qua các Network Share được cấu hình với quyền truy cập quá lỏng lẻo. Bên cạnh đó, cần lưu ý rằng cho đến nay lỗ hổng nổi tiếng EternalBlue vẫn tiếp tục đe dọa các hệ thống Windows chưa được vá đang sử dụng SMBv1, và thường là bước khởi đầu để các cuộc tấn công bằng Ransomware làm tê liệt hoạt động của doanh nghiệp.
Server Message Block (SMB) là giao thức được Windows sử dụng để truy cập các tài nguyên được chia sẻ (Shared Resources) như tệp và máy in. Đây là giao thức được sử dụng rộng rãi trong các doanh nghiệp ở mọi quy mô, từ nhỏ đến lớn. Hình minh họa dưới đây thể hiện mô hình hoạt động của SMB.
![[Windows Fundamentals.png|center|border|650]]
Nhiều người thường nhầm lẫn rằng NTFS Permissions và Share Permissions là một. Trên thực tế, chúng không giống nhau, mặc dù cả hai thường được áp dụng đồng thời trên cùng một thư mục chia sẻ (Shared Folder). Dưới đây là các quyền có thể được cấu hình để kiểm soát truy cập đối với một thư mục được chia sẻ trên hệ thống Windows sử dụng NTFS.
**Share Permissions**

| Permissions  | Description                                                                                                                                |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Full Control | Cho phép người dùng thực hiện tất cả thao tác của Change và Read, đồng thời có quyền thay đổi NTFS Permissions của các tệp và thư mục con. |
| Change       | Cho phép đọc, chỉnh sửa, xoá và tạo mới tệp cũng như thư mục con.                                                                          |
| Read         | Chỉ cho phép xem nội dung của tệp và các thư mục con.                                                                                      |
**NTFS Basic Permissions**

| Permissions          | Description                                                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Full Control         | Cho phép tạo, chỉnh sửa, di chuyển và xoá tệp hoặc thư mục đồng thời thay đổi các quyền NTFS áp dụng cho các thư mục được phép. |
| Modify               | Cho phép hoặc từ chối quyền xem và chỉnh sửa tệp, thư mục, bao gồm cả việc thêm hoặc xóa tệp.                                   |
| List Folder Contents | Cho phép hoặc từ chối quyền xem danh sách các tệp và thư mục con.                                                               |
| Read                 | Cho phép hoặc từ chối quyền đọc nội dung của tệp.                                                                               |
| Write                | Cho phép hoặc từ chối quyền ghi dữ liệu vào tệp hoặc tạo tệp mới trong thư mục.                                                 |
| Special Permissions  | Bao gồm nhiều quyền nâng cao để kiểm soát chi tiết hơn đối với tệp và thư mục.                                                  |
**NTFS Special Permissions**

| Permissions                    | Description                                                                                                                                                                               |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Full Control                   | Cho phép hoặc từ chối toàn quyền đối với tệp và thư mục, bao gồm thay đổi NTFS Permissions.                                                                                               |
| Traverse Folder / Execute File | Cho phép hoặc từ chối truy cập vào thư mục con trong một cấu trúc thư mục ngay cả khi không có quyền xem nội dung của thư mục cha. Đồng thời cho phép hoặc từ chối thực thi chương trình. |
| List Folder / Read Data        | Cho phép hoặc từ chối xem danh sách các tệp, thư mục con và mở tệp để đọc nội dung.                                                                                                       |
| Read Attributes                | Cho phép hoặc từ chối xem các thuộc tính cơ bản của tệp hoặc thư mục như **System**, **Archive**, **Read-only** và **Hidden**.                                                            |
| Create Files / Write Data      | Cho phép hoặc từ chối tạo tệp mới trong thư mục và ghi dữ liệu vào tệp.                                                                                                                   |
| Create Folders / Append Data   | Cho phép hoặc từ chối tạo thư mục con và thêm dữ liệu vào cuối tệp mà không ghi đè dữ liệu hiện có.                                                                                       |
| Write Attributes               | Cho phép hoặc từ chối thay đổi các thuộc tính của tệp. Quyền này không bao gồm việc tạo tệp hoặc thư mục mới.                                                                             |
| Write Extended Attributes      | Cho phép hoặc từ chối thay đổi các thuộc tính mở rộng của tệp hoặc thư mục.                                                                                                               |
| Delete Subfolders and Files    | Cho phép hoặc từ chối xóa các thư mục con và tệp, nhưng không bao gồm thư mục cha.                                                                                                        |
| Delete                         | Cho phép hoặc từ chối xóa thư mục cha, thư mục con và các tệp.                                                                                                                            |
| Read Permissions               | Cho phép hoặc từ chối xem các quyền được gán cho tệp hoặc thư mục.                                                                                                                        |
| Change Permissions             | Cho phép hoặc từ chối thay đổi quyền truy cập của tệp hoặc thư mục.                                                                                                                       |
| Take Ownership                 | Cho phép hoặc từ chối quyền chiếm quyền sở hữu (**Ownership**) của tệp hoặc thư mục. Chủ sở hữu của một đối tượng có toàn quyền thay đổi các quyền được gán cho đối tượng đó.             |
Cần lưu ý rằng NTFS Permissions chỉ áp dụng trên chính hệ thống lưu trữ tệp hoặc thư mục. Theo mặc định, các thư mục được tạo trên phân vùng NTFS sẽ kế thừa quyền (Inheritance) từ thư mục cha. Tuy nhiên, quản trị viên có thể vô hiệu hóa cơ chế kế thừa này để cấu hình các quyền tùy chỉnh cho từng thư mục hoặc thư mục con.
Trong khi đó, Share Permissions chỉ được áp dụng khi thư mục được truy cập thông qua giao thức SMB, tức là thông thường từ một máy tính khác trên mạng. Điều này có nghĩa là nếu người dùng đăng nhập trực tiếp vào máy tính hoặc truy cập thông qua RDP, họ chỉ cần tuân theo NTFS Permissions khi truy cập thư mục trên hệ thống tệp, còn Share Permissions sẽ không được áp dụng.
Chính vì vậy, **NTFS Permissions** mang lại cho quản trị viên khả năng kiểm soát chi tiết (**Granular Access Control**) hơn rất nhiều đối với các thao tác mà người dùng được phép thực hiện trên từng tệp hoặc thư mục.
### Creating a Network Share
Để hiểu rõ nguyên lý hoạt động của SMB và mối quan hệ giữa SMB với NTFS Permissions, chúng ta sẽ thực hành tạo một Network Share trên máy Windows 10. Chúng ta sẽ tạo một Shared Folder bằng cách tạo một thư mục mới trên Desktop của máy Windows 10.
Cần lưu ý rằng trong hầu hết các môi trường doanh nghiệp lớn, các thư mục chia sẻ thường không được đặt trên máy tính cá nhân mà được lưu trữ trên Storage Area Network (SAN), Network Attached Storage (NAS) hoặc trên một phân vùng riêng của máy chủ chạy hệ điều hành như Windows Server.
Nếu trong quá trình kiểm thử xâm nhập ta phát hiện một Network Share trên một hệ điều hành Windows Desktop, điều này thường cho thấy một trong hai khả năng: hoặc đây là hệ thống của một doanh nghiệp nhỏ, hoặc đây là một Beachhead System đã bị Pentester hoặc kẻ tấn công chiếm quyền và đang được sử dụng làm điểm trung gian để thu thập và đánh cắp dữ liệu.
**Creating the Folder**
![[Screenshot 2026-07-22 at 03.07.39.png|center|border|650]]
Chúng ta sẽ sử dụng tùy chọn Advanced Sharing để cấu hình thư mục chia sẻ.
![[Screenshot 2026-07-22 at 03.10.28.png|center|border|650]]
**Making the Folder a Share**
![[Screenshot 2026-07-22 at 03.12.28.png|center|border|650]]
Có thể thấy rằng Share Name mặc định sẽ được đặt trùng với tên của thư mục. Ngoài ra, Windows cũng cho phép giới hạn số lượng người dùng được phép kết nối đồng thời đến thư mục chia sẻ này. Trong môi trường thực tế, quản trị viên nên cấu hình giá trị này dựa trên số lượng người dùng thường xuyên cần truy cập tài nguyên được chia sẻ.
Tương tự như NTFS Permissions, các tài nguyên chia sẻ cũng có một Access Control List (ACL) riêng. Có thể xem đây là SMB Permissions List. Cần lưu ý rằng đối với các tài nguyên được chia sẻ, cả Share Permissions (SMB) và NTFS Permissions đều được áp dụng đồng thời lên tài nguyên đó.
ACL bao gồm các Access Control Entries (ACE). Thông thường, mỗi ACE sẽ chứa một người dùng hoặc nhóm người dùng, còn được gọi là Security Principal, vì đây là cơ chế phù hợp để quản lý và theo dõi quyền truy cập vào các tài nguyên được chia sẻ.
**Share Permissions ACL (Sharing Tab)**
![[Screenshot 2026-07-22 at 03.15.18.png|center|border|650]]



## Windows Services & Processes
### Windows Services
Windows Services là một thành phần cốt lõi của hệ điều hành Windows, cho phép tạo và quản lý các tiến trình chạy nền. Không giống các ứng dụng thông thường chỉ hoạt động khi người dùng đăng nhập, Windows Services có thể tự động khởi động trong quá trình boot hệ thống mà không cần bất kỳ sự tương tác nào từ người dùng.
Sau khi được khỏi động, các dịch vụ này tiếp tục hoạt động trong nền ngay cả khi người dùng đã đăng xuất khỏi hệ thống. Cơ chế này giúp duy trì liên tục các chức năng quan trọng của hệ điều hành và các ứng dụng máy chủ.
Ngoài các dịch vụ đực tích hợp sẵn trong Windows, nhà phát triển cũng có thể triển khai ứng dụng dưới dạng Windows Service. Ví dụ, một hệ thống giám sát mạng, phần mềm chống mã độc, hệ thống thu thập nhật lý, hoặc tác nhân EDR thường được cài đặt dưới dạng dịch vụ để đảm bảo hoạt động liên tục và có khả năng tự khởi động sau mỗi lần hệ thống khởi động lại.
Windows Services chịu trách nhiệm vận hành nhiều chức năng quan trọng của hệ điều hành bao gồm:
- Cung cấp các dịch vụ mạng.
- Thực hiện chuẩn đoán và giám sát hệ thống.
- Quản lý thông tin xác thực và chứng thực người dùng.
- Cung cấp nhiều dịch vụ nền thiết yếu khác phục vụ cho hoạt động ổn định của hệ điều hành.
Việc quản lý Windows Services được thực hiện thông qua Service Control Manager (SCM), thành phần chịu trách nhiệm khởi tạo, dừng, cấu hình và giám sát toàn bộ dịch vụ trên hệ thống. Người quản trị có thể truy cập SCM thông qua Microsoft Management Console (MMC) với tiện ích `services.msc` .
Giao diện `services.msc` cung cấp khả năng theo dõi và quản lý từng dịch vụ đã được cài đặt, đồng thời hiển thị các thông tin quan trọng như:
- Name: tên dịch vụ.
- Description: mô tả chức năng của dịch vụ.
- Status: trạng thái hiện tại (Running, Stopped,...).
- Startup Type: chế độ khởi động (Automatic, Automatic (Deylayed Start), Manual, Disabled).
- Log On As: Tài khoản mà dịch vụ sử dụng để thực thi (ví dụ: LocalSystem, LocalService, NetworkService, hoặc tài khoản người dùng).
Bên cạnh giao diện đồ hoạ, quản trị viên cũng có thể quản lý Windows Services thông qua dòng lệnh bằng công cụ `sc.exe` hoặc sử dụng các Powershell Cmdlets chẳng hạn như `Get-Service` để liệt kê, truy vấn trạng thái và tự động hoá việc quản lý dịch vụ.
![[Screenshot 2026-07-25 at 06.26.04.png|center|border|650]]
Mỗi Windows Service có thể tồn tại ở nhiều trạng thái khác nhau như Running, Stopped, Paused, Starting, Stopping. Ngoài trạng thái hoạt động, mỗi dịch vụ còn có Startup Type, xác định thời điểm dịch vụ được kích hoạt như Automatic, Automatic (Delayed Start), Manual, Disabled. Windows chia dịch vụ thành ba nhóm chính Local Services, Network Services, System Services. Thông thường việc tạo, chỉnh sửa hoặc xoá Windows Services chỉ được phép đối với người dùng có quyền quản trị. Trong pentest, các cấu hình sai quyền là một trong những kỹ thuật Privilege Escalation phổ biến, cho phép kẻ tấn công leo thang đặc quyền lên mức SYSTEM.

| Services              | Description                                                                                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| smss.exe              | Session Manager Subsystem: quản lý các phiên làm việc của hệ thống trong quá trình khởi động Windows                                                          |
| csrss.exe             | Client/ Server Runtime Subsystem là thành phần ở chế độ người dùng (User mode) của Windows Subsystem, chịu trách nhiệm xử lý console tạo tiến trình và luồng. |
| wininit.exe           | Khởi tạo môi trường Windows sau quá trình boot và khởi động các dịch vụ hệ thống quan trọng.                                                                  |
| logonui.exe           | Giao diện đăng nhập (Windows Logon Interface) hiển thị màn hình đăng nhập cho người dùng.                                                                     |
| lsass.exe             | Local Security Authority Subsystem Service (LSASS) thực hiện xác thực người dùng, áp dụng chính sách bảo mật và quản lý Access Token.                         |
| services.exe          | Service Control Manager (SCM) quản lý khởi động dùng và giám sát toàn bộ Windows Services.                                                                    |
| winlogon.exe          | Xử lý quá trình đăng nhập, Secure Attention Sequence, tải hồ sơ người dùng và khoá màn hình.                                                                  |
| System                | Tiến trình nền đại diện cho Windows Kernel                                                                                                                    |
| svchost.exe (RPCSS)   | Tiến trình host chứa nhiều dịch vụ chạy từ DLL, sử dụng Remote Procedure Call (RPC)                                                                           |
| svhost.exe (DCOM/PnP) | Tiến trình Host chạy các dịch vụ sử dụng Distributed Component Object Model và Plug and Play                                                                  |
> Lưu ý: `svhost.exe` không phải là một dịch vụ riêng lẻ mà là Service Host Process, được sử dụng để lưu trữ nhiều Windows Services viết dưới dạng DLL.

### Processes
Process là một chương trình đang được thực thi trên hệ thống Windows. Các tiến trình có thể được hệ điều hành tự động tạo trong quá trình khởi động, được khởi chạy bởi người dùng, hoặc được tạo bởi các ứng dụng hoặc dịch vụ khác. Phần lớn tiến trình của ứng dụng có thể bị kết thục mà không ảnh hưởng nghiêm trọng đến hệ điều hành. Tuy nhiên, một số tiến trình là Critical Processes, nếu bị dừng sẽ khiến Windows hoạt động không ổn định dẫn đến Blue Screen of Death (BSOD).
Các tiến trình quan trọng bao gồm Windows Logon Application, System, System Idle Process, Windows Start-Up Application, Client Server Runtime Process (CSRSS), Windows Session Manager (SMSS), Service Host (svchost.exe), và Local Security Authority Subsystem Service (LSASS).
### Local Security Authority Subsystem Service (LSASS)
`Isass.exe` là tiến trình chịu trách nhiệm thực thi các chính sách bảo mật trên hệ thống Windows.
Khi người dùng cố gắng đăng nhập vào hệ thống, tiến trình này sẽ xác thực thông tin đăng nhập và tạo Access Token dựa trên mức quyền của người dùng. LSASS cũng chịu trách nhiệm xử lý các yêu cầu thay đổi mật khẩu của tài khoản người dùng.
Mọi sự kiện liên quan đến tiến trình này, chẳng hạn như đăng nhập, đăng xuất, và các lần xác thực khác, đều được ghi lại trong Windows Security Log.
LSASS là một mục tiêu có giá trị rất cao đối với các pentester vì có nhiều công cụ có khả năng trích xuất cả clear text lẫn hash mật khẩu được lưu trong bộ nhớ của tiến trình.
### Sysinternals Tools
Sysinternals tools là một bộ công cụ di động dành cho Windows do Microsoft phát triển, được sử dụng để quản trị và phân tích hệ thống. Phần lớn các công cụ trong bộ này không yêu cầu cài đặt trước khi sử dụng. 
Các công cụ có thể được tải trực tiếp từ trang web của Microsoft hoặc chạy ngay từ thư mục chia sẻ trực tuyến bằng cách nhập đường dẫn sau vào Windows Explorer:
```powershell
\\live.sysinternals.com\tools
```
Ví dụ, chúng ta có thể chạy **procdump.exe** trực tiếp từ thư mục chia sẻ này mà không cần tải tệp về máy:
```powershell
C:\htb> \\live.sysinternals.com\tools\procdump.exe -accepteula
```
`ProcDump` là công cụ dùng để giám sát một tiến trình và tạo memory dump khi tiến trình đáp ứng các điều kiện được chỉ định, chẳng hạn như sử dụng CPU vượt ngưỡng hoặc xảy ra ngoại lệ (exception).
Bộ Sysinternals còn bao gồm nhiều công cụ hữu ích khác như Process Explorer, phiên bản nâng cao của Task Manager; Process Monitor, dùng để giám sát hoạt động của hệ thống tệp, Registry và mạng; TCPView, dùng để theo dõi các kết nối mạng; và PsExec, hỗ trợ quản lý hoặc kết nối tới các hệ thống từ xa thông qua giao thức SMB.
Đối với pentester, các công cụ này rất hữu ích để xác định những tiến trình đáng chú ý, tìm kiếm các khả năng Privilege Escalation cũng như hỗ trợ Lateral Movement trong quá trình kiểm thử xâm nhập.
### Task Manager
**Windows Task Manager** là công cụ mạnh mẽ được tích hợp sẵn để quản lý hệ thống Windows. Công cụ này cung cấp thông tin về các tiến trình đang chạy, hiệu năng hệ thống, các dịch vụ (services), chương trình khởi động cùng Windows, người dùng đang đăng nhập cùng các tiến trình của họ và mức sử dụng tài nguyên của hệ thống.
Task Manager có thể được mở bằng nhiều cách, chẳng hạn như nhấp chuột phải vào thanh Taskbar và chọn Task Manager, nhấn Ctrl + Shift + Esc, nhấn Ctrl + Alt + Delete rồi chọn Task Manager, tìm kiếm Task Manager trong Start Menu hoặc nhập lệnh `taskmgr` từ Command Prompt hoặc PowerShell.
![[Screenshot 2026-07-29 at 15.30.15.png|center|border|650]]
#### Processes tab
Hiển thị danh sách các ứng dụng và tiến trình nền đang hoạt động, cùng với mức sử dụng CPU, Memory, Disk, Network và Power của từng tiến trình.
#### Performance tab
Hiển thị biểu đồ và các thông số như mức sử dụng CPU, thời gian hoạt động của hệ thống (System Uptime), mức sử dụng bộ nhớ, ổ đĩa, mạng và GPU. Từ tab này, người dùng cũng có thể mở Resource Monitor, công cụ cung cấp thông tin chi tiết hơn về việc sử dụng CPU, Memory, Disk và Network.
#### App History tab
Hiển thị lịch sử sử dụng tài nguyên của các ứng dụng đối với tài khoản người dùng hiện tại trong một khoảng thời gian.
#### Startup tab
Hiển thị các ứng dụng được cấu hình để tự động khởi động cùng Windows cũng như mức ảnh hưởng của chúng đến thời gian khởi động hệ thống.
#### Users tab
Hiển thị những người dùng đang đăng nhập cùng các tiến trình và mức sử dụng tài nguyên tương ứng của từng phiên đăng nhập.
#### Details tab
Hiển thị thông tin chi tiết về từng tiến trình, bao gồm tên tiến trình, Process ID (PID), trạng thái, tài khoản người dùng đang chạy tiến trình, mức sử dụng CPU và bộ nhớ.
#### Services tab
Hiển thị tên, PID, mô tả và trạng thái của từng dịch vụ đã được cài đặt trên hệ thống. Từ tab này cũng có thể mở trực tiếp công cụ Services của Windows.
![[Windows Fundamentals-1.png|center|border|650]]
### Process Explorer
Process Explorer là một thành phần thuộc bộ công cụ Sysinternals. Công cụ này cho phép người dùng xem những Handle và DLL mà một tiến trình đã nạp trong quá trình thực thi.
Process Explorer hiển thị danh sách tất cả các tiến trình đang chạy. Từ đó, người dùng có thể xem các Handle mà tiến trình đang sử dụng trong một cửa sổ hoặc xem các DLL và các tệp được ánh xạ vào bộ nhớ (memory-mapped files) trong cửa sổ khác.
Ngoài ra, Process Explorer còn hỗ trợ tìm kiếm để xác định tiến trình nào đang sử dụng một Handle hoặc DLL cụ thể. Công cụ này cũng rất hữu ích trong việc phân tích mối quan hệ parent-child process, giúp xác định các tiến trình con được tạo bởi một ứng dụng và hỗ trợ khắc phục các sự cố như orphan process (tiến trình mồ côi) vẫn còn tồn tại sau khi tiến trình cha đã kết thúc.
## Service Permissions
Service cho phép quản lý các tiến trình chạy lâu dài và là một thành phần rất quan trọng của hệ điều hành Windows. Tuy nhiên, các quản trị viên hệ thống thường bỏ qua chúng như một vector tấn công có thể bị lợi dụng để nạp các DLL độc hại, thực thi ứng dụng mà không cần quyền quản trị viên, leo thang đặc quyền hoặc thậm chí duy trì khả năng truy cập lâu dài.
Những vector tấn công này thường xuất hiện do cấu hình sai quyền của Service, có thể bắt nguồn từ phần mềm của bên thứ ba hoặc do những sai sót trong quá trình cài đặt và cấu hình của quản trị viên.
Bước đầu tiên để nhận thức được tầm quan trọng của **Service Permissions** là hiểu rằng chúng tồn tại và luôn phải lưu ý đến chúng trong quá trình quản trị hệ thống.
Trên các hệ điều hành Windows Server, các dịch vụ mạng quan trọng như DHCP và Active Directory Domain Services (AD DS) thường được cài đặt bằng chính tài khoản của quản trị viên đang thực hiện quá trình cài đặt.
Trong quá trình cài đặt, dịch vụ sẽ được cấu hình để chạy bằng thông tin xác thực và quyền hạn của một tài khoản người dùng được chỉ định. Theo mặc định, tài khoản này chính là tài khoản của người dùng hiện đang đăng nhập vào hệ thống.
Ngoài việc quan tâm đến quyền của Service, chúng ta cũng cần chú ý đến quyền của thư mục chứa chương trình thực thi. Nếu quyền NTFS của thư mục này được cấu hình quá lỏng lẻo, kẻ tấn công có thể thay thế tệp thực thi gốc (Executable) bằng một chương trình hoặc DLL độc hại. Đây là một kỹ thuật phổ biến được sử dụng trong nhiều cuộc tấn công Privilege Escalation trên Windows.
### Examining Services using services.msc
Trong phần Processes and Services, công cụ `services.msc` cho phép xem và quản lý hầu hết mọi thông tin liên quan đến các dịch vụ trên Windows. Hãy lấy ví dụ với dịch vụ **Windows Update (wuauserv)**.
![[Screenshot 2026-07-29 at 15.55.50.png|center|border|650]]
Khi mở thuộc tính của dịch vụ này, chúng ta sẽ thấy nhiều thông tin quan trọng có thể xem và cấu hình. Trong số đó, Service Name đặc biệt hữu ích vì tên này thường được sử dụng khi quản lý dịch vụ thông qua các công cụ dòng lệnh như `sc.exe`, `PowerShell` hoặc `net`.
Một trường thông tin khác cũng rất quan trọng là Path to Executable, đây là đường dẫn đầy đủ đến chương trình sẽ được thực thi khi dịch vụ khởi động.
Nếu quyền NTFS của thư mục chứa tệp thực thi được cấu hình không an toàn, kẻ tấn công có thể thay thế chương trình gốc bằng một chương trình độc hại. Khi dịch vụ được khởi động, chương trình độc hại này cũng sẽ được thực thi với quyền của dịch vụ.
![[Screenshot 2026-07-29 at 15.57.41.png|center|border|650]]
Trong cửa sổ Properties của Service, tab Log On cho biết dịch vụ sẽ chạy dưới tài khoản nào. Phần lớn các dịch vụ trên Windows mặc định chạy bằng tài khoản LocalSystem, đây là tài khoản có mức đặc quyền cao nhất trên một máy Windows cục bộ. Tuy nhiên, không phải mọi ứng dụng đều cần đến quyền LocalSystem. Do đó, trước khi triển khai một ứng dụng mới trong môi trường Windows, quản trị viên nên nghiên cứu kỹ yêu cầu của ứng dụng để xác định mức quyền tối thiểu cần thiết. Đây là một thực hành tốt nhằm tuân thủ Principle of Least Privilege (Nguyên tắc đặc quyền tối thiểu), tức là chỉ cấp đúng lượng quyền cần thiết để ứng dụng hoạt động.  Windows cung cấp một số Built-in Service Account thường được sử dụng để chạy dịch vụ, bao gồm: LocalService, NetworkService, LocalSystem. Ngoài các tài khoản tích hợp sẵn này, quản trị viên cũng có thể tạo các tài khoản mới chỉ với mục đích duy nhất là chạy một dịch vụ cụ thể. Những tài khoản này thường được gọi là Service Accounts.
![[Windows Fundamentals-2.png|Center|Border|650]]
### Examining services using sc
Lệnh `sc` cũng có thể được sử dụng để cấu hình và quản lý các Services, ví dụ như dưới đây:
![[Screenshot 2026-07-29 at 22.04.54.png|center|border|650]]
Lệnh `sc qc` được sử dụng để truy vấn cấu hình của một dịch vụ. Đây là lúc việc biết chính xác tên của các dịch vụ trở nên hữu ích. Nếu muốn truy vấn một dịch vụ trên một máy tính trong mạng, chúng ta chỉ cần chỉ định hostname hoặc địa chỉ IP ngay sau lệnh `sc`.
```dotnet
C:\Users\htb-student>sc //hostname or ip of box query ServiceName
```
Chúng ta cũng có thể sử dụng **sc** để khởi động và dừng các dịch vụ.
![[Screenshot 2026-07-29 at 22.08.08.png|center|border|550]]
Lưu ý rằng thao tác này bị từ chối vì chúng ta không chạy lệnh trong ngữ cảnh có quyền quản trị. Nếu mở Command Prompt với quyền Administrator, chúng ta sẽ được phép thực hiện thao tác này.
![[Screenshot 2026-07-29 at 22.11.33.png|center|border|650]]
Nếu chúng ta đang điều tra một hệ thống bị nghi ngờ nhiễm malware, `sc` cho phép nhanh chóng tìm kiếm và phân tích các dịch vụ thường bị nhắm mục tiêu cũng như các dịch vụ mới được tạo. Ngoài ra, công cụ này cũng thân thiện với việc viết script hơn nhiều so với việc sử dụng các công cụ giao diện đồ hoạ như `services.msc`.
Một cách hữu ích khác để kiểm tra quyền của Service bằng `sc` là sử dụng lệnh `sdshow`: 
![[Screenshot 2026-07-29 at 22.16.15.png|center|border|650]]
Có thể thấy kết quả đầu ra có vẻ rất khó hiểu, thậm chí có thể khiến chúng ta nghĩ rằng nhập sai lệnh, nhưng thực tế chuỗi ký tự này đều có ý nghĩa.
Mọi đối tượng được đặt tên (Name Object) trong Windows đều là Securable Object, và ngay cả một đối tượng không có tên cũng có thể là Securable Object. Nếu một đối tượng có thể được bảo vệ trong Windows thì nó sẽ có một Security Descriptor.
Security Descriptor xác định chủ sở hữu  của đối tượng và nhóm chính (Primary Group) đồng thời chứa một Discretionary Access Control List (DACL) và một System Access Control List (SACL).
Thông thường DACL được sử dụng để kiểm soát quyền truy cập vào một đối tượng, còn SACL được sử dụng để ghi nhận và ghi log các lần truy cập vào đối tượng đó. Trong phần này chúng ta sẽ tập trung vào DACL, tuy nhiên các khái niệm tương tự cũng được áp dụng cho SACL.
```markdown
D:(A;;CCLCSWRPLORC;;;AU)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)
```
Chuỗi ký tự được ghép lại với nhau và phân tách bằng các dấu ngoặc đơn này được biểu diễn theo định dạng gọi là  Security Descriptor Definition Language (SDDL).
Chúng ta thường có xu hướng đọc từ trái sang phải vì đó là cách tiếng Anh được viết, nhưng khi làm việc với máy tính điều này có thể khác. Hãy đọc toàn bộ Security Descriptor của dịch vụ Windows Update (wuauserv) theo thứ tự bắt đầu từ ký tự đầu tiên và cặp dấu ngoặc đầu tiên. 
```markdown
D: (A;;CCLCSWRPLORC;;;AU)
```
Các ký tự phía sau là các quyền trong DACL.
```markdown
AU:
```
Đại diện cho Authenticated Users.
```markdown
A;;
```
Cho biết quyền truy cập được Allow.
```markdown
CC
```
Tên đầy đủ là `SERVICE_QUERY_CONFIG`, cho phép truy vấn cấu hình của dịch vụ từ Service Control Manager (SCM).
```markdown
SW
```
Tên đầy đủ là `SERVICE_ENUMERATE_DEPENDENTS`, cho phép liệt kê danh sách các dịch vụ phụ thuộc.
```markdown
RP
```
Tên là `SERVICE_START` cho phép khởi động dịch vụ.
```markdown
LO
```
Tên đầy đủ `SERVICE_INTERROGATE`, cho phép truy vấn trạng thái hiện tại của dịch vụ.
```markdown
RC
```
Tên đầy đủ là `READ_CONTROL`, cho phép đọc Security Descriptor của dịch vụ.
Trong quá trình đọc Security Descriptor rất dễ bị rối bởi thứ tự các ký tự có vẻ ngẫu nhiên. Tuy nhiên, hãy nhớ rằng thực chất chúng ta đang xem các Access Control Entry (ACE) trong một Access Control List (ACL). Mỗi cặp gồm hai ký tự nằm giữa các dấu chấm phẩy biểu thị một hành động được phép thực hiện bởi một người dùng hoặc nhóm cụ thể.
`;;CCLCSWRPLORC;;;`: sau dấu phẩy cuối cùng, các ký tự sẽ xác định Security Principal (Người dùng hoặc nhóm) được phép thực hiện các hành động đó.
`;;;AU`: Ký tự nằm ngay sau dấu ngoặc mở và trước cặp dấu chấm phẩy đầu tiên xác định các hành động đó được Allow hay Deny.
`A;;`: Toàn bộ Security Descriptor của dịch vụ Windows Update (wuauserv) này bao gồm ba nhóm Access Control Entry (ACE) vì có ba Security Principal khác nhau. Mỗi Security Principal được gán một tập quyền riêng.
### Examine service permissions using PowerShell
Sử dụng PowerShell cmdlet `Get-Acl` chúng ta có thể kiểm tra quyền của một Service bằng cách truy cập đến đường dẫn của dịch vụ trong Registry.
![[Screenshot 2026-07-29 at 22.54.05.png|center|border|650]]
Lưu ý rằng lệnh này trả về quyền của từng tài khoản dưới dạng dễ đọc, đồng thời cũng hiển thị thông tin dưới dạng SDDL. Ngoài ra, SID đại diện cho từng Security Principal (Người dùng hoặc nhóm) cũng xuất hiện trong chuỗi SDDL. Đây là thông tin mà chúng ta không có được khi sử dụng lệnh `sc` trong cmd.
## Windows Sessions
### Interactive
Interactive Sessions hay Local Logon Session là một phiên đăng nhập được khởi tạo khi người dùng xác thực vào hệ thống cục bộ hoặc miền bằng cách nhập thông tin xác thực của mình.
Một Interactive Logon có thể được thực hiện bằng cách đăng nhập trực tiếp vào hệ thống, yêu cầu một phiên đăng nhập thứ cấp bằng lệnh runas thông qua dòng lệnh, hoặc thông qua kết nối RDP.
### Non-interactive
Các Non-interactive Account trong Windows khác với các tài khoản người dùng thông thường ở chỗ chúng không yêu cầu thông tin đăng nhập.
Có ba loại Non-interactive Account trong Windows, bao gồm Local System Account, Local Service Account và Network Service Account.
Các tài khoản này thường được hệ điều hành Windows sử dụng để tự động khởi động các services và ứng dụng mà không cần bất kỳ sự tương tác nào từ người dùng.
Các tài khoản này không có mật khẩu đi kèm và thường được sử dụng để khởi động các dịch vụ khi hệ thống khởi động hoặc để thực thi các tác vụ theo lịch (Scheduled Tasks).
Có một số điểm khác biệt giữa ba loại tài khoản này:

| Account                 | Description                                                                                                                                                                                                                                                                                                                            |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Local System Account    | Còn được biết đến với tên `NT AUTHORITY\SYSTEM`, đây là tài khoản có mức đặc quyền cao nhất trong hệ thống Windows. Tài khoản này được sử dụng cho nhiều tác vụ liên quan đến hệ điều hành, chẳng hạn như khởi động các dịch vụ của Windows. Mức đặc quyền của tài khoản này cao hơn cả các tài khoản thuộc nhóm Local Administrators. |
| Local Service Account   | Được biết đến với tên `NT AUTHORITY\LocalService`, đây là phiên bản có mức đặc quyền thấp hơn của tài khoản SYSTEM và các quyền tương tự như một tài khoản người dùng cục bộ. Tài khoản này chỉ được cấp các chức năng giới hạn và có thể khởi động một số dịch vụ.                                                                    |
| Network Service Account | Được biết đế với tên `NT AUTHORITY\NetworkService`, tài khoản này tương tự như một tài khoản người dùng Domain thông thường. Trên máy tính cục bộ, nó có các quyền tương tự Local Service Account. Ngoài ra, tài khoản này có thể thiết lập phiên xác thực, đối với một số dịch vụ mạng nhất định.                                     |
## Interacting with the Windows Operating System
### Graphical User Interface
Graphical User Interface (GUI) được giới thiệu vào cuối những năm 1970 bởi phòng nghiên cứu Xerox Palo Alto. Sau đó, GUI được tích hợp vào các hệ điều hành Apple và Microsoft nhằm giải quyết những khó khăn về khả năng sử dụng đối với người dùng phổ thông, những có thể gặp trở ngại khi làm việc với giao diện dòng lệnh (Command Line).
Phần lớn người dùng máy tính Windows thông thường hầu như không cần tương tác với hệ điều hành thông qua dòng lệnh.
Đúng như tên gọi, GUI cung cấp cho người dùng một giao diện đồ họa với khả năng Point-and-Click (trỏ và nhấp chuột), cho phép họ tương tác với hệ điều hành cũng như các ứng dụng và dịch vụ đã được cài đặt.
Việc ra đời của GUI đã giúp máy tính trở nên phổ biến và dễ tiếp cận hơn với nhiều đối tượng người dùng, vì họ có thể sử dụng máy tính mà không cần ghi nhớ các lệnh hoặc bất kỳ ngôn ngữ lập trình nào.
Các quản trị viên hệ thống cũng thường xuyên sử dụng công cụ dựa trên GUI để quản lý AD, cấu hình IIS hoặc làm việc với các cơ sở dữ liệu.
 
### Remote Desktop Protocol (RDP)
RDP (Remote Desktop Protocol) là một giao thức độc quyền của Microsoft, cho phép người dùng kết nối tới hệ thống từ xa thông qua kết nối tới một hệ thống từ xa thông qua kết nối mạng và truy cập giao diện đồ hoạ của hệ thống đó.
Người dùng sẽ sử dụng phần mềm RDP Client để kết nối tới một máy tính đang chạy RDP Server.
RDP sử dụng cổng 3389 để thiết lập một kênh truyền chuyên dụng nhằm trao đổi dữ liệu giữa máy khách và máy chủ. Khi kết nối thông qua RDP, người dùng có thể sử dụng giao diện đồ hoà của hệ thống từ xa giống như đang ngồi trực tiếp trước máy tính và đăng nhập cục bộ. 
RDP thường được các quản trị viên hệ thống sử dụng để quản lý máy chủ hoặc máy tính từ xa một cách nhanh chóng. Ngoài ra, người dùng cũng có thể truy cập vào máy tính làm việc của mình khi đang đi công tác hoặc làm việc tại nhà sau khi kết nối tới Virtual Private Network (VPN).
### Windows Command Line
Các Command-line Interface mang lạ cho người dùng khả năng kiểm soát hệ thống ở mức cao hơn và có thể được sử dụng để thực hiện nhiều tác vụ hàng ngày, công việc quản trị cũng như xử lý sự cố.
CLI cũng có thể được tận dụng để tự động hoá nhiều công việc nhằm thực hiện chúng nhanh hơn, chẳng hạn như thêm nhiều người vào một Domain chỉ trong một lần thực thi. 
Trong hệ điều hành Windows, hai phương pháp chính để tương tác với hệ thống thông qua dòng lệnh là Command Prompt (CMD) và PowerShell.
### CMD
Command Prompt (`cmd.exe`) được sử dụng để nhập và thực thi các lệnh. Người dùng có thể nhập lệnh đơn lẻ như `ipconfig` để xem thông tin về địa chỉ IP hoặc thực hiện các tác vụ nâng cao hơn như thiết lập Scheduled Tasks, tạo Scripts và Batch Files.
Command Prompt có thể được mở từ **Start Menu**, bằng cách nhập `cmd` vào hộp thoại **Run**, hoặc trực tiếp khởi chạy tệp thực thi từ:
```markdown
C:\Windows\system32\cmd.exe
```
Sau khi khởi chạy `cmd.exe`, chúng ta có thể nhập `help` để xem danh sách các lệnh hiện có.
![[Screenshot 2026-07-30 at 16.44.52.png|center|border|650]]
### PowerShell
Windows PowerShell là một command shell được Microsoft thiết kế nhằm hướng tới các quản trị viên hệ thống. Tương tự Windows Command Line, PowerShell cung cấp một giao diện dòng lệnh tương tác (Interactive Command Prompt), đồng thời cũng cung cấp một môi trường viết script mạnh mẽ.
PowerShell được xây dựng dựa trên **.NET Framework**, nền tảng được sử dụng để xây dựng và chạy các ứng dụng trên Windows. Điều này khiến PowerShell trở thành một công cụ rất mạnh để tương tác trực tiếp với hệ điều hành.
Tương tự Command Prompt, PowerShell cho phép chúng ta truy cập trực tiếp vào File System, và chúng ta có thể thực thi phần lớn các lệnh tương tự như khi sử dụng trong một phiên cmd shell.
### Cmdlets
PowerShell sử dụng **Cmdlets**, là những công cụ nhỏ thực hiện một chức năng cụ thể được tích hợp trong shell. 
Có hơn **100 Cmdlets cốt lõi (Core Cmdlets)**, ngoài ra còn có rất nhiều Cmdlets khác được viết thêm. Chúng ta cũng có thể tự tạo Cmdlets của riêng mình để thực hiện các tác vụ phức tạp hơn. PowerShell cũng hỗ trợ cả những Script đơn giản và Script phức tạp, được sử dụng cho các tác vụ quản trị hệ thống, tự động hóa và nhiều mục đích khác.
Cmdlets có dạng `Verb-Noun`. Ví dụ, lệnh `Get-ChildItem` có thể được sử dụng để liệt kê thư mục hiện tại. Cmdlets cũng nhận các Arguments hoặc Flags. Chúng ta có thể nhập `Get-ChildItem -` rồi nhấn phím Tab để lần lượt xem qua các argument có sẵn.
Một lệnh như:
```powershell
Get-ChildItem -Recurse
```
sẽ hiển thị nội dung của thư mục làm việc hiện tại cùng với tất cả các thư mục con bên trong nó.
![[Screenshot 2026-07-30 at 17.02.53.png|center|border|650]]
Một ví dụ khác:
```
Get-ChildItem -Path C:\Users\Administrator\Documents
```
dùng để lấy nội dung của một thư mục khác. Cuối cùng, chúng ta có thể kết hợp nhiều argument như sau để liệt kê đệ quy (Recursively) nội dung của tất cả các thư mục con bên trong một thư mục khác
### Aliases
Trong PowerShell, nhiều cmdlet cũng có các aliases. Ví dụ, bí danh của cmdlet `Set-Location` , được sử dụng để thay đổi thư mục, có thể là `cd` hoặc `sl`. Trong khi đó, các aliases của `Get-ChildItem` là `ls` và `gci`. Chúng ta có thể xem tất cả các bí danh hiện có bằng cách nhập `Get-Alias`.
![[Screenshot 2026-07-30 at 17.16.16.png|center|border|650]]
Chúng ta cũng có thể tự thiết lập bí danh bằng `New-Alias` và truy vấn bí danh của bất kỳ cmdlet nào bằng `Get-Alias -Name`.
![[Screenshot 2026-07-30 at 17.21.04.png|center|border|650]]
PowerShell cung cấp một hệ thống trợ giúp dành cho cmdlet, hàm, script và các khái niệm. Hệ thống này không được cài đặt sẵn theo mặc định. Tuy nhiên, chúng ta có thể chạy lệnh `Get-Help <cmdlet-name> -Online` để mở tài liệu trợ giúp trực tuyến của một cmdlet hoặc hàm trong trình duyệt web. Ngoài ra, có thể sử dụng Update-Help để tải xuống và cài đặt các tệp trợ giúp cục bộ.
Việc nhập một lệnh như Get-Help Get-AppPackage sẽ chỉ trả về thông tin trợ giúp rút gọn nếu các tệp Help chưa được cài đặt.
![[Screenshot 2026-07-30 at 17.21.52.png|center|border|650]]
Việc nhập một lệnh như Get-Help Get-AppPackage sẽ chỉ trả về thông tin trợ giúp rút gọn nếu các tệp Help chưa được cài đặt.
![[Screenshot 2026-07-30 at 17.23.18.png|center|border|650]]
### Running Scripts
PowerShell ISE (Integrated Scripting Environment – Môi trường Scripting Tích hợp) cho phép người dùng viết script PowerShell trực tiếp trong quá trình làm việc. Công cụ này cũng cung cấp chức năng tự động hoàn thành và tra cứu các lệnh PowerShell. PowerShell ISE cho phép chúng ta viết và thực thi script ngay trong cùng một console, qua đó hỗ trợ quá trình gỡ lỗi (debug) nhanh chóng.

Chúng ta có thể thực thi script PowerShell theo nhiều cách khác nhau. Nếu biết các hàm cần sử dụng, có thể chạy script trực tiếp trên máy cục bộ hoặc sau khi nạp script vào bộ nhớ thông qua một download cradle, như ví dụ dưới đây.
![[Screenshot 2026-07-30 at 17.28.19.png|center|border|650]]Một cách phổ biến để làm việc với script trong PowerShell là import (nạp) script để tất cả các hàm của nó trở nên khả dụng trong phiên console PowerShell hiện tại: `Import-Module .\PowerView.ps1`. Sau đó, chúng ta có thể bắt đầu sử dụng các lệnh và duyệt qua các tùy chọn, hoặc nhập `Get-Module` để liệt kê tất cả module đã được nạp cùng với các lệnh tương ứng.
```powershell
PS C:\htb> Get-Module | select Name,ExportedCommands | fl

Name             : Appx
ExportedCommands : {[Add-AppxPackage, Add-AppxPackage], [Add-AppxVolume, Add-AppxVolume], [Dismount-AppxVolume,
                   Dismount-AppxVolume], [Get-AppxDefaultVolume, Get-AppxDefaultVolume]...}

Name             : Microsoft.PowerShell.LocalAccounts
ExportedCommands : {[Add-LocalGroupMember, Add-LocalGroupMember], [Disable-LocalUser, Disable-LocalUser],
                   [Enable-LocalUser, Enable-LocalUser], [Get-LocalGroup, Get-LocalGroup]...}

Name             : Microsoft.PowerShell.Management
ExportedCommands : {[Add-Computer, Add-Computer], [Add-Content, Add-Content], [Checkpoint-Computer,
                   Checkpoint-Computer], [Clear-Content, Clear-Content]...}

Name             : Microsoft.PowerShell.Utility
ExportedCommands : {[Add-Member, Add-Member], [Add-Type, Add-Type], [Clear-Variable, Clear-Variable], [Compare-Object,
                   Compare-Object]...}

Name             : PSReadline
ExportedCommands : {[Get-PSReadLineKeyHandler, Get-PSReadLineKeyHandler], [Get-PSReadLineOption,
                   Get-PSReadLineOption], [Remove-PSReadLineKeyHandler, Remove-PSReadLineKeyHandler],
                   [Set-PSReadLineKeyHandler, Set-PSReadLineKeyHandler]...}
```
### Execution Policy
Đôi khi chúng ta sẽ gặp trường hợp không thể thực thi script trên một hệ thống. Nguyên nhân là do một tính năng bảo mật có tên execution policy (chính sách thực thi), được thiết kế nhằm hạn chế việc thực thi các script độc hại. Các chính sách có thể có gồm:

| Chính sách   | Mô tả                                                                                                                                                                                                                                                                                                                      |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AllSigned    | Tất cả script đều có thể được thực thi, nhưng script và các tệp cấu hình phải được nhà phát hành đáng tin cậy ký số. Áp dụng cho cả script từ xa và script cục bộ. Hệ thống sẽ hiển thị lời nhắc trước khi thực thi các script được ký bởi những nhà phát hành chưa được xác định là đáng tin cậy hoặc không đáng tin cậy. |
| Bypass       | Không có script hoặc tệp cấu hình nào bị chặn và người dùng không nhận được cảnh báo hay lời nhắc.                                                                                                                                                                                                                         |
| Default      | Thiết lập chính sách thực thi mặc định: Restricted đối với máy Windows dành cho máy trạm và RemoteSigned đối với máy chủ Windows.                                                                                                                                                                                          |
| RemoteSigned | Script có thể được thực thi, nhưng các script được tải xuống từ Internet phải có chữ ký số. Script được tạo cục bộ không bắt buộc phải có chữ ký số.                                                                                                                                                                       |
| Restricted   | Cho phép thực thi từng lệnh riêng lẻ nhưng không cho phép chạy script. Tất cả loại tệp script, bao gồm tệp cấu hình (.ps1xml), tệp script module (.psm1) và PowerShell profile (.ps1), đều bị chặn.                                                                                                                        |
| Undefined    | Không có execution policy nào được thiết lập cho phạm vi hiện tại. Nếu execution policy của TẤT CẢ các phạm vi đều được đặt thành Undefined, chính sách thực thi mặc định Restricted sẽ được sử dụng.                                                                                                                      |
| Unrestricted | Đây là chính sách thực thi mặc định đối với các máy tính không chạy Windows và không thể thay đổi. Chính sách này cho phép thực thi script chưa được ký, nhưng sẽ cảnh báo người dùng trước khi chạy các script không thuộc vùng mạng nội bộ (local intranet).                                                             |

```powershell
PS C:\htb> Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine    RemoteSigned
```
Execution policy không được thiết kế như một cơ chế kiểm soát bảo mật nhằm hạn chế hành động của người dùng. Người dùng có thể dễ dàng bypass (vượt qua) chính sách này bằng cách nhập trực tiếp nội dung script vào cửa sổ PowerShell, tải xuống và gọi thực thi script, hoặc chỉ định script dưới dạng một lệnh được mã hóa. Chính sách cũng có thể bị bypass bằng cách thay đổi execution policy (nếu người dùng có đủ quyền) hoặc thiết lập execution policy cho phạm vi của tiến trình hiện tại (process scope). Cách này có thể được thực hiện bởi hầu hết người dùng vì không yêu cầu thay đổi cấu hình hệ thống và chỉ có hiệu lực trong thời gian tồn tại của phiên làm việc.

Dưới đây là ví dụ về việc thay đổi execution policy cho tiến trình (phiên làm việc) hiện tại.
```powershell
PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): Y
```

Giờ đây, chúng ta có thể thấy rằng execution policy đã được thay đổi.

```powershell
PS C:\htb>  Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process          Bypass
  CurrentUser       Undefined
 LocalMachine    RemoteSigned
```
## Windows Management Instrumentation (WMI)
WMI là một phân hệ của PowerShell, cung cấp cho các quản trị hệ thống những công cụ mạnh mẽ để giám sát hệ thống. Mục tiêu của WMI là hợp nhất việc quản lý thiết bị và ứng dụng trên các mạng doanh nghiệp. VMI là một thành phần cốt lõi của hệ điều hành Windows và đã được tích hợp sẵn kể từ Windows 2000 bao gồm các thành phần sau:

| Tên thành phần     | Mô tả                                                                                                                                                                             |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WMI service        | Tiến trình Windows Management Instrumentation, tự động khởi chạy khi hệ thống khởi động và đóng vai trò trung gian giữa các WMI provider, WMI repository và các ứng dụng quản lý. |
| Managed objects    | Bất kỳ thành phần logic hoặc vật lý nào có thể được quản lý thông qua WMI.                                                                                                        |
| WMI providers      | Các đối tượng có chức năng giám sát sự kiện/ dư liệu liên quan đến một đối tượng cụ thể.                                                                                          |
| Classes            | Được các WMI provider sử dụng để truyền dữ liệu đến WMI service.                                                                                                                  |
| Methods            | Được gắn với các class và cho phép thực hiện thao tác. Ví dụ method có thể được sử dụng để khởi động/ dừng tiến trình trên các máy tính từ xa.                                    |
| WMI repository     | Một cơ sở dữ liệu lưu trữ toàn bộ dữ liệu tĩnh liên quan đến WMI.                                                                                                                 |
| CIM Object Manager | Thành phần tiếp nhận yêu cầu dữ liệu từ các WMI provider và trả dữ liệu đó về cho ứng dụng đã yêu cầu.                                                                            |
| WMI API            | Cho phép các ứng dụng truy cập vào hạ tầng WMI.                                                                                                                                   |
| WMI Consumer       | Gửi các truy vấn đến các đối tượng thông qua CIM Object Manager                                                                                                                   |
Một số ứng dụng của VMI bao gồm:
- Thu thập thông tin trạng thái của hệ thống cục bộ hoặc từ xa.
- Cấu hình các thiết lập bảo mật trên máy tính hoặc ứng dụng từ xa.
- Thiết lập và thay đổi quyền của người dùng và nhóm.
- Thiết lập hoặc sửa đổi các thuộc tính của hệ thống.
- Thực thi mã.
- Lập lịch thực thi tiến trình.
- Thiết lập cơ chế ghi nhật ký.
Tất cả các tác vụ. trên đều có thể được thực hiện thông qua sự kết hợp giữa PowerShell và Windows Managemetn Instrumentation Command-Line (WMIC).
WMI có thể được sử dụng thông qua Windows Command Prompt bằng cách nhập lệnh `WMIC` để mở một trình thông dịch lệnh, hoặc bằng cách thực thi trực tiếp các lệnh WMIC. Ví dụ `wmic computersystem get name`. Lệnh tên được sử dụng để lấy hostname của hệ thống. Chúng ta cũng có thể xem danh sách tất cả các lệnh và aliases được WMIC hỗ trợ bằng lệnh.
```powershell
WMIC /?
```
![[Screenshot 2026-07-31 at 15.05.09.png|center|border|650]]
Ví dụ dưới đây hiển thị thông tin về hệ điều hành.
![[Screenshot 2026-07-31 at 15.07.25.png|center|border|650]]
WMIC sử dụng các aliases cùng với các đông từ, trang từ và tham số chuyển đổi đi kèm.
Trong ví dụ trên:
- `list` được sư dụng để hiển thị dữ liệu.
- `brief` là một trạng từ dùng để hiển thị tập thuộc tính cốt lõi thay vì toàn bộ thông tin.
WMI cũng có thể được sử dụng thông qua PowerShell bằng module `Get-WmiObject`. Module này được dùng để truy xuất các instances của lớp WMI hoặc lấy thông tin ề các lớ WMI hiện có. Module này có thể làm việc với máy cục bộ cũng như máy từ xa.
![[Screenshot 2026-07-31 at 15.13.27.png|center|border|650]]
Chúng ta cũng có thể sử dụng module `Invoke-WmiMethod` được dùng để gọi các phương thức của các đối tượng WMI.
Ví dưới đây thực hiện thao tác rename một tệp. Có thể thấy lệnh đã được thực thi thành công vì giá trị `ReturnValue` bằng `0` biểu thị thao tác đã hoàn tất mà không xảy ra lỗi.
```markdown
PS C:\htb> Invoke-WmiMethod -Path "CIM_DataFile.Name='C:\users\public\spns.csv'" 
-Name Rename -ArgumentList "C:\Users\Public\kerberoasted_users.csv"

__GENUS          : 2
__CLASS          : __PARAMETERS
__SUPERCLASS     :
__DYNASTY        : __PARAMETERS
__RELPATH        :
__PROPERTY_COUNT : 1
__DERIVATION     : {}
__SERVER         :
__NAMESPACE      :
__PATH           :
ReturnValue      : 0
PSComputerName   :
```
## Microsoft Management Console (MMC)
Microsoft Management Console có thể được sử dụng để tập hợp các snap-in nhằm quản lý phần cứng, phần mềm, và các thành phần mạng trên một máy chủ chạy Windows.
MMC được giới thiệu từ Windows Server 2000 và được hỗ trợ trên tất cả các phiên bản Windows. Ngoài ra, MMC còn cho phép quản trị viên xây dựng các công cụ quản trị tuỳ chỉnh và phân phối chúng cho người dùng.
MMC hoạt động dựa trên khái niệm snap-in, cho phép quản trị viên tạo một bảng điều khiển được tuỳ chỉnh, chỉ bao gồm những công cụ cần thiết để quản lý nhiều dịch vụ khác nhau. Các snap-in này có thể được thêm vào nhằm quản lý hệ thống cục bộ cũng như hệ thống từ xa.
Chúng ta có thể Microsoft Management Console (MMC) bằng cách nhập lệnh `mmc` vào Start Menu.
Khi khởi động MMC lần đầu tiên, giao diện của nó sẽ trống và chưa chứa bất kỳ snap-in hay công cụ quản trị nào. Sau đó, quản trị viên có thể thêm các snap-in cần thiết để xây dựng một môi trường quản trị phù hợp với nhu cầu quản lý hệ thống.
![[Screenshot 2026-07-31 at 15.36.24.png|center|border|650]]
Sau đây, chúng ta có thể truy cập File -> Add or Remove Snap-ins để thêm hoặc bỏ các snap-in, từ đó bắt đầu tuỳ chỉnh bảng điểu khiển quản trị theo nhu cầu.
![[Screenshot 2026-07-31 at 15.38.25.png|center|border|650]]
Khi bắt đầu thêm các snap-in, hệ thống sẽ yêu cầu chúng ta lựa chọn phạm vi quản lý của snap-in đó: quản lý máy tính cục bộ (local computer) hay một máy tính khác trên mạng.
![[Screenshot 2026-07-31 at 15.40.40.png|center|border|650]]
Sau khi hoàn tất việc thêm các snap-in, chúng sẽ xuất hiện ở khung bên trái của cửa sổ MMC. Từ đây, chúng ta có thể lưu tập hợp các snap-in này dưới dạng tệp `.msc`, để toàn bộ các snap-in sẽ được tải tự động trong lần mở MMC tiếp theo.

Theo mặc định, các tệp `.msc` được lưu trong thư mục Windows Administrative Tools thuộc Start Menu. Trong những lần khởi động MMC sau, chúng ta có thể lựa chọn mở bất kỳ giao diện quản trị (view) nào mà mình đã tạo trước đó.
![[Windows Fundamentals-3.png|center|border|650]]
## Windows Subsystem for Linux (WSL)
Windows Subsystem for Linux (WSL) là một tính năng cho phép các tệp thực thi của Linux chạy trực tiếp trên Windows 10 và Windows Server 2019. 
Ban đầu, WSL được phát triển nhằm phục vụ các lập trình viên cần sử dụng Bash, Ruby và các công cụ dòng lệnh (command-line tools) của Linux như `sed`, `awk`, `grep`, v.v., ngay trên môi trường làm việc Windows.
Phiên bản thứ hai của WSL (WSL 2), được phát hành vào tháng 5 năm 2019, đã giới thiệu một nhân Linux (Linux kernel) thực thụ, hoạt động dựa trên một tập hợp các tính năng của Hyper-V.
WSL có thể được cài đặt bằng cách thực thi lệnh PowerShell dưới đây với quyền Administrator: `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`. Sau khi tính năng này được kích hoạt, chúng ta có thể tải về một bản phân phối (distribution) Linux từ Microsoft Store và tiến hành cài đặt, hoặc tải thủ công bản phân phối Linux mong muốn, sau đó giải nén và cài đặt thông qua dòng lệnh.
WSL cài đặt một ứng dụng có tên `Bash.exe`, có thể được khởi chạy chỉ bằng cách nhập lệnh `bash` trong cửa sổ dòng lệnh của Windows để mở Bash shell. Từ shell này, chúng ta sẽ có trải nghiệm tương tự như trên một hệ thống Linux thực, bao gồm cả cấu trúc thư mục chuẩn của Linux.
```powershell
PS C:\htb> ls /

bin dev home lib lLib64 media opt root sbin srv tmp var
boot etc init 1lib32 Libx32 mnt proc run Snap sys usr
```
Chúng ta có thể truy cập ổ đĩa C$ và các ổ đĩa khác của hệ điều hành Windows thông qua thư mục `/mnt`, giúp việc chuyển đổi giữa môi trường WSL và Windows diễn ra một cách liền mạch.
Sau khi truy cập vào Bash shell, chúng ta có thể tương tác với WSL tương tự như khi làm việc trên bất kỳ hệ điều hành Linux nào, chẳng hạn như:
- Thực thi các lệnh Linux.
- Cài đặt hoặc cập nhật các gói phần mềm (packages).
- Quản trị hệ thống và thực hiện các tác vụ thông thường khác.
```powershell
PS C:\htb> uname -a

Linux WS01 4.4.0-18362-Microsoft #476-Microsoft Fri Nov 01 16:53:00
PST 2019 x86_64 x86_64 x86_64 GNU/Linux
```

## Desktop Experience và Server Core
**Windows Server Core** lần đầu tiên được giới thiệu cùng với Windows Server 2008 như một môi trường máy chủ tối giản (minimalistic server environment), chỉ bao gồm các chức năng cốt lõi của Windows Server.
Nhờ đó, Server Core có yêu cầu quản trị thấp hơn, bề mặt tấn công (attack surface) nhỏ hơn, đồng thời tiêu thụ ít dung lượng lưu trữ và bộ nhớ hơn so với phiên bản Desktop Experience (GUI).
Trên Server Core, toàn bộ các tác vụ cấu hình và bảo trì hệ thống đều được thực hiện thông qua dòng lệnh (Command Prompt), PowerShell hoặc quản trị từ xa bằng Microsoft Management Console (MMC) hay Remote Server Administration Tools (RSAT).
Mặc dù Server Core hướng đến việc giảm thiểu kích thước hệ thống bằng cách loại bỏ giao diện đồ họa (GUI), một số chương trình đồ họa vẫn được hỗ trợ, bao gồm: Registry Editor, Notepad, System Information, Windows Installer, Task Manager và PowerShell.
Ngoài ra, Server Core cũng hỗ trợ một số công cụ thuộc bộ Sysinternals Suite, chẳng hạn như: Active Directory Explorer, Process Explorer, Process Monitor và TCPView.
Kể từ Windows Server 2019, người dùng phải lựa chọn Server Core hoặc Desktop Experience ngay trong quá trình cài đặt hệ điều hành và không thể chuyển đổi ngược lại sau khi cài đặt hoàn tất (ví dụ: chuyển từ Server Core sang Desktop Experience).
Sau khi cài đặt, quá trình cấu hình ban đầu của Server Core có thể được thực hiện thông qua `SConfig`, một giao diện dựa trên văn bản (text-based interface) được viết bằng VBScript và thực thi bằng WScript.
`SConfig` được sử dụng để thực hiện nhiều tác vụ quản trị phổ biến, bao gồm:
- Cấu hình mạng (Networking).
- Kiểm tra và cài đặt các bản cập nhật Windows.
- Quản lý tài khoản người dùng.
- Cấu hình quản trị từ xa (Remote Management).
- Kích hoạt Windows.
- Và nhiều tác vụ quản trị khác.
![[Windows Fundamentals-4.png|center|border|650]]
Một số ứng dụng máy chủ không thể chạy trên Server Core, bao gồm:
- Microsoft System Center Virtual Machine Manager 2019 (SCVMM)
- System Center Data Protection Manager 2019
- SharePoint Server 2019
- Project Server 2019
Tóm lại, Server Core có ưu điểm là nhẹ hơn, tiêu thụ ít tài nguyên hệ thống hơn và có bề mặt tấn công nhỏ hơn. Tuy nhiên, nó cũng có độ khó quản trị cao hơn và tồn tại một số hạn chế, chẳng hạn như không thể thực hiện một số tác vụ quản trị thông qua các chương trình GUI.
Trong thực tế, việc lựa chọn Desktop Experience hay Server Core cho một máy chủ cần dựa trên:
- Nhu cầu nghiệp vụ của tổ chức.
- Mục đích sử dụng của máy chủ.
- Trình độ và kinh nghiệm của đội ngũ quản trị viên chịu trách nhiệm vận hành hệ thống.
Bảng dưới đây trình bày một số ứng dụng phổ biến được hỗ trợ trên Server Core và Desktop Experience. Đây chỉ là danh sách các ứng dụng thông dụng và không phải danh sách đầy đủ.

| Ứng dụng                              | Server Core  | Desktop Experience |
| ------------------------------------- | ------------ | ------------------ |
| Command Prompt                        | Có sẵn       | Có sẵn             |
| Windows PowerShell / Microsoft .NET   | Có sẵn       | Có sẵn             |
| Regedit                               | Có sẵn       | Có sẵn             |
| Diskmgmt.msc                          | Không hỗ trợ | Có sẵn             |
| Server Manager                        | Không hỗ trợ | Có sẵn             |
| Mmc.exe                               | Không hỗ trợ | Có sẵn             |
| Eventvwr                              | Không hỗ trợ | Có sẵn             |
| Services.msc                          | Không hỗ trợ | Có sẵn             |
| Control Panel                         | Không hỗ trợ | Có sẵn             |
| Windows Explorer                      | Không hỗ trợ | Có sẵn             |
| Taskmgr                               | Có sẵn       | Có sẵn             |
| Internet Explorer hoặc Microsoft Edge | Không hỗ trợ | Có sẵn             |
| Remote Desktop Services               | Có sẵn       | Có sẵn             |
## Windows Security
Security là một chủ đề đặc biệt quan trọng trong hệ điều hành Windows. Các hệ thống Windows bao gồm nhiều thành phần hoạt động cùng nhau, tạo nên một bề mặt tấn công đáng kể. Do tích hợp sẵn nhiều ứng dụng, tín năng và nhiều lớp cấu hình khác nhau, Windows rất dễ bị cấu hình sai, khiến hệ thống vẫn có thể trở thành mục tiêu tấn công ngay ca khi đã được cập nhật đầy đủ các bản vá bảo mật.
Windows cũng tích hợp nhiều tính năng có thể bị lạm dụng và trong suốt quá trình phát triển đã tồn tại nhiều lỗ hổng nghiêm trọng. Điều này dẫn đến sự xuất hiện nhiều kỹ thuật khai thác từ xa và cục bộ có hiệu quả cao và được sử dụng rộng rã.
Trong những năm qua, Microsoft đã không ngừng cải thiện cơ chế bảo mật của Windows. Khi mức độ kết nối giữa các hệ thống ngày càng tăng và các tác nhân tấn công ngày càng tinh vi hơn, Microsoft cũng liên tục bổ sung các tính năng mới nhằm giúp quản trị viên hệ thống tăng cường bảo mật , đồng thời ngăn chặn và phát hiện các hành vi xâm nhập hoặc lạm dụng hệ thống.
Windows tuân theo một số nguyên tắc bảo mật để kiểm soát việc xác thực và phân quyền truy cập trong toàn bộ hệ thống. Các nguyên tắc này được áp dụng cho nhiều thực thể khác nhau như người dùng, máy tính trong mạng, luồng xử lý, tiến trình, mỗi thực thể chỉ được phép thực hiện những hành động mà nó đã được cấp quyền.
Mô hình bảo mật này được thiết kế nhằm giảm thiểu nguy cơ truy cập trái phép, từ đó khiến việc khai thác hệ thống của attacker hoặc phần mềm độc hại trở nên khó khăn hơn.
### Security Identifier (SID)
Mỗi security principal trong hệ thống được gán một Security Identifier duy nhất. Windows sẽ tự động tạo SID cho từng đối tượng.
Điều này có nghĩa là, ngay cả khi trên hệ thống tồn tại tài khoản người dùng có cùng tên hoặc cùng thông tin hiển thị, WIndows vẫn phân biệt chính xác hai toàn khoản này và các quyền tương ứng của chúng thông qua SID.
SID là một chuỗi giá trị có độ dài khác nhau và được lưu trữ trong cơ sở dữ liệu bảo mật của Windows. SID của người dùng sẽ được thêm vào Access Token của họ để xác định tất cả các hành động mà người dùng đó được uỷ quyền thực hiện.
Một SID bao gồm hai thành phần chính:
- Identifier Authority: Cơ quan định danh
- Relative Identifier (RID): định danh tương đối
Trong môi trường AD, SID còn bao gồm thêm Domain SID, dùng để định danh miền, mà đối tượng đó thuộc về.![[Screenshot 2026-08-03 at 09.52.41.png|center|border|650]]SID được cấu thành theo mẫu sau:
```powershell
(SID)-(revision level)-(identifier-authority)-(subauthority1)-(subauthority2)-(etc)
```
Hãy cùng phân tích từng thành phần của **SID**.

| Giá trị                          | Ý nghĩa              | Mô tả                                                                                                                                                                                       |
| -------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| S                                | SID                  | Xác định chuỗi này là một Security Identifier (SID)                                                                                                                                         |
| 1                                | Revision Level       | Phiên bản của định danh SID. Cho đến nay, giá trị này chưa từng thay đổi và luôn là 1                                                                                                       |
| 5                                | Identifier Authority | Một chuỗi 48-bit dùng để xác định cơ quan cấp phát đã tạo SID này                                                                                                                           |
| 21                               | Subauthority1        | Một giá trị biến đổi dùng để xác định mối quan hệ của người dùng hoặc nhóm với authority đã tạo SID. Giá trị này cho biết authority đã tạo tài khoản người dùng theo thứ tự nào.            |
| 2614195641-1726409526-3792725429 | Subauthority2        | Xác định máy tính hoặc domain đã tạo SID này.                                                                                                                                               |
| 1002                             | Subauthority3        | Relative Identifier (RID) dùng để phân biệt tài khoản này với các tài khoản khác. RID cũng cho biết người dùng là người dùng thông thường, Guest, Administrator hoặc thuộc một nhóm nào đó. |
## Security Accounts Manager (SAM) và Access Control Entries (ACE)
Security Accounts Manager (SAM) chịu trách nhiệm cấp quyền cho các đối tượng trong mạng nhằm thực thi các tiến trình hoặc các tác vụ cụ thể.
Các quyền truy cập này được quản lý thông qua Access Control Entries (ACE) nằm trong Access Control Lists (ACL). ACL chứa các ACE dùng để xác định người dùng, nhóm hoặc tiến trình nào được phép truy cập vào một tệp hoặc được phép thực thi một tiến trình.
Các quyền truy cập đối với một đối tượng có thể bảo vệ được xác định bởi Security Descriptor, trong đó ACL được chia thành hai loại:
- Discretionary Access Control List (DACL).
- System Access Control List (SACL).
Mỗi luồng và tiến trình được tạo hoặc khởi chạy bởi người dùng đều phải trải qua quá trình cấp quyền. Một thành phần quan trọng của quá trình này là Access Token, được Local Security Authority xác thực.
Ngoài SID, Access Token còn chứa nhiều thông tin liên quan bảo mật khác. Việc hiểu rõ các cơ chế này là một phần quan trọng trong quá trình học các sử dụng và vượt qua các cơ chế bảo mật của Windows trong giai đoạn leo thang đặc quyền.
## User Account Control (UAC)
User Account Control (UAC) là một tính năng bảo mật của Windows được thiết kế nhằm ngăn chặn phần mềm độc hại thực thi hoặc can thiệp vào các tiến trình có khả năng gây tổn hại đến máy tính hoặc dữ liệu hệ thống.
Trong UAC có một cơ chế gọi là Admin Approval Mode, được thiết kế để ngăn chặn việc cài đặt các phần mềm không mông muốn mà không có sự chấp thuận của quản trị viện, đồng thời ngăn các thay đồi ảnh hưởng đến toàn bộ hệ thống được thực hiện. Chắc hẳn bạn đã từng thấy hộp thoại xác nhận (consent prompt) khi cài đặt một phần mềm. Do quá trình cài đặt yêu cầu quyền quản trị (Administrator privileges), Windows sẽ hiển thị một cửa sổ yêu cầu người dùng xác nhận việc tiếp tục cài đặt. Đối với người dùng thông thường không có quyền quản trị, quá trình thực thi sẽ bị từ chối hoặc hệ thống sẽ yêu cầu nhập mật khẩu của tài khoản quản trị viên. Hộp thoại xác nhận này sẽ tạm dừng việc thực thi các script hoặc tệp thực thi (binary) mà phần mềm độc hại hoặc kẻ tấn công cố gắng chạy cho đến khi người dùng nhập mật khẩu hoặc xác nhận cho phép thực thi.
Để hiểu cách UAC hoạt động, trước tiên chúng ta cần tìm hiểu:
- Cấu trúc của UAC.
- Cơ chế hoạt động của UAC.
- Những điều kiện kích hoạt hộp thoại xác nhận (consent prompt).
Sơ đồ dưới đây (được điều chỉnh từ tài liệu nguồn) minh họa cách User Account Control (UAC) hoạt động.![[Windows Fundamentals-5.png|center|border|650]]
### Registry
Windows Register là một cơ sở dữ liệu phân cấp có vai trò đặc biệt quan trọng đối với hệ điều hành Windows. Registry lưu trữ các thết lập cấp thấp của hệ điều hành Windows cũng như của các ứng dụng lựu chọn sử dụng cơ chế lưu trữ này.
Registry được chia thành hai nhóm dữ liệu chính:
- Dữ liệu dành riêng cho máy tính (computer-specific data).
- Dữ liệu dành riêng cho người dùng (user-specific data).
Chúng ta có thể mở Registry Editor bằng cách nhập lệnh `regedit` tại Command Prompt, PowerShell hoặc trong thanh tìm kiếm của Windows.![[Screenshot 2026-08-03 at 10.49.51.png|center|border|650]]Cấu trúc dạng cây của Windows Registry bao gồm các khóa gốc (root keys). Bên trong mỗi root key là các khóa con (subkeys), và trong mỗi subkey sẽ chứa các giá trị (values), nơi lưu trữ dữ liệu cấu hình của hệ thống và ứng dụng.
Windows Registry hỗ trợ 11 kiểu giá trị (value types) khác nhau có thể được tạo và lưu trữ trong một subkey.

| Giá trị                 | Kểu dữ liệu                                                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| REG_BINARY              | Dữ liệu nhị phân (**binary data**) ở bất kỳ định dạng nào.                                                                                                                                                                                                                                                                                                                                                        |
| REG_DWORD               | Một số nguyên 32-bit                                                                                                                                                                                                                                                                                                                                                                                              |
| REG_DWORD_LITTLE_ENDIAN | Một số nguyên 32-bit theo định dạng little-endian. Windows được thiết kế để hoạt động trên các kiến trúc máy tính sử dụng little-endian, do đó kiểu giá trị này được định nghĩa là REG_DWORD trong các tệp tiêu đề (header files) của Windows.                                                                                                                                                                    |
| REG_DWORD_BIG_ENDIAN    | Một số nguyên 32-bit theo định dạng big-endian. Một số hệ điều hành UNIX hỗ trợ kiến trúc big-endian.                                                                                                                                                                                                                                                                                                             |
| REG_EXPAND_SZ           | Một chuỗi ký tự kết thúc bằng ký tự NULL (_null-terminated string_), chứa các tham chiếu chưa được mở rộng đến biến môi trường (environment variables), ví dụ: `%PATH%`. Chuỗi này có thể ở định dạng Unicode hoặc ANSI, tùy thuộc vào hàm được sử dụng. Để mở rộng các tham chiếu đến biến môi trường, sử dụng hàm `ExpandEnvironmentStrings`.                                                                   |
| REG_LINK                | Một chuỗi Unicode kết thúc bằng ký tự NULL, chứa đường dẫn đích (target path) của một liên kết tượng trưng (symbolic link) được tạo thông qua hàm `RegCreateKeyEx` với tùy chọn `REG_OPTION_CREATE_LINK`.                                                                                                                                                                                                         |
| REG_MULTI_SZ            | Một chuỗi gồm nhiều **chuỗi con kết thúc bằng ký tự NULL, và toàn bộ chuỗi được kết thúc bằng một chuỗi rỗng (`\0`). Ví dụ: `String1\0String2\0String3\0LastString\0\0`. Ký tự `\0` đầu tiên kết thúc chuỗi thứ nhất, ký tự tiếp theo kết thúc chuỗi thứ hai và cứ tiếp tục như vậy; ký tự `\0` cuối cùng kết thúc toàn bộ chuỗi. Lưu ý rằng ký tự kết thúc cuối cùng phải được tính vào tổng độ dài của chuỗi.** |
| REG_NONE                | Không xác định kiểu dữ liệu (No defined value type).                                                                                                                                                                                                                                                                                                                                                              |
| REG_QWORD               | Một số nguyên 64-bit.                                                                                                                                                                                                                                                                                                                                                                                             |
| REG_QWORD_LITTLE_ENDIAN | Một số nguyên 64-bit theo định dạng little-endian. Windows được thiết kế để hoạt động trên các kiến trúc little-endian, do đó kiểu dữ liệu này được định nghĩa là REG_QWORD trong các tệp tiêu đề của Windows.                                                                                                                                                                                                    |
| REG_SZ                  | Một chuỗi ký tự kết thúc bằng ký tự NULL (_null-terminated string_). Chuỗi này có thể ở định dạng Unicode hoặc ANSI, tùy thuộc vào hàm được sử dụng.                                                                                                                                                                                                                                                              |
Mỗi thư mục nằm dưới Computer được gọi là một khóa (key). Tất cả các khóa gốc (root keys) đều bắt đầu bằng tiền tố `HKEY`. Một khóa như `HKEY_LOCAL_MACHINE` được viết tắt là `HKLM`.
HKLM chứa toàn bộ các thiết lập liên quan đến hệ thống cục bộ (local system). Khóa gốc này bao gồm sáu khóa con (subkeys):
- SAM
- SECURITY
- SYSTEM
- SOFTWARE
- HARDWARE
- BCD
Các khóa con này đều được nạp vào bộ nhớ (loaded into memory) trong quá trình khởi động hệ thống (boot time), ngoại trừ HARDWARE, vì khóa này được tải động (dynamically loaded).
![[Windows Fundamentals-6.png|center|border|650]]Toàn bộ Windows Registry của hệ thống được lưu trữ trong nhiều tệp (registry hive files) trên hệ điều hành. Chúng ta có thể tìm thấy các tệp này trong thư mục:
![[Screenshot 2026-08-03 at 10.58.42.png|center|border|650]]
### Run và RunOnce Registry Keys
Ngoài ra, Windows còn có các registry hive, chứa một nhóm logic gồm các key (khóa), subkey (khóa con) và value (giá trị) nhằm hỗ trợ việc tải các phần mềm và tệp vào bộ nhớ khi hệ điều hành khởi động hoặc người dùng đăng nhập.
Các registry hive này rất hữu ích trong việc duy trì quyền truy cập  vào hệ thống. Chúng được gọi là các Run và RunOnce Registry Keys.
Windows Registry bao gồm bốn khóa sau:
```
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
```
Dưới đây là một ví dụ về khóa `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run` khi đã đăng nhập vào một hệ thống.
![[Screenshot 2026-08-03 at 11.02.23.png|center|border|650]]
