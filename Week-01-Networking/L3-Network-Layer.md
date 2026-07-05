# Layer 3 — Network Layer (Tầng Mạng)

> **OSCP Prep — Week 1: Networking**
> Góc nhìn: SOC Tier 3 / Pentester mindset

---

## 1. Vai trò trong mô hình OSI

Layer 3 nằm trên Layer 2. Nhiệm vụ chính:

> **Định tuyến (routing) gói tin từ nguồn đến đích XUYÊN QUA nhiều mạng khác nhau, dựa trên địa chỉ logic (IP address).**

So sánh với L2:
- **L2** chỉ lo hop-by-hop (cùng 1 LAN, dùng MAC)
- **L3** lo end-to-end (qua nhiều mạng/router, dùng IP)

**Ẩn dụ:** L2 là gửi thư trong cùng 1 con phố (biết số nhà). L3 là gửi thư xuyên quốc gia — cần mã bưu chính (IP), hệ thống bưu điện (router) chuyển tiếp qua nhiều trạm.

---

## 2. IP Address — Địa chỉ logic

### IP vs MAC — Phân biệt rõ ràng

| | MAC Address | IP Address |
|:---|:---|:---|
| **Tầng** | Layer 2 | Layer 3 |
| **Loại** | Địa chỉ vật lý (burned-in vào NIC) | Địa chỉ logic (gán bằng phần mềm, thay đổi được) |
| **Phạm vi** | Chỉ có ý nghĩa trong **cùng 1 LAN** | Có ý nghĩa **toàn cầu**, xuyên qua nhiều mạng |
| **Ai dùng** | Switch (tra MAC table → forward) | Router (tra routing table → route) |
| **Thay đổi khi đi qua router?** | **CÓ** — mỗi hop thay đổi Src/Dst MAC | **KHÔNG** — Src/Dst IP giữ nguyên từ đầu đến cuối |

Điểm quan trọng nhất cần nhớ:

```
PC-A (10.0.0.1) gửi đến Server (10.0.2.5) qua Router

Hop 1: PC-A → Router
  L2: Src MAC = PC-A,     Dst MAC = Router port 1   ← THAY ĐỔI mỗi hop
  L3: Src IP  = 10.0.0.1, Dst IP  = 10.0.2.5        ← GIỮ NGUYÊN

Hop 2: Router → Server
  L2: Src MAC = Router port 2, Dst MAC = Server      ← THAY ĐỔI
  L3: Src IP  = 10.0.0.1,     Dst IP  = 10.0.2.5    ← GIỮ NGUYÊN
```

MAC thay đổi mỗi hop, IP giữ nguyên end-to-end. Đây là khác biệt cốt lõi.

### IPv4 Address

- **32 bit**, chia thành 4 octet, mỗi octet 8 bit
- Biểu diễn dạng **dotted decimal**: `192.168.1.100`
- Tổng cộng: 2^32 = ~4.3 tỷ địa chỉ (không đủ cho thế giới → lý do ra đời IPv6 và NAT)

```
  192    .   168    .    1     .   100
11000000  10101000  00000001  01100100
```

### Private IP vs Public IP

**Private IP** là các dải IP chỉ dùng trong mạng nội bộ (LAN), **không được route trên Internet**.

| Dải Private | Phạm vi | Dùng cho |
|:---|:---|:---|
| `10.0.0.0/8` | 10.0.0.0 – 10.255.255.255 | Mạng lớn (doanh nghiệp) |
| `172.16.0.0/12` | 172.16.0.0 – 172.31.255.255 | Mạng vừa |
| `192.168.0.0/16` | 192.168.0.0 – 192.168.255.255 | Mạng nhỏ (gia đình, văn phòng nhỏ) |

**Public IP** là IP được cấp phát bởi ISP, **duy nhất trên toàn Internet**, dùng để giao tiếp ra bên ngoài.

```
Mạng nhà ông:
[PC: 192.168.1.100] ──→ [Router] ──→ Internet
      Private IP          NAT          Public IP: 113.x.x.x
                     (đổi Private → Public)
```

