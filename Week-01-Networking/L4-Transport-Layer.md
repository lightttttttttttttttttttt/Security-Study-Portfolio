# Layer 4 — Transport Layer (Tầng Giao vận)

> **OSCP Prep — Week 1: Networking**
> Góc nhìn: SOC Tier 3 / Pentester mindset

---

## 1. Vai trò trong mô hình OSI

Layer 4 nằm trên Layer 3. Nhiệm vụ chính:

> **Cung cấp kết nối end-to-end giữa hai ứng dụng, kiểm soát luồng dữ liệu, đảm bảo (hoặc không đảm bảo) việc truyền tin cậy — thông qua hai protocol chính: TCP và UDP.**

L3 lo đưa gói tin đến đúng máy. L4 lo đưa dữ liệu đến đúng **ứng dụng** trong máy đó, và quản lý chất lượng truyền.

---

## 2. TCP vs UDP

| | TCP | UDP |
|:---|:---|:---|
| Viết tắt | Transmission Control Protocol | User Datagram Protocol |
| Kết nối | Có (connection-oriented) | Không (connectionless) |
| Đảm bảo nhận | Có — mất gói thì gửi lại | Không — mất thì thôi |
| Thứ tự gói | Đảm bảo đúng thứ tự | Không đảm bảo |
| Tốc độ | Chậm hơn (overhead nhiều) | Nhanh hơn (gửi thẳng) |
| Dùng cho | HTTP/S, SSH, FTP, Email | DNS, Livestream, Gaming, VoIP |

**Tại sao gaming/livestream dùng UDP?**

Nếu đang chơi game, nhận frame bị mất → không cần gửi lại frame cũ (đã lỗi thời rồi), cứ nhận frame mới tiếp. Gửi lại còn tệ hơn vì gây lag.

---

## 3. Port — Số phòng trong tòa nhà

```
IP   = địa chỉ tòa nhà (đến đúng máy)
Port = số phòng        (đến đúng ứng dụng trong máy đó)
```

### Port thông dụng cần nhớ

| Port | Protocol | Ghi chú |
|:---|:---|:---|
| 20, 21 | FTP | 20 = data, 21 = control |
| 22 | SSH | Remote shell an toàn |
| 23 | Telnet | Remote shell KHÔNG mã hóa — nguy hiểm |
| 25 | SMTP | Gửi email |
| 53 | DNS | Phân giải tên miền (UDP + TCP) |
| 80 | HTTP | Web không mã hóa |
| 110 | POP3 | Nhận email |
| 143 | IMAP | Nhận email |
| 443 | HTTPS | Web mã hóa (TLS) |
| 445 | SMB | File share Windows — hay bị exploit |
| 3306 | MySQL | Database |
| 3389 | RDP | Remote Desktop Windows |
| 8080 | HTTP alt | Web server dev thường dùng |

> **SOC note:** Các port như 23 (Telnet), 445 (SMB), 3389 (RDP) xuất hiện trong log là dấu hiệu đáng ngờ — attacker thường target những port này.

### Dải port

| Dải | Tên | Ghi chú |
|:---|:---|:---|
| 0 – 1023 | Well-known ports | Reserved cho system/protocol chuẩn |
| 1024 – 49151 | Registered ports | Ứng dụng đăng ký (MySQL, RDP...) |
| 49152 – 65535 | Ephemeral ports | OS tự gán cho client khi kết nối ra ngoài |

---

## 4. TCP 3-Way Handshake

Trước khi truyền data, TCP phải thiết lập kết nối qua 3 bước:

```
Client                          Server
  |                               |
  |──── SYN ─────────────────────→|  Bước 1: "Tôi muốn kết nối"
  |                               |
  |←─── SYN-ACK ─────────────────|  Bước 2: "OK, tôi chấp nhận"
  |                               |
  |──── ACK ─────────────────────→|  Bước 3: "Xác nhận, bắt đầu nào"
  |                               |
  |════ Bắt đầu truyền data ══════|
```

- **SYN** (Synchronize) — yêu cầu kết nối
- **ACK** (Acknowledgment) — xác nhận đã nhận
- **SYN-ACK** — vừa đồng ý vừa xác nhận

Bước 3 là ACK, **không phải data**. Data bắt đầu chạy sau khi handshake hoàn tất.

### Đóng kết nối — 4-Way Termination

```
Client          Server
  |─── FIN ────→|   "Tôi xong rồi"
  |←─── ACK ───|   "OK"
  |←─── FIN ───|   "Tôi cũng xong"
  |─── ACK ────→|   "OK, đóng thôi"
```

---

## 5. TCP Header — Các trường quan trọng

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|           Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 6 Flags cần nhớ

| Flag | Ý nghĩa |
|:---|:---|
| **SYN** | Bắt đầu kết nối |
| **ACK** | Xác nhận đã nhận |
| **FIN** | Kết thúc kết nối bình thường |
| **RST** | Reset — cắt kết nối đột ngột (lỗi hoặc từ chối) |
| **PSH** | Push — gửi data lên application ngay, không buffer |
| **URG** | Urgent — data khẩn cấp, xử lý trước |

### Sequence Number và Acknowledgment Number

