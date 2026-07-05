# Layer 2 — Data Link Layer (Tầng Liên kết Dữ liệu)

> **OSCP Prep — Week 1: Networking**  
> Góc nhìn: SOC Tier 3 / Pentester mindset

---

## 1. Vai trò trong mô hình OSI

Layer 2 nằm **ngay trên** Layer 1. Nhiệm vụ chính:

> **Đóng gói dữ liệu thành frame, đánh địa chỉ vật lý (MAC), kiểm soát truy cập đường truyền, và phát hiện lỗi — để truyền dữ liệu tin cậy giữa hai thiết bị TRỰC TIẾP KẾT NỐI (hop-by-hop) trên cùng một mạng LAN.**

Layer 2 chỉ lo **hop-by-hop** (giữa hai thiết bị liền kề). Truyền **end-to-end** qua nhiều mạng là việc của Layer 3 (IP).

**Ẩn dụ:** Nếu L1 là đường ống nước, thì L2 là **hệ thống đánh địa chỉ nhà** trên cùng một con phố. Biết gửi cho nhà nào trên phố này, nhưng không biết gì về thành phố khác.

---

## 2. Hai sublayer của Layer 2

### a) LLC — Logical Link Control (IEEE 802.2)
- Giao tiếp với **Layer 3** (IP, IPv6, ARP...)
- Multiplexing: nhiều protocol L3 dùng chung một interface vật lý
- Xác định protocol tầng trên cần nhận dữ liệu (dựa vào **EtherType**)

### b) MAC — Media Access Control (IEEE 802.3)
- Giao tiếp với **Layer 1**
- Quản lý địa chỉ MAC, đóng gói frame
- Kiểm soát truy cập đường truyền (CSMA/CD)
- Phát hiện lỗi (FCS/CRC)

```
┌─────────────────────────┐
│      Layer 3 (IP)       │
├─────────────────────────┤
│   LLC (Logical Link)    │ ← Nói chuyện với L3
├─────────────────────────┤
│   MAC (Media Access)    │ ← Nói chuyện với L1
├─────────────────────────┤
│    Layer 1 (Physical)   │
└─────────────────────────┘
```

---

## 3. Địa chỉ MAC (MAC Address)

### Cấu trúc

- **48 bit** (6 byte), hex, phân cách bởi `:` hoặc `-`
- VD: `AA:BB:CC:DD:EE:FF`

```
┌──────────────────────┬──────────────────────┐
│   OUI (24 bit)       │   NIC Specific       │
│   = Nhà sản xuất     │   (24 bit)           │
│   (IEEE cấp phát)    │   = Serial do NSX    │
├──────────────────────┴──────────────────────┤
│  AA : BB : CC : DD : EE : FF               │
│  └─ OUI ──┘   └── Device ID ──┘            │
└─────────────────────────────────────────────┘
```

- **OUI (3 byte đầu):** Định danh nhà sản xuất. VD: `00:50:56` → VMware, `DC:A6:32` → Raspberry Pi
- **NIC Specific (3 byte sau):** Serial unique trong OUI đó

### Các loại MAC đặc biệt

| Loại | Giá trị | Ý nghĩa |
|:---|:---|:---|
| **Unicast** | Bit thấp nhất byte đầu = 0 | Gửi cho MỘT thiết bị |
| **Multicast** | Bit thấp nhất byte đầu = 1 | Gửi cho MỘT NHÓM |
| **Broadcast** | `FF:FF:FF:FF:FF:FF` | Gửi cho TẤT CẢ trên segment |

**Nhận biết nhanh:** Ký tự hex thứ 2 — **chẵn** (0,2,4,6,8,A,C,E) = Unicast, **lẻ** = Multicast.

### MAC có thay đổi được không?

- MAC được gán **cứng (burned-in)** vào NIC → nhưng phần mềm **có thể spoof**
- `ip link set dev eth0 address XX:XX:XX:XX:XX:XX`
- **Bảo mật:** MAC filtering KHÔNG đáng tin cậy vì MAC spoofing quá dễ

---

## 4. Ethernet Frame — Cấu trúc chi tiết (IEEE 802.3)

```
┌──────────┬─────┬────────────┬────────────┬───────────┬─────────────────┬─────┐
│ Preamble │ SFD │ Dest MAC   │ Source MAC │ EtherType │    Payload      │ FCS │
│ 7 bytes  │ 1B  │  6 bytes   │  6 bytes   │  2 bytes  │  46-1500 bytes  │ 4B  │
└──────────┴─────┴────────────┴────────────┴───────────┴─────────────────┴─────┘
```

### Giải thích từng trường

#### Preamble (7 bytes)
- `10101010` lặp 7 lần → **đồng bộ clock** giữa bên gửi/nhận
- Không tính là phần frame "thật"

