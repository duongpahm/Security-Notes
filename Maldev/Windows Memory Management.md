## Virtual Memory & Paging
Trong các hệ điều hành hiện đại, bộ nhớ không được ánh xạ trực tiếp bộ nhớ vật lý (tức là RAM). Thay vào đó, các tiến trình sử dụng địa chỉ bộ nhớ ảo (virtual memory addresses), sau đó các địa chỉ này được ánh xạ tới địa chỉ bộ nhớ vật lý (physical memory addresses). Có nhiều lý do cho cơ chế này, nhưng mục tiêu cuối cùng là tôi ưu và tiết kiệm tối đa bộ nhớ vật lý. Bộ nhớ ảo có thể được ánh xạ tới bộ nhớ vật lý nhưng cũng có thể được lưu trữ trên ổ đĩa, Nhờ cơ chế địa chỉ hóa bộ nhớ ảo (virtual memory addressing), nhiều tiến trình có thể cùng chia sẻ một địa chỉ vật lý, trong khi mỗi tiến trình vẫn sở hữu một địa chỉa bộ nhớ ảo duy nhất. Bộ nhớ ảo dựa trên khái niệm Memory Paging, trong đó bộ nhớ được chia thành các khối có kích thước 4KB được gọi là page.![[Pasted image 20260824082449.png|center]]
## Page State
Các trang nằm trong không gian địa chỉ ảo của một tiến trình có thể ở trong một trong 3 trạng thái:
1. Free: trang chưa được commit cũng chưa được reserve. Trang không thể được tiến trình truy cập. Trang này có thể được reserve, commit, hoặc đồng thời reserve và commit. Việc cố gắng đọc từ hoặc ghi vào mộ trang có trạng thái Free có thể dẫn đến ngoiaj lệ vi phạm quyền truy cập.
2. Reserved: trang đã được đặt trước để sử dụng trong tương lai. Phạm vi địa chỉ của trang không thể được sử dụng bởi các hàm cấp phát khác. Trang không thể được truy cập và chưa có bộ nhớ vật lý được liên kết với nó. Trang này có thể được chuyển sang trạng thái commited.
3. Commited: dung lượng bộ nhớ đã được cấp phát từ tổng dung lượng RAM và paging file trên ổ đĩa. Trang có thể được truy cập và quyền truy cập được kiểm soát bởi một trong những các hằng số bảo vệ bộ nhớ. Hệ thống chỉ khởi tạo và tải mỗi trang đã commited vào bộ nhớ vật lý khi có lần đầu tiên tiến trình cố gắng đọc hoặc ghi vào trang đó. Khi tiến trình kết thúc, hệ thống sẽ giải phóng vùng lữu trữ được cấp phát cho các trang đã commited.
## Page Protection Options
Sau khi các trang đã được commited, chúng cần được thiết lập tùy chọn bảo vệ. Danh sách các hằng số bảo vệ bộ nhớ có thể được tìm thấy tại đây, một số ví dụ được liệt kê bên dưới:
- `PAGE_NOACCESS`: vô hiệu hóa toàn bộ quyền truy cập đối với vùng trang đã commited. Việc cố gắng read, write hoặc execute trên vùng bộ nhớ này sẽ dẫn đến vi phạm quyền truy cập.
- `PAGE_EXECUTE_READWRITE`: Cho phép Read, Write và Execute. Tùy chọn này rất không khuyến khích sử dụng và nhìn chung một IoC (Indicaotr of Compromise) vì việc một vùng bộ nhớ đồng thời có quyền read và execute là không phổ biến.
- `PAGE_READONLY`: Cho phép read-only đối với vùng trang đã commited. Việc cố gắng ghi vào vùng bộ nhớ này sẽ dẫn đến vi phạm quyền truy cập.
## Memory Protection
Các hệ điều hành hiện đại nhìn chung đều được tích hợp các cơ chế bảo vệ bộ nhớ nhằm ngăn chặn các hoạt động khai thác lỗ hổng và tấn công. Đây cũng là những cơ chế quan trọng cần lưu ý vì chúng có khả năng sẽ xuất hiện trong quá trình xây dựng hoặc debug malware.
- Data Execution Preventio (DEP): là một tính năng bảo vệ bộ nhớ ở cấp hệ thống, được tích hợp vào hệ điều hành kể từ Windows XP và Windows Server 2003. Nếu tùy chọn bảo vệ trang được thiết lập thành `PAGE_READONLY`, DEP sẽ ngăn không cho code được thực thi trong vùng bộ nhớ đó.
- Address Space Layout Randomization (ASLR): là một kỹ thuật bảo vệ bộ nhớ được sử dụng nhằm ngăn chặn việc khai thác các lỗ hổng gây hỏng bộ nhớ (memory corruption vulnerabilities). ASLR thực hiện ngẫu nhiên hóa vị trí trong không gian địa chỉ của các vùng dữ liệu quan trọng trong một tiến trình, bao gồm địa chỉ cơ sở của executable, cũng như vị trí của stack, heap, và các thư viện.
## x86 vs x64 Memory Space
Khi làm việc với các tiến trình Windows, cần xác định tiến trình đang sử dụng kiến trúc x86 hay x64. Các tiến trình x86 có không gian bộ nhớ nhỏ hơn, ở mức 4GB (`0xFFFFFFFF`), trong khi x64 có không gian bộ nhớ lớn hơn rất nhiều lên tới 128TB (`0xFFFFFFFFFFFF`).
### Allocating Memory Example 
Ví dụ dưới đây trình bày một đoạn mã nguồn nhỏ nhằm giúp hiểu rõ hơn cách tương tác với bộ nhớ Windows thông qua các hàm C và Windows API. Bước đầu tiên khi tương tác với bộ nhớ là cấp phát bộ nhớ. Đoạn mã dưới đây minh họa một số phương thức để cấp phát bộ nhớ, về bản chất là dánh trước một vùng nhớ bên tỏng tiến trình đang chạy:

