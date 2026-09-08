Cốt lõi của bất kỳ Windows Domain nào là Active Directory Domain Servers hay AD DS. Dịch vụ này đóng vai trò như một danh mục tập trung dùng để lưu trữ thông tin về tất cả đối tượng tồn tại trong mạng. Trong số nhiều loại đối tượng được AD hỗ trợ, có thể kể đến users, group, machines, printers, file sharing và nhiều đối tượng khác.
## Active Directory Infomation
### User
**User** là một trong những loại đối tượng phổ biến nhất trong AD. Người dùng thuộc nhóm các đối tượng security principals (thực thể bảo mật), nghĩa là chúng có thể được miền xác thực và có thể gán quyền đối với cá tài nguyên như tệp tin hoặc máy in. Có thể hiểu rằng một security principals là một đối tượng có khả năng thực hiện hành động lên các tài nguyên trong mạng.
User có thể được sử dụng đại diện cho hai loại thực thể
- **People**: người dùng thường đại diện cho các cá nhân trong tổ chức cần truy cập vào hệ thống mạng, chẳng hạn như nhân viên.
- **Service**: Người dùng cũng có thể được định nghĩa để phục vụ cho các dịch vụ như IIS hoặc MSSQL. Mỗi dịch vụ đều cần một tài khoản người dùng để vận hành. Tuy nhiên, tài khoản dịch vụ khác với người dùng thông thường vì chúng chỉ được cấp những đặc quyền cần thiết để chạy dịch vụ đó.
### Machine
**Machine** là một loại đối tượng khác trong AD. Mỗi machine khi tham gia vào miền AD sẽ được tạo một đối tượng máy tính tương ứng. Machine cũng được xem là security principals và được gián một tài khoản giống như người dùng thông thường. Tuy nhiên, tài khoản này thường có quyền hạn tương đối hạn chế trong chính miền đó.
Bản thân các tài khoản machine là quản trị viên cục bộ trên máy tính được gán tương ứng. Thông thường, các tài khoản này không được thiết kế để con người truy cập trực tiếp, ngoại trừ chính máy tính đó. Tuy nhiên, cũng giống như bất kỳ tài khoản nào khác, nếu có mật khẩu, bạn vẫn có thể sử dụng tài khoản đó để đăng nhập.
**Note:** Mật khẩu của tài khoản máy tính được tự động luân phiên thay đổi và thường bao gồm 120 ký tự ngẫu nhiên.
Việc nhận diện tài khoản machine tương đối dễ dàng vì chúng tuân theo một quy tắc đặt tên cụ thể. Tên tài khoản machine là tên của machine kèm theo ký hiệu đô la ở cuối. Ví dụ, một machine có tên DC01 sẽ có tài khoản máy tính tương ứng là DC01$.
### Security Groups
Nếu đã quen với Windows, bạn có thể định nghĩa các nhóm người dùng để gán quyền truy cập vào tệp tin hoặc các tài nguyên khác cho toàn bộ nhóm thay vì từng người dùng riêng lẻ. Cách làm này giúp việc quản trị trở nên hiệu quả hơn, vì khi thêm người dùng vào một nhóm đã tồn tại, người dùng đó sẽ tự động kế thừa toàn bộ quyền hạn của nhóm. Security Group cũng được xem là security principals, do đó chúng có thể được cấp quyền đối với các tài nguyên trong mạng.
Nhóm có thể bao gồm cả người dùng và máy tính làm thành viên. Khi cần thiết, nhóm cũng có thể chứa các nhóm khác.
Một số nhóm được tạo mặc định trong miền nhằm cung cấp các đặc quyền cụ thể cho người dùng. Dưới đây là một số nhóm quan trọng:

| Security Group     | Description                                                                                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Domain Admins      | Người dùng thuộc nhóm này có quyền quản trị trên toàn bộ miền. Theo mặc định, họ có thể quản trị bất kỳ máy tính nào trong miền, bao gồm cả các Domain Controllar.             |
| Server Operators   | Người dùng trong nhóm này có thể quản trị các Domain Controller. Tuy nhiên, họ không thể thay đổi thành viên của các nhóm quản trị.                                            |
| Backup Operators   | Người dùng trong nhóm này được phép truy cập bất kỳ tệp tin nào, bất kể quyền truy cập thông thường. Nhóm này thường được dùng để thực hiện sao lưu dữ liệu trên các máy tính. |
| Account Operators  | Người dùng trong nhóm này có thể tạo hoặc chỉnh sửa các tài khoản khác trong miền.                                                                                             |
| Domain Users       | Bao gồm tất cả các tài khoản người dùng hiện có trong miền                                                                                                                     |
| Domain Computers   | Bao gồm tất cả các máy tính hiện có trong miền                                                                                                                                 |
| Domain Controllers | Bao gồm tất cả các Domain Controller trong miền.                                                                                                                               |