**NAT (Network Address Translation):** Router đổi Private IP → Public IP khi gửi ra Internet, và ngược lại khi nhận về. Đây là lý do nhiều thiết bị trong nhà dùng chung 1 Public IP.

### Các địa chỉ IP đặc biệt

| Địa chỉ | Ý nghĩa |
|:---|:---|
| `127.0.0.1` | Loopback — gửi cho chính mình, không ra mạng |
| `0.0.0.0` | "Tất cả interfaces" hoặc "chưa có IP" (tùy ngữ cảnh) |
| `255.255.255.255` | Limited broadcast — broadcast trong LAN hiện tại |
| `169.254.x.x` | APIPA — tự gán khi không tìm được DHCP server |

---

## 3. Subnet & Subnetting

### Subnet Mask là gì?

Subnet mask xác định **phần nào của IP là Network, phần nào là Host**.

```
IP:          192.168.1.100
Subnet Mask: 255.255.255.0    (hoặc /24)

Chuyển sang binary:
IP:   11000000.10101000.00000001.01100100
Mask: 11111111.11111111.11111111.00000000
      └──── Network (24 bit) ────┘└Host─┘

Network Address: 192.168.1.0   (phần host = toàn 0)
Broadcast:       192.168.1.255 (phần host = toàn 1)
Host khả dụng:   192.168.1.1 – 192.168.1.254
Số host:         2^8 - 2 = 254  (trừ network address và broadcast)
```

### CIDR Notation

`/24` nghĩa là **24 bit đầu là Network**, còn lại là Host.

| CIDR | Subnet Mask | Số host khả dụng |
|:---|:---|:---|
| /8 | 255.0.0.0 | 16,777,214 |
| /16 | 255.255.0.0 | 65,534 |
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /30 | 255.255.255.252 | 2 |
| /32 | 255.255.255.255 | 1 (single host) |

### Ví dụ tính Subnetting: `192.168.1.100/26`

```
/26 → 26 bit network, 6 bit host
Subnet mask: 255.255.255.192

Số host/subnet: 2^6 - 2 = 62 host khả dụng
Số subnet trong /24: 2^(26-24) = 4 subnet

Các subnet:
  192.168.1.0   – 192.168.1.63    (Subnet 1)
  192.168.1.64  – 192.168.1.127   (Subnet 2)  ← 100 nằm ở đây
  192.168.1.128 – 192.168.1.191   (Subnet 3)
  192.168.1.192 – 192.168.1.255   (Subnet 4)

Với IP 192.168.1.100:
  Network Address: 192.168.1.64
  Broadcast:       192.168.1.127
  Host range:      192.168.1.65 – 192.168.1.126
  Subnet mask:     255.255.255.192
```

**Công thức nhớ nhanh:**
- Số host = `2^(32 - prefix) - 2`
- Block size (khoảng cách giữa các subnet) = `256 - octet cuối của mask`
  - /26 → mask octet cuối = 192 → block = 256 - 192 = **64**
  - Các subnet bắt đầu tại: 0, 64, 128, 192

---

## 4. IPv4 Header — Cấu trúc chi tiết

IPv4 header tối thiểu **20 bytes**, tối đa 60 bytes (nếu có Options).

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |    DSCP   |ECN|         Total Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|    Fragment Offset      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |       Header Checksum         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source IP Address                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination IP Address                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (nếu có)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Giải thích từng trường quan trọng

#### Version (4 bit)
- `4` = IPv4, `6` = IPv6

#### IHL — Internet Header Length (4 bit)
- Chiều dài header tính bằng **số lượng word 32-bit**
- Giá trị thường gặp: `5` (= 5 × 4 = **20 bytes**, không có Options)
- Giá trị tối đa: `15` (= 15 × 4 = 60 bytes)

#### DSCP / ECN (8 bit tổng)
- **DSCP (6 bit):** Đánh dấu ưu tiên cho QoS (Quality of Service)
- **ECN (2 bit):** Thông báo tắc nghẽn mà không cần drop packet

#### Total Length (16 bit)
- Tổng chiều dài **toàn bộ packet** (header + data), tính bằng byte
- Tối đa: 2^16 - 1 = **65,535 bytes**

