## Introduction
Windows API cung cấp cho các nhà phát triển một phương thức để ứng dụng của họ tương tác với hệ điều hành Windows. Ví dụ, nếu ứng dụng cần hiển thị nội dung trên màn hình, sửa đổi một tệp hoặc truy vấn Registry, tất cả thao tác này đều có thể thực hiện thông qua Windows API. 
## Windows Data Types
Windows có nhiều kiểu dữ liệu ngoài những kiểu dữ liệu phổ biến như `int`, `float`. Các kiểu dữ liệu này được Microsoft tài liệu hòa và có thể xem trong tài liệu [Windows Data Types](https://learn.microsoft.com/en-us/windows/win32/winprog/windows-data-types). Một số kiểu dữ liệu phổ biến được liệt kê dưới đây:
- `DWORD`: Một số nguyên không dấu 32-bit, được sử dụng trên hệ thống 32-bit và 65-bit, biểu diễn các giá trị trừ `0` đến `(2^32 - 1)`.
```c
DWORD dwVariable = 42;
```
- `size_t`: Được sử dụng để biểu diễn kích thước của một đối tượng. Trên hệ thống 32-bit, đây là số nguyên không dấu 32-bit, biểu diễn các giá trị từ `0` đến `(2^32 - 1)`. Trên hệ thống 64-bit, đây là số nguyên không dấu 64-bit, biểu diễn các giá trị từ `0` đến `(2^64 - 1)`.
```c
SIZE_T sVariable = sizeof(int);
```
- `VOID`: Biểu thị sự không tồn tại của một kiểu dữ liệu
```c
void* pVariable = NULL; // Tương đương với PVOID
```
- `PVOID`: Một con trỏ 32-bit hoặc 4 byte có thể trỏ tới bất kỳ kiểu dữ liệu nào trên hệ thống 32-bit. Trên hệ thống 64-bit, đây là con trỏ 64-bit hoặc 8 byte có thể trỏ tới bất kỳ kiểu dữ liệu nào.
```c
PVOID pVariable = &SomeData;
```
- `HANDLE`: Một giá trị dùng để xác định một đối tượng cụ thể do hệ điều hành quản lý chẳng hạn như tệp, tiến trình hoặc luồng:
```c
HANDLE hFile = CreateFile(...);
```
- `HMODULE`: một handle tới một module. Đây chính là địa chỉ cơ sở của module trong bộ nhớ. Một ví dụ về module có thể là tệp DLL hoặc EXE/
```c
HMODULE hModule = GetModuleHandle(...);
```
- `LPCSTR/PCSTR`: Một con trỏ tới một chuỗi ký tự hằng, kết thúc bằng `NULL`, sử udnjg các ký tự Windows 8-bit (ANSI). Chữ `L` là viết tắt mang tính lịch sử của `long`, bắt nguồn từ thời kỳ lập trình Windows 16-bit. Ngày nay, tiền tố này không ảnh hưởng đến kiểu dữ liệu, nhưng vấn được duy trì trong quy ước đặt tên. Chữ `C` biểu thị "constant" hoặc biến read-only. Cả hai kiểu này tương đương với `const char*`.
```c
LPCSTR lpString = "Hello World!";
PCSTR pcString = "Hello World!";
```
- `LPSTR/PSTR`: Tương tự `LPCSTR` và `PCSTR` nhưng điểm khác biệt là `LPSTR` và `PSTR` không trỏ tới một biến hằng, mà trỏ tới một chuỗi có thể đọc và ghi. Cả hai kiểu dữ liệu này tương đương với `char*`.
```c
LPSTR lpString = "Hello World!";
PSTR pString = "Hello World!";
```
- `LPCWSTR/PCWSTR`: Một con trỏ tới chuỗi ký tự Unicode hẳng, kết thúc bằng `NULL`, sử dụng ký tự Unicode 16-bit của Windows. Cả hai kiểu dữ liệu này tương đương với `const wchar*`.
```c
LPCWSTR lpwcString = L"Hello, world!"; 
PCWSTR pcwString = L"Hello, world!";
```
- `PWSTR/LPWSTR`: Tương tự `LPCWSTR` và `PCWSTR` nhưng `PWSTR` và `LPWSTR` không trỏ tới một biến hằng mà trỏ tới một chuỗi có thể read và write. Cả hai kiểu dữ liệu này đều tương đương với `wchar*`.
```c
LPWSTR lpwString = L"Hello, world!"; 
PWSTR pwString = L"Hello, world!";
```
- `wchar_t`: Tương tự `wchar`, được sử dụng để biếu diễn ký tự wide.
```c
wchar_t wChar = L'A'; 
wchar_t* wcString = L"Hello, world!";
```
- `ULONG_PTR`: Biểu diễn một số nguyên không dấu có kích thước bằng kích thước của một con trỏ trên kiến trúc được chỉ định. Điều này có nghĩa là trên hệ thống 32-bit, `ULONG_PTR` có kích thước 32 bit, còn khi trên hệ thống 64-bit nó có kích thước 64 bit. `ULONG_PTR` thường được sử dụng trong các phép toán số học liên quan đến con trỏ ví dụ `PVOID`. Trước khi thực hiện phép toán số học, con trỏ sẽ được ép kiểu sang `ULONG_PTR`. Cách tiếp cận này được sử dụng để tránh thao tác trực tiếp tên con trỏ, vốn có thể dẫn đến compilation errors.
```c
PVOID Pointer = malloc(100);
// Pointer = Pointer + 10; // không được phép
Pointer = (ULONG_PTR)Pointer + 10 // Allow
```
## Data Type Pointers
Windows API cho phép nhà phát triển khai báo một kiểu dữ liệu trực tiếp hoặc con trỏ tới kiểu dữ liệu đó. Điều này được thể hiện trong tên của các kiểu dữ liệu: những kiểu dữ liệu bắt đầu bằng `P` biểu thị con trỏ tới kiểu dữ liệu tương ứng, trong khi những kiêu dữ liệu không bắt đầu bằng `P` biểu thị chính kiểu dữ liệu đó.
Điều này sẽ trở nên hữu ích khi làm việc với các hàm Windows API có tham số là con trỏ tới một kiểu dữ liệu. Các ví dụ duois đây minh họa mối quan hệ giũa kiểu dữ liệu bắt đầu bằng `P` và kiểu dữ liệu tương ứng không phải là con trỏ:
- `PHANDLE` tương đương với `HANDLE*`
- `PSIZE_T` tương đương với `SIZE_T*`
- `PDWORD` tương đương với `DWORD*`
## ANSI & Unicode Functions
Phần lớn các hàm Windows API có hai phiên bản với kết thúc bằng `A` hoặc `W`. Ví dụ `CreateFileA` và `CreateFileW`. Các hàm kết thúc bằng `A` biểu thị ANSI, trong khi các hàm kết thúc bằng `W` biểu thị Unicode hoặc Wide.
Điểm khác biệt chính cần lưu ý là các hàm ANSI sẽ nhận các kiểu dữ liệu ANSI làm tham số, khi phù hợp, trong khi các hàm Unicode sẽ nhận các kiểu dữ liệu Unicode. Ví dụ, tham số đầu tiên của `CreateFileA` là `LPCSTR`, tức là con trỏ tới chuỗi ký tự hằng, kết thúc bằng `NULL`, sử dụng ký tự ANSI 8-bit của Windows. Ngược lại, tham số đầu tiên của `CreateFileW` là `LPCWSTR`, tức là con trỏ tới chuỗi ký tự Unicode hằng, kết thúc bằng `NULL`, sử dụng ký tự Unicode 16-bit.
Hơn nữa, số lượng byte cần thiết để lưu trữ dữ liệu cũng sẽ khác nhau tùy thuộc vào phiên bản hàm được sử dụng.
```c
char str1[] = "maldev"; // 7 byte (maldev + null byte)

char str2[] = L"maldev"; // 14 byte, mỗi ký tự chiếm 2 byte và null cũng chiếm 2 byte
```
## In and Out Parameters
Windows API có các tham số `IN` và `OUT`. Tham số `IN` là tham số được truyền vào hàm và được sử dụng làm dữ liệu đầu vào. Trong khi đó, tham số `OUT` là tham số được sử dụng để trả về một giá trị cho bên gọi hàm. Các tham số đầu ra thường được truyền bằng tham chiếu thông qua con trỏ.
Ví dụ, đoạn mã dưới đây định nghĩa một hàm `HackTheWorld`, nhận vào một con trỏ tới kiểu dữ liệu số nguyên và thiết lập giá trị của nó thành  123. Đây được xem là một tham số `OUT` vì tham số này được sử dụng để tra về một giá trị
```c
BOOL HackTheWorld(OUT int* num){
	// Thiết lập giá trị của num thành 123
	*num = 123;
	// Trả về giá trị boolean
	return TRUE;
}

int main(){
	int a = 0;
	// 'HackTheWorld' sẽ trả về true
	// 'a' sẽ chứa giá trị 123
	HackTheWorld(&a);
}
```
Cần lưu ý rằng việc sử dụng các từ khóa `OUT` hoặc `IN` nhằm giúp ta dễ dàng hiểu hàm yêu cầu gì và hàm sẽ xử lý các tham số đó thế nào. Tuy nhiên việc loại bỏ các từ khóa này vẫn không ảnh hưởng đến việc tham số được xem là tham số đầu vào hay đầu ra.
### Windows API Example 
Sau khi đã tìm hiểu cơ bản các kiến thức nền tảng về Windows API, ta sẽ tìm hiểu các sử dụng hàm `CreateFileW`
#### Find the API Reference
Điều quan trọng luôn là tham khảo tài liệu chính thức khi không chắc chắn về chức năng của một hàm hoặc các đối số mà hàm yêu cầu. Luôn độc kỹ mô tả của hàm à đánh giá xem hàm đó có thực hiện đúng tác vụ mong muốn hay không. Tài liệu của `CreateFileW` [here](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilew).
#### Analyze Return Type & Parameters
Bước tiếp theo là xem xét các tham số của hàm cùng với kiểu dữ liệu trả về. Tài liệu cho biết: nếu hàm thực thi thành công, giá trị trả về một handle đang mở tới tệp, thiết bị, named pipe hoặc mail slot được chỉ định. Do đó, `CreateFileW` trả về kiểu dữ liệu `HANDLE` tới đối tượng được chỉ định.
Ngoài ra, hãy lưu ý rằng tất cả các tham số của hàm đều là tham số `in`. Điều này có nghĩa là hàm không trả về dữ liệu thông qua các tham số này, vì tất cả chúng đều là tham số đầu vào. Cần lưu ý rằng các từ khóa nằm trong dấu ngoặc vuông, chẳng hạn như `in`, `out`, và `optional`, chỉ nhằm mục đích tham chiếu cho dev và không có tác dụng thực tế đến hoạt động của hàm.
```c
HANDLE CreateFileW( 
	[in] LPCWSTR lpFileName, 
	[in] DWORD dwDesiredAccess, 
	[in] DWORD dwShareMode, 
	[in, optional] LPSECURITY_ATTRIBUTES lpSecurityAttributes, 
	[in] DWORD dwCreationDisposition, 
	[in] DWORD dwFlagsAndAttributes, 
	[in, optional] HANDLE hTemplateFile 
);
```

####  Use The Function
Đoạn mã mẫu dưới đây minh họa cách sử dụng `CreateFileW`. Đoạn mã sẽ tạo một tệp văn bản có tên **`maldev.txt`** trên **Desktop của người dùng hiện tại**.
```c
// Biến này được sử dụng để lưu handle tới đối tượng tệp 
// 'INVALID_HANDLE_VALUE' chỉ được sử dụng để khởi tạo biến
HANDLE hFile = INVALID_HANDLE_VALUE;

// Đường dẫn đầy đủ của tệp cần tạo.
// Trong C, cần sử dụng hai dấu gạch chéo ngược để escape một dấu gạch chéo ngược.
LPCWSTR filePath = L"C:\\Users\\admin\\Desktop\\maldev.txt";

// Gọi CreateFileW với đường dẫn của tệp
// Các tham số bổ sung được lấy trực tiếp từ tài liệu.
hFile = CreateFileW(filePath, GENERIC_ALL, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);

// Khi thất bại, CreateFileW trả về INVALID_HANDLE_VALUE
// GetLastError() là một Windows API dùng để lấy mã lỗi
// của hàm WinAPI được thực thi trước đó.
if (hFile == INVALID_HANDLE_VALUE){ 
	printf("[-] CreateFileW Api Function Failed With Error : %d\n", GetLastError()); 
	return -1; 
}

```

![[Pasted image 20260824170259.png]]

#### Windows API Debugging Errors
Khi các hàm thất bại, chúng thường trả về một mã lỗi không cung cấp nhiều thông tin chi tiết. Ví dụ, nếu `CreateFileW` thất bại, hàm sẽ trả về `INVALID_HANDLE_VALUE` cho biết tệp không thể được tạo. Để hiểu rõ nguyên nhân khiến tệp không thể được tạo, cần lấy error code bằng hàm `GetLastError`. Sau khi lấy được error code, cần tra cứu code đó trong Windows System Error Codes List. Một số mã lỗi phổ biến:
- `5` - `ERROR_ACCESS_DENIED`: Từ chối truy cập
- `2` - `ERROR_FILE_NOT_FOUND`: Không tìm thấy tệp
- `87` - `ERROR_INVALID_PARAMETER`: Tham số không hợp lệ
#### Windows Native API Debugging Errors
Các NTAPI chủ yếu được export từ `ntdll.dll`. Không giống Windows API, các hàm này không thể sử dụng `GetLastError` để lấy error code. Thay vào đó, chúng trả về trực tiếp mã trạng thái lỗi, được biểu diễn bằng kiểu dữ liệu `NTSTATUS`.
`NTSTATUS` được sử dụng để biểu thị trạng thái của một system call hoặc một hàm và được định nghĩa là một giá trị số nguyên không dấu 32-bit. Một system call hoặc một hàm và được định nghĩa là một giá trị số nguyên không dấu 32-bit. Một system call thực thi thành công sẽ trả về giá trị `STATUS_SUCCESS`, có giá trị là `0`. Ngược lại nếu lời gọi thất bại, nó sẽ trả về giá trị khác `0`. Để tìm hiểu nguyên nhân của lỗi, cần tra cứu tài liệu của Microsoft về các giá trị `NTSTATUS`.
Đoạn mã dưới đây minh họa cách kiểm tra lỗi đối với system call:
```c
NTSTATUS STATUS = NativeSyscallExample(...);

if (STATUS != STATUS_SUCCESS){
	// In mã lỗi dưới dạng số nguyên không dấu ở định dạng hexadecimal
	printf("[!] NativeSyscallExample Failed With Status : 0x%0.8X \n", STATUS);
}
// NativeSyscallExample thực thi thành công
```
#### NT_SUCCESS Macro
Một phương thức khác để kiểm tra giá trị trả về của NTAPI là sử dụng macro `NT_SUCCESS`. Macro này trả về `TRUE` nếu hàm thực thi thành công và `FALSE` nếu hàm thất bại.
```c
#define NT_SUCCESS(Status) (((NTSTATUS)(Status)) >= 0)
```
Ví dụ dưới đây minh họa cách sử dụng macro này:
```
NTSTATUS STATUS = NativeSyscallExample(...);

if (!NT_SUCCESS(STATUS)){
	// In mã lỗi dưới dạng số nguyên không dấu ở định dạng hexadecimal
	printf("[!] NativeSyscallExample Failed With Status : 0x%0.8X \n", STATUS);
}
```