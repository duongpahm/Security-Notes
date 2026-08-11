Các cuộc tấn công phía client sử dụng file container là một phương thức phân phối mã độc có tính ẩn mình cao và hiệu quả trong việc đạt được việc thực thi mã. Bằng cách khai thác hành vi của các tệp nhị phân được Microsoft ký số và cách chúng xử lý quá trình nạp thư viện DLL, các attacker có thể vượt qua các cơ chế phòng vệ phổ biến như Mark-of-the-Web (MotW) và thực thi mã độc trên hệ thống.
Trước tiên chúng ta sẽ xác định các tệp nhị phân được Microsoft ký số phù hợp, những tệp không bị ảnh hưởng bởi các hạn chế của MotW khi được giải nén từ một tệp Zip. Các tệp nhị phân này có thể bị lợi dụng để nạp các DLL độc hại được đặt trong cùng thư mục. Sau khi xác định được mục tiêu phù hợp, chúng ta sẽ chuyển sang xây dựng một proxy DLL giả mạo.
Cuối cùng, chúng ta sẽ mô phổng một kịch bản tấn công hoàn chỉnh bằng cách đóng gói tệp nhị phân đã được ký số cùng với proxy DLL độc hại vào một tệp ZIP. Khi nạn nhân giải nén và thực thi tệp nhị phân, chương trình sẽ kích hoạt payload, đồng thời vẫn tiếp tục hoạt động bình thường nhằm duy trì sự tin cậy của người dùng và tránh bị phát hiện.
## DLL Sideloading using Signed Microsoft Binaries
Chúng ta sẽ tìm hiểu cách thực hiện các cuộc tấn công DLL sideloading bằng cách lợi dụng các tệp nhị phân đáng tin cậy đã được Microsoft ký số. Trước tiên, chúng ta sẽ xác định những tệp nhị phân không chịu sự ràng buộc của cơ chế Mark-of-the-Web khi được giải nén từ các tệp ZIP. Các tệp này có thể được tận dụng để nạp DLL độc hại từ thư mục cục bộ mà không hiển thị cảnh báo hoặc yêu cầu người dùng xác nhận.
Tiếp theo, chúng ta sẽ sử dụng các công cụ như `ProcMon` để phân tích hành vi nạp DLL của tệp nhị phân đã chọn và xác định rằng chương trình cố gắng nạp các thư viện bị thiếu từ thư mục làm việc hay không. Sau khi xác nhận được hành vi này, chúng ta sẽ triển khai kỹ thuật DLL proxying, cho phép chuyển tiếp các lời gọi hàm đến DLL hợp lệ trong khi đồng thời chèn thêm một payload tuỳ chỉnh.
### Finding the Target Application and DLL
Trong giai đoạn truy cập ban đầu, kỹ thuật DLL sideloading có thể được sử dụng để nguỵ trang payload bên trong một tệp ZIP được gửi dưới dạng tệp đính kèm email. Bằng cách đặt một tệp nhị phân đáng tin cậy, đã được Microsoft ký số, cùng với một DLL độc hại, kẻ tấn công có thể tạo ra một container trông có vẻ vô hại đối với người dùng và thường có khả năng vượt qua các bộ lọc bảo mật phổ biến.
**DLL sideloading** là một kỹ thuật thực thi mã khai thác cách các ứng dụng Windows nạp Dynamic Link Libraries (DLL) - tức các thư viện liên kết động. Khi một ứng dụng nạp DLL mà không chỉ định đường dẫn đầy đủ, bộ nạp Windows sẽ tuân theo một thứ tự tìm kiếm DLL đã được định nghĩa sẵn. Điểm quan trọng là quá trình tìm kiếm này bắt đầu từ thư mục chứa ứng dụng.
Các attacker có thể lợi dụng hành vi này bằng cách đặt một DLL độc hại trong cùng thư mục với một tệp thực thi hợp lệ đã được ký số. Nếu tệp thực thi cố gắng nạp một DLL không tồn tại trong hệ thống hoặc không được chỉ định bằng đường dẫn cố định, Windows sẽ nạp DLL của attacker từ thư mục hiện tại. Điều này dẫn đến việc thực thi mã trong ngữ cảnh của tệp nhị phân đáng tin cậy mà không kích hoạt cảnh báo cho user.
Theo mặc định, Windows tìm kiếm DLL theo thứ tự sau:
1. Thư mục mà ứng dụng được nạp từ đó
2. Thư mục hệ thống, `System32`
3. Thư mục hệ thống 16-bit
4. Thư mục Windows
5. Thư mục làm việc hiện tại
6. Các thư mục được liệt kê trong biến môi trường `PATH`
Để giảm thiểu rủi ro từ DLL sideloading, nhà phát triển nên sử dụng lời gọi API an toàn như `LoadLibraryExA()` với các cờ `LOAD_LIBRARY_SEARCH_*`, đồng thời cấu hình rõ ràng đường dẫn tìm kiếm DLL bằng `SetDefaultDllDirectories()` hoặc `AddDllDirectory()`.
Mặc dù tệp nhị phân chủ có thể xác minh chữ ký số của DLL trước khi nạp, cơ chế này thường không được bắt buộc triển khai trong thực tế. Phần lớn các ứng dụng Windows giả định rằng bất kỳ DLL nào nằm trong đường dẫn mong đợi đều đáng tin cậy và không thực hiện thêm các bước xác minh chữ ký tại thời điểm nạp. 
Với những thông tin trên, chúng ta có thể bắt đầu lựa chọn tệp nhị phân mục tiêu để triển khai DLL sideloading. Vì payload sẽ được gửi qua Internet đến nạn nhân, chúng ta cần hiểu khác niệm **Mark-of-the-Web (MotW)**.
**MotW** là một tính năng bảo mật trong Windows, giúp bảo vệ người dùng khỏi các tệp có khả năng không an toàn được tải xuống từ Internet. Khi một tệp được tải xuống thông qua trình duyệt web, client email hoặc một ứng dụng có khả năng nhận diện nguồn Internet, Windows sẽ thêm một siêu dữ liệu đặc biệt có tên **Zone identifier** dưới dạng một **Alternate Data Stream (ADS)** bổ sung.
Ví dụ, chúng ta đăng nhập vào máy `dev01` bằng tài khoản `offsec:lab` và phân tích ADS bằng PuTTY, một trình khách SSH phổ biến có thể được tải xuống từ Internet.
```powershell
PS C:\Users\offsec\Downloads> Get-Content -Path .\putty.exe -Stream Zone.Identifier
[ZoneTransfer]
ZoneId=3
ReferrerUrl=https://www.chiark.greenend.org.uk/
HostUrl=https://the.earth.li/~sgtatham/putty/0.83/w64/putty.exe
```
Giá trị `ZoneId=3` cho biết tệp nguồn gốc Internet Zone. Đây là cờ quan trọng được Windows sử dụng để xác định liệu có cần áp dụng cảnh báo bảo mật hoặc các hạn chế khi mở tệp hay không.
Do tệp thực thi này có `ZoneId=3`, Windows sẽ xem nó là tệp không đáng tin cậy. Nếu người dùng cố gắng chạy tệp này, tuỳ thuộc vào chính sách hệ thống, họ có thể thấy cảnh báo SmartScreen hoặc một thông báo bảo mật yêu cầu xác nhận trước khi thực thi.
Có nhiều cách để bypass MotW, một phương pháp phổ biến là đặt một tệp nhị phân được Microsoft ký số cùng với một DLL độc hại bên trong tệp ZIP. Khi nạn nhân giải nén nội dung ra máy cục bộ, thay vì chạy trực tiếp tệp nhị phân từ trong file nén, các tệp sau khi giải nén có thể không kế thừa cờ MotW, tuỳ thuộc vào công cụ giải nén hoặc phiên bản Windows.
Ngoài ra, ngay cả khi MotW vẫn được giữ lại, việc sử dụng một tệp nhị phân được Microsoft ký số có thể giúp tránh bị phát hiện, vì các tệp này thường được hệ điều hành và người dùng tin cậy.
Để tối đa hoá khả năng thành công, chúng ta sẽ sử dụng một cách tiếp cận kết hợp: đóng gói một tệp nhị phân được Microsoft ký số cùng với một DLL độc hại bên trong một tệp ZIP.
Microsoft có nhiều tệp nhị phân đã ký số có thể bị kẻ tấn công lợi dụng. Trong trường hợp này vào tệp thực thi `OneDrive.exe`, thường năm trong thư mục `C:\Program Files\Microsoft OneDrive`. Chúng ta sẽ sử dụng tệp thực thi này, nhưng đặt nó trong thư mục `C:\Tools`. Đây là một tệp nhị phân được Microsoft ký số và được biết là có hành vi nạp DLL nhất định từ thư mục hiện tại, khiến nó trở thành một ứng viên phù hợp cho kỹ thuật DLL sideloading.
Tiếp theo, chúng ta sẽ phân tích cách OneDrive nạp DLL. Ta sẽ sử dụng Process Monitor từ bộ Sysinternals Suite để quan sát chi tiết hành vi tra cứu và nạp DLL.
Bây giờ, ta có thể khởi chạy ProcMon từ thư mục `C:\Tools` và cấu hình một bộ lọc để theo dõi cách `OneDrive.exe` cố gắng nạp DLL. Mục tiêu là xác định các thao tác `CreateFile` thất bại với kết quả `NAME_NOT_FOUND`. Điều này cho biết ứng dụng đã cố gắng nạp một DLL nhưng không tìm thấy tệp đó tại đường dẫn mong đợi.
Ta cấu hình các bộ lọc ProcMon để chỉ tập trung vào hành vi nạp DLL liên quan đến tiến trình OneDrive. Bằng cách đặt `Process Name` chứa “OneDrive”, ta cô lập hoạt động của tệp nhị phân mục tiêu. Sau đó, ta lọc thao tác `CreateFile`, thao tác ghi nhận các nỗ lực truy cập tệp, bao gồm cả việc nạp DLL. Để xác định các cơ hội sideloading tiềm năng, ta chỉ bao gồm các sự kiện có kết quả `NAME_NOT_FOUND`, biểu thị rằng tệp bị thiếu. Tiếp theo, ta thu hẹp phạm vi bằng cách yêu cầu `Path` kết thúc bằng `.dll`, nhằm đảm bảo chỉ theo dõi các yêu cầu liên quan đến thư viện động. Cuối cùng, ta loại trừ các mục có `Process Name` là `Procmon.exe` để tránh làm nhiễu nhật ký bởi chính hoạt động của ProcMon.
![[Screenshot 2026-07-07 at 15.07.51.png|center|border|500]]
Kết quả quan sát cho thấy `OneDrive.exe` cố gắng nạp nhiều DLL từ thư mục làm việc hiện tại, bao gồm **Secur32.dll**, **VERSION.dll**, **WININET.dll**, **WTSAPI32.dll** cùng một số DLL khác. Tuy nhiên, tất cả các yêu cầu này đều trả về trạng thái **NAME_NOT_FOUND**, cho thấy các DLL tương ứng không tồn tại tại vị trí được tìm kiếm.
![[Screenshot 2026-07-07 at 15.08.49.png|center|border]]
Trong kịch bản tấn công này, chúng ta lựa chọn **Secur32.dll** làm mục tiêu. Đây là một DLL phụ thuộc mà chương trình kỳ vọng sẽ tồn tại, nhưng hiện không có trong thư mục làm việc. Do đó chúng ta có thể tạo một DLL giả mạo mang cùng tên **Secur32.dll**, đồng thời triển khai đầy đủ các hàm exports mà chương trình mong đợi. Khi **OneDrive.exe** nạp DLL này, mã độc của chúng ta sẽ được thực thi trước, trong khi các lời gọi hợp lệ vẫn được chuyển tiếp đến **Secur32.dll** của hệ thống gốc.

