### Introduction
**Portable Executable (PE)** là định dạng tệp được sử dụng cho các tệp thực thi trên Windows. Một số phần mở rộng của tệp PE bao gồm `.exe`, `.dll`, `.sys` và `.scr`. 
### PE Structure
Sơ đồ bên dưới minh họa một cấu trúc đơn giản hóa của Portable Executable. Mỗi header được thể hiện trong hình đều được định nghĩa dưới dạng một cấu trúc dữ liệu (data structure) chứa thông tin về tệp PE. ![[Pasted image 20260824171817.png|center]]
#### DOS Header (IMAGE_DOS_HEADER)
Header đầu tiên của tệp PE luôn bắt đầu bằng hai byte `0x4D` và `0x5A`, thường được gọi là MZ. Hai byte này đại diện cho chữ ký DOS header (DOS header signature), được sử dụng để xác nhận rằng tệp đang được phân tích hoặc kiểm tra là một PE hợp lệ. DOS header là một cấu trúc dữ liệu được định nghĩa như sau:
```C
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // Offset to the NT header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```
Các thành phần quan trọng nhất của cấu trúc này là `e_magic` và `e_lfanew`.
- `e_magic` có kích thước 2 byte với giá trị cố định là `0x5A4D` hoặc MZ.
- `e_lfanew` là một giá trị có kích thước 4 byte, chứa offset tới một vị trí bắt đầu của NT Header. Lưu ý rằng `e_lfanew` luông nằm tại offset `0x3C`.![[Pasted image 20260824173226.png|center|650]]
#### DOS Stub
Trước khi chuyển sang cấu trúc NT Header, có DOS Stub. Đây là một thông báo lỗi hiển thị dòng "This program cannot be run in DOS mode" trong trường trường hợp chương trình được tải trong DOS mode hoặc "Disk Operating Mode". Điều đáng lưu ý là thông báo lỗi này có thể được thay đổi bởi lập trình viên tại thời điểm biên dịch tệp PE, tuy nhiên thông báo mặc định này vẫn thường được giữ nguyên.
![[Pasted image 20260824173416.png||border|650]]

