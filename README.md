# Security Notes

Repository lưu trữ ghi chú cá nhân, writeup lab và tài liệu kỹ thuật về Red Teaming, Pentest, Malware Development và Malware Analysis. Toàn bộ nội dung được tổ chức dưới dạng một [Obsidian](https://obsidian.md/) Vault.

---

## Cấu trúc nội dung

### 1. Active Directory
Ghi chú và kỹ thuật khai thác môi trường Active Directory từ Linux và Windows:
- **Recon & Enumeration:** External recon, credentialed enumeration từ Linux, trích xuất Password Policies, kiểm tra Security Controls.
- **Attacks & Poisoning:** LLMNR / NBT-NS Poisoning, Password Spraying (cách build target user list và spray nội bộ).
- **Protocols & Auth:** NTLM Authentication, Kerberos, LDAP, DNS, MSRPC.

### 2. Maldev (Malware Development)
Nghiên cứu cơ chế hoạt động của hệ điều hành Windows và kỹ thuật phát triển mã độc:
- **Windows Internals:** Kiến trúc Windows, tiến trình (Processes), quản lý bộ nhớ (Virtual Memory), Windows API, Undocumented Structures.
- **PE & DLL:** Cấu trúc Portable Executable (PE Format), Dynamic-Link Library (DLL).
- **Payload Placement & Execution:** Kỹ thuật đặt payload trong các section `.data`, `.rdata`, `.rsrc`, `.text`.
- **Evasion & Defense:** Kỹ thuật mã hóa payload (XOR,...), phân tích cơ chế phát hiện của EDR/AV (Detection Mechanisms).

### 3. Code (C / C++)
Source code mẫu và solution Visual Studio phục vụ phần thực hành Maldev:
- `DataPayloadPlacement`: Đặt shellcode trong section `.data` / `.rdata`.
- `ResourcePayloadPlacement`: Đóng gói payload vào resource `.rsrc` và load lúc runtime.
- `TextPayloadPlacement`: Kỹ thuật đặt payload trong section `.text`.
- `Xor`: Chương trình mã hóa và giải mã payload sử dụng phép toán XOR.

### 4. Malware Analysis
Quy trình và nhật ký phân tích mẫu mã độc:
- **Tĩnh (Static Analysis):** Phân tích PE header, trích xuất strings, import/export tables, kiểm tra chữ ký số.
- **Động (Dynamic Analysis):** Theo dõi hành vi tiến trình, network traffic, thay đổi registry và file system trong môi trường sandbox/lab.

### 5. HTB (Hack The Box)
Ghi chép kỹ thuật giải lab và khai thác hệ thống từ HTB Academy / CTF:
- **Windows Privilege Escalation:** Attacking the OS, Credential Theft, User & Group Privileges, Bypass UAC, Restricted Environments.
- **Linux Privilege Escalation:** Kỹ thuật leo thang đặc quyền trên môi trường Linux.
- **Pivoting & Tunneling:** Dynamic port forwarding qua SSH, SOCKS proxy, chuyển hướng traffic giữa các dải mạng.
- **File Transfer:** Tổng hợp các phương pháp tải công cụ và dữ liệu giữa máy tấn công và target (Windows & Linux).

### 6. Notes & OSEP
Ghi chú chuyên sâu theo định hướng chứng chỉ OSEP và AV Evasion:
- Kỹ thuật Client-Side Attacks sử dụng file container (ISO, VHD).
- Kỹ thuật Phishing qua JScript, macro Microsoft Office, Calendar invites.
- Cơ chế vượt qua phần mềm diệt virus (Antivirus Evasion).

### 7. THM (TryHackMe)
- Writeup và ghi chú các phòng lab TryHackMe (Active Directory, Secure AI Systems).

### 8. Dashboard
- Dashboard quản lý tiến độ học tập và truy vấn tự động các bài lab qua Dataview.

### 9. Attachments
- Thư mục lưu trữ toàn bộ hình ảnh, sơ đồ và ảnh chụp bằng chứng thực hành trong quá trình làm lab.

---

## Quy trình đồng bộ Git (Multi-device Workflow)

Để đồng bộ giữa các máy làm việc:

```bash
# Trước khi bắt đầu ghi chép (kéo cập nhật mới nhất về máy)
git pull origin main

# Sau khi ghi chép xong (lưu và đẩy lên GitHub)
git add .
git commit -m "Update notes: [nội dung cập nhật]"
git push origin main
```

*Lưu ý: File trạng thái làm việc cục bộ của Obsidian (`workspace.json`) cùng các thư mục tạm/build của Visual Studio (`.vs/`, `x64/`, `Debug/`) đã được loại trừ trong `.gitignore` để tránh xung đột khi làm việc trên nhiều thiết bị.*

---

## Miễn trừ trách nhiệm (Disclaimer)

Toàn bộ thông tin, tài liệu và mã nguồn trong repository này được biên soạn cho mục đích học tập, nghiên cứu bảo mật và kiểm thử an toàn thông tin trong môi trường phòng lab hoặc hệ thống được cấp phép. Tác giả không chịu trách nhiệm cho bất kỳ hành vi lạm dụng nào vi phạm pháp luật.
