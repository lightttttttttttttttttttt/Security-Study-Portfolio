# 🐧 Linux CLI For Hackers & SOC Analysts
> **Tâm pháp:** "Trong Linux, mọi thứ đều là File (từ ổ cứng, bàn phím, cho đến process). Không có giao diện (GUI) không phải là điểm yếu, mà là để tốc độ ra đòn nhanh nhất."

Nếu ông xác định thi OSCP, **bắt buộc** ông phải sống chung với cái màn hình đen xì này. Dưới đây là những lệnh sống còn, không cần học thuộc lòng nhưng phải hiểu bản chất để gõ thành phản xạ.

---

## 1. Tứ Đại Bổn Thuật (Điều hướng cơ bản)
Muốn tấn công/giám sát thì phải biết mình đang đứng ở đâu và xung quanh có gì.

*   `pwd` (Print Working Directory): Trả lời câu hỏi *"Mình đang ở thư mục nào?"* (Vì không có thanh địa chỉ như Windows).
*   `ls -la`: Liệt kê tất cả file. **Bắt buộc luôn phải thêm cờ `-la`**.
    *   `-l`: Hiển thị chi tiết (quyền, chủ sở hữu, dung lượng, thời gian).
    *   `-a`: Hiển thị cả **file ẩn** (trong Linux, file bắt đầu bằng dấu chấm `.` là file ẩn, bọn mã độc hay dùng trò này để trốn).
*   `cd` (Change Directory): Đi lại giữa các thư mục.
    *   `cd /`: Về thư mục gốc (Root - to nhất).
    *   `cd ~`: Về nhà (Thư mục Home của user hiện tại).
    *   `cd ..`: Lùi ra ngoài 1 thư mục.
*   `cat <tên file>`: Đọc và in toàn bộ nội dung file ra màn hình (Hay dùng để đọc file cấu hình, file pass, cờ root.txt).

---

## 2. Tìm diệt & Lùng sục (Find & Grep)
Thi OSCP vào được máy nạn nhân rồi thì phải đi bới rác tìm password hoặc file nhạy cảm. Còn làm SOC thì phải mò log.

*   **`grep`**: Bóp nghẹt hàng triệu dòng log để lấy đúng thứ mình cần.
    *   *Ví dụ SOC:* Tìm xem thằng nào đăng nhập thất bại.
    *   `cat /var/log/auth.log | grep "Failed password"`
    *   *(Cái dấu `|` gọi là "Pipe" - Ống nước. Nó lấy kết quả của lệnh bên trái, đút vào mồm lệnh bên phải).*

*   **`find`**: Tìm file theo vô vàn điều kiện (tên, kích thước, phân quyền).
    *   *Ví dụ:* `find / -name "pass*.txt"` (Tìm từ thư mục gốc `/` tất cả file có tên bắt đầu chữ pass).

---

## 3. Hệ thống Phân quyền (Permissions) - Chìa khóa OSCP
Linux quản lý file cực kỳ chặt. Quyền được chia làm 3 nhóm: **User** (Chủ sở hữu) - **Group** (Nhóm) - **Others** (Bọn còn lại).
Và 3 loại quyền: **R** (Read - Đọc) / **W** (Write - Ghi) / **X** (Execute - Chạy).

*Khi ông gõ `ls -la`, ông sẽ thấy một đoạn mã đầu tiên:*
`-rwxr-xr--`
*   Ký tự 1: `-` là file bình thường, `d` là thư mục (directory).
*   3 ký tự tiếp (`rwx`): Quyền của chủ file (Được đọc, ghi, chạy).
*   3 ký tự tiếp (`r-x`): Quyền của nhóm (Đọc, Chạy - Cấm ghi).
*   3 ký tự cuối (`r--`): Quyền của người ngoài (Chỉ được đọc).

### Lệnh thay đổi quyền:
*   `chmod +x script.sh`: Bơm quyền "Execute" cho file để nó chạy được. (Tải mã độc về mà không `chmod +x` thì nó nằm im).
*   `chown root:root file.txt`: Đổi chủ sở hữu file sang cho ông nội `root`.

---

## 4. Tuyệt kỹ SUID (Đặc quyền tối thượng)
Trả lời câu hỏi ban nãy: **SUID là gì và tại sao lệnh `find / -perm -4000` lại bá đạo?**

*   **Bản chất:** Khi một file được gắn cờ SUID (hiện chữ `s` thay vì chữ `x` ở quyền User, ví dụ: `-rwsr-xr-x`), bất cứ ai chạy file này cũng sẽ **TẠM THỜI MƯỢN ĐƯỢC QUYỀN CỦA CHỦ SỞ HỮU FILE ĐÓ**.
*   **Điểm chết người (Privilege Escalation):** Nếu chủ file là `root`, và cái file đó lại là một công cụ có thể thực thi lệnh (ví dụ như `nmap`, `bash`, `vim`), ông (một user cùi bắp) có thể chạy công cụ đó -> nó sẽ mượn quyền `root` -> ông dùng nó sinh ra một shell mới -> BOOM! Ông có quyền Root!
*   **Lệnh `find / -perm -4000 2>/dev/null`:**
    *   `find /`: Tìm từ gốc rễ.
    *   `-perm -4000`: Lọc những file có gắn cờ SUID (mã là 4000).
    *   `2>/dev/null`: Đem tất cả những dòng báo lỗi "Permission denied" (Mày cùi bắp không được vào đây tìm) ném vào "Hố đen" (`/dev/null`) để màn hình chỉ hiện ra kết quả xịn, không bị rác.

---

## 5. Mổ xẻ Log với Awk + Sort + Uniq (Vũ khí của SOC)
Trả lời câu hỏi: Làm sao lấy IP lỗi 404 nhiều nhất từ file log 5GB?

Trong file log web (`access.log`), nội dung thường có dạng:
`192.168.1.10 - - [20/Jul/2026] "GET /admin HTTP/1.1" 404 1205`
*(Cột 1 là IP, cột 9 là mã lỗi HTTP 404)*

**Lệnh ma thuật:**
`cat access.log | awk '$9 == 404 {print $1}' | sort | uniq -c | sort -nr`

**Giải mã cơ chế:**
1.  `cat access.log`: Đọc file log.
2.  `awk '$9 == 404 {print $1}'`: `awk` là thánh thái thịt. Nó chia dòng văn bản thành các cột (mặc định cách nhau bằng dấu cách). Câu này bảo: "Nếu cột 9 là số 404 thì in ra màn hình cái cột số 1 (chính là IP)".
3.  `sort`: Sắp xếp các IP đó theo thứ tự Alphabet (Bắt buộc phải sort trước khi đếm).
4.  `uniq -c`: Gom các dòng IP giống nhau đứng cạnh nhau thành 1, và `c` (count) đếm số lượng. (Ví dụ: `50 192.168.1.10`).
5.  `sort -nr`: Sort lại một lần nữa. `-n` là theo Số lượng, `-r` là Reverse (Từ lớn đến bé).

=> Kết quả in ra trên cùng sẽ là thằng IP bị 404 nhiều nhất. Đây chính là cách viết Tool gom Log SOC không cần giao diện!
