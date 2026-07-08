# Layer 5 — Session Layer (Tầng Phiên)

> **OSCP Prep — Week 1: Networking**
> Góc nhìn: SOC Tier 3 / Pentester mindset

---

## 1. Vai trò trong mô hình OSI

Layer 5 nằm trên Layer 4 (Transport) và dưới Layer 6 (Presentation). 

> **Nhiệm vụ chính: Thiết lập, duy trì, quản lý và đóng các phiên giao tiếp (sessions) giữa hai ứng dụng trên hai máy tính khác nhau.**

Nếu L4 (TCP/UDP) lo việc thiết lập đường truyền gói tin, thì L5 lo việc **giữ trạng thái** của cuộc hội thoại đó (ví dụ: ai nói trước, ai nói sau, khi nào dừng, phiên làm việc kéo dài bao lâu).

Trong mô hình TCP/IP thực tế, L5, L6, L7 thường được gộp chung vào **Application Layer (Tầng Ứng dụng)**. Tuy nhiên, khi phân tích bảo mật, L5 có những giao thức cực kỳ quan trọng.

---

## 2. Các giao thức quan trọng (SOC Focus)

| Giao thức | Port | Mô tả & Ứng dụng |
|:---|:---|:---|
| **NetBIOS** (Network Basic Input/Output System) | 137, 138, 139 | Cho phép các ứng dụng trên các máy tính khác nhau giao tiếp trong mạng LAN. Cốt lõi của mạng Windows cũ. |
| **SMB** (Server Message Block) | 445 (TCP) | Dùng để chia sẻ file, máy in, serial ports trong mạng Windows. Rất hay bị lợi dụng để tấn công. |
| **RPC** (Remote Procedure Call) | 135 (TCP) | Cho phép một chương trình máy tính thực thi code trên một máy tính khác (remote) y như đang chạy local. |

---

## 3. Các kỹ thuật tấn công & Khai thác (Pentest Mindset)

Trong môi trường Enterprise (Active Directory) hoặc bài lab OSCP, giao thức ở L5 (nhất là SMB/RPC) là mục tiêu hàng đầu.

### 3.1. Null Session / Anonymous Enumeration
- **Khái niệm:** Attacker kết nối vào SMB (port 445) hoặc RPC (port 135) bằng username rỗng (`""`) và password rỗng (`""`).
- **Hậu quả:** Có thể liệt kê (enumerate) được danh sách Users, Groups, Network Shares, Password Policies trên máy chủ Windows hoặc Domain Controller.
- **Tool sử dụng:** `enum4linux`, `smbclient`, `rpcclient`.

### 3.2. SMB Relay Attack
- **Khái niệm:** Kẻ tấn công đứng ở giữa (Man-in-the-Middle), đánh chặn gói tin xác thực SMB (NTLM hash) từ máy nạn nhân, rồi "chuyển tiếp" (relay) gói tin đó sang một máy chủ khác để đăng nhập với tư cách nạn nhân.
- **Hậu quả:** Chiếm quyền điều khiển máy đích mà không cần biết mật khẩu (miễn là nạn nhân có quyền Admin trên máy đích).

### 3.3. Session Hijacking (Cướp phiên)
- Lấy cắp Session ID (thường thấy ở Web - Layer 7, nhưng bản chất là cướp phiên giao tiếp đã được xác thực ở L5) để mạo danh người dùng hợp lệ.

---

## 4. Dấu hiệu nhận biết trong Log (SOC Tier 3)

- **Event ID 4624 (Logon):** Theo dõi các kết nối Logon Type 3 (Network Logon) qua port 445 (SMB) sử dụng tài khoản `ANONYMOUS LOGON`. Đây là dấu hiệu của Null Session.
- **Traffic Port 139/445 tăng đột biến:** Có thể máy tính đang bị rà quét (scanning) danh sách file shares bằng công cụ tự động.
- **RPC Endpoint Mapper (Port 135):** Thường bị rà quét bằng Nmap để lấy danh sách các dịch vụ RPC đang chạy ngầm trên hệ thống Windows.