```
// Allocating a memory buffer of 100 bytes

// Method 1 - Using malloc()
PVOID pAddress = malloc(100);

// Method 2 - Using HeapAlloc()
PVOID pAddress = HeapAlloc(GetProcessHeap(), 0, 100);

// Method 3 - Using LocalAlloc()
PVOID pAddress = LocalAlloc(LPTR, 100);
```

Các hàm cấp phát bộ nhớ trả về địa chỉ cơ sở, về bản chất là một con trỏ trỏ tới vị trí bắt đầu của khối bộ nhớ đã được cấp phát. Với các đoạn mã trên, `pAddress` sẽ chứa địa chỉ cơ sở của khối bộ nhớ đã được cấp phát. Thông qua con trỏ này có thể thực hiên nhiều thao tác khác nhau như read, write và execute. Loại thao tác có thể thực hiện phụ thuộc vào quyền bảo vệ được thiết lập cho vùng bộ nhớ đã cấp phát.
![[Pasted image 20260824094852.png]]
Khi bộ nhớ được cấp, vùng nhớ đó có thể trống hoặc chứa dữ liệu ngẫu nhiên. Một số hàm cấp phát bộ nhớ cung cấp tùy chọn đặt toàn bộ vùng nhớ về giá trị 0 (zero out) ngay trong quán trình cấp phát.
![[Pasted image 20260824095541.png]]
### Writing To Memory Example
Bước tiếp theo sau khi cấp phát bộ nhớ thường ghi dữ liệu vào vùng đệm đó. Có nhiều phương thức có thể được sử dụng để ghi dữ liệu vào bộ nhớ, nhưng trong ví dụ này, hàm `memcpy` được sử dụng:

```
PVOID pAddress = HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, 100);

CHAR* cString = "Hello World!";

memcpy(pAddress, cString, strlen(cString));
```
`HeapAlloc` sử dụng cở `HEAP_ZERO_MEMORY`, khiến vùng bộ nhớ được cấp phát được khởi tạo với giá trị 0. Sau đó, chuỗi được sao chép vào vùng bộ nhớ đã cấp phát bằng hàm `memcpy`. Tham số cuối cùng của `memcpy` là số byte cần được sao chép. Tiếp theo, kiểm tra lại buffer để xác minh rằng dữ liệu đã được ghi thành công.
![[Pasted image 20260824100551.png]]
### Freeing Allocated Memory
Khi ứng dụng hoàn tất việc sử dụng một buffer đã được cấp phát. nên deallocate hoặc giải phóng vùng đệm đso để tránh xảy ra memory leak. Tùy thuộc vào hàm được sử dụng để cấp phát bộ nhớ, sẽ có một hàm giải phóng bộ nhớ tương ứng. Ví dụ:
- Cấp phát bằng `malloc` yêu cầu sử dụng hàm `free`.
- Cấp phát bằng `HeapAlloc` yêu cầu sử dụng hàm `HeapFree`.
- Cấp phát bằng `LocalAlloc` yêu cầu sử dụng hàm `LocalFree`.

![[Pasted image 20260824105032.png]]

![[Pasted image 20260824105720.png]]