#### SFD — Start Frame Delimiter (1 byte)
- `10101011` — hai bit cuối `11` báo: **"Frame bắt đầu ngay sau byte này"**

#### Destination MAC (6 bytes)
- MAC **bên nhận** (Unicast, Multicast, hoặc Broadcast)
- Switch dùng trường này để quyết định **forward ra cổng nào**

#### Source MAC (6 bytes)
- MAC **bên gửi** — **luôn là Unicast**
- Switch dùng trường này để **học** (cập nhật MAC Address Table)

#### EtherType / Length (2 bytes)
- Giá trị **≥ 0x0600** → EtherType (protocol L3):
  - `0x0800` = IPv4
  - `0x0806` = ARP
  - `0x86DD` = IPv6
  - `0x8100` = 802.1Q VLAN tag
- Giá trị **< 0x0600** → Length (frame IEEE 802.3 cũ)

#### Payload / Data (46–1500 bytes)
- Dữ liệu từ Layer 3 trở lên
- **MTU = 1500 bytes** (Ethernet chuẩn)
- **Tối thiểu 46 bytes** — ngắn hơn thì thêm **padding** (0x00)
- Lý do minimum: frame quá ngắn có thể xong trước khi collision được phát hiện

> **Jumbo Frame:** Một số mạng hỗ trợ MTU tới **9000 bytes** (data center, iSCSI). Tất cả thiết bị trên đường truyền phải hỗ trợ, không thì frame bị drop.

#### FCS — Frame Check Sequence (4 bytes)
- Chứa **CRC-32**
- Bên gửi tính CRC (Dest MAC → Payload) → ghi vào FCS
- Bên nhận tính lại → khớp = OK, không khớp = **DROP** (không sửa, không gửi lại)

### Kích thước Frame

| Thành phần | Kích thước |
|:---|:---|
| Preamble + SFD | 8 bytes |
| Header (Dest + Src + EtherType) | 14 bytes |
| Payload | 46–1500 bytes |
| FCS | 4 bytes |
| **Tổng (không tính Preamble/SFD)** | **64–1518 bytes** |

- < 64 bytes → **Runt frame** (thường do collision) → drop
- \> 1518 bytes (non-jumbo) → **Giant frame** → drop

---

## 5. Switch — Thiết bị Layer 2

### MAC Address Table (CAM Table)

```
┌───────────────────┬──────┬─────────┐
│    MAC Address    │ Port │  VLAN   │
├───────────────────┼──────┼─────────┤
│ AA:BB:CC:11:22:33 │ Fa0/1│  10     │
│ DD:EE:FF:44:55:66 │ Fa0/3│  10     │
│ 11:22:33:44:55:66 │ Fa0/5│  20     │
└───────────────────┴──────┴─────────┘
```

### 3 hành động xử lý frame

#### 1. LEARN
- Nhận frame → đọc **Source MAC** → ghi vào bảng kèm port
- Entry có **aging timer** (mặc định 300s). Hết hạn → xóa

#### 2. FORWARD
- Đọc **Dest MAC** → tra bảng → tìm thấy → forward ra **đúng port**
- Dest = Broadcast → forward ra **tất cả port** (trừ port nhận)

#### 3. FLOOD
- **Không tìm thấy** Dest MAC (unknown unicast) → forward ra **tất cả port** (trừ port nhận)
- Khi thiết bị đích reply → switch learn được → lần sau không flood nữa

```
Ví dụ:

Bước 1: PC-A (AAAA) → PC-C (CCCC) qua port 1
        Switch LEARN: AAAA ở port 1
        CCCC chưa biết → FLOOD ra port 2, 3, 4

Bước 2: PC-C reply từ port 3
        Switch LEARN: CCCC ở port 3
        AAAA ở port 1 → FORWARD ra port 1

Bước 3: PC-A gửi lại cho PC-C
        CCCC ở port 3 → FORWARD ra port 3
        Không flood nữa!
```

---

## 6. Broadcast Domain (Miền quảng bá)

> **Broadcast Domain = phạm vi mà frame broadcast (`FF:FF:FF:FF:FF:FF`) được chuyển tiếp tới.**

| Thiết bị | Chia Collision Domain? | Chia Broadcast Domain? |
|:---|:---|:---|
| **Hub** | ❌ Không | ❌ Không |
| **Switch** | ✅ Có (mỗi port) | ❌ Không (trừ VLAN) |
| **Router** | ✅ Có (mỗi interface) | ✅ Có (mỗi interface) |

---

## 7. ARP — Address Resolution Protocol

### Vấn đề
L3 biết IP đích. Nhưng L2 cần **MAC đích** để gửi frame. ARP là cầu nối:

> **ARP: "Tôi biết IP của anh, cho tôi MAC của anh."**

### Quy trình chi tiết

