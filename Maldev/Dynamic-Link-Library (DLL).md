### What is a DLL?
DLL (Dynamic-Link-Library) là các thư viện dùng chung chứa các hàm thực thi hoặc dữ liệu có thể được nhiều ứng dụng sử dụng đồng thời. DLL được sử dụng để export các hàm nhằm cho phép một tiến trình sử dụng chúng. Không giống như tệp `.exe`, tệp `.dll` không thể tự thực thi mã lệnh. Thay vào, thư viện DLL cần được một chương trình khác gọi (invoke) để thực thi mã bên trong. Như ta đã biết, `CreateFileW` được export từ `kernel32.dll`; do đó, nếu một tiến trình muốn gọi hàm này, trước tiên nó cần load `kernel32.dll` vào không gian đại chỉ của mình.
Một số DLL được tự động load vào mọi tiến trình theo mặc định, vì các DLL này export những hàm cần thiết để tiến trình có thể thực thi đúng cách. Một ví dụ về các DLL này là `ntdll.dll`, `kernel32.dll` và `kernelbase.dll`.
![[Pasted image 20260826104643.png]]
### System-Wide DLL Base Address
Hệ điều hành Windows sử dụng một địa chỉ cơ sở DLL trê toàn hệ thống để load một số DLL tại cùng một địa chỉ cơ sở trong không gian địa chỉ ảo của tất cả tiến trình trên một máy tính, nhằm tối ưu hóa việc sử dụng bộ hớ và cải thiện hiệu năng hệ thống. Hình ảnh bên dưới cho thấy `kernel32.dll` được load tại cùng một địa chỉ (`0x7fff9fad0000`) trong nhiều tiến trình đang chạy.
![[Pasted image 20260826110214.png]]