### Proxying the Calls
Sau khi xác định `Secur32.dll` là một thư viện dependency bị thiếu mà `OneDrive.exe` cố gắng nạp, chúng ta có thể tiến hành xây dựng kịch bản tấn công bằng kỹ thuật DLL Sideloading, DLL độc hại phải mô phỏng DLL hợp hợp pháp một cách đủ chính xác nhằm tránh làm ứng dụng bị lỗi hoặc kết thúc một cách bất thường. Điều này đồng nghĩa với việc  DLL giả mạo cần export đầy đủ các hàm mà ứng dụng mong đợi, đồng thời forward các lời gọi hợp lệ đến `Secur32.dll` thật, trong khi vẫn vâm thầm thực thi payload độc hại.

Kỹ thuật này được gọi là DLL Proxying cho phép duy trì hoạt động bình thường của ứng dụng trong khi chèn và thực thi mã độc mà không làm thay đổi hành vi bề ngoài của chương trình.
Trước khi bắt đầu với DLL proxy độc hại, chúng cần tìm hiểu nguyên lý hoạt động của kỹ thuật này để có cái nhìn tổng quan.

**DLL Proxying** là kỹ thuật trong đó kẻ tấn công tạo ra một phiên bản giả mạp của DLL hợp pháp mà ứng dụng mục tiêu dự kiến sẽ nạp. DLL giả mạo này xuất cùng các hàm exports như DLL gốc, khiến ứng dụng không nhận thấy bất kỳ sự khác biệt nào trong quá trình thực thi. Tuy nhiên, phía sau cơ chế đó DLL proxy sẽ forward các lời gọi hợp lệ đến DLL hệ thống thật, đồng thời thực hiện các hành vi độc hại theo ý đồ của các attacker.
Kỹ thuật này hoạt động do nhiều ứng dụng trên Windows nạp DLL mà không chỉ định đường dẫn tuyệt đối, thay vào đó dựa vào thứ tự tìm kiếm DLL (DLL Search Order). Nếu dứng dụng tìm thấy một DLL mang đúng tên như mong đợi trong thư mục làm việc hiện tại, nó sẽ nạp DLL đó mà không xác minh tính hợp lệ hoặc nguồn gốc của tệp.

