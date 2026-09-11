### Introduction
Khi tham chiếu đến tài liệu Windows về một cấu trúc, có thể bắt gặp nhiều thành phần được đánh dấu là `reserved` bên trong cấu trúc đó. Các thành phân `reserved` này thường được biểu diễn dưới dạng mảng có kiểu dữ liệu `BYTE` hoặc `PVOID`. Cách triển khai này được Microsoft sử dụng nhằm duy trì tính bảo mật và hạn chế người dùng tìm hiểu cấu trúc nội bộ, từ đó tránh việc người dùng thực hiện các thay đổi đối với những thành phần `reserved` này.
### PEB Structure Example
Process Environment Block (PEB) là một cấu trúc dữ liệu chứa thông tin về Windows process.Tuy nhiên, tài liệu của Microsoft về cấu trúc PEB hiển thị một số thành viên dưới dạng `reserved`, khiến việc truy cập các thành viên này trở nên khó khăn.
```c
typedef struct _PEB {
  BYTE                          Reserved1[2];
  BYTE                          BeingDebugged;
  BYTE                          Reserved2[1];
  PVOID                         Reserved3[2];
  PPEB_LDR_DATA                 Ldr;
  PRTL_USER_PROCESS_PARAMETERS  ProcessParameters;
  PVOID                         Reserved4[3];
  PVOID                         AtlThunkSListPtr;
  PVOID                         Reserved5;
  ULONG                         Reserved6;
  PVOID                         Reserved7;
  ULONG                         Reserved8;
  ULONG                         AtlThunkSListPtr32;
  PVOID                         Reserved9[45];
  BYTE                          Reserved10[96];
  PPS_POST_PROCESS_INIT_ROUTINE PostProcessInitRoutine;
  BYTE                          Reserved11[128];
  PVOID                         Reserved12[1];
  ULONG                         SessionId;
} PEB, *PPEB;
```
#### Finding Reserved Members
Một cách để xác định các reserved members của PEB đang lưu trữ những gì là sử dụng lệnh `!peb` trong WinDbg.
![[Pasted image 20260827192042.png]]
Để tham khảo cấu trúc PEB đầy đủ hơn, có thể xem định nghĩa [PEB structure](https://github.com/winsiderss/systeminformer/blob/master/phnt/include/ntpebteb.h#L56) của Process Hacker.
### Alternative Documentation
Như đã đề cập trước đó, một số module sẽ không sử dụng tài liệu của Microsoft mà thay vào đó tham khảo các nguồn tài liệu khác.
- [**Process Hacker's Header Files**](https://github.com/winsiderss/systeminformer/tree/master/phnt/include): Các tệp header chứa nhiều định nghĩa cấu trúc nội bộ của Windows.
- [**undocumented.ntinternals.net**](http://undocumented.ntinternals.net/): Cung cấp thông tin về các cấu trúc và thành phần undocumented của Windows. Tuy nhiên, một số cấu trúc có thể đã **lỗi thời**.
- [**ReactOS's Documentation**](https://doxygen.reactos.org/globals_type.html): Cung cấp tài liệu và định nghĩa liên quan đến các thành phần bên trong hệ điều hành Windows.
- [**Vergilius Project**](https://www.vergiliusproject.com/): Mặc dù chủ yếu tập trung vào các **cấu trúc Windows kernel**, đây vẫn là một nguồn tài liệu có giá trị.