### Why Use DLLs?
Có một số lí do chính khiến DLL được sử dụng phổ biến trong Windows:
1. Modularization of Code: thay vì có một tệp thực thi khổng lồ chứa toàn bộ chức năng, mã lệnh được chia thành nhiều thư viện độc lập, trong đó mỗi thư viện tập trung vào một chức năng cụ thể. Việc module hóa giúp các nhà phát triển dễ dàng hơn trong quá trình phát triển và debug.
2. Code Reuse: DLL thúc đẩy việc tái sử dụng mã vì thư viện có thể được nhiều tiến trình gọi và sử dụng.
3. Efficient Memory Usage: Khi nhiều tiến trình cần cùng một DLL, chúng ta có thể tiết kiệm bộ nhớ bằng cách chia sẻ DLL đó thay vì load một bản sao riêng vào bộ nhớ của từng tiến trình.
### DLL Entry Point
DLL có thể tùy chọn chỉ định một hàm entry point, hàm này thực thi mã khi một sự kiện nhất định xảy ra, chẳng hạn như khi một tiến trình load thư viện DLL. Có 4 trường hợp mà entry point có thể nhận được:
- `DLL_PROCESS_ATTACH`: Một tiến trình đang load DLL.
- `DLL_THREAD_ATTACH`: Một tiến trình đang tạo một thread mới.
- `DLL_THREAD_DETACH`: Một thread kết thúc bình thường.
- `DLL_PROCESS_DETACH`: Một tiến trình unload DLL.
### Sample DLL Code
Đoạn mã dưới đây minh họa một cấu trúc mã DLL điển hình:
```c
BOOL APIENTRY DllMain(
    HANDLE hModule,          // Handle to DLL module
    DWORD ul_reason_for_call,// Reason for calling function
    LPVOID lpReserved        // Reserved
) {

    switch (ul_reason_for_call) {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
            // Do something here
            break;

        case DLL_THREAD_ATTACH:  // A process is creating a new thread.
            // Do something here
            break;

        case DLL_THREAD_DETACH:  // A thread exits normally.
            // Do something here
            break;

        case DLL_PROCESS_DETACH: // A process unloads the DLL.
            // Do something here
            break;
    }

    return TRUE;
}
```
### Exporting a Function
DLL có thể export các hàm để ứng dụng hoặc tiến trình gọi chúng sử dụng. Để export một hàm, hàm đó cần được khai báo bằng các từ khóa `extern` và `__declspec(dllexport)`. Ví dụ về một hàm được export `HelloWorld` được minh họa bên dưới.
```c
////// sampleDLL.dll //////
extern __declspec(dllexport) void HelloWorld(){
	// Function code here
}
```
### Dynamic Linking
Có thể sử dụng các WinAPI `LoadLibrary`, `GetModuleHandle`, và `GetProcAddress` để import một hàm từ DLL. Kỹ thuật này gọi là dynamic linking. Đây là phương pháp load và liên kết DLL tại thời điểm runtime, thay vì liên kết chúng tại thời điểm biên dịch bằng linker và import address table.
#### Loading a DLL
Việc gọi một hàm như `MessageBoxA` trong một ứng dụng sẽ khiến Windows OS load DLL export hàm `MessageBoxA` vào không gian địa chỉ bộ nhớ của tiến trình gọi; trong trường hợp này là `user32.dll`. Việc load `user32.dll` được OS thực hiện tự động khi tiến trình khời chạy, chứ không phải do mã chương trình thực hiện.
Tuy nhiên, trong một số trường hợp, chẳng hạn làm `HelloWorld` trong `sampleDLL.dll`, DLL có thể chưa được load vào bộ nhớ. Để ứng dụng có thể gọi hàm `HelloWorld`, trước tiên nó cần lấy handle của DLL đang export hàm đó. Nếu ứng dụng chưa load `sampleDLL.dll` vào bộ nhớ, cần sử dụng WinAPI `LoadLibrary`, như bên dưới:
```c
HMODULE hModule = LoadLibraryA("sampleDLL.dll); // hModule now contain sampleDLL.dll's handle
```
#### Retrieving a Function's Address
Nếu `sampleDLL.dll` đã được load vào bộ nhớ và đã lấy được handle , bước tiếp theo là lấy địa chỉ của hàm. Việc này được thực hiện bằng WinAPI `GetProcAddress`, hàm này nhận vào handle của DLL export hàm và tên của hàm.
```c
PVOID pHelloWorld = GetProcAddress(hModule, "HelloWorld");
```
#### Invoking The Function
Sau khi địa chỉ của `HelloWorld` được lưu vào biến `pHelloWorld`, bước tiếp theo là thực hiện type-cast địa chỉ này thành function pointer của hàm `HelloWorld`. Function pointer này cần thiết để có thể invoke hàm.
```c
type void (WINAPI* HelloWorldFunctionPointer)();

void call(){
	HMODULE hModule = LoadLibrary("sampleDLL.dll");
	PVOID pHelloWorld = GetProcAddress(hModule, "HelloWorld");
	// Type-casting the 'pHelloWorld' variable to be of type 'HelloWorldFunctionPointer' 
	HelloWorldFunctionPointer HelloWorld = (HelloWorldFunctionPointer)pHelloWorld;
	HelloWorld(); 
	// Calling the 'HelloWorld' function via its function pointer
}
```
### Dynamic Linking Example
Đoạn mã dưới đây minh họa một ví dụ đơn giản khác về dynamic linking, trong đó hàm `MessagenBoxA` được gọi. Đoạn mã giả định rằng `user32.dll`, DLL export hàm này, chưa được load vào bộ nhớ. Như đã đề cập, nếu một DLL chưa được load vào bộ nhớ cần sử dụng `LoadLibrary` để load DLL đó vào address space của tiến trình.
```c
typedef int (WINAPI* MessageBoxAFunctionPointer)( // Constructing a new data type, that will represent MessageBoxA's function pointer 
  HWND          hWnd,
  LPCSTR        lpText,
  LPCSTR        lpCaption,
  UINT          uType
);

void call(){
    // Retrieving MessageBox's address, and saving it to 'pMessageBoxA' (MessageBoxA's function pointer)
    MessageBoxAFunctionPointer pMessageBoxA = (MessageBoxAFunctionPointer)GetProcAddress(LoadLibraryA("user32.dll"), "MessageBoxA");
    if (pMessageBoxA != NULL){
        // Calling MessageBox via its function pointer if not null    
        pMessageBoxA(NULL, "MessageBox's Text", "MessageBox's Caption", MB_OK); 
    }
}
```
### Function Pointers
Các kiểu dữ liệu function pointer sẽ sử dụng quy ước đặt tên dựa trên của WinAPI, với tiền tố `fn`, trong đó `fn` là viết tắt của function pointer.
Ví dụ, kiểu dữ liệu `MessageBoxAFunctionPointer` ở trên sẽ được biểu diễn dưới dạng `fnMessageBoxA`. Quy ước này được sử dụng nhằm duy trì tính đơn giản và cải thiện khả năng đọc hiểu.
### Rundll32.exe
Có một số cách để chạy các hàm được export mà không cần sử dụng phương pháp lập trình. Một kỹ thuật phổ biến là sử dụng binary `rundll32.exe`.
`Rundll32.dll` là một Windows binary được tích hợp sẵn được sử dụng để chạy một hàm được export từ tệp DLL. Để chạy một hàm được export, sử dụng lệnh:
```c
rundll32.exe <dllname>, <function exported to run>
```
Ví dụ, `User32.dll` export hàm `LockWorkStation`, hàm này có chức năng khóa máy. Để chạy hàm này, sử dụng lệnh:
```c
rundll32.exe user32.dll,LockWorkStation
```
### Creating a DLL File With Visual Studio
Để tạo một tệp DLL, khởi chạy Visual Studio và tạo một project mới. Khi xuất hiện danh sách project templates, chọn tùy chọn `Dynamic-Link Library (DLL)`:
![[Pasted image 20260826164742.png]]
Tiếp theo, chọn vị trí lưu các tệp của project. Sau khi hoàn tất, đoạn mã C sau sẽ xuất hiện.
![[Pasted image 20260827103332.png]]

Templete DLL được cung cấp sẵn đi kèm với các tệp `framework.h`, `pch.h` và `pch.cpp` được gọi là Precompiled Header. Đây là các tệp được sử dụng nhằm tăng tốc quá trình biên dịch project, đặc biệt với các project có quy mô lớn. Trong trường hợp này, nhiều khả năng các tệp không cần thiết, do đó nên xóa các tệp này. để thực hiện, chọn tệp cần xóa, nhấn phím Delete sau đó chọn tùy chọn **`Delete`**.
![[Pasted image 20260827103909.png]]
![[Pasted image 20260827104102.png]]
Sau khi xóa các Precompiled Headers, cần thay đổi các thiết lập mặc định của compiler để xác nhận rằng Precompiled Headers không được sử dụng trong project.
![[Pasted image 20260827104240.png]]

![[Pasted image 20260827104324.png]]

Thay đổi tùy chọn **`Precompiled Header`** thành **`Not Using Precompiled Headers`**, sau đó nhấn **`Apply`**.
![[Pasted image 20260827104448.png]]

Cuối cùng, thay đổi tệp `dllmain.cpp` thành `dllmain.c`. Để biên dịch chương trình, chọn `Build > Build Solution`. Sau đó, một tệp DLL sẽ được tạo trong thư mục `Release` hoặc `Debug`, tùy thuộc vào cấu hình biên dịch.
![[Pasted image 20260827104720.png]]
