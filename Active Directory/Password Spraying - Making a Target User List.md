Để thực hiện thành công một password spraying attack, trước tiên chúng ta cần có danh sách những domain user hợp lệ để thử authentication. Có một số phương pháp để thu thập danh sách các user mục tiêu hợp lệ:
- Tận dụng SMB NULL session để lấy danh sách đầy đủ domain user từ Domain Controller.
- Sử dụng LDAP anonymous bind để thực hiện truy vấn LDAP một cách anonymous và lấy danh sách domain user.
- Sử dụng tool như `Kerbrute` để xác thực user bằng cách sử dụng wordlist từ nguồn GitHub repo [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames), hoặc thu thập thông tin bằng tool như [linkedin2username](https://github.com/initstring/linkedin2username) để tạo danh sách các user có khả năng tồn tại.
- Sử dụng một tập hợp credentials từ Linux hoặc Windows attack system, được client cung cấp hoặc thu được bằng các phương thức khác, chẳng hạn như LLMNR/NBT-NS response poisoning sử dụng `Responder`, hoặc thậm chí từ một password spray thành công với một wordlist nhỏ hơn.
Bất kể chọn phương pháp nào, chúng ta cũng cần đặc biệt quan tâm đến domain password policy. Nếu có SMB NULL session, LDAP anonymous bind hoặc một tập hợp credential hợp lệ, chúng ta có thể enumerate password policy.
Có password policy trong tay rất hữu ích vì minimum password length và việc password complexity có được enable hay không sẽ giúp chúng ta xây dựng danh sách password để sử dụng trong các lần password spray.
Việc biết account lockout threshold và bad password timer sẽ cho chúng ta biết có thể thực hiện bao nhiêu lần spray trong một khoản thời gian mà không lockout account cũng như cần chờ bao nhiêu phút giữa các lần password spray.
Một lần nữa, nếu không biết password policy, chúng ta luôn có thể yêu cầu client cung cấp thông tin này. Nếu client không cung cấp, chúng ta có thể thực hiện một password spraying attempt có mục tiêu rất cụ thể như một "hail mary" nếu tất cả các phương án khác để có foothold đều đã cạn kiệt.
Ngoài ra, chúng ta cũng có thể thử một lần spray sau mỗi vài giờ nhằm hạn chế khả năng lockout account. Bất kể lựa chọn phương pháp nào, và dù có password policy hay không, chúng ta luôn phải ghi lại toàn bộ hoạt động, bao gồm nhưng không giới hạn ở:
- Các account được nhắm mục tiêu.
- Domain Controller được sử dụng trong attack.
- Thời gian thực hiện spray.
- Ngày thực hiện spray.
- Password đã thử.
Điều này giúp chúng ta đảm bảo không thực hiện trùng lặp các hoạt động. Nếu xảy ra account lockout hoặc client phát hiện các authentication attempt đáng ngờ, chúng ta có thể cung cấp các ghi chú của mình để họ đối chiếu với hệ thống logging và xác nhận rằng không có hoạt động độc hại nào đang diễn ra trong network.
## SMB NULL Session to Pull User List
Nếu đang ở trong một internal machine nhưng không có valid domain credentials, chúng ta có thể kiểm tra SMB NULL session hoặc LDAP anonymous bind trên Domain Controller.
Cả hai phương pháp đều cho phép ta thu thập danh sách chính xác của tất cả user trong AD cũng như password policy. Nếu đã có credentials của một domain user hoặc quyền `SYSTEM` trên Windows hosts, chúng ta có thể dễ dàng query AD để lấy những thông tin này. Có thể thực hiện việc này bằng account `SYSTEM` vì account này có khả năng impersonate computer. Computer object được xem như một domain user account, mặc dù có một số khác biệt, chẳng hạn như authentication thông qua forest trust.
Nếu không có valid domain account, đồng thời SMB NULL session và LDAP anonymous bind đều không khả dụng, chúng ta có thể xây dựng user list bằng các nguồn bên ngoài như email harvesting và LinkedIn. Danh sách user này sẽ không đầy đủ bằng những phương pháp trên, nhưng vẫn có thể đủ để giúp chúng ta giành được quyền truy cập vào Active Directory. 
Một số tool có khả năng tận dụng SMB NULL session và LDAP anonymous bind bao gồm [enum4linux](https://github.com/portcullislabs/enum4linux), [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) và [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec), cùng nhiều tool khác. Bất kể sử dụng tool nào, chúng ta sẽ cần thực hiện một chút filtering để làm sạch output và thu được danh sách chỉ chứa username, mỗi username nằm trên một dòng.
Với `enum4linux`, chúng ta có thể thực hiện điều này bằng flag `-U`.
![[Pasted image 20260906071526.png]]Chúng ta có thể sử dụng command `enumdomusers` sau khi kết nối anonymous bằng `rpcclient`.
![[Pasted image 20260906071626.png]]
Cuối cùng, chúng ta có thể sử dụng `CrackMapExec` với flag `--users`. Đây là một tool hữu ích vì ngoài việc liệt kê user, nó còn hiển thị `badpwdcount`, tức số lần login không hợp lệ. Nhờ đó, chúng ta có thể loại bỏ khỏi danh sách những account đang tiến gần đến lockout threshold. 
Tool cũng hiển thị `baddpwdtime`, tức thời điểm của lần password attempt không hợp lệ gần nhất. Thông tin này giúp chúng ta xác định một account còn bao lâu nữa trước khi `badpwdcount` được reset.
Trong môi trường có nhiều Domain Controller, giá trị này được duy trì riêng biệt trên từng Domain Controller.
Để có được tổng số lần password sai chính xác của một account, chúng ta phải query từng Domain Controller rồi tính tổng các giá trị, hoặc query Domain Controller đang giữ **PDC Emulator FSMO role**.
![[Pasted image 20260906071749.png]]
## Gathering Users with LDAP Anonymous
Chúng ta có thể sử dụng nhiều tool khác nhau để thu thập user khi phát hiện **LDAP anonymous bind**. Một số ví dụ bao gồm [windapsearch](https://github.com/ropnop/windapsearch) và [ldapsearch](https://linux.die.net/man/1/ldapsearch). Nếu lựa chọn sử dụng `ldapsearch`, chúng ta cần chỉ định một **LDAP search filter** hợp lệ. Có thể tìm hiểu thêm về các search filter này trong module [Active Directory LDAP](https://academy.hackthebox.com/course/preview/active-directory-ldap).
![[Pasted image 20260906071859.png]]
Các tool như `windapsearch` giúp quá trình này đơn giản hơn, mặc dù chúng ta vẫn nên hiểu cách tự xây dựng LDAP search filter.

Ở đây, chúng ta có thể chỉ định anonymous access bằng cách cung cấp username rỗng với flag `-u`, đồng thời sử dụng flag `-U` để yêu cầu tool chỉ lấy thông tin về user.
```powershell
w3bis2bi@htb[/htb]$ ./windapsearch.py --dc-ip 172.16.5.5 -u "" -U

[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+] Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+] ...success! Binded as: 
[+]  None

[+] Enumerating all AD users
[+] Found 2906 users: 

cn: Guest

cn: Htb Student
userPrincipalName: htb-student@inlanefreight.local

cn: Annie Vazquez
userPrincipalName: avazquez@inlanefreight.local

cn: Paul Falcon
userPrincipalName: pfalcon@inlanefreight.local

cn: Fae Anthony
userPrincipalName: fanthony@inlanefreight.local

cn: Walter Dillard
userPrincipalName: wdillard@inlanefreight.local

<SNIP>
```
## Enumerating Users with Kerbrute
Nếu chúng ta hoàn toàn không có quyền truy cập từ vị trí của mình trong internal network, chúng ta có thể sử dụng `Kerbrute` để enumerate các AD account hợp lệ và thực hiện password spraying.
Tool này sử dụng Kerberos Pre Authentication, đây là một phương thức nhanh hơn và có khả năng stealthier hơn để thực hiện password spraying. Phương thức này không tạo ra Windows event ID [4625: An account failed to log on](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625), tức logon failure thường được hệ thống giám sát.
Tool gửi các TGT request đến Domain Controller mà không có Kerberos Pre Authentication để thực hiện username enumeration. Nếu KDC phản hồi với lỗi `PRINCIPAL UNKNOWN`, username đó không hợp lệ. Khi KDC yêu cầu Kerberos Pre Authentication, điều này cho biết username tồn tại và tool sẽ đánh dấu username đó là hợp lệ. Phương thức username enumeration này không tạo ra logon failure và không làm account bị lockout. 
Tuy nhiên, sau khi đã có danh sách user hợp lệ và chuyển sang sử dụng tool này để password spraying, các Kerberos Pre Authentication attempt thất bại sẽ được tính vào số lần login thất bại của account và có thể dẫn đến account lockout. Do đó, chúng ta vẫn phải thận trọng bất kể lựa chọn phương thức nào. Hãy thử phương pháp này bằng wordlist [jsmith.txt](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt), chứa 48.705 username phổ biến có thể có theo format `flast`. 
GitHub repo [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames) là một resource rất hữu ích cho dạng attack này và chứa nhiều username list khác nhau mà chúng ta có thể sử dụng để enumerate username hợp lệ bằng `Kerbrute`.
![[Pasted image 20260906072432.png]]
Chúng ta đã kiểm tra hơn 48.000 username chỉ trong hơn 12 giây và phát hiện hơn 50 username hợp lệ. Sử dụng `Kerbrute` để username enumeration sẽ tạo ra event ID [4768: A Kerberos authentication ticket (TGT) was requested](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768). Event này chỉ được trigger nếu [Kerberos event logging](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-kerberos-event-logging) được enable thông qua Group Policy. Defender có thể cấu hình SIEM để phát hiện sự gia tăng bất thường của event ID này, đây có thể là dấu hiệu của một attack. 
Nếu không thể tạo được một username list hợp lệ bằng bất kỳ phương pháp nào đã đề cập ở trên, chúng ta có thể quay lại external information gathering và tìm kiếm company email address hoặc sử dụng tool như [linkedin2username](https://github.com/initstring/linkedin2username) để kết hợp các username có khả năng tồn tại từ LinkedIn page của công ty.
## Credentialed Enumeration to Build our User List
Khi có valid credentials, chúng ta có thể sử dụng bất kỳ tool nào đã đề cập trước đó để xây dựng user list.
Một phương pháp nhanh và đơn giản là sử dụng CrackMapExec.
![[Pasted image 20260906072730.png]]