## Active Directory Users and Computers
Để cấu hình người dùng, nhóm hoặc máy tính trong AD, chúng ta cần đăng nhập vào Domain Controller và mở Active Directory Users and Computers từ menu Start.
![[Screenshot 2026-06-21 at 19.24.52.png|center|border|600]]

Công cụ này sẽ mở ra một cửa sổ cho phép bạn quan sát cấu trúc phân cấp của người dùng, máy tính, và nhóm tồn tại trong miền. Các đối tượng này được tổ chức trong các Organizational Units hay OU, tức là các đối tượng dạng vùng chứa cho phép phân loại người dùng và máy tính. OU chủ yếu sử dụng định nghĩa các tập hợp người dùng có yêu cầu chính sách tương tự nhau. Ví dụ, nhân viên thuộc bộ phận Công nghệ thông tin. Cần lưu ý rằng một người dùng chỉ có thể thuộc về một OU tại một thời điểm.
![[Screenshot 2026-06-21 at 19.40.09.png|center|border|600]]
Khi kiểm tra máy của chúng ta, có thể thấy đã tồn tại một OU có tên THM, bên trong có năm OU con dành cho các bộ phân IT, Management, Marketing, R&D và Sales. Việc thiết kế OU mô phỏng theo cơ cấu tổ chức của doanh nghiệp là rất phổ biến, vì điều này giúp triển khai hiệu quả các chính sách nền tảng áp dụng cho toàn bộ phòng bạn. Tuy nhiên, mặc dù đây thường là mô hình được mong đợi, ta vẫn có thể định nghĩa OU theo cách tuỳ ý. Ta có thể nhấp chuột phải vào OU THM vầ tạo 1 OU mới bên dưới có tên là Student 
![[Screenshot 2026-06-21 at 19.45.10.png|center|border|600]]
Nếu mở bất kỳ OU nào, ta có thể thấy được các người dùng mà OU đó chứa và thực hiện các tác vụ cơ bản như tạo, xoá hoặc chỉnh sửa người dùng khi cần thiết. Ta cũng có thể đặt lại mật khẩu nếu cần, đây là một thao tác rất hữu ích đối với bộ phận hỗ trợ kỹ thuật.
![[Screenshot 2026-06-21 at 19.51.10.png|center|border|600]]
Ta có thể thấy ngoài OU THM, còn có các vùng chứ mặc định khác. Các vùng chứa này được Windows tự động tạo ra và có chức năng như sau:
- **Builtin**: Chứa các nhóm mặc định có sẵn cho bất kỳ máy Windows nào.
- **Computers**: Bất kỳ máy tính nào tham gia vào mạng sẽ được đưa vào đây theo mặc định. Ta có thể di chuyển chúng sang vị trí khác nếu cần.
- **Domain Controller**: OU mặc định chứa các Domain Controller trong mạng.
- **Users**: Chứa các người dùng và nhóm mặc định được áp dụng trong phạm vi toàn miền.
- **Managed Service Accounts**: Chứa các tài khoản được sử dụng bởi các dịch vụ trong miền Windows
### Security Groups vs OUs
**OU** hữu ích trong việc áp dụng chính sách cho người dùng và máy tính. Các chính sách này bao gồm những cấu hình cụ thể liên quan đến từng nhóm người dùng tuỳ theo vai trò của họ trong doanh nghiệp. Cần nhớ rằng một người dùng chỉ có thể là thành viên của một OU tại một thời điểm, vì việc áp dụng hai tập chính sách khác nhau cho cùng một người dùng thường không hợp lý.
Ngược lại, security groups được sử dụng để cấp quyền đối với tài nguyên. Ví dụ, ta sẽ sử dụng nhóm nếu muốn cho phép một số người dùng truy cập vào thư mục chia sẻ hoặc máy in mạng, một người dùng có thể thuộc nhiều nhóm khác nhau, điều này là cần thiết để cấp quyền truy cập đến nhiều loại tài nguyên khác nhau.
## Managing Users in AD
### Deleting extra OUs and users
Điều đầu tiên ta nên nhận thấy là trong cấu hình Active Directory hiện tại có một **OU phòng ban bổ sung** không xuất hiện trong sơ đồ tổ chức. Chúng ta được thông báo rằng phòng ban này đã bị đóng do cắt giảm ngân sách, vì vậy nó cần được xóa khỏi domain.
Nếu ta thử nhấp chuột phải và xóa OU đó, bạn sẽ nhận được lỗi. Theo mặc định, các OU được bảo vệ để tránh bị xóa nhầm. 
![[Screenshot 2026-07-12 at 19.57.48.png|center|border|600]]

