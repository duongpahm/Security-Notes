### Payload Encryption
Payload Encryption trong malware là một kỹ thuật được attacker sử dụng để che giấu malicious code được chứa trong một tệp độc hại. Attack sử dụng nhiều thuật toán mã hóa khác nhau để che giấu mã độc, khiến các security solution khó phát hiện hoạt động độc hại của tệp hơn.
Encryption cũng giúp malware duy trì trạng thái ẩn và tránh bị phát hiện trên hệ thống người dùng trong thời gian dài hơn. Việc mã hóa các thành phần của malware gần như luôn cần thiết khi đối phó với các security solution hiện đại.
### Encryption Pros and Cons
Encryption có thể giúp né tránh signature-based detection khi sử dụng các đoạn code và payload đã bị signature, nhưng có thể không hiệu quả trước các hình thức detection khác, chẳng hạn như runtime analysis và heuristic analysis.
Cần lưu ý rằng càng nhiều dữ liệu được mã hóa bên tong một tệp thì entropy của tệp càng cao. Một tệp có entropy cao có thể khiến security solution đánh dấu tệp là suspicious, hoặc ít nhất xem tệp đó là đáng ngờ và tiến hành phân tích/ giám sát bổ sung. 
### Encryption Types
Các  thuật toán mã hóa được sử dụng phổ biến trong malware là `XOR`, `AES`, `RC4`.
## Payload Encryption - XOR
XOR encryption là phương pháp đơn giản nhất để sử dụng và nhẹ nhất để triển khai, do đó trở thành một lựa chọn phổ biến trong malware. XOR nhanh hơn AES và RC4, đồng thời không yêu cầu sử dụng thêm thư viện hoặc Windows API. Ngoài ra, XOR là một thuật toán mã hóa hai chiều, cho phép sử dụng cùng một hàm cho cả quá trình mã hóa và giải mã.
### XOR Encryption
Đoạn code dưới đây minh họa một hàm mã hóa XOR cơ bản. Hàm này đơn giản thực hiện phép XOR trên từng byte của shellcode với một khóa có kích thước 1 byte.
```C
/*
	- pShellcode : Base address of the payload to encrypt 
	- sShellcodeSize : The size of the payload 
	- bKey : A single arbitrary byte representing the key for encrypting the payload
*/
VOID XorByOneKey(IN PBYTE pShellcode, IN SIZE_T sShellcodeSize, IN BYTE bKey) {
	for (size_t i = 0; i < sShellcodeSize; i++){
		pShellcode[i] = pShellcode[i] ^ bKey;
	}
}
```
### Securing The Encryption Key
Một số công cụ và security solution có thể thực hiện brute-force key từ đó làm lộ shellcode đã được giải mã. Để khiến quá trình đoán key trở nên khó khăn hơn với các công cụ này, đoạn code bên dưới thực hiện một thay đổi nhỏ và mở rộng keyspace của key bằng cách đưa `i` trở thành một phần của key. Khi keyspace lớn hơn đáng kể, việc brute-force key sẽ trở nên khó khăn hơn.
```c
/*
	- pShellcode : Base address of the payload to encrypt 
	- sShellcodeSize : The size of the payload 
	- bKey : A single arbitrary byte representing the key for encrypting the payload
*/
VOID XorByiKeys(IN PBYTE pShellcode, IN SIZE_T sShellcodeSize, IN BYTE bKey) {
	for (size_t i = 0; i < sShellcodeSize; i++) {
		pShellcode[i] = pShellcode[i] ^ (bKey + i);
	}
}
```
Đoạn code trên vẫn có thể được tăng cường mức độ bảo vệ (hardened) hơn nữa. Đoạn code bên dưới thực hiện quá trình mã hóa bằng một key, trong đó từng byte của key được sử dụng lặp lại, qua đó khiến việc phá mã (crack) key trở nên khó khăn hơn.
```c
/*
	- pShellcode : Base address of the payload to encrypt 
	- sShellcodeSize : The size of the payload 
	- bKey : A random array of bytes of specific size
	- sKeySize : The size of the key
*/
VOID XorByInputKey(IN PBYTE pShellcode, IN SIZE_T sShellcodeSize, IN PBYTE bKey, IN SIZE_T sKeySize) {
	for (size_t i = 0, j = 0; i < sShellcodeSize; i++, j++) {
		if (j > sKeySize){
			j = 0;
		}
		pShellcode[i] = pShellcode[i] ^ bKey[j];
	}
}
```

![[Pasted image 20260828142546.png]]