#### NT Header (IMAGE_NT_HEADERS)
NT Header rất quan trọng vì nó bao gồm hai image header khác là `FileHeader` và `OptionalHeader`, chứa một lượng lớn thông tin về tệp PE. Tương tự như DOS Header, NT Header cũng chứa signature member, được sử dụng để xác thực nó. Thông thường, thành phần  signature này bằng chuỗi `"PE"` được biểu diễn bởi hai byte `0x50` và `0x45`. Tuy nhiên, vì signature có kiểu dữ liệu `DWORD` nên nó sẽ được biểu diễn dưới dạng `0x50450000`, vẫn là `"PE"`, nhưng được đệm thêm hai byte `0`. Có thể truy cập NT Header bằng cách sử dụng thành viên `e_lfanew` bên trong DOS Header.
Cấu trúc NT Header thay đổi tùy thuộc vào kiến trúc máy tính
**32-bit Version:**
```C
typedef struct _IMAGE_NT_HEADERS {
  DWORD                   Signature;
  IMAGE_FILE_HEADER       FileHeader;
  IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

**64-bit Version:**
```C
typedef struct _IMAGE_NT_HEADERS64 {
    DWORD                   Signature;
    IMAGE_FILE_HEADER       FileHeader;
    IMAGE_OPTIONAL_HEADER64 OptionalHeader;
} IMAGE_NT_HEADERS64, *PIMAGE_NT_HEADERS64;
```
Điểm khác biệt duy nhất nằm tại cấu trúc dữ liệu `OptionalHeader`, cụ thể là `IMAGE_OPTIONAL_32` và  `IMAGE_OPTIONAL_HEADER64`.![[Pasted image 20260824173948.png|border|650]]

#### File Header (IMAGE_FILE_HEADER)
Tiếp theo là File Header, có thể được truy cập từ cấu trúc dữ liệu NT Header:
```C
typedef struct _IMAGE_FILE_HEADER {
  WORD  Machine;
  WORD  NumberOfSections;
  DWORD TimeDateStamp;
  DWORD PointerToSymbolTable;
  DWORD NumberOfSymbols;
  WORD  SizeOfOptionalHeader;
  WORD  Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```
Các thành viên quan trọng nhất của cấu trúc này là:
- `NumberOfSections`: Số lượng section trong tệp PE
- `Characteristics`: Các flags xác định một số thuộc tính  của tệp thực thi, chẳng hạn như tệp đó là dll hay ứng dụng console.
- `SizeOfOptionalHeader`: kích thước Optional Header nằm ngay trong đó.![[Pasted image 20260824175118.png|center|border|650]]
#### Optional Header (IMAGE_OPTIONAL_HEADER)
**Optional Header** là một thành phần quan trọng. Mặc dù có tên là "optional", nó lại thiết yếu đối với quá trình thực thi của tệp PE. Nó được gọi là optional vì một số loại tệp không chứa header này.
Optional Header có hai phiên bản, một phiên bản dành cho hệ thống 32-bit, một phiên bản dành cho hệ thống 64-bit.  Cả hai phiên bản có các thành viên trong cấu trúc dữ liệu gần như giống nhau, với điểm khác biệt chính nằm ở kích thước của một số thành viên. `ULONGLONG` được sử dụng trong phiên bản 64-bit, trong khi `DWORD` được sử dụng trong phiên bản 32-bit. Ngoài ra phiên bản 32-bit có một số thành viên không tồn tại trong phiên bản 64-bit.
32-bit Version:
```C
typedef struct _IMAGE_OPTIONAL_HEADER {
  WORD                 Magic;
  BYTE                 MajorLinkerVersion;
  BYTE                 MinorLinkerVersion;
  DWORD                SizeOfCode;
  DWORD                SizeOfInitializedData;
  DWORD                SizeOfUninitializedData;
  DWORD                AddressOfEntryPoint;
  DWORD                BaseOfCode;
  DWORD                BaseOfData;
  DWORD                ImageBase;
  DWORD                SectionAlignment;
  DWORD                FileAlignment;
  WORD                 MajorOperatingSystemVersion;
  WORD                 MinorOperatingSystemVersion;
  WORD                 MajorImageVersion;
  WORD                 MinorImageVersion;
  WORD                 MajorSubsystemVersion;
  WORD                 MinorSubsystemVersion;
  DWORD                Win32VersionValue;
  DWORD                SizeOfImage;
  DWORD                SizeOfHeaders;
  DWORD                CheckSum;
  WORD                 Subsystem;
  WORD                 DllCharacteristics;
  DWORD                SizeOfStackReserve;
  DWORD                SizeOfStackCommit;
  DWORD                SizeOfHeapReserve;
  DWORD                SizeOfHeapCommit;
  DWORD                LoaderFlags;
  DWORD                NumberOfRvaAndSizes;
  IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

**64-bit Version:**
```C
typedef struct _IMAGE_OPTIONAL_HEADER64 {
  WORD                 Magic;
  BYTE                 MajorLinkerVersion;
  BYTE                 MinorLinkerVersion;
  DWORD                SizeOfCode;
  DWORD                SizeOfInitializedData;
  DWORD                SizeOfUninitializedData;
  DWORD                AddressOfEntryPoint;
  DWORD                BaseOfCode;
  ULONGLONG            ImageBase;
  DWORD                SectionAlignment;
  DWORD                FileAlignment;
  WORD                 MajorOperatingSystemVersion;
  WORD                 MinorOperatingSystemVersion;
  WORD                 MajorImageVersion;
  WORD                 MinorImageVersion;
  WORD                 MajorSubsystemVersion;
  WORD                 MinorSubsystemVersion;
  DWORD                Win32VersionValue;
  DWORD                SizeOfImage;
  DWORD                SizeOfHeaders;
  DWORD                CheckSum;
  WORD                 Subsystem;
  WORD                 DllCharacteristics;
  ULONGLONG            SizeOfStackReserve;
  ULONGLONG            SizeOfStackCommit;
  ULONGLONG            SizeOfHeapReserve;
  ULONGLONG            SizeOfHeapCommit;
  DWORD                LoaderFlags;
  DWORD                NumberOfRvaAndSizes;
  IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;
```
Optional Header chứa rất nhiều thông tin có thể sử dụng trong quá trình phân tích PE. Dưới đây là một số thành phần của cấu trúc thường được sử dụng:
- `Magic`: mô tả kiến trúc của image file, xác định image là 32-bit hay 64-bit![[Pasted image 20260824175851.png|center|border|600]]
-  `MajorOperatingSystemVersion`: số phiên bản major của hệ điều hành được yêu cầu ví dụ `11`, `10`.
- `MinorOperatingSystemVersion`: số phiên bản minor của hệ điều hành được yêu cầu, ví dụ `1511`, `1507`, `1607`.
- `SizeOfCode`: kích thước của section `.text`
- `AddressOfEntryPoint`: offset tới entry point của tệp, thường là hàm `main`.
- `BaseOfCode`: offset tới vị trí bắt đầu của section `.text`
- `SizeOfImage`: Kích thước của image file tính bằng byte.
- `ImageBase`: Xác định địa chỉ ưu tiên mà tại đó ứng dụng sẽ được nạp vào bộ nhớ khi thực thi. Tuy nhiên, do các cơ chế bảo ệ bộ nhớ của Windows như Address Space Layout Randomization (ASLR), rất hiếm một image được ánh xạ vào đúng địa chỉ ưu tiên của nó, vì Windows PE Loader sẽ ánh xạ tệp vào một địa chỉ khác. Việc Windows PE Loader cấp phát địa chỉ ngẫu nhiên này có thể gây ra vấn đề trong quá trình triển khai các kỹ thuật ở phần sau, vì một số địa chỉ được xem là cố định đã bị thay đổi. Windows PE Loader sau đó sẽ thực hiện **PE relocation** để điều chỉnh các địa chỉ này.
- `DataDirectory`: một trong những thành phần quan trọng nhất của Optional Header. Đây là một mảng `IMAGE_DATA_DIRECTORY` chứa các directory trong một tệp PE.
##### Data Directory
Data Directory có thể được truy cập thông qua thành phần cuối cùng của Optional Header. Đây là một mảng có kiểu dữ liệu là `IMAGE_DATA_DIRECTORY`, với cấu trúc dữ liệu như sau:
```C
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;
    DWORD   Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```
Mảng Data Directory có kích thước `IMAGE_NUMBEROF_DIRECTORY_ENTRIES` là một hằng số có giá trị `16`. Một phần tử trong mảng đại diện cho một data directory cụ thể, chứa một số thông tin về một PE Section hoặc một Data Table (nơi lưu trữ các thông tin cụ thể về PE). Có thể truy cập một data directory cụ thể bằng cách sử dụng index của nó trong mảng.
```C
#define IMAGE_DIRECTORY_ENTRY_EXPORT          0   // Export Directory
#define IMAGE_DIRECTORY_ENTRY_IMPORT          1   // Import Directory
#define IMAGE_DIRECTORY_ENTRY_RESOURCE        2   // Resource Directory
#define IMAGE_DIRECTORY_ENTRY_EXCEPTION       3   // Exception Directory
#define IMAGE_DIRECTORY_ENTRY_SECURITY        4   // Security Directory
#define IMAGE_DIRECTORY_ENTRY_BASERELOC       5   // Base Relocation Table
#define IMAGE_DIRECTORY_ENTRY_DEBUG           6   // Debug Directory
#define IMAGE_DIRECTORY_ENTRY_ARCHITECTURE    7   // Architecture Specific Data
#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR       8   // RVA of GP
#define IMAGE_DIRECTORY_ENTRY_TLS             9   // TLS Directory
#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG    10   // Load Configuration Directory
#define IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT   11   // Bound Import Directory in headers
#define IMAGE_DIRECTORY_ENTRY_IAT            12   // Import Address Table
#define IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT   13   // Delay Load Import Descriptors
#define IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR 14   // COM Runtime descriptor
```
![[Pasted image 20260824181505.png]]
Hai phần dưới đây sẽ trình bày ngắn gọn về hai data directory quan trọng, đó là `Export Directory` và `Import Address Table`.
- **Export Directory** của một PE là mộ cấu trúc dữ liệu chứa thông tin về các hàm và biến được export từ tệp thực thi. Nó chứa địa chỉ của hàm và biến được export, cho phép các tệp thwujc thi khác truy cập tới các hàm và dữ liệu này. Export Directory thường được tìm thấy trong các DLL thực hiện export hàm, ví dụ `kernel32.dll` export hàm `CreateFileA`.
- **Import Address Table (IAT)** là một cấu trúc dữ liệu trong PE chứa thông tin về địa chỉ của các hàm được import từ các tệp thực thi khác. Các địa chỉ này được sử dụng để truy cập tới các hàm và dữ liệu trong những tệp thực thi khác, ví ví dụ `Application.exe` import hàm `CreateFileA` từ `kernel32.dll`.
#### PE Sections
PE Section chứa mã lệnh và dữ liệu được sử dụng để tạo nên một chương trình thực thi. Mỗi PE section được gán một tên riêng và thường chứa mã lệnh thực thi, dữ liệu hoặc thông tin tài nguyên. Không có một số lượng PE section cố định vì các compiler khác nhau có thể thêm, loại bỏ hoặc hợp nhất các section tùy thuộc vào cấu hình. Một section cũng có thể thêm thủ công sau đó. Do đó, số lượng section mang tính dynamic và `IMAGE_FILE_HEADER.NumberOfSections` được sử dụng để xác định số lượng section đó.
Các PE section dưới đây là những section quan trọng nhất và tồn tại trong hầu hết các PE:
- `.text`: Chứa mã lệnh thực thi, tức mã nguồn đã được biên dịch thành mã máy.
- `.data`: Chứa dữ liệu đã được khởi tạo, chẳng hạn như các biến được khởi tạo trong mã chương trình.
- `.rdata`: Chứa dữ liệu read-only. Đây là các biến hằng được khai báo với tiền tố `const`.
- `.idata`: Chứa các bảng import. Đây là các bảng chứ thông tin liên quan đến những hàm được gọi trong mã chương trình. Windows PE Loader sử dụng thông tin này để xác định các tệp DLL cần được tải vào tiến trình, cùng với các hàm được sử dụng từ mỗi DLL.
- `.reloc`: Chứa thông tin về cách điều chỉnh các địa chỉ bộ nhớ, để chương trình có thể được tải vào bộ nhớ mà không xảy ra lỗi.
- `.rsrc`: Được sử dụng để lữu trữ các tài nguyên như icon và bitmap.
 Mỗi PE section có một cấu trúc dữ liệu `IMAGE_SECTION_HEADER` chứa thông tin quan trọng về section đó. Các cấu trúc này được lưu bên dưới NT Headers trong tệp PE và được xếp liên tiếp với nhau, trong đó mỗi cấu trúc đại diện cho một section. 
```C
typedef struct _IMAGE_SECTION_HEADER {
  BYTE  Name[IMAGE_SIZEOF_SHORT_NAME];
  union {
    DWORD PhysicalAddress;
    DWORD VirtualSize;
  } Misc;
  DWORD VirtualAddress;
  DWORD SizeOfRawData;
  DWORD PointerToRawData;
  DWORD PointerToRelocations;
  DWORD PointerToLinenumbers;
  WORD  NumberOfRelocations;
  WORD  NumberOfLinenumbers;
  DWORD Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

Khi xem xét các thành phần của cấu trúc, có thể thấy tất cả các thành phần đều có giá trị và vai trò quan trọng:
- `Name` :Tên của section, ví dụ `.text`, `.data`, `.rdata`.
- `PhysicalAddress` hoặc `VirtualSize`: Kích thước của section khi được tải vào bộ nhớ.
- `VirtualAddress`:Offset tới vị trí bắt đầu của section trong bộ nhớ.
