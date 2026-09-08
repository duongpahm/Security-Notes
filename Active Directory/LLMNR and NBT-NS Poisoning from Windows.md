LLMNR và NBT NS poisoning cũng có thể được thực hiện từ một Windows host. Trong phần này sẽ tìm hiểu về tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) và thử capture thêm một tập credentials khác.
## Inveigh - Overview
Nếu chúng ta sử dụng một Windows host làm attack box, client cung cấp cho chúng ta một Windows box để thực hiện testing, hoặc chúng ta giành được quyền truy cập vào một Windows với local admin thông qua một phương thức attack khác và muốn tiếp tục bằng cách mở rộng quyền truy cập, tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) có cách hoạt động tương tự Responder, nhưng được viết bằng PowerShell và C#. Inveigh có khả năng lắng nghe IPv4 và IPv6 cùng nhiều protocol khác, bao gồm `LLMNR`, `DNS`, `MDNS`, `NBNS`, `DHCPv6`, `ICMPv6`, `HTTP`, `HTTPS`, `SMB`, `LDAP`, `WebDAV`, và `ProxyAuth`. 
Chúng ta có thể bắt đầu với phiên bản PowerShell như sau, sau đó liệt kê tất cả các parameter có thể sử dụng. Có một [wiki](https://github.com/Kevin-Robertson/Inveigh/wiki/Parameters) liệt kê toàn bộ parameter và hướng dẫn cách sử dụng.
### Using Inveigh
![[Pasted image 20260906012049.png]]
Hãy bắt đầu Inveigh với LLMNR và NBNS spoofing, đồng thời hiển thị output trên console và ghi output vào một file. Chúng ta sẽ giữ nguyên các thiết lập mặc định còn lại, có thể xem tại [đây](https://github.com/Kevin-Robertson/Inveigh#parameter-help).
![[Pasted image 20260906012304.png]]

## C# Inveigh (InveighZero)
Phiên bản PowerShell của Inveigh là phiên bản ban đầu và hiện không còn được cập nhật. Tác giả của tool duy trì phiên bản C#, trong đó kết hợp mã C# PoC ban đầu cùng với phiên bản C# được port từ phần lớn mã nguồn của phiên bản PowerShell. Trước khi có thể sử dụng phiên bản C# của tool, chúng ta phải compile executable. Để tiết kiệm thời gian, lab đã cung cấp sẵn cả phiên bản PowerShell và executable đã được compile của tool trong thư mục `C:\Tools` trên target host. Tuy nhiên, việc tự mình thực hiện quá trình compile bằng Visual Studio vẫn rất đáng để thực hành và cũng là một best practice.
![[Pasted image 20260906012728.png]]
Như chúng ta có thể thấy, tool khởi động và hiển thị những option nào được bật theo mặc định cũng như những option nào không được bật. Các option có `[+]` là những option đang được enable, trong khi những option có `[ ]` phía trước là các option đang bị disable. Output trên console trong quá trình tool chạy cũng cho biết option nào đang bị disable và do đó không gửi response. Ví dụ bên trên cho thấy mDNS đang bị disable.

Chúng ta cũng thấy thông báo `Press ESC to enter/exit interactive console`, một chức năng rất hữu ích khi tool đang chạy. Console cung cấp quyền truy cập vào các credentials và hash đã capture, cho phép chúng ta stop Inveigh và thực hiện nhiều thao tác khác.
Chúng ta có thể nhấn phím `esc` để truy cập console trong khi Inveigh vẫn đang chạy.
```
<SNIP>

[+] [20:10:24] LLMNR(A) request [academy-ea-web0] from 172.16.5.125 [response sent]
[+] [20:10:24] LLMNR(A) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [response sent]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[.] [20:10:24] TCP(1433) SYN packet from 172.16.5.125:61310
[.] [20:10:24] TCP(1433) SYN packet from 172.16.5.125:61311
C(0:0) NTLMv1(0:0) NTLMv2(3:9)> HELP
```
Sau khi nhập `HELP` và nhấn Enter, chúng ta sẽ được hiển thị một số option:
```
=============================================== Inveigh Console Commands ===============================================

Command                           Description
========================================================================================================================
GET CONSOLE                     | get queued console output
GET DHCPv6Leases                | get DHCPv6 assigned IPv6 addresses
GET LOG                         | get log entries; add search string to filter results
GET NTLMV1                      | get captured NTLMv1 hashes; add search string to filter results
GET NTLMV2                      | get captured NTLMv2 hashes; add search string to filter results
GET NTLMV1UNIQUE                | get one captured NTLMv1 hash per user; add search string to filter results
GET NTLMV2UNIQUE                | get one captured NTLMv2 hash per user; add search string to filter results
GET NTLMV1USERNAMES             | get usernames and source IPs/hostnames for captured NTLMv1 hashes
GET NTLMV2USERNAMES             | get usernames and source IPs/hostnames for captured NTLMv2 hashes
GET CLEARTEXT                   | get captured cleartext credentials
GET CLEARTEXTUNIQUE             | get unique captured cleartext credentials
GET REPLYTODOMAINS               | get ReplyToDomains parameter startup values
GET REPLYTOHOSTS                | get ReplyToHosts parameter startup values
GET REPLYTOIPS                  | get ReplyToIPs parameter startup values
GET REPLYTOMACS                 | get ReplyToMACs parameter startup values
GET IGNOREDOMAINS               | get IgnoreDomains parameter startup values
GET IGNOREHOSTS                 | get IgnoreHosts parameter startup values
GET IGNOREIPS                   | get IgnoreIPs parameter startup values
GET IGNOREMACS                  | get IgnoreMACs parameter startup values
SET CONSOLE                     | set Console parameter value
HISTORY                         | get command history
RESUME                          | resume real time console output
STOP                            | stop Inveigh
```
Chúng ta có thể nhanh chóng xem các hash đã capture và loại bỏ những hash trùng user bằng cách nhập `GET NTLMV2UNIQUE`.
![[Pasted image 20260906013120.png]]
Chúng ta có thể nhập `GET NTLMV2USERNAMES` để xem những username nào đã được thu thập. Điều này rất hữu ích nếu chúng ta muốn có một danh sách user để thực hiện thêm enumeration và xác định những account nào đáng để thử crack offline bằng Hashcat.
![[Pasted image 20260906013246.png]]

![[Pasted image 20260906013424.png]]
![[Pasted image 20260906013446.png]]

## Remediation
MITRE ATT&CK liệt kê kỹ thuật này với mã [ID: T1557.001](https://attack.mitre.org/techniques/T1557/001), `Adversary in the Middle: LLMNR/NBT NS Poisoning and SMB Relay`.
Có một số phương pháp để giảm thiểu nguy cơ từ attack này. Để đảm bảo các spoofing attack này không thể xảy ra, chúng ta có thể disable LLMNR và NBT NS. Tuy nhiên, cần lưu ý rằng với một thay đổi đáng kể đối với environment như vậy, luôn nên tiến hành testing từng bước và cẩn thận trước khi triển khai trên toàn bộ hệ thống. Với vai trò penetration tester, chúng ta có thể đề xuất các remediation step này, nhưng cần trao đổi rõ với client rằng họ nên kiểm thử kỹ các thay đổi để đảm bảo việc disable cả hai protocol không làm ảnh hưởng đến hoạt động của network.
Chúng ta có thể disable LLMNR trong Group Policy bằng cách truy cập:
`Computer Configuration --> Administrative Templates --> Network --> DNS Client`
![[Pasted image 20260906013757.png]]sau đó enable `"Turn OFF Multicast Name Resolution."`
![[Pasted image 20260906013901.png]]
NBT NS không thể được disable thông qua Group Policy mà phải được disable locally trên từng host. Chúng ta có thể thực hiện việc này bằng cách mở `Network and Sharing Center` trong `Control Panel`, chọn `Change adapter settings`, nhấp chuột phải vào adapter để mở properties, chọn `Internet Protocol Version 4 (TCP/IPv4)`, sau đó nhấn `Properties`. Tiếp theo chọn `Advanced`, mở tab `WINS` và cuối cùng chọn `Disable NetBIOS over TCP/IP`.
![[Pasted image 20260906014207.png]]

![[Pasted image 20260906014350.png]]

Mặc dù không thể disable NBT NS trực tiếp thông qua GPO, chúng ta có thể tạo một PowerShell script trong:`Computer Configuration --> Windows Settings --> Script (Startup/Shutdown) --> Startup`
với nội dung tương tự như sau:
```powershell
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose}
```

![[Pasted image 20260906014533.png]]
Để triển khai thay đổi này trên toàn bộ host trong domain, chúng ta có thể tạo một GPO bằng `Group Policy Management` trên Domain Controller, lưu script trong thư mục scripts của SYSVOL share và gọi script thông qua UNC path, chẳng hạn như:
![[Pasted image 20260906014703.png]]
Sau khi GPO được áp dụng cho các OU cụ thể và những host đó được restart, script sẽ được thực thi trong lần reboot tiếp theo và disable NBT NS, với điều kiện script vẫn tồn tại trên SYSVOL share và host có thể truy cập nó thông qua network.
Các biện pháp mitigation khác bao gồm filtering network traffic để block LLMNR/NetBIOS traffic và enable SMB Signing nhằm ngăn chặn NTLM relay attack. Network intrusion detection và prevention system cũng có thể được sử dụng để giảm thiểu hoạt động này. Ngoài ra, network segmentation có thể được sử dụng để cô lập những host cần LLMNR hoặc NetBIOS được enable nhằm hoạt động chính xác.