- **Sequence Number:** Đánh số thứ tự byte. Bên nhận dùng để ráp lại đúng thứ tự nếu gói đến lộn xộn.
- **Acknowledgment Number:** Bên nhận báo lại "tôi đã nhận đến byte số X, gửi tiếp từ X+1".

```
Client gửi: Seq=100, Data=50 bytes
Server ACK: Ack=150  ("tôi nhận đến byte 149, gửi tiếp từ 150")
```

### Window Size và Flow Control

**Window Size** = số byte tối đa bên gửi được phép gửi trước khi cần ACK.

Cơ chế này gọi là **Sliding Window** — tránh gửi quá nhanh làm tràn buffer bên nhận:

```
Window = 3 packet:

Gửi [1][2][3] → đợi ACK
Nhận ACK(1)   → gửi thêm [4]
Nhận ACK(2)   → gửi thêm [5]
...
```

---

## 6. Socket 5-tuple — Server phân biệt hàng trăm kết nối

Câu hỏi: Server đang lắng nghe port 80. 100 client cùng kết nối vào — server phân biệt từng client bằng cách nào?

Mỗi kết nối TCP được định danh bởi **5 thứ** (Socket 5-tuple):

```
(Source IP, Source Port, Dest IP, Dest Port, Protocol)
```

Ví dụ:

```
Client A: (192.168.1.10 : 52341,  1.2.3.4 : 80, TCP)  ← kết nối riêng
Client B: (192.168.1.20 : 48192,  1.2.3.4 : 80, TCP)  ← kết nối riêng
Client C: (192.168.1.10 : 55678,  1.2.3.4 : 80, TCP)  ← kết nối riêng
```

Dest IP và Dest Port giống nhau (đều là `1.2.3.4:80`), nhưng **Source Port khác nhau** → server phân biệt được từng kết nối.

Source Port của client là **ephemeral port** — OS tự gán ngẫu nhiên trong dải 49152–65535 mỗi khi client tạo kết nối mới.

---

## 7. UDP Header — Đơn giản hơn nhiều

```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Chỉ 8 bytes header — không có Sequence Number, ACK, Window, Flags. Gửi thẳng, không hỏi han.

---

## 8. Bảo mật Layer 4 (SOC Tier 3)

| Mối đe dọa | Kỹ thuật | Phát hiện / Phòng thủ |
|:---|:---|:---|
| **SYN Flood** | Gửi hàng nghìn SYN, không gửi ACK cuối → server giữ half-open connection → cạn resource → DoS | SYN cookies, rate limiting SYN, firewall stateful |
| **Port Scanning** | Gửi SYN đến nhiều port để tìm port mở (Nmap SYN scan) | IDS phát hiện scan pattern, firewall block |
| **Session Hijacking** | Đoán Sequence Number để inject packet vào connection của người khác | Randomize Sequence Number, TLS encryption |
| **RST Attack** | Gửi RST giả mạo để cắt kết nối của người khác | TLS, Sequence Number validation |
| **UDP Flood** | Gửi UDP packet hàng loạt đến random port → victim gửi ICMP Unreachable liên tục → cạn bandwidth | Rate limiting, firewall |

### Nmap scan techniques liên quan đến TCP flags

```bash
# SYN scan (stealth scan) — phổ biến nhất
nmap -sS target

# Full connect scan — hoàn thành 3-way handshake
nmap -sT target

# UDP scan
nmap -sU target

# NULL scan — không có flag nào
nmap -sN target

# XMAS scan — FIN + PSH + URG
nmap -sX target

# FIN scan
nmap -sF target

# Xem port và service
nmap -sV -p 22,80,443 target

# Xem OS fingerprint (dựa vào TCP behavior)
nmap -O target
```

### Lệnh hữu ích cho SOC

```bash
# Xem tất cả connection đang mở
ss -tulnp          # Linux (thay thế netstat)
netstat -tulnp     # Linux cũ
netstat -ano       # Windows

# Filter theo port cụ thể
ss -tulnp | grep :443

# Xem connection theo state (ESTABLISHED, TIME_WAIT, SYN_SENT...)
ss -s

# Capture TCP handshake
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'

# Capture SYN flood
tcpdump -i eth0 'tcp[tcpflags] == tcp-syn'

# Capture reset packet
tcpdump -i eth0 'tcp[tcpflags] & tcp-rst != 0'
```

---

## Tóm tắt Layer 4

| Khái niệm | Một câu |
|:---|:---|
| TCP | Reliable, có handshake, đảm bảo nhận đủ và đúng thứ tự |
| UDP | Nhanh, không đảm bảo, dùng khi mất gói không ảnh hưởng nhiều |
| Port | Xác định ứng dụng nào trong máy nhận data |
| 3-way handshake | SYN → SYN-ACK → ACK |
| Flags | SYN/ACK/FIN/RST/PSH/URG — mỗi cái có vai trò riêng |
| Sequence/ACK Number | Đảm bảo thứ tự và xác nhận đã nhận |
| Window Size | Kiểm soát tốc độ gửi, tránh tràn buffer |
| Socket 5-tuple | Cách server phân biệt hàng trăm kết nối cùng lúc |
| Ephemeral port | Port tạm OS tự gán cho client, dải 49152–65535 |