Để xóa OU, chúng ta cần bật tùy chọn **Advanced Features** trong menu **View**. Thao tác này sẽ hiển thị thêm các vùng chứa cho phép vô hiệu hoá cơ chế bảo vệ chống xoá. Nhấp chuột phải vào OU, chọn properties, sau đó tuỳ chọn bảo vệ trong thẻ Object. Sau khi tắt cơ chế này, xoá OU, mọi người dùng, nhóm hoặc OU con bên dưới cũng sẽ bị xoá theo.

![[Screenshot 2026-07-12 at 20.08.22.png|center|border|600]]

![[Screenshot 2026-07-12 at 20.11.27.png|center|border|600]]
Sau khi xóa OU dư thừa, hãy đối chiếu danh sách người dùng của từng phòng ban với sơ đồ tổ chức và tạo mới hoặc xóa tài khoản khi cần thiết để đồng bộ với cơ cấu hiện hành.

### Delegation (Uỷ quyền quản trị)
Active Directory cho phép cấp một số người dùng quyền quản trị hạn chế trên các OU cụ thể. Cơ chế này, gọi là uỷ quyền (delegation), cho phép giao tiếp các tác vụ quản trị nhất định mà không cần đến đặc quyền của Domain Administrator.
Một trường hợp phổ biến là trao quyền cho bộ phận hỗ trợ CNTT đặt lại mật khẩu của người dùng thông thường. Theo sơ đồ tổ chức, Phillip phụ trách hỗ trợ CNTT, do đó, nên uỷ quyền cho anh ấy quyền đặt lại mật khẩu đối với các OU Sales, Marketing và Management.
Để uỷ quyền cho OU Sales, nhấp chuột phải vào OU đó chọn Delegate Control. Trong trình hướng dẫn, thêm người dùng Philip (có thể nhập "philip" và dùng Check Names để tránh sai sót)
![[Screenshot 2026-07-16 at 09.56.33.png|center|border|650]]

![[Screenshot 2026-07-16 at 09.58.27.png|center|border|650]]

![[Active Directory-1.png|center|border|550]]
Sau khi hoàn tất, Phillip sẽ có khả năng đặt lại mật khẩu cho người dùng thuộc bộ phận Sales. Bạn có thể lặp lại quy trình tương tự cho các OU khác nếu muốn.
Tiếp theo, đăng nhập bằng tài khoản của Phillip qua RDP để thử đặt lại mật khẩu của Sophie. Sử dụng tên đăng nhập `THM\phillip` với thông tin xác thực đã được cung cấp. Do Phillip không có đủ quyền để sử dụng công cụ Active Directory Users and Computers, việc đặt lại mật khẩu sẽ được thực hiện thông qua PowerShell:
![[Screenshot 2026-07-16 at 10.06.30.png|center|border|550]]

Sau đó, buộc Phillip đổi mật khẩu ở lần đăng nhập kế tiếp bằng lệnh:
![[Screenshot 2026-07-16 at 10.09.54.png|center|border|600]]

