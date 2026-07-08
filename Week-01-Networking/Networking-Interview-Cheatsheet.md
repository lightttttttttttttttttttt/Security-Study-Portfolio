# 🎯 Networking Interview & OSCP Cheatsheet

> **Focus:** Những gì cốt lõi nhất hay bị hỏi khi phỏng vấn SOC Tier 3 và dùng nhiều nhất trong thực chiến OSCP. Không hàn lâm, đi thẳng vào ứng dụng.

---

## 1. Network Layer (Layer 3) - Routing & IP

### 1.1. Subnetting & Private IPs
- **Private IP (Bắt buộc thuộc):** 
  - `10.0.0.0/8` (Mạng Enterprise lớn)
  - `172.16.0.0/12` (Mạng vừa)
  - `192.168.0.0/16` (Mạng nhỏ/Gia đình)
- **Câu hỏi phỏng vấn:** "Tại sao không route được Private IP ra Internet?" -> Vì các Router public được cấu hình để drop gói tin chứa địa chỉ này. Phải dùng **NAT** (Network Address Translation) để bọc nó vào Public IP.

### 1.2. TTL & OS Fingerprinting
- **Thực chiến OSCP:** Khi gõ `ping 10.10.10.x`, nhìn chỉ số `TTL` trả về để đoán hệ điều hành của mục tiêu (rất quan trọng trước khi chọn Exploit).
  - **TTL ≈ 128:** Windows.
  - **TTL ≈ 64:** Linux/Unix.
  - **TTL ≈ 255:** Thiết bị mạng (Cisco Router).

### 1.3. ARP (Layer 2/3 Boundary)
- **Câu hỏi phỏng vấn:** "ARP Spoofing hoạt động như thế nào?"
- **Đáp án:** Kẻ tấn công gửi các gói tin ARP Reply giả mạo vào mạng LAN, lừa máy nạn nhân tin rằng MAC address của kẻ tấn công chính là MAC address của Default Gateway (Router). Từ đó, toàn bộ traffic của nạn nhân sẽ đi qua máy kẻ tấn công (Man-in-the-Middle).

---

## 2. Transport Layer (Layer 4) - TCP/UDP & Ports

### 2.1. Phân biệt TCP vs UDP
- **TCP:** Bắt tay 3 bước (3-Way Handshake), đảm bảo dữ liệu tới nơi, có Flow Control (Sliding Window), chậm nhưng chắc.
- **UDP:** Bắn và quên (Fire & Forget), không kiểm tra lỗi, nhanh nhưng dễ mất gói. (Dùng cho DNS, Video streaming, SNMP).

### 2.2. TCP 3-Way Handshake & Port Scanning
- **3-Way Handshake chuẩn:** `SYN` -> `SYN-ACK` -> `ACK`.
- **Thực chiến (Nmap Stealth Scan - `-sS`):** 
  Nmap gửi `SYN` -> Đích trả `SYN-ACK` -> Nmap ném **`RST` (Reset)** để ngắt luôn thay vì gửi `ACK`. 
  *Tại sao?* Vì chưa hoàn tất kết nối, ứng dụng ở L7 (như Apache) sẽ không ghi log (Tránh bị phát hiện).

### 2.3. Top Ports phải thuộc nằm lòng
| Port | Dịch vụ | Ghi chú (Pentest) |
|:---:|:---|:---|
| **21** | FTP | Tìm Anonymous login (không cần pass). |
| **22** | SSH | Brute-force, tìm key rò rỉ. |
| **25** | SMTP | Enum usernames. |
| **53** | DNS | Zone Transfer (AXFR) lôi toàn bộ domain con. |
| **80/443**| HTTP/HTTPS| Entry point số 1. Web hacking. |
| **139/445**| SMB | Khai thác **Null Session**, **SMB Relay**, EternalBlue (MS17-010). Trái tim của Windows AD. |
| **3389** | RDP | Remote Desktop. BlueKeep exploit. |

---

## 3. Application & Session Layers (L5-L7)

### 3.1. SMB (Server Message Block - Port 445)
- **Hay hỏi nhất:** "Làm sao để dò quét danh sách user trong mạng Windows mà không có password?"
- **Đáp án:** Dùng kỹ thuật **Null Session** (Anonymous Logon) - kết nối tới SMB với User rỗng, Pass rỗng.

### 3.2. HTTP vs HTTPS (Port 80/443)
- **Câu hỏi phỏng vấn:** "TLS nằm ở tầng mấy và mã hóa payload như thế nào?"
- **Đáp án:** TLS nằm ở L6 (Presentation), mã hóa toàn bộ phần Header của HTTP và Payload. Attacker bắt được gói tin chỉ thấy data bị xáo trộn (thường là rác hoặc Base64). 
- **Bypass:** Attacker hay dùng kỹ thuật **SSL Stripping** (ép HTTPS xuống HTTP để đọc trộm clear-text).

### 3.3. Base64
- Thường xuyên gặp trong log (PowerShell, Malware). Đặc điểm nhận dạng: Chuỗi text ngẫu nhiên có chữ/số/kí tự đặc biệt và **kết thúc bằng dấu `=` hoặc `==`**.

---

> **Tóm lại cho OSCP:** 
> - Thấy IP -> Gõ Ping xem TTL đoán hệ điều hành.
> - Thấy Port 445 -> Dò Null Session ngay lập tức.
> - Quét port thì luôn dùng Stealth Scan (`-sS`) để né log Firewall.
