### Introduction
Lưu trữ payload trong section `.rsrc` là một trong những lựa chọn tốt nhất vì đây là phần lớn các binary trong thực tế lưu trữ dữ liệu.
Đây cũng là một phương pháp gọn gàng hơn đối với malware author, bởi vì các payload có thước lớn không thể được lưu trữ trong section `.data` hoặc `.rdata` do giới hạn về kích thước, dẫn đến lỗi từ Visual Studio trong quá trình biên dịch.
### .rsrc Section
Các bước dưới đây mình sẽ minh họa cách lưu trữ payload trong section `.rsrc`:
1. Trong Visual Studio, nhấp chuột phải vào Resource files, sau đó chọn `Add > New Item`.
![[Pasted image 20260828082229.png]]
2. Nhấp vào **`Resource File`**.
![[Pasted image 20260828082332.png]]
3. Thao tác này sẽ tạo một sidebar mới có tên Resource View. Nhấp chuột phải vào tệp `.rc` (tên mặc định là `Resource.rc`), sau đó chọn tùy chọn `Add Resource`.
![[Pasted image 20260828082421.png|center]]
4. .Click 'Import'.
![[Pasted image 20260828082734.png]]
5. Chọn tệp **`calc.ico`**, đây là **payload dạng raw** đã được đổi tên để sử dụng phần mở rộng **`.ico`**.
![[Pasted image 20260828082829.png]]
6. Một hộp thoại sẽ xuất hiện và yêu cầu xác định resource type. Nhập `RCDATA` (không bao gồm dấu ngoặc kép).
![[Pasted image 20260828092617.png]]

7. Sau khi nhấp `OK`, payload sẽ được hiển thị dưới dạng binary thô (raw binary format) bên trong project Visual Studio.
![[Pasted image 20260828082916.png]]
8. Khi thoát khỏi Resource View, tệp header `resource.h` sẽ được hiển thị và được đặt tên tương ứng với tệp `.rc` ở Bước 2. Tệp này chứa một define statement tham chiếu đến ID của payload trong resource section (`IDR_RCDATA1`). Đây là thành phần quan trọng để có thể truy xuất payload từ resource section ở các bước sau.
![[Pasted image 20260828093226.png]]

Sau khi biên dịch, payload sẽ được lưu trữ trong section `.rsrc`, nhưng không thể truy cập trực tiếp. Thay vào đó, cần sử dụng một số WinAPI để truy xuất payload:
- [`FindResourceW`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-findresourcew): Lấy vị trí của dữ liệu được chỉ định được lữu trữ trong resource section dựa trên một ID cụ thể được truyền vào.
- [`LoadResource`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadresource): Lấy handle `HGLOBAL` của resource data. Handle này có thể được sử dụng để lấy base address của resource được chỉ định trong bộ nhớ.
- [`LockResource`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-lockresource): Lấy pointer tới dữ liệu được chỉ định trong resource section thông qua handle của resource đó.
- [`SizeofResource`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-sizeofresource): Lấy kích thước của dữ liệu được chỉ định trong section.
Đoạn code bên dưới sẽ sử dụng các Windows API trên để truy cập section `.rsrc` và lấy địa chỉ cùng kích thước của payload.
```c
#include <Windows.h>
#include <stdio.h>
#include "resource.h"

int main() {

	HRSRC		hRsrc                   = NULL;
	HGLOBAL		hGlobal                 = NULL;
	PVOID		pPayloadAddress         = NULL;
	SIZE_T		sPayloadSize            = NULL;

	
	// Get the location to the data stored in .rsrc by its id *IDR_RCDATA1*
	hRsrc = FindResourceW(NULL, MAKEINTRESOURCEW(IDR_RCDATA1), RT_RCDATA);
	if (hRsrc == NULL) {
		// in case of function failure 
		printf("[!] FindResourceW Failed With Error : %d \n", GetLastError());
		return -1;
	}

	// Get HGLOBAL, or the handle of the specified resource data since its required to call LockResource later
	hGlobal = LoadResource(NULL, hRsrc);
	if (hGlobal == NULL) {
		// in case of function failure 
		printf("[!] LoadResource Failed With Error : %d \n", GetLastError());
		return -1;
	}

	// Get the address of our payload in .rsrc section
	pPayloadAddress = LockResource(hGlobal);
	if (pPayloadAddress == NULL) {
		// in case of function failure 
		printf("[!] LockResource Failed With Error : %d \n", GetLastError());
		return -1;
	}

	// Get the size of our payload in .rsrc section
	sPayloadSize = SizeofResource(NULL, hRsrc);
	if (sPayloadSize == NULL) {
		// in case of function failure 
		printf("[!] SizeofResource Failed With Error : %d \n", GetLastError());
		return -1;
	}
	
	// Printing pointer and size to the screen
	printf("[i] pPayloadAddress var : 0x%p \n", pPayloadAddress);
	printf("[i] sPayloadSize var : %ld \n", sPayloadSize);
	printf("[#] Press <Enter> To Quit ...");
	getchar();
	return 0;
}
```

Sau khi biên dịch và chạy đoạn code trên, địa chỉ của payload cùng với kích thước của nó sẽ được in ra màn hình. Cần lưu ý rằng địa chỉ này nằm trong section `.rsrc`, đây là vùng bộ nhớ chỉ đọc (read-only memory). Mọi nỗ lực thay đổi hoặc chỉnh sửa dữ liệu trong vùng này sẽ gây ra lỗi access violation.
Để chỉnh sửa payload, cần cấp phát một buffer có kích thước bằng với kích thước của payload, sau đó sao chép payload sang buffer này. Buffer mới này là nơi có thể thực hiện các thay đổi đối với payload, chẳng hạn như giải mã (decrypt) payload.
### Updating .rsrc Payload
Vì payload không thể được chỉnh sửa trực tiếp trong resource section, nó phải được chuyển sang một buffer tạm thời. Để thực hiện việc này, bộ nhớ có kích thước bằng payload được cấp phát bằng `HeapAlloc`, sau đó payload được chuyển thành resource section sang temporary buffer bằng `memcpy`.
```c
// Allocating memory using a HeapAlloc call
PVOID pTmpBuffer = HeapAlloc(GetProcessHeap(), 0, sPayloadSize);
if (pTmpBuffer != NULL){
	// copying the payload from resource section to the new buffer 
	memcpy(pTmpBuffer, pPayloadAddress, sPayloadSize);
}

// Printing the base address of our buffer (pTmpBuffer)
printf("[i] pTmpBuffer var : 0x%p \n", pTmpBuffer);
```
Vì `pTmpBuffer` hiện đang trỏ tới một vùng bộ nhớ có quyền ghi (writable memory region) chứa payload, nên có thể thực hiện việc decrypt payload hoặc thực hiện bất kỳ thay đổi nào đối với payload.
![[Pasted image 20260828104451.png]]
Tiếp tục quá trình thực thi, payload được lưu vào temporary buffer.
![[Pasted image 20260828104827.png]]

