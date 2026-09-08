Trước khi bắt đầu pentest, việc thực hiện external recon đối với mục tiêu có thể mang lại nhiều lợi ích. Hoạt động này có thể phục vụ nhiều mục đích, chẳng hạn:
- Xác thực những thông tin mà khách hàng cung cấp trong scoping document.
- Đảm bảo rằng chúng ta đang thực hiện các hoạt động trên đúng scope khi remote.
- Tìm kiếm những thông tin được công khai trên Internet có thể ảnh hưởng đến kết quả của bài kiểm thử, chẳng hạn như thông tin xác thực bị rò rỉ.
## What Are We Looking For?
Khi thực hiện external recon, có một số loại thông tin quan trọng cần tìm kiếm. Những thông tin này không phải lúc nào cũng được công khai, nhưng việc kiểm tra xem chúng ta có thể thu thập được gì là rất hữu ích.
Nếu bị mắc kẹt trong quá trình pentest, việc quay lại xem những gì có thể thu thập được thông qua passive recon đôi khi có thể cung cấp manh mối để tiếp tục.
Ví dụ, password breach data có thể cung cấp credentials có khả năng được sử dụng để truy cập VPN hoặc một dịch vụ externally facing khác.

| Data Point             | Mô tả                                                                                                                                                                                                                                                                                          |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IP Space           | Xác định ASN hợp lệ của mục tiêu, các netblock đang được tổ chức sử dụng cho infrastructure public-facing, cloud presence, hosting providers, DNS records, v.v.                                                                                                                        |
| Domain Information | Dựa trên IP data, DNS và thông tin đăng ký domain để xác định ai quản trị domain. Tìm các subdomain, các domain services có thể truy cập công khai như mail server, DNS, website, VPN portal, v.v. Đồng thời tìm hiểu các cơ chế phòng thủ đang được sử dụng, chẳng hạn SIEM, AV, IPS/IDS. |
| Schema Format      | Tìm hiểu email account format, AD username format, thậm chí cả password policy nếu có thể. Những thông tin này có thể giúp xây dựng username list hợp lệ để kiểm tra các dịch vụ external-facing bằng password spraying, credential stuffing, brute forcing, v.v.                  |
| Data Disclosures   | Tìm kiếm các file được công khai như `.pdf`, `.ppt`, `.docx`, `.xlsx`, v.v. để tìm thông tin về target. Ví dụ: thông tin về intranet, user metadata, network shares, software/hardware quan trọng, credentials bị commit lên GitHub hoặc AD username format xuất hiện trong metadata của PDF.  |
| Breach Data        | Tìm kiếm usernames, passwords hoặc các thông tin quan trọng khác đã bị công khai trong các vụ data breach và có thể giúp attacker có được foothold.                                                                                                                                            |
## Where Are We Looking?
Danh sách các loại dữ liệu ở trên có thể được thu thập được bằng nhiều cách khác nhau. Có rất nhiều website và công cụ có thể cung cấp một hần