```
PC-A (10.0.0.1, MAC: AAAA) muốn gửi cho PC-B (10.0.0.2, MAC: ???)

Bước 1: Kiểm tra ARP Cache → CHƯA CÓ

Bước 2: ARP REQUEST (Broadcast)
        ┌─────────────────────────────────────────┐
        │ Dest MAC:   FF:FF:FF:FF:FF:FF (Bcast)   │
        │ Src MAC:    AAAA                         │
        │ EtherType:  0x0806 (ARP)                 │
        ├─────────────────────────────────────────┤
        │ Opcode:     1 (Request)                  │
        │ Sender MAC: AAAA  |  Sender IP: 10.0.0.1│
        │ Target MAC: 0000  |  Target IP: 10.0.0.2│
        └─────────────────────────────────────────┘
        → Gửi ra TẤT CẢ thiết bị trên LAN

Bước 3: PC-C (10.0.0.3): "Không phải tôi" → bỏ qua
        PC-B (10.0.0.2): "Là tôi!" → xử lý

Bước 4: ARP REPLY (Unicast cho PC-A)
        ┌─────────────────────────────────────────┐
        │ Dest MAC:   AAAA (Unicast)               │
        │ Src MAC:    BBBB                         │
        │ Opcode:     2 (Reply)                    │
        │ Sender MAC: BBBB  |  Sender IP: 10.0.0.2│
        │ Target MAC: AAAA  |  Target IP: 10.0.0.1│
        └─────────────────────────────────────────┘

Bước 5: PC-A cập nhật ARP Cache:
        10.0.0.2 → BBBB (Dynamic, TTL ~120-300s)
```

### Lệnh xem ARP Table

```bash
# Windows
arp -a

# Linux
arp -n
ip neigh show
```

---

## 8. ARP Spoofing / ARP Poisoning

### Nguyên lý

ARP **không có xác thực**. Ai cũng có thể gửi ARP Reply không cần Request trước (**Gratuitous ARP**).

### Kịch bản MitM

```
Bình thường:
[Victim 10.0.0.10] ←→ [Gateway 10.0.0.1]

Attacker (10.0.0.50, MAC: XXXX) gửi:
1. Cho Victim: "10.0.0.1 có MAC = XXXX" (giả làm Gateway)
2. Cho Gateway: "10.0.0.10 có MAC = XXXX" (giả làm Victim)

Kết quả:
[Victim] → [Attacker] → [Gateway] → Internet
         ←             ←
→ MAN-IN-THE-MIDDLE hoàn chỉnh!
```

### Công cụ (biết để phòng thủ)

```bash
arpspoof -i eth0 -t 10.0.0.10 10.0.0.1
bettercap -iface eth0 -eval "arp.spoof on"
ettercap -T -M arp:remote /10.0.0.10// /10.0.0.1//
```

### Phòng thủ

| Biện pháp | Hiệu quả |
|:---|:---:|
| **Dynamic ARP Inspection (DAI)** | ★★★★★ |
| **DHCP Snooping** (nền tảng cho DAI) | ★★★★ |
| **802.1X (Port-based NAC)** | ★★★★ |
| **Static ARP Entry** | ★★★ (khó scale) |
| **ARP monitoring** (arpwatch, XArp) | ★★★ (detect, không prevent) |
| **HTTPS/SSH** (encrypt end-to-end) | ★★★★ (defense in depth) |

> **SOC Tier 3 dấu hiệu phát hiện:**
> - MAC flapping (cùng MAC trên nhiều port)
> - Gratuitous ARP bất thường tần suất cao
> - Một MAC claim nhiều IP
> - DAI drop logs tăng đột biến
> - Trong pcap: nhiều ARP Reply mà không có Request tương ứng

---

## 9. VLAN — Virtual LAN

### VLAN là gì?

Chia **một switch vật lý thành nhiều broadcast domain logic** mà không cần thêm phần cứng.

```
Port 1,2,3 → VLAN 10 (Kế toán)   ── Broadcast Domain A
Port 4,5,6 → VLAN 20 (Kỹ thuật)  ── Broadcast Domain B

Hai VLAN KHÔNG nói chuyện được ở L2!
```

### Tại sao cần?
1. **Giảm broadcast domain** → giảm traffic thừa
2. **Bảo mật** → cách ly phòng ban
3. **Linh hoạt** → không phụ thuộc vị trí vật lý

### 802.1Q — VLAN Tagging

Khi frame qua **trunk port** (giữa các switch), thêm **4 bytes tag**:

