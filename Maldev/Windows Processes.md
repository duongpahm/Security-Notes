### What is a Windows Process?
Windows Process là một chương trình hoặc ứng dụng đang chạy trên máy Windows. Một process có thể được khởi chạy bởi người dùng hoặc bởi chính hệ thống. Process sử dụng các tài nguyên như bộ nhớ, dung lượng ổ đĩa và thời gian CPU để hoàn thành một tác vụ.
### Process Threads
Windows process được cấu thành từ một hoặc nhiều thread và các thread này có thể chạy đồng thời. Một thread là một tập các chỉ thị có thể được thực thi độc lập bên trong một process. Các thread trong cùng một process có thể giao tiếp và chia sẻ dữ liệu với nhau. Thread được hệ điều hành lập lịch thực thi và được quản lý trong ngữ cảnh của một process.
### Process Memory
Windows process cũng sử dụng bộ nhớ để lưu trữ dữ liệu và các instruction. Bộ nhớ này được cấp phát cho process khi process được tạo và dung lượng bộ nhớ được cấp phát có thể được thiết lập bởi chính process.
Hệ điều hành quản lý bộ nhớ bằng cả virtual memory và physical memory. Virtual memory cho phép hệ điều hành sử dụng nhiều bộ nhớ hơn lượng bộ nhớ vật lý thực tế hiện có bằng tạo ra một virtual address space mà các ứng dụng có thể truy cập. Các virtual address space này được chia thành các page sau đó được cấp phát cho các process.
### Memory Types
Process có thể nhiều loại bộ nhớ khác nhau
- Private memory: dành riêng cho một process duy nhất và không thể được chia sẻ bởi các process khác. Loại bộ nhớ này được sử dụng để lưu trữ đặc thù của process.
- Mapped memory: có thể được chia sẻ giữa hai hoặc nhiều process. Nó được sử dụng để chia sẻ dữ liệu giữa các process, chẳng hạn như shared libraries, shared memory segments và shared files. Mapped memory có thể được các process khác nhìn thấy nhưng được bảo vệ khỏi việc các process khác sửa đổi.
- Image memory: chứa code và data của một executable file. Nó được sử dụng để lưu trữ code và data mà process sử dụng, chẳng hạn như program code, data và resources. Image memory thường liên quan đến các DLL được load vào address space của process.
### Process Environment Block (PEB)
Process Environment Block (PEB) là một cấu trúc dữ liệu trong Windows chứa thông tin về một process, chẳng hạn như parameters, startup information, thông tin về các heap đã được cấp phát và các DLL đã được load, cùng nhiều thông tin khác.
PEB được hệ điều hành sử dụng để lưu trữ thông tin về các process trong quá trình chúng đang chạy, đồng thời được Windows Loader sử dụng để khởi chạy các ứng dụng.
PEB cũng lưu trữ các thông tin về process như Process ID và đường dẫn đến executable. Mỗi process được tạo ra đều có PEB riêng, chứa tập hợp thông tin riêng của process đó.
### PEB Structure
Cấu trúc PEB trong C được thể hiện như dưới đây. Các thành phần được đánh dấu là **Reserved** có thể được bỏ qua.
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
Các thành phần không được đánh dấu Reserved được giải thích dưới đây.
#### BeingDebugged
`BeingDebugged` là một flag trong cấu trúc PEB, cho biết process có đang được debug hay không. Giá trị của nó được thành `1 (TRUE)` khi process đang được debug và `0 (FALSE)` khi process không được debug.
Thành phần này được Windows Loader sử dụng để xác định liệu ứng dụng có được khởi chạy với debugger được attach hay không.
#### Ldr
`Ldr` là một pointer trỏ tới cấu trúc `PEB_LDR_DATA` bên trong Process Environment Block. Cấu trúc này chứa thông tin về các dynamic-link library module đã được load của process. Nó bao gồm danh sách các DLL được load trong process, base address của từng DLL và kích thước của từng module. Windows Loader sử dụng cấu trúc này để theo dõi các DLL đã được load trong process. Cấu trúc `PEB_LDR_DATA` như dưới đây:
```c
typedef struct _PEB_LDR_DATA {
  BYTE       Reserved1[8];
  PVOID      Reserved2[3];
  LIST_ENTRY InMemoryOrderModuleList;
} PEB_LDR_DATA, *PPEB_LDR_DATA;
```
`Ldr` có thể được sử dụng để tìm base address của một DLL cụ thể, cũng xác định những hàm nằm trong vùng bộ nhớ của DLL đó.
#### ProcessParameters
`ProcessParameters` là một cấu trúc dữ liệu trong PEB chứa các command-line parameters được truyền vào process khi process được tạo. Windows Loader thêm các parameters này vào cấu trúc PEB của process. `ProcessParameters` là một con trỏ tới cấu trúc `RTL_USER_PROCESS_PARAMETERS`:
```c
typedef struct _RTL_USER_PROCESS_PARAMETERS {
  BYTE           Reserved1[16];
  PVOID          Reserved2[10];
  UNICODE_STRING ImagePathName;
  UNICODE_STRING CommandLine;
} RTL_USER_PROCESS_PARAMETERS, *PRTL_USER_PROCESS_PARAMETERS;
```
`ProcessParameters` sẽ được sử dụng trong các kỹ thuật như command-line spoofing.
#### AtlThunkSListPtr & AtlThunkSListPtr32
`AtlThunkSListPtr` và `AtlThunkSListPtr32`được module ATL (Active Template Library) sử dụng để lưu một pointer tới linked list của các thunking function. Thunking function được sử dụng để gọi các hàm được triển khai trong một address space khác. Các hàm này thường đại diện cho những hàm được export từ một tệp DLL. Linked list của các thunking function được ATL sử dụng để quản lý quá trình thunking.
#### PostProcessInitRoutine
Trường `PostProcessInitRoutine` trong cấu trúc PEB được sử dụng để lưu pointer tới một hàm được hệ điều hành gọi sau khi quá trình TLS (Thread Local Storage) initialization đã hoàn tất đối với tất cả các thread trong process. Hàm này có thể được sử dụng để thực hiện các tác vụ initialization bổ sung cần thiết cho process. 
#### SessionId
`SessionId` trong PEB là một định danh duy nhất được gán cho một session duy nhất. Nó được sử dụng để theo dõi hoạt động của người dùng trong session.
### Thread Environment Block (TEB)
Thread Environment Block (TEB) là một cấu trúc dữ liệu trong Windows dùng để lưu trữ thông tin về một thread. Nó chứa môi trường của thread, security context và các thông tin liên quan khác. TEB được lưu trữ trong stack của thread và được Windows kernel sử dụng để quản lý các thread.
### TEB Structure
Cấu trúc TEB trong C được thể hiện bên dưới. Các thành phần được đánh dấu là Reserved có thể được bỏ qua.
```c
typedef struct _TEB {
  PVOID Reserved1[12];
  PPEB  ProcessEnvironmentBlock;
  PVOID Reserved2[399];
  BYTE  Reserved3[1952];
  PVOID TlsSlots[64];
  BYTE  Reserved4[8];
  PVOID Reserved5[26];
  PVOID ReservedForOle;
  PVOID Reserved6[4];
  PVOID TlsExpansionSlots;
} TEB, *PTEB;
```
#### ProcessEnvironmentBlock (PEB)
Đây là một pointer trỏ tới cấu trúc PEB. PEB nằm bên trong Thread Environment Block và được sử dụng để lưu trữ thông tin về process đang chạy hiện tại.
#### TlsSlots
TLS (Thread Local Storage) Slots là các vị trí trong TEB được sử dụng để lưu trữ dữ liệu dành riêng cho từng thread (thread-specific data). Mỗi thread trong Windows có TEB riêng, và mỗi TEB có tập hợp các TLS slots. Ứng dụng có thể sử dụng các slot này để lưu trữ dữ liệu thuộc về thread đó, chẳng hạn như thread-specific variables, thread-specific handles, thread-specific states, v.v.
#### TlsExpansionSlots
TLS Expansion Slots trong TEB là một tập hợp các pointer được sử dụng để lưu trữ thread-local storage data của một thread. Các TLS Expansion Slots được dánh riêng cho việc sử dụng bởi systems DLLs.
### Process And Thread Handles
Trên hệ điều hành Windows, mỗi process có một process identifier, hay Process ID riêng biệt, được hệ điều hành cấp phát khi process được tạo, PID được sử dụng để phân biệt một process đang chạy với các process khác.
Khái niệm tương tự cũng áp dụng cho thread đang chạy. Mỗi thread đang chạy có một ID duy nhất, được sử dụng để phân biệt nó với các thread khác đang tồn tại trên hệ thống, bất kể thread đó thuộc process nào. 
Các identifier này có thể được sử dụng để mở handle tới một process hoặc thread thông qua các WinAPI:
- `OpenProcess`: mở handle tới một process object hiện có thông qua identifier của process.
- `OpenThread`: mở handle tới một thread object hiện có thông qua identifier của thread.
Handle đã mở có thể được sử dụng để thực hiện các thao tác tiếp theo với Windows object tương ứng, chẳng hạn như suspend một process hoặc thread. Handle luôn phải được đóng sau khi không còn cần sử dụng để tránh xảy ra handle leaking. Việc này được thực hiện bằng cách gọi WinAPI `CloseHandle`.