## Managing Computer in AD
Theo mặc định, mọi máy tính khi gia nhập Domain ngoại trừ Domain Controller (DC) sẽ được đặt vào container có tên Computers. Nếu kiểm tra trên Domain Controller, sẽ thấy một số thiết bị đã được lưu tại đây.
Trong container chứa computers, có thể quan sát thấy các servers, laptop và PCs/ workstations tương ứng với người dùng trong mạng.
![[Screenshot 2026-07-16 at 10.24.06.png|center|border|650]]
Tuy nhiên việc lưu trữ toàn bộ thiết bị trong cùng một container không phải là phương án tối ưu, bởi mỗi loại thiết bị thường yêu cầu những Group Policy (GPO) và cấu hình bảo mật khác nhau. Chẳng hạn, máy chủ cần các chính sách bảo mật nghiêm ngặt hơn máy trạm của người dùng.
Mặc dù Microsoft không quy định một mô hình tổ chức duy nhất cho Active Directory, nhưng cách tiếp cận phổ biến và hiệu quả là phân tách thiết bị theo mục đích sử dụng.
Thông thường, các thiết bị trong Domain sẽ được chia thành ít nhất ba nhóm sau:
**Workstations**
Máy trạm là loại thiết bị phổ biến nhất trong môi trường AD. Đây là những máy tính mà người dùng sử dụng để: đăng nhập vào Domain Logon, thực hiện công việc hàng ngày, truy cập Internet và các tài nguyên mạng. 
Lưu ý về bảo mật:
- Không nên để các tài khoản có đặc quyền cao, chẳng hạn Domain Administrator, đăng nhập trực tiếp trên các máy trạm.
- Máy trạm là mục tiêu tấn công phổ biến nhất của tin tặc thông qua phishing, malware hoặc ransomware. Nếu tài khoản quản trị đăng nhập trên các thiết bị này, thông tin xác thực có thể bị đánh cắp.
**Server**
Máy chủ là loại thiết bị phổ biến thứ hai trong môi trường AD. Chúng có nhiệm vụ cung cấp các dịch vụ cho người dùng, các máy chủ khác, các ứng dụng trong hệ thống. Ví dụ: File Server, Web Server, Database Server, DHCP Server, DNS Server, Application Server. Do vai trò quan trọng của chúng, máy chủ thường được áp dụng chính sách bảo mật và quản lý riêng biệt.
**Domain Controller**
Domain Controller là thành phần cốt lõi của hạ tầng AD. Chức năng của DC bao gồm: xác thực người dùng (authentication), quản lý Active Directory, lưu trữ cơ sở dữ liệu AD, cung cấp các dịch vụ như Kerberos, LDAP và DNS. Đây được xem là những thiết bị nhạy cảm nhất trong toàn bộ hệ thống, vì chúng lưu trữ: hash mật khẩu của tất cả tài khoản người dùng, thông tin về nhóm bảo mật của domain, cơ sở dữ liệu Active Directory (NTDS.dit). Nếu Domain Controller bị xâm nhập, gần như toàn bộ Domain sẽ bị kiểm soát. 
Để chuẩn hoá cấu trúc AD, chúng ta sẽ tạo thêm hai Organization Unit (OU) mới: Workstations và Server. Lưu ý không cần tạo OU cho Domain Controller vì Windows tự động tạo OU này trong quá trình cài đặt AD. Hai OU mới cần được tạo trực tiếp bên dưới tên miền `thm.local` , không nằm trong OU khác. 
![[Screenshot 2026-07-16 at 10.46.04.png|center|border|650]]
## Group Policies
Cho đến thời điểm này, chúng ta đã tổ chức Users và Computer vào các OU. Tuy nhiên, mục đích thực sự của việc phân chia này là để có thể triển khai các chính sách khác nhau cho từng OU một cách độc lập. Nhờ đó quản trị viên có thể áp dụng các cấu hình hệ thống và security baseline phù hợp với từng phòng ban hoặc từng nhóm thiết bị.
Windows quản lý các chính sách này thông qua Group Policy Objects (GPO).
**Group Policy Object** là một tập hợp các thiết lập được sử dụng để quản lý tập trung người dùng và máy tính trong môi trường AD.
![[Screenshot 2026-07-16 at 10.51.19.png|Center|border|550]]
Một GPO có thể chứa Computer Configuration áp dụng cho máy tính và được thực thi khi máy khởi động, và User Configuration áp dụng cho tài khoản người dùng và được thực thi khi người dùng đăng nhập. Nhờ GPO, quản trị viên có thể: thiết lập chính sách mật khẩu, cấu hình firewall, khoá control panel, tắt USB, cài đặt phần mềm, triển khai máy in, thiết lập Desktop, áp dụng các chính sách khác nhau không cần cấu hình thủ công trên từng máy.
![[Screenshot 2026-07-16 at 11.07.59.png|center|border|600]]
Để quản lý GPO, sử dụng công cụ Group Policy Management (GPMC), có thể mở từ Start Menu.
Khi khởi động GPMC, sẽ thấy toàn bộ cây OU của Domain AD. Quy trình triển khai GPO gồm hai bước:
1. Tạo một GPO trong mục Group Policy Objects.
2. Liên kết GPO đó với OU hoặc Domain mà muốn áp dụng.
Trong hệ thống hiện tại có ba GPO mặc định: Default Domain Policy, Default Domain Controllers Policy, và RDP Policy. Trong đó, Default Domain Policy được liên kết với toàn bộ miền `thm.local` , RDP Policy cũng áp dụng cho toàn bộ domain, Default Domain Controllers Policy chỉ áp dụng cho OU Domain Controllers.
Hãy cùng xem xét **Default Domain Policy** để tìm hiểu bên trong một **GPO** có những gì. Tab đầu tiên sẽ thấy khi chọn một GPO sẽ hiển thị Scope tức là vị trí mà GPO được liên kết trong AD. Đối với chính sách hiện tại, chúng ta có thể thấy rằng nó chỉ được liên kết với domain `thm.local`:
![[Screenshot 2026-07-16 at 11.28.52.png|center|border|600]]
Như ta có thể thấy, ta cũng có thể áp dụng Security Filtering cho các GPO để chúng chỉ được áp dụng cho những người dùng hoặc máy tính cụ thể nằm trong một OU. Theo mặc định, GPO sẽ được áp dụng cho nhóm Authenticated Users, bao gồm tất cả người dùng và máy tính đã được xác thực trong miền.
Tab **Settings** chứa toàn bộ nội dung thực tế của **GPO** và cho biết những cấu hình cụ thể mà GPO sẽ áp dụng. Như đã đề cập trước đó, mỗi **GPO** đều bao gồm các cấu hình chỉ áp dụng cho **Computer Configurations** và các cấu hình chỉ áp dụng cho **User Configurations**. Trong trường hợp này, **Default Domain Policy** chỉ chứa các **Computer Configurations**.
![[Screenshot 2026-07-16 at 11.30.02.png|center|border|600]]
Ta có thể tự do khám phá GPO và mở rộng các mục cấu hình có sẵn bằng cách nhấp vào liên kết "show" ở phía bên phải của từng cấu hình. Trong trường hợp này, Default Domain Policy chứa các cấu hình cơ bản nên được áp dụng cho hầu hết các miền (domains), bao gồm chính sách mật khẩu (Password Policy) và chính sách khóa tài khoản (Account Lockout Policy).
![[Screenshot 2026-07-16 at 11.34.40.png|center|border|600]]
Vì GPO này được áp dụng cho toàn bộ domain, nên bất kỳ thay đổi nào đối với nó cũng sẽ ảnh hưởng đến tất cả các máy tính trong domain. Có thể thay đổi chính sách độ dài tối thiểu của mật khẩu (Minimum Password Length Policy) để yêu cầu người dùng phải sử dụng mật khẩu có ít nhất 10 ký tự. Để thực hiện việc này, hãy nhấp chuột phải vào GPO và chọn Edit.
![[Screenshot 2026-07-16 at 11.37.18.png|center|border|600]]

