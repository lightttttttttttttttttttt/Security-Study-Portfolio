# Layer 1 — Physical Layer (Tầng Vật lý)

> **OSCP Prep — Week 1: Networking**  
> Góc nhìn: SOC Tier 3 / Pentester mindset

---

## 1. Vai trò trong mô hình OSI

Layer 1 là **tầng thấp nhất** trong mô hình OSI 7 lớp. Nhiệm vụ duy nhất:

> **Truyền các bit thô (raw bits) — chuỗi 0 và 1 — qua một phương tiện vật lý từ điểm A đến điểm B.**

Layer 1 **không biết** gì về:
- Địa chỉ (MAC, IP) → đó là việc của L2, L3
- Gói tin, frame → đó là việc của L2 trở lên
- Ý nghĩa của dữ liệu → nó chỉ truyền tín hiệu

**Ẩn dụ:** Layer 1 giống như **hệ thống ống nước trong tòa nhà**. Nó không biết nước đang chảy đi đâu, phục vụ ai — nó chỉ đảm bảo nước (tín hiệu) chảy được từ đầu này sang đầu kia.

---

## 2. Phương tiện truyền dẫn (Transmission Media)

### a) Cáp đồng (Copper Cable)

| Loại | Đặc điểm | Tốc độ | Khoảng cách max |
|:---|:---|:---|:---|
| **UTP (Unshielded Twisted Pair)** | Phổ biến nhất, rẻ. Các cặp dây xoắn để giảm nhiễu điện từ (EMI). | 1 Gbps (Cat5e), 10 Gbps (Cat6a) | ~100m |
| **STP (Shielded Twisted Pair)** | Thêm lớp vỏ bọc kim loại chống nhiễu. Dùng trong môi trường nhiều EMI. | Tương tự UTP | ~100m |
| **Coaxial** | Lõi đồng bọc lớp cách điện + kim loại. Cũ, ít dùng trong LAN hiện đại. | Tùy chuẩn | Vài trăm mét |

**Tại sao dây xoắn (Twisted)?**  
Khi hai dây dẫn song song truyền tín hiệu, chúng tạo ra trường điện từ gây nhiễu lẫn nhau (**crosstalk**). Bằng cách xoắn hai dây lại, trường điện từ của chúng **triệt tiêu nhau**, giảm nhiễu đáng kể. Mỗi cặp dây xoắn với tốc độ xoắn (twist rate) khác nhau để tránh crosstalk giữa các cặp trong cùng một sợi cáp.

### b) Cáp quang (Fiber Optic)

Truyền dữ liệu bằng **ánh sáng** (xung quang học) thay vì tín hiệu điện.

| Loại | Đặc điểm | Khoảng cách |
|:---|:---|:---|
| **Single-mode (SMF)** | Lõi rất nhỏ (~9μm), một tia sáng duy nhất, dùng laser. Đắt hơn, đi xa hơn. | Hàng chục km |
| **Multi-mode (MMF)** | Lõi lớn hơn (~50-62.5μm), nhiều tia sáng phản xạ bên trong. Rẻ hơn, data center. | ~550m (10G), ~2km (1G) |

**Ưu điểm so với cáp đồng:**
- **Miễn nhiễm EMI** hoàn toàn (ánh sáng, không phải điện)
- Tốc độ cao hơn, khoảng cách xa hơn
- **Khó bị tap (nghe lén vật lý)** hơn cáp đồng → quan trọng về bảo mật

> **SOC Tier 3 note:** Cáp quang khó bị tap hơn nhiều so với cáp đồng. Tap quang (optical tap) cần thiết bị chuyên dụng và thường gây suy hao tín hiệu có thể phát hiện được. Cáp đồng thì chỉ cần một cái vampire tap hoặc hub là xong.

### c) Wireless (Không dây)

Truyền dữ liệu qua **sóng vô tuyến (radio frequency)**.

| Chuẩn | Tần số | Tốc độ lý thuyết |
|:---|:---|:---|
| 802.11n (Wi-Fi 4) | 2.4 GHz / 5 GHz | ~600 Mbps |
| 802.11ac (Wi-Fi 5) | 5 GHz | ~3.5 Gbps |
| 802.11ax (Wi-Fi 6) | 2.4 / 5 / 6 GHz | ~9.6 Gbps |

**Bảo mật:** Wireless là **shared medium** mở hoàn toàn — bất kỳ ai trong phạm vi phủ sóng đều có thể bắt được tín hiệu. Đây là lý do mã hóa (WPA2/WPA3) và authentication là bắt buộc.

---

## 3. Tín hiệu & Mã hóa tín hiệu (Signaling & Encoding)

Layer 1 cần **chuyển đổi** giữa dữ liệu số (bit 0/1) và tín hiệu vật lý:

| Phương tiện | Loại tín hiệu | Cách biểu diễn |
|:---|:---|:---|
| Cáp đồng | Tín hiệu điện (voltage) | Thay đổi mức điện áp. VD: +5V = 1, 0V = 0 |
| Cáp quang | Xung ánh sáng | Có ánh sáng = 1, không có = 0 |
| Wireless | Sóng radio | Thay đổi tần số/biên độ/pha (modulation) |

### Line Encoding