```
Frame có 802.1Q tag:
┌──────────┬──────────┬──────────┬───────────┬─────────┬─────┐
│ Dest MAC │ Src MAC  │ 802.1Q   │ EtherType │ Payload │ FCS │
│          │          │ Tag (4B) │           │         │     │
└──────────┴──────────┴──────────┴───────────┴─────────┴─────┘

802.1Q Tag (4 bytes):
┌─────────────────┬─────┬─────┬──────────────┐
│ TPID (2B)       │ PCP │ DEI │ VLAN ID      │
│ = 0x8100        │ 3b  │ 1b  │ 12 bits      │
│                 │     │     │ (1-4094)     │
└─────────────────┴─────┴─────┴──────────────┘
```

### Access Port vs Trunk Port

| Loại | Chức năng | Tag? |
|:---|:---|:---|
| **Access** | Kết nối end device. Thuộc 1 VLAN. | Không tag |
| **Trunk** | Giữa switch-switch. Nhiều VLAN. | Có 802.1Q tag |

### VLAN Hopping Attack

> **SOC Alert:**
> 1. **Switch Spoofing:** Giả làm switch → thương lượng trunk → nhận mọi VLAN  
>    → Phòng: Tắt DTP, `switchport mode access`
> 2. **Double Tagging:** 2 lớp 802.1Q tag → switch gỡ tag ngoài → frame nhảy VLAN  
>    → Phòng: Không dùng VLAN 1 làm native VLAN

---

## 10. STP — Spanning Tree Protocol (IEEE 802.1D)

### Vấn đề: Layer 2 Loop

Đường dự phòng giữa switch → vòng lặp → **Broadcast Storm** → mạng sập!

```
[Switch A] ════════ [Switch B]
    ║                   ║
    ╚═══════════════════╝   ← LOOP!
    
Hậu quả:
1. Broadcast Storm (frame nhân bản vô hạn)
2. MAC Table Instability (MAC flapping)
3. Multiple Frame Delivery (nhận nhiều bản copy)
```

### STP giải quyết

Block một số port → tạo **cây logic không vòng lặp**:

```
[Switch A] ════════ [Switch B]
    ║                   ║
    ╚══════╳════════════╝  ← BLOCKED bởi STP
    
Link chính chết → STP mở block → chuyển sang link dự phòng.
```

### Thuật toán STP (tóm tắt)
1. **Bầu Root Bridge** (Bridge ID thấp nhất thắng)
2. **Chọn Root Port** (cost thấp nhất đến Root Bridge, trên mỗi non-root switch)
3. **Chọn Designated Port** (cost thấp nhất trên mỗi segment)
4. **Block còn lại**

### Port States

| State | Nhận BPDU? | Học MAC? | Forward? | Thời gian |
|:---|:---|:---|:---|:---|
| **Blocking** | ✅ | ❌ | ❌ | 20s |
| **Listening** | ✅ | ❌ | ❌ | 15s |
| **Learning** | ✅ | ✅ | ❌ | 15s |
| **Forwarding** | ✅ | ✅ | ✅ | — |

Convergence STP cổ điển: **~50 giây**. RSTP (802.1w): **~6 giây**.

---

## 11. Bảo mật Layer 2 (SOC Tier 3)

| Mối đe dọa | Kỹ thuật | Dấu hiệu |
|:---|:---|:---|
| **ARP Spoofing** | ARP Reply giả → redirect traffic | MAC flapping, gratuitous ARP bất thường |
| **MAC Flooding** | Random source MAC → tràn CAM → switch thành hub | CAM full, port security violations |
| **VLAN Hopping** | Double tagging / switch spoofing | DTP từ access port, double 802.1Q tag |
| **STP Manipulation** | BPDU giả → chiếm Root Bridge | Topology change, BPDU Guard violations |
| **DHCP Starvation** | Spam DHCP Discover → hết IP pool | Pool cạn, nhiều request từ 1 port |

### Switch Hardening Best Practices

```
! Port Security
switchport port-security
switchport port-security maximum 2
switchport port-security violation shutdown

! DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10,20

! Dynamic ARP Inspection
ip arp inspection vlan 10,20

! BPDU Guard
spanning-tree bpduguard enable

! Root Guard
spanning-tree guard root

! Tắt DTP
switchport mode access
switchport nonegotiate

! Tắt CDP/LLDP trên access port
no cdp enable
no lldp transmit
no lldp receive

! Shutdown unused ports
interface range Fa0/10-24
 shutdown
```

---

## Tóm tắt L1 vs L2

| Tiêu chí | Layer 1 (Physical) | Layer 2 (Data Link) |
|:---|:---|:---|
| **PDU** | Bit | Frame |
| **Địa chỉ** | Không có | MAC Address (48-bit) |
| **Thiết bị** | Hub, Repeater | Switch, Bridge |
| **Phát hiện lỗi** | Không | Có (FCS/CRC-32) |
| **Tấn công đặc trưng** | Physical tap, Jamming | ARP Spoofing, MAC Flooding, VLAN Hopping |