Thao tác này sẽ mở ra một cửa sổ mới, tại đó bạn có thể điều hướng và chỉnh sửa tất cả các cấu hình hiện có. Để thay đổi Minimum Password Length, hãy truy cập Computer Configurations → Policies → Windows Settings → Security Settings → Account Policies → Password Policy và thay đổi giá trị của chính sách tương ứng.
![[Screenshot 2026-07-16 at 11.39.48.png|center|border|650]]
Như ta có thể thấy, có rất nhiều **chính sách** có thể được cấu hình trong một **GPO**. Việc giải thích chi tiết từng chính sách là điều không thể thực hiện trong phạm vi một bài học này, tuy nhiên bạn nên dành thời gian tự khám phá vì nhiều chính sách có ý nghĩa khá trực quan và dễ hiểu. Nếu cần tìm hiểu thêm về bất kỳ chính sách nào, bạn chỉ cần nhấp đúp vào chính sách đó và đọc nội dung trong thẻ **Explain**, nơi cung cấp mô tả chi tiết về chức năng và tác động của chính sách.
![[Screenshot 2026-07-16 at 11.42.01.png|center|border|400]]
### GPO Distribution
Các Group Policy Object (GPO) được phân phối trong mạng thông qua một thư mục chia sẻ mạng có tên `SYSVOL`, được lưu trữ trên Domain Controller. Theo thông lệ, tất cả người dùng và máy tính trong domain đều có quyền truy cập vào thư mục chia sẻ này qua mạng để đồng bộ (synchronize) các GPO của mình theo định kỳ. Theo mặc định, thư mục chia sẻ SYSVOL ánh xạ tới đường dẫn `C:\Windows\SYSVOL\sysvol\` trên mỗi Domain Controller trong hệ thống.
Sau khi thực hiện bất kỳ thay đổi nào đối với GPO, có thể mất tối đa 2 giờ để các máy tính trong miền tự động đồng bộ và áp dụng các chính sách mới. Nếu muốn buộc một máy tính cụ thể đồng bộ GPO ngay lập tức, bạn có thể chạy lệnh sau trên chính máy tính đó:
![[Screenshot 2026-07-16 at 11.56.22.png|center|border|600]]

