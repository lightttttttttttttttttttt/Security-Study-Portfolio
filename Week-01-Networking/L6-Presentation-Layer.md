# Layer 6 — Presentation Layer (Tầng Trình diễn)

> **OSCP Prep — Week 1: Networking**
> Góc nhìn: SOC Tier 3 / Pentester mindset

---

## 1. Vai trò trong mô hình OSI

Layer 6 nằm trên Layer 5 (Session) và dưới cùng là Layer 7 (Application).

> **Nhiệm vụ chính: Đóng vai trò là "Người phiên dịch" của mạng. Nó chuẩn bị dữ liệu (format, translate, mã hóa, nén) để ứng dụng có thể đọc được, hoặc để truyền đi an toàn.**

Nếu L7 là màn hình hiển thị trang web, thì L6 là thằng chịu trách nhiệm giải nén hình ảnh (JPEG/PNG), giải mã ký tự (UTF-8), và giải mã bảo mật (SSL/TLS) trước khi ném lên L7.

---

## 2. Các chức năng chính (SOC Focus)

### 2.1. Mã hóa và Giải mã (Encryption / Decryption)
- Chức năng quan trọng nhất về mặt bảo mật. L6 chịu trách nhiệm xáo trộn dữ liệu bằng thuật toán để kẻ gian bắt được gói tin (qua Wireshark) cũng không đọc được.
- **Protocol tiêu biểu:** SSL (Secure Sockets Layer) và TLS (Transport Layer Security). Khi duyệt web `https://`, TLS hoạt động ở đây để mã hóa gói tin HTTP.

### 2.2. Encode / Decode (Mã hóa biểu diễn)
- Chuyển đổi dữ liệu từ định dạng này sang định dạng khác để hệ thống có thể đọc hiểu hoặc truyền tải mà không bị lỗi.
- **Ví dụ:** Base64, Hex, ASCII, EBCDIC, UTF-8.

### 2.3. Nén dữ liệu (Compression)
- Thu gọn kích thước gói tin trước khi gửi để tiết kiệm băng thông (ví dụ: GZIP).

---

## 3. Các kỹ thuật tấn công & Khai thác (Pentest Mindset)

Trong SOC và Pentest, L6 thường là nơi attacker cố tình "che giấu" dấu vết của mình.

### 3.1. Obfuscation (Làm rối mã / payload)
- **Khái niệm:** Kẻ tấn công biến đổi các đoạn mã độc (như PowerShell script, shellcode) thành một mớ ký tự hỗn độn để lách qua Antivirus, WAF (Web Application Firewall) hoặc IDS/IPS.
- **Kỹ thuật hay dùng:** Dùng Base64 encoding kết hợp với XOR cipher, hoặc URL encoding.
- **Ví dụ log PowerShell:** `powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -EncodedCommand JABzACAAPQAgAE4AZQB3...` (Cục mã kia thực chất là payload được encode Base64 ở L6).

### 3.2. SSL Stripping (Downgrade Attack)
- **Khái niệm:** Kẻ tấn công (đứng giữa - MitM) chặn yêu cầu HTTPS của người dùng và ép kết nối hạ cấp xuống HTTP (không có TLS/SSL).
- **Hậu quả:** Toàn bộ dữ liệu truyền đi ở dạng rõ (clear-text) ở L6, dễ dàng bị đọc lén mật khẩu và session cookie.

### 3.3. Che giấu qua mã hóa TLS (Malware C2 over HTTPS)
- Malware (Mã độc) ngày nay rất khôn ngoan. Nó giao tiếp với máy chủ điều khiển (C2 - Command & Control) thông qua HTTPS thay vì HTTP.
- Vì gói tin bị mã hóa bởi TLS ở L6, các hệ thống Firewall thông thường không thể nhìn xuyên qua để biết bên trong có chứa lệnh thực thi độc hại hay không (trừ khi dùng cơ chế SSL Inspection/Decryption).

---

## 4. Dấu hiệu nhận biết trong Log (SOC Tier 3)

- **Base64 Payload:** Bất cứ chuỗi string nào dài bất thường, chứa toàn chữ HOA, chữ thường, số, và hay kết thúc bằng dấu `=` hoặc `==`. Gặp là copy mang đi decode ngay.
- **Bất thường ở TLS Certificates:** Cảnh báo về chứng chỉ SSL/TLS tự ký (Self-signed certificate), chứng chỉ hết hạn, hoặc Issuer (người cấp) lạ hoắc trong traffic mạng. Attacker hay dùng Self-signed cert cho C2 server.