#### Identification (16 bit)
- ID duy nhất cho mỗi packet gốc
- Dùng để **ghép lại các fragment** thuộc cùng 1 packet

#### Flags (3 bit)
- Bit 0: Reserved (luôn = 0)
- **Bit 1: DF (Don't Fragment)** — nếu = 1, router KHÔNG được phân mảnh. Nếu packet quá lớn → drop và gửi ICMP "Fragmentation Needed"
- **Bit 2: MF (More Fragments)** — nếu = 1, còn fragment phía sau. Fragment cuối cùng có MF = 0

#### Fragment Offset (13 bit)
- Vị trí của fragment trong packet gốc, tính bằng đơn vị **8 bytes**
- Fragment đầu tiên: offset = 0

#### TTL — Time to Live (8 bit)

> **TTL = số hop tối đa mà packet được phép đi qua.**

- Mỗi router xử lý packet → **giảm TTL đi 1**
- TTL = 0 → router **DROP packet** + gửi **ICMP Time Exceeded** về cho nguồn
- Mục đích: **ngăn packet đi vòng vòng mãi** (routing loop) → chiếm bandwidth vô hạn

```
Ví dụ: PC-A gửi packet với TTL = 64

PC-A (TTL=64) → Router1 (TTL=63) → Router2 (TTL=62) → ... → Server (TTL=52)

Nếu có routing loop:
Router1 (TTL=3) → Router2 (TTL=2) → Router1 (TTL=1) → Router2 (TTL=0) → DROP!
                                                         + gửi ICMP Time Exceeded
```

**TTL mặc định theo OS:**

| OS | TTL mặc định |
|:---|:---|
| Linux | 64 |
| Windows | 128 |
| Cisco/Network devices | 255 |

> **SOC Tier 3 note:** Nhìn TTL trong packet capture có thể **đoán OS** của máy gửi. Nếu nhận packet có TTL = 118, khả năng cao là Windows (128 - 10 hop = 118). Đây là kỹ thuật **OS Fingerprinting** thụ động.

#### Protocol (8 bit)
- Cho biết payload chứa protocol gì ở Layer 4:

| Giá trị | Protocol |
|:---|:---|
| 1 | ICMP |
| 6 | TCP |
| 17 | UDP |
| 47 | GRE |
| 50 | ESP (IPsec) |

#### Header Checksum (16 bit)
- Checksum **chỉ cho header**, không tính data
- Mỗi router phải **tính lại** (vì TTL thay đổi mỗi hop)

#### Source IP / Destination IP (32 bit mỗi trường)
- IP nguồn và IP đích
- **Giữ nguyên** suốt hành trình (trừ khi NAT can thiệp)

---

## 5. Fragmentation (Phân mảnh)

### Tại sao cần?

Mỗi link có **MTU (Maximum Transmission Unit)** — kích thước tối đa của packet L3 mà link đó chấp nhận. Ethernet chuẩn: MTU = 1500 bytes.

Nếu packet lớn hơn MTU của link tiếp theo → router phải **chia nhỏ (fragment)** packet.

```
Packet gốc: 4000 bytes, MTU = 1500

Fragment 1: 1500 bytes (Header 20 + Data 1480)
  ID = 1234, MF = 1, Offset = 0

Fragment 2: 1500 bytes (Header 20 + Data 1480)
  ID = 1234, MF = 1, Offset = 185  (1480 / 8 = 185)

Fragment 3: 1060 bytes (Header 20 + Data 1040)
  ID = 1234, MF = 0, Offset = 370  (2960 / 8 = 370)

Tổng data: 1480 + 1480 + 1040 = 4000 - 20 (header gốc) = 3980 ✓
```

### Ai fragment? Ai ráp lại?

- **Router** fragment khi packet > MTU của next hop
- **Máy đích (destination host)** ráp lại — KHÔNG phải router trung gian
- Nếu **1 fragment bị mất** → toàn bộ packet bị drop (phải gửi lại từ đầu)

### DF Flag và Path MTU Discovery

Hiện đại, người ta tránh fragmentation bằng **Path MTU Discovery (PMTUD)**:
1. Gửi packet với **DF = 1** (Don't Fragment)
2. Nếu router gặp MTU nhỏ hơn → drop packet + gửi **ICMP "Fragmentation Needed"** kèm MTU của link đó
3. Máy gửi nhận ICMP → giảm kích thước packet → gửi lại
4. Lặp lại đến khi tìm được MTU nhỏ nhất trên toàn đường đi

> **SOC Tier 3 note:** Attacker dùng **fragmentation** để bypass firewall/IDS. Packet bị chia nhỏ đến mức mỗi fragment không chứa đủ header L4 (TCP/UDP port) → firewall không đọc được port → cho qua. Đây là lý do IDS/firewall hiện đại phải có khả năng **reassemble fragments** trước khi kiểm tra.

---

## 6. ICMP — Internet Control Message Protocol

### ICMP là gì?

ICMP là protocol **báo lỗi và chẩn đoán** của L3. Nó không truyền dữ liệu người dùng — chỉ truyền **thông báo điều khiển**.

ICMP nằm trong IP packet (Protocol = 1).

### Các loại ICMP message quan trọng

| Type | Code | Tên | Ý nghĩa |
|:---|:---|:---|:---|
| 0 | 0 | Echo Reply | Phản hồi ping |
| 3 | 0 | Destination Unreachable — Net Unreachable | Không tìm được route đến mạng đích |
| 3 | 1 | Destination Unreachable — Host Unreachable | Tìm được mạng nhưng host không phản hồi |
| 3 | 3 | Destination Unreachable — Port Unreachable | Host nhận được nhưng port đó không mở (UDP) |
| 3 | 4 | Fragmentation Needed (DF set) | Packet quá lớn, DF=1, không fragment được |
| 3 | 13 | Administratively Prohibited | Bị firewall/ACL chặn |
| 5 | x | Redirect | Router báo cho host biết có route tốt hơn |
| 8 | 0 | Echo Request | Gửi ping |
| 11 | 0 | Time Exceeded — TTL expired | TTL = 0, packet bị drop |

### Công cụ dùng ICMP

**ping** — gửi Echo Request, nhận Echo Reply:
```bash
ping 8.8.8.8

# Output:
# Reply from 8.8.8.8: bytes=32 time=25ms TTL=118
# → TTL=118 → Windows server (128-10 hop), 25ms latency
```

**traceroute / tracert** — tìm đường đi qua các router:
```bash
# Linux
traceroute 8.8.8.8

# Windows
tracert 8.8.8.8

# Nguyên lý: Gửi packet với TTL = 1, 2, 3, ...
# TTL=1 → Router 1 drop + gửi ICMP Time Exceeded (lộ IP router 1)
# TTL=2 → Router 2 drop + gửi ICMP Time Exceeded (lộ IP router 2)
# TTL=3 → ... đến khi tới đích
# → Biết được toàn bộ đường đi!
```

> **SOC Tier 3 note:**
> - **ICMP Tunnel:** Attacker giấu dữ liệu trong payload của ICMP Echo Request/Reply để exfiltrate data → bypass firewall chỉ block TCP/UDP. Tool: `icmpsh`, `ptunnel`.
> - **Ping Sweep:** Gửi ICMP Echo Request đến toàn bộ dải IP để tìm host sống → recon phase. Tool: `nmap -sn 10.0.0.0/24`.
> - **ICMP Redirect Attack:** Gửi ICMP Redirect giả để đổi route của victim → MitM ở L3.
> - Nhiều tổ chức **block ICMP** hoàn toàn → ping không phản hồi không có nghĩa host chết.

---

## 7. Routing — Định tuyến

### Router là gì?

Router là thiết bị L3. Nhận packet → đọc **Destination IP** → tra **Routing Table** → forward ra interface phù hợp.

Mỗi interface của router là một **broadcast domain** và **collision domain** riêng.

### Routing Table

```bash
# Linux
ip route show
# hoặc
route -n

# Windows
route print
```

Ví dụ routing table:
```
Destination     Gateway         Interface
10.0.1.0/24     0.0.0.0         eth0        ← Directly connected
10.0.2.0/24     10.0.1.1        eth0        ← Qua gateway 10.0.1.1
0.0.0.0/0       10.0.1.254      eth0        ← Default route (gateway mặc định)
```

### Default Gateway

> **Default Gateway = router mà máy gửi packet đến khi destination IP KHÔNG nằm trong cùng subnet.**

```
PC (192.168.1.100/24, Gateway: 192.168.1.1)

Gửi cho 192.168.1.50 → cùng subnet → gửi trực tiếp (ARP → tìm MAC → gửi frame)
Gửi cho 10.0.2.5    → KHÁC subnet → gửi đến Gateway (192.168.1.1) → router lo tiếp
```

Quy trình quyết định:
1. Destination IP **AND** Subnet Mask = Network Address của mình? → cùng subnet → gửi trực tiếp
2. Không? → gửi đến Default Gateway

### Static vs Dynamic Routing

| Loại | Mô tả | Dùng khi |
|:---|:---|:---|
| **Static** | Admin cấu hình tay từng route | Mạng nhỏ, đơn giản |
| **Dynamic** | Router tự trao đổi route với nhau qua protocol (OSPF, BGP, RIP, EIGRP) | Mạng lớn, phức tạp |

---

## 8. Bảo mật Layer 3 (SOC Tier 3)

| Mối đe dọa | Kỹ thuật | Phát hiện / Phòng thủ |
|:---|:---|:---|
| **IP Spoofing** | Giả mạo Source IP để ẩn danh hoặc bypass ACL | Ingress/Egress filtering (BCP38), uRPF (Unicast Reverse Path Forwarding) |
| **ICMP Tunnel** | Giấu dữ liệu trong ICMP payload để exfiltrate | Giám sát kích thước ICMP payload bất thường, block ICMP không cần thiết |
| **ICMP Redirect** | Gửi ICMP Redirect giả → đổi route của victim | Disable ICMP redirect accept trên host |
| **Route Hijacking (BGP)** | Quảng bá route giả trên BGP → chiếm traffic | RPKI, BGP route monitoring |
| **Fragmentation Attack** | Fragment nhỏ để bypass firewall/IDS | Firewall/IDS có khả năng reassemble trước khi inspect |
| **Ping of Death** | Gửi ICMP packet > 65535 bytes → crash hệ thống cũ | OS hiện đại đã patch, vẫn cần patch management |
| **Smurf Attack** | Gửi ICMP Echo Request đến broadcast address với spoofed source → DDoS | Block directed broadcast, ingress filtering |

### Lệnh hữu ích cho SOC

```bash
# Xem routing table
ip route show          # Linux
route print            # Windows

# Xem ARP + IP neighbor
ip neigh show          # Linux
arp -a                 # Windows

# Kiểm tra kết nối
ping -c 4 10.0.0.1               # Linux
ping -n 4 10.0.0.1               # Windows
traceroute 10.0.0.1              # Linux
tracert 10.0.0.1                 # Windows

# Xem interface và IP
ip addr show           # Linux
ipconfig /all          # Windows

# Packet capture (filter ICMP)
tcpdump -i eth0 icmp
tcpdump -i eth0 'ip[6:2] & 0x3fff != 0'   # Bắt fragmented packets
```

---

## Tóm tắt Layer 3

| Khái niệm | Một câu |
|:---|:---|
| **IP Address** | Địa chỉ logic 32-bit, xác định thiết bị trên mạng, giữ nguyên end-to-end |
| **Subnet Mask** | Xác định phần Network vs Host trong IP |
| **IPv4 Header** | 20 bytes minimum, chứa Src/Dst IP, TTL, Protocol, Flags |
| **TTL** | Số hop tối đa, giảm 1 mỗi router, = 0 thì drop |
| **Fragmentation** | Chia packet khi > MTU, đích ráp lại, né được bằng PMTUD |
| **ICMP** | Protocol báo lỗi/chẩn đoán, ping/traceroute dùng nó |
| **Router** | Thiết bị L3, đọc Dst IP + routing table → forward |
| **Default Gateway** | Router gửi đến khi đích nằm ngoài subnet |
| **Private/Public IP** | Private dùng nội bộ (10.x, 172.16-31.x, 192.168.x), Public dùng trên Internet |