- **NRZ (Non-Return-to-Zero):** Đơn giản nhất, +V = 1, -V = 0. Vấn đề: chuỗi dài toàn 1 hoặc toàn 0 → mất đồng bộ.
- **Manchester Encoding:** Mỗi bit biểu diễn bằng một **chuyển trạng thái** (transition) ở giữa chu kỳ bit. Ethernet 10BASE-T dùng cách này. Ưu điểm: tự đồng bộ (self-clocking).
- **4B/5B, 8B/10B:** Mã hóa 4 bit thành 5 bit (hoặc 8→10) để đảm bảo luôn có đủ transitions cho đồng bộ + phát hiện lỗi cơ bản.

---

## 4. Băng thông, Throughput, và Latency

| Thuật ngữ | Nghĩa | Ẩn dụ |
|:---|:---|:---|
| **Bandwidth** | Dung lượng tối đa lý thuyết (VD: 1 Gbps) | Số làn trên đường cao tốc |
| **Throughput** | Lượng dữ liệu thực tế truyền được / đơn vị thời gian | Số xe thực tế đi qua trong 1 giờ |
| **Latency** | Thời gian để một bit đi từ nguồn đến đích | Thời gian một chiếc xe đi hết đường |

**Throughput < Bandwidth** luôn luôn, vì: overhead protocol, collision/retransmission, congestion, lỗi truyền.

---

## 5. Các thiết bị Layer 1

### Hub (Bộ chia)
- Nhận tín hiệu từ một cổng → **broadcast ra TẤT CẢ các cổng còn lại**
- **Không thông minh**: không đọc địa chỉ, không biết gửi cho ai
- Tạo ra **một Collision Domain chung** cho tất cả thiết bị

**Ẩn dụ:** Hub giống **loa phóng thanh trong phòng**. Ai nói gì cả phòng đều nghe. Hai người nói cùng lúc → collision.

### Repeater (Bộ lặp)
- Nhận tín hiệu suy giảm → **khuếch đại/tái tạo** → truyền tiếp
- Mở rộng khoảng cách truyền dẫn
- Cũng **không thông minh**

---

## 6. Collision Domain (Miền va chạm)

> **Collision Domain = phạm vi mà nếu hai thiết bị truyền cùng lúc, tín hiệu sẽ va chạm và hỏng.**

```
Hub với 5 máy:
[PC1]──┐
[PC2]──┤
[PC3]──┼── [HUB] ── 1 Collision Domain chung
[PC4]──┤
[PC5]──┘

Switch với 5 máy:
[PC1]──port1──┐
[PC2]──port2──┤
[PC3]──port3──┼── [SWITCH] ── 5 Collision Domains riêng biệt
[PC4]──port4──┤
[PC5]──port5──┘
```

---

## 7. Half-Duplex vs Full-Duplex

| Chế độ | Mô tả | Ví dụ |
|:---|:---|:---|
| **Half-Duplex** | Chỉ truyền **HOẶC** nhận, không đồng thời | Bộ đàm, Hub-based Ethernet |
| **Full-Duplex** | Truyền **VÀ** nhận đồng thời | Switch-based Ethernet, điện thoại |

Hub → Half-Duplex → cần CSMA/CD.  
Switch + Full-Duplex → **không collision** → CSMA/CD tắt.

---

## 8. CSMA/CD (Carrier Sense Multiple Access with Collision Detection)

Giao thức truy cập đường truyền của Ethernet khi Half-Duplex (Hub environment).

### Quy trình:

```
1. LISTEN (Carrier Sense)
   → "Nghe" đường truyền. Có ai đang truyền không?
   
2. Rảnh → TRANSMIT
   Bận → ĐỢI, quay lại bước 1

3. TRONG KHI truyền → vẫn NGHE (Collision Detection)
   → Tín hiệu bị méo/biến dạng?
   
4. PHÁT HIỆN COLLISION:
   a. DỪNG truyền
   b. Gửi JAM SIGNAL (32-bit) → thông báo collision
   c. ĐỢI thời gian NGẪU NHIÊN (Binary Exponential Backoff)
   d. Quay lại bước 1
```

### Binary Exponential Backoff

- Collision lần n → chờ ngẫu nhiên trong `[0, 2^n - 1]` × slot time
- **Tối đa 16 lần** collision → bỏ cuộc, báo lỗi lên tầng trên

**Ẩn dụ:** Cuộc họp không có người điều phối. Hai người nói cùng lúc → cả hai dừng → đợi ngẫu nhiên → thử lại. Càng va nhiều → đợi càng lâu.

---

## 9. Bảo mật ở Layer 1 (SOC Tier 3)

| Mối đe dọa | Mô tả | Phát hiện / Phòng thủ |
|:---|:---|:---|
| **Physical Tap** | Nghe lén trực tiếp trên cáp | Kiểm tra vật lý, OTDR (quang), camera + access control |
| **Rogue Device** | Thiết bị không phép cắm vào mạng | 802.1X (Port-based NAC), MAC filtering |
| **Jamming (Wireless)** | Phát nhiễu gián đoạn Wi-Fi | Wireless IDS/IPS, RF anomaly detection |
| **Evil Twin AP** | AP giả mạo cùng SSID | Wireless IDS, EAP-TLS (certificate-based auth) |

> **Mindset:** Phần lớn tấn công L1 đòi hỏi **physical access**. Nếu attacker có physical access, họ bypass được mọi control phần mềm. Physical security là lớp phòng thủ đầu tiên.