Về bản chất, một DLL proxy thường thực hiện ba nhiệm vụ chính:
1. Export cùng tập các hàm như DLL gốc để đáp ứng yêu cầu của ứng dụng.
2. Forward toàn bộ các lời gọi hàm đến DLL hợp pháp nhằm duy trì hoạt động bình thường của chương trình.
3. Thực thi mã độc theo hai cách phổ biến:
    - Trong quá trình DLL được nạp, chẳng hạn bên trong hàm **DllMain**.
    - Khi một hàm cụ thể của DLL được gọi.
![[Client Side Attacks with File Container.png|center|border]]
Theo mặc định, Windows **không xác minh chữ ký số (Digital Signature)** của một DLL trước khi nạp, trừ khi chính ứng dụng triển khai cơ chế kiểm tra chữ ký, điều mà trên thực tế vẫn chưa phổ biến. Do đó, ngay cả một DLL độc hại không được ký số vẫn có thể được một tệp thực thi đã được Microsoft ký số nạp mà không tạo ra bất kỳ cảnh báo bảo mật nào.
Sau khi đã nắm vững nguyên lý của **DLL Proxying**, chúng ta sẽ chuyển sang phần thực hành. Bước đầu tiên là xây dựng bộ khung (skeleton) cơ bản cho DLL proxy.

