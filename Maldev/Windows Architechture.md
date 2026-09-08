## Windows Architechture
Một bộ xử lý bên trong máy tính chạy hệ điều hành Windows có thể hoạt động ở hai chế độ khác nhau: User Mode và Kernel Mode. Các ứng dụng hoạt động trong User Mode, trong khi các thành phần của hệ điều hành hoạt động trong Kernel Mode. Khi một ứng dụng muốn thực hiện một tác vụ, chẳng hạn như tạo một tệp, ứng dụng không thể tự mình thực hiện thao tác này. Kernel là thực thể duy nhất có quyền và khả năng thực hiện tác vụ đó. Vì vậy, ứng dụng phải tuân theo một luồng gọi hàm cụ thể yêu cầu kernel thực hiện thao tác.
![[Pasted image 20260824072156.png]]
1. User Processes: Một chương trình/ ứng dụng được người dùng thực thi, chẳng hạn như Notepad, Google Chrome hoặc Microsoft Word.
2. Subsystem DLLs: Các DLL chứa những hàm API được gọi bởi tiến trình người dùng. Một ví dụ là `kernel32.dll`, cung cấp hàm `CreateFile` thuộc Windows API. Các DLL của phân hệ phổ biến khác bao gồm `ntdll.dll`, `advapi32.dll` và `user32.dll`.
3. `Ntdll.dll`: là một dll có phạm vị toàn hệ thống và lớp thấp nhất khả dụng trong User Mode. Đây là một DLL đặc biệt đảm nhận việc chuyển tiếp từ User Mode sang Kernel Mode. Cơ chế này thường được gọi là Native API hoặc NTAPI.
4. Executive Kernel: Đây là thành phần được biết đến như Windows Kernel, có nhiệm vụ gọi các driver và module khác có sẵn trong Kernel Mode để hoàn thành các tác vụ. Windows Kernel được lưu trữ mọt phần trong tệp `ntoskrnl.exe`, nằm tại `C:\Windows\System32`.
## Function Call Flow
Hình ảnh dưới đây minh họa một ví dụ về ứng dụng thực hiện thao tác tạo tệp. Luồng xử lý bắt đầu khi ứng dụng user gọi hàm `CreateFile` thuộc WinAPI, hàm này được cung cấp trong `kernel32.dll`. `kernel32.dll` là một DLL quan trọng, cung cấp quyền truy cập vào WinAPI cho các ứng dụng và do đó thường được tải bởi phần lớn các ứng dụng. Tiếp theo, `CreateFile` gọi hàm NTAPI tương ứng là `NtCreateFile` được cung cấp thông qua `ntdll.dll`. Sau đó, `ntdll.dll` thực thi lệnh assembly `sysenter` (x86) hoặc `syscall` (x64), qua đó chuyển quyền thực thi từ User Model sang Kernel Mode. Tiếp theo, hàm `NtCreateFile` trong kernel được sử dụng để gọi các kernel driver và module nhằm thực hiện tác vụ được yêu cầu.
![[Pasted image 20260824074618.png]]
### Function Call Flow Example
Ví dụ này minh họa luồng gọi hàm được quan sát thông qua debugger. Quá trình này được thực hiện bằng cách gắn debugger vào một binary thực hiện thao tác tạo tệp thông qua Windows API `CreateFileW`. Ứng dụng người dùng gọi hàm **`CreateFileW`** thuộc **WinAPI**.
![[Pasted image 20260824074902.png]]
**`CreateFileW`** tiếp theo gọi hàm **NTAPI** tương ứng là **`NtCreateFile`**.
![[Pasted image 20260824074918.png]]
Cuối cùng, hàm **`NtCreateFile`** sử dụng lệnh assembly **`syscall`** để chuyển từ User Mode sang Kernel Mode. Sau đó, **kernel** sẽ là thành phần trực tiếp thực hiện thao tác tạo tệp.
![[Pasted image 20260824074955.png]]
### Directly Invoking the Native API (NTAPI)
Điều quan trọng cần lưu ý là các ứng dụng có thể gọi trực tiếp các `syscall` mà không cần đi qua Windows API. Windows API về bản chất chỉ đóng vai trò như một lớp wrapper cung cấp giao diện để gọi Native API. Tuy nhiên Native API khó sử dụng hơn vì Microsoft không chính thức công bố tài liệu đầy đủ về API này. Hơn nữa, Microsoft khuyến cáo không nên sử dụng trực tiếp các hàm của Native API, vì chúng có thể được thay đổi bất kỳ lúc nào mà không có thông báo trước. 