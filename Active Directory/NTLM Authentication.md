## LM Hash
LAN Manager (LM/LANMAN) hash là cơ chế lưu trữ mật khẩu cũ nhất của Windows, ra mắt từ 1987 trên hệ điều hành OS/2. Hashd được lưu trữ trong database SAM trên một máy lẻ, hoặc trong `NTDS.DIT` trên Domain Controller. Do các lỗi hổng bảo mật nghiêm trọng trong thuật toán băm, LM đã bị tắt mặc định kể từ Windows Vista/Server 2008.
Cơ chế hoạt động:
1. Mật khẩu tối đa 14 ký tự, không phân biệt hoa/ thường, toàn bộ được chuyển thành chữ in hoa trước khi băm.
2. Nếu mật khẩu ngắn hơn 14 ký tự, nó được đệm thêm ký tự `NULL` cho đủ độ dài.
3. Chuỗi 14 ký tự được cắt thành 2 đoạn 7 ký tự.
4. Mỗi khóa  DES được dùng để mã hóa một chuỗi cố định, tạo ra một khối ciphertext 8 byte.
5. Hai khối 8 byte này được ghép lại thành LM hash hoàn chỉnh, tổng cộng 16 byte.
Vì sao đây là thuật toán yếu:
- VÌ chuyển hết thành chữ hoa, keyspace khả dụng chỉ còn 69 ký tự, thu hẹp đáng kể không gian tìm kiếm so với mật khẩu gốc có phân biệt hoa/ thường.
- Vì bị bẻ đôi thành 7+7 thay vì xử lý nguyên khối 14 ký tự, attacker chỉ cần bruteforce 7 ký tự hai lần thay vì 14 ký tự 1 lần giúp nhẹ hơn rất nhiều cho một hệ thống có một hoặc nhiều GPU.
- Nếu mật khẩu thực tế chỉ có 7 ký tự hoặc ít hơn, nửa sau của LM hash luôn luôn là một giá trị cố định, có thể nhận ngay ra bằng mắt thường mà không cần dùng `Hashcat`.
- Việc sử dụng LM hash có thể bị vô hiệu hóa hoàn toàn thông qua Group Policy.
## NT Hash (NTLM hash)
NT LAN Manager (NTLM) hash thường gọi tắt là NT hash là cơ chế lưu trữ mật khẩu được dùng trên các hệ thống Windows hiện đại. Đây là một giao thức challenge-response, xác thực bằng cách trao đổi ba loại message: client gửi một `NEGOTIATE MESSAGE` tới server, server phản hồi bằng một `CHALLENGE MESSAGE` để xác minh danh tính client, và cuối cùng client phản hồi bằng `AUTHENTICATE MESSAGE`. Các hash này được lưu cục bộ trong database SAM, hoặc trong `NTDS.DIT` trên Domain Controller.
Giao thức có hai loại giá trị hash mật khẩu để lựa chọn khi xác thực: LM hash và NT hash (chính là hash MD4 của giá trị mật khẩu dạng little-endian UTF-16).
![[Pasted image 20260906001039.png]]
```text
NT hash = MD4( UTF-16-LE( password ) )
```
Nghĩa là băm MD4 trực tiếp trên toàn bộ chuỗi mật khẩu đã chuyển sang (UTF-16 little-endian) không cắt khúc như LM hash.
**Điểm mạnh và điểm yếu**:
- Vì hỗ trợ toàn bộ bảng ký tự Unicode, NT hash mạnh hơn đáng kể so với LM hash.
- Tuy vậy, chúng vẫn có thể bị bruteforce tương đối nhanh bằng các công cụ như Hashcat: các cuộc tấn công dùng GPU đã cho thấy toàn bộ keyspace NTLM 8 ký tự có thể bị brute-force trong chưa đầy 3 giờ. Ngay cả mật khẩu dài (15+ ký tự) cũng có thể bị crack bằng tấn công từ điển offline kết hợp với các rule biến đổi.
- NTLM cũng dễ bị tấn công pass-the-hash: kẻ tấn công chỉ cần có được NT hash là có thể xác thực vào hệ thống mà tài khoản đó có quyền local admin mà không cần biết giá trị cleartext của mật khẩu.
Cấu trúc một dòng NTLM hash đầy đủ (dạng dump từ SAM/NTDS):
```text
Rachel:500:aad3c435b514a4eeaad3b935b51304fe:e46b9e548fa0d122de7f59fb6d48eaa2:::
```
Trong đó:
- `Rachel`: username.
- `500`: Relative Identifier (RID); RID 500 là RID đã biết trước, luôn gắn với tài khoản administrator.
- `aad3c435b514a4eeaad3b935b51304fe` là LM hash; nếu LM hash bị tắt trên hệ thống, giá trị này không dùng được vào việc gì.
- `e46b9e548fa0d122de7f59fb6d48eaa2`: **NT hash**; hash này có thể crack offline để tìm ra cleartext (tùy độ dài/độ mạnh mật khẩu), hoặc dùng trực tiếp cho tấn công pass-the-hash.
## NTLMv1 (Net-NTLMv1)
Giao thức NTLM thực hiện challenge/response giữa server và client bằng NT hash. NTLMv1 dùng cả LM hash lần NT hash, điều này khiến nó dễ bị crack offline hơn sau khi bắt được hash bằng công cụ `Responder`, hoặc thông một cuộc tấn công NTLM relay. Giao thức này dùng cho xác thực mạng, và bản thân Net-NTLMv1 hash được tạo ra từ một thuật toán challenge/response: server gửi cho client một số ngẫu nhiên 8 byte (challenge), và client trả về một response 24 byte. Điểm qua trọng là các hash này khoogn thể dùng tấn công pass-the-hash khác hẳn với NT hash.
Cơ chế (thuật toán Challenge & Response của v1):
```
C = 8-byte server challenge, ngẫu nhiên
K1 | K2 | K3 = LM-hash hoặc NT-hash | 5-byte-0 (đệm thêm)
response = DES(K1,C) | DES(K2,C) | DES(K3,C)
```
Hash mật khẩu (LM hoặc NT) được chia nhỏ thành ba khóa DES (K1, K2, K3), mỗi khóa mã hóa challenge C, ba kết quả được ghép lại thành một response 24 byte.
Dạng mẫu một NTLMv1 hash đầy đủ:
```
u4-netntlm::kNS:338d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41:cb8086049ec4736c
```
NTLMv1 là nền tảng cho xác thực NTLM hiện đại. Giống như bất kỳ giao thức nào, nó có những điểm yếu và dễ bị crack cũng như các tấn công khác. đây chính là lý do NTLMv2 ra đời để cải tiến trên nền tảng đó.
## NTLMv2 (Net-NTLMv2)
Giao thức NTLMv2 lần đầu được giới thiệu trong Windows NT 4.0 SP4, được tạo ra như một phiên bản thay thế mạnh hơn NTLMv1, và đã là mặc định trong Windows kể từ Server 2000. NTLMv2 được gia cố để chống lại một số kiểu tấn công spoofing mà NTLMv1 để mắc phải. NTLMv2 gửi hai response cho challenge 8 byte nhận từ server. Các response này chưa một hash HMAC-MD5 16 byte của challenge, một challenge được sinh ngẫu nhiên từ phía client, và một hash HMAC-MD5 của thông tin đăng nhập của người dùng. Một response thứ hai được gửi kèm, sử dụng một client challenge có độ dài biến đổi bao gồm cả thời gian hiện tại, một giá trị ngẫu nhiên 8 byte và tên domain.
Cơ chế (thuật toán Challenge & Response của v2):
```
SC = 8-byte server challenge, ngẫu nhiên 
CC = 8-byte client challenge, ngẫu nhiên 
CC* = (X, time, CC2, domain name) 
v2-Hash = HMAC-MD5(NT-Hash, user name, domain name) 
LMv2 = HMAC-MD5(v2-Hash, SC, CC) 
NTv2 = HMAC-MD5(v2-Hash, SC, CC*) 
response = LMv2 | CC | NTv2 | CC*
```
So với NTLMv1, các dev đã cải tiến bằng cách làm cho NTLMv2 khó crack hơn và trang bị một thuật toán mạnh mẽ hơn, gồm nhiều giai đoạn tính toán, đây chính alf lý do NTLMv2 chỉ dùng NT hash (không dùng LM hash) và thêm username, domain name, timestamp vào phép tính.
**Dạng mẫu một NTLMv2 hash đầy đủ:**
```
admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0000000052920b85f78d013c31cdb3b92f5d765c783030
```
Giống NTLMv1, các Net-NTLMv2 hash bắt được qua mạng cũng **không thể dùng cho pass-the-hash** , chỉ có thể crack offline hoặc relay.
## Domain Cached Credentials (MSCache2)
Trong một môi trường AD, các phương thức xác thực đều yêu cầu máy đang cố truy cập phải giao tiếp được với Domain Controller. Microsoft phát triển thuật toán MS Cachev1 và v2 (còn gọi là Domain Caced Credentials - DCC) để giải quyết tình huống một máy đã join trong doamin nhưng không thể giao tiếp được với Domain Controller (ví dụ do sự cố mạng hoặc lỗi kỹ thuật khác), khiến xác thực NTLM/ Kerberis thông thường không hoạt động để truy cập vào máy đó. **Cơ chế:** Máy tự lưu lại **10 hash gần nhất** của bất kỳ user domain nào từng đăng nhập thành công vào máy, tại registry key `HKEY_LOCAL_MACHINE\SECURITY\Cache`. Đây hoàn toàn là cơ chế cục bộ, không cần liên lạc với DC nữa một khi hash đã được lưu sẵn.
**Đặc điểm khai thác:**
- Các hash này không thể dùng cho tấn công pass-the-hash.
- Hơn nữa, hash rất chậm để crack bằng công cụ, ngay cả khi dùng một giàn GPU cực mạnh vì vậy các nỗ lực crack thường cần nhằm rất có mục tiệu hoặc dựa và việc mật khẩu vốn đã yếu.
- Những hash này có thể lấy được sau khi kẻ tấn công/pentester đã có quyền **local admin** trên máy.
**Dạng mẫu:**

```
$DCC2$10240#bjones#e4e938d12fe5974dc42a90120bd9c90f
```

**Bảng so sáng**

| Hash/Protocol | Kỹ thuật mã hóa        | Mutual Authentication | Loại message                | Bên thứ ba tin cậy |
| ------------- | ---------------------- | --------------------- | --------------------------- | ------------------ |
| NTLM          | Symmetric key          | Không                 | Random number               | Domain Controller  |
| NTLMv1        | Symmetric key          | Không                 | MD4 hash, random number     | Domain Controller  |
| NTLMv2        | Symmetric key          | Không                 | MD4 hash, random number     | Domain Controller  |
| Kerberos      | Symmetric + asymmetric | **Có**                | Encrypted ticket (DES, MD5) | DC / KDC           |
## Kết luận
1. **LM và NT** là **thuật toán băm** tức là cách lưu trữ mật khẩu thành hash (trong SAM trên máy lẻ, hoặc `NTDS.DIT` trên Domain Controller).
2. **NTLM, NTLMv1, NTLMv2** là **giao thức xác thực**, dùng LM/NT hash làm nguyên liệu để chứng minh qua mạng rằng một bên biết đúng mật khẩu, mà không gửi cleartext đi. Trong đó **NTLMv1 dùng được cả LM hoặc NT hash**, còn **NTLMv2 chỉ dùng NT hash**.
3. **Không loại hash nào ở trên dùng salt** kể cả NTLMv2, dù có thêm username/domain/time vào phép tính thì đó cũng không phải salt đúng nghĩa.
4. **Kerberos** vẫn là giao thức được ưu tiên hơn trong AD hiện đại, chủ yếu vì có _mutual authentication_  điều mà NTLM, NTLMv1, NTLMv2 đều thiếu.