Trên máy phát triển chạy **Windows 11**, hãy tạo một dự án **C++ Dynamic-Link Library (DLL)** mới bằng **Visual Studio**. Sau khi dự án được tạo, mở tệp **dllmain.cpp**:
```cpp
#include "pch.h"

#include <Windows.h>

#ifdef _WIN64
#define DLLPATH "\\\\.\\GLOBALROOT\\SystemRoot\\System32\\secur32.dll"
#else
#define DLLPATH "\\\\.\\GLOBALROOT\\SystemRoot\\SysWOW64\\secur32.dll"
#endif // _WIN64

#pragma comment(linker, "/EXPORT:GetUserNameExW=" DLLPATH ".GetUserNameExW")

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
    switch (fdwReason)
    {
    case DLL_PROCESS_ATTACH:
    {
        MessageBoxA(NULL, "Executing from Malicious DLL", "Executing from Malicious DLL", 0);

    }
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

Trong đoạn mã này, trước tiên chúng ta đưa vào tệp pch.h, đây là precompiled header được Visual Studio sử dụng nhằm giảm thời gian biên dịch của dự án. Tiếp theo, thư viện `<Windows.h>` được khai báo để có thể sử dụng các hàm của Windows API, chẳng hạn như `MessageBoxA`, cùng với các kiểu dữ liệu và hằng số cần thiết để xử lý các sự kiện của DLL trong hàm DllMain. Đồng thời, chương trình cũng định nghĩa đường dẫn tới `secur32.dll` hợp pháp dựa trên kiến trúc của hệ điều hành mục tiêu (32-bit hoặc 64-bit).

![[Screenshot 2026-07-07 at 15.58.50.png|border|center]]

Tiếp theo, chúng ta chỉ thị cho linker export hàm `GetUserNameExW` từ DLL proxy và forward lời gọi của hàm này đến `secur32.dll` hợp pháp của hệ thống. Đây là một bước quan trọng nhằm đảm bảo ứng dụng vẫn hoạt động ổn định. Việc cung cấp đúng hàm mà ứng dụng mong đợi sẽ giúp tránh tình trạng chương trình bị lỗi hoặc kết thúc đột ngột khi cố gắng giải quyết các dependencies . 