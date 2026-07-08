# Layer 7 — Application Layer (Tầng Ứng dụng)

> **OSCP Prep — Week 1: Networking**
> Góc nhìn: SOC Tier 3 / Pentester mindset

---

## 1. Vai trò trong mô hình OSI

Layer 7 là tầng cao nhất, nơi người dùng (hoặc phần mềm) tương tác trực tiếp với mạng.

> **Nhiệm vụ chính: Cung cấp giao diện mạng cho các ứng dụng (Web browser, Email client, FTP client). Nó định nghĩa cách các ứng dụng giao tiếp với nhau.**

Đây là **chiến trường chính** của OSCP và Web Pentest. 90% các lỗ hổng bảo mật hiện đại (OWASP Top 10) nằm ở Layer 7 (SQLi, XSS, RCE, IDOR...).

---

## 2. Các giao thức cốt lõi (Must-know cho OSCP)

### 2.1. HTTP/HTTPS (Port 80 / 443 - TCP)
Là xương sống của Web. Điểm vào (Entry point) phổ biến nhất khi boot một box OSCP.

- **Methods:** `GET` (lấy data), `POST` (gửi data), `PUT` (sửa), `DELETE` (xóa), `OPTIONS` (xem server hỗ trợ method gì).
- **Status Codes (Quan trọng khi fuzzing/directory buster):**
  - `200 OK`: Thành công (Tìm thấy file/thư mục).
  - `301/302 Redirect`: Chuyển hướng.
  - `401 Unauthorized`: Chưa đăng nhập.
  - `403 Forbidden`: Cấm truy cập (Có thư mục tồn tại nhưng không được phép xem -> Có thể cố bypass bằng HTTP header).
  - `404 Not Found`: Không tồn tại.
  - `500 Internal Server Error`: Lỗi logic code server (Dấu hiệu tốt để test SQLi hoặc khai thác lỗi).
- **Headers:** Nơi giấu nhiều lỗ hổng (Ví dụ: `User-Agent`, `Host`, `Cookie`, `X-Forwarded-For`).

### 2.2. DNS (Domain Name System - Port 53 - UDP/TCP)
Danh bạ của Internet, chuyển đổi Tên miền (Domain) sang IP. DNS chạy UDP cho các query bình thường, nhưng chạy TCP cho Zone Transfer.

- **Khai thác (DNS Enumeration):**
  - **Zone Transfer (AXFR):** Trọng tâm của OSCP. Nếu server cấu hình sai, attacker có thể tải toàn bộ danh sách sub-domains của một công ty bằng lệnh `dig axfr domain.com @DNS_Server_IP`.

### 2.3. FTP (File Transfer Protocol - Port 21 - TCP)
Truyền tải file không mã hóa.
- **Khai thác:** 
  - **Anonymous Login:** Luôn luôn kiểm tra xem server có cho phép đăng nhập bằng user `anonymous` và mật khẩu rỗng hay không. Thông thường trong bài lab sẽ giấu file cấu hình, password, hoặc SSH keys trong này.
  - Giao thức clear-text, dễ bị sniff (Wireshark) để lộ mật khẩu.

### 2.4. DHCP (Dynamic Host Configuration Protocol - Port 67/68 - UDP)
Cấp phát IP tự động cho máy trạm.
- **Khai thác (Layer 2/7):**
  - **DHCP Starvation:** Gửi hàng vạn request xin IP giả, làm kiệt quệ pool IP của Router.
  - **Rogue DHCP Server:** Attacker dựng máy chủ DHCP giả, cấp IP và cấp luôn Default Gateway là máy của Attacker (Dẫn đến MitM).

---

## 3. Dấu hiệu nhận biết trong Log (SOC Tier 3)

- **HTTP Logs:** 
  - Thấy quá nhiều mã `404 Not Found` từ cùng một IP trong thời gian ngắn -> Dấu hiệu của Directory Brute-force (Gobuster/Dirb).
  - Thấy các ký tự lạ như `' OR 1=1`, `<script>`, `../../../` trong URL (Mã `200` hoặc `500`) -> Dấu hiệu tấn công SQLi, XSS, LFI.
- **DNS Logs:** Truy vấn DNS tới các tên miền rất dài, vô nghĩa (ví dụ: `87a9b...c3.attacker.com`) -> Dấu hiệu của **DNS Tunneling** hoặc Data Exfiltration (tuồn dữ liệu ra ngoài qua ngõ DNS để lách Firewall).
