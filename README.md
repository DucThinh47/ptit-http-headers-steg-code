# Lab: Giấu tin trong mạng sử dụng giao thức HTTP
## Lý thuyết
**1. Kỹ thuật "Mã hóa Cú pháp" (Syntactic Encoding) qua Viết hoa/thường**

Phần đầu tiên (giấu tin trong trường "Host") minh họa một nguyên tắc rất tinh vi: sự khác biệt giữa cú pháp (syntax) và ngữ nghĩa (semantics).
- `Ngữ nghĩa (Ý nghĩa)`: Đối với máy chủ web, Host: example.com và HOST: example.com là hoàn toàn giống nhau. Tên trường HTTP không phân biệt chữ hoa/thường (case-insensitive) theo tiêu chuẩn RFC.
- `Cú pháp (Cách viết)`: Tuy nhiên, ở cấp độ bit, chuỗi "host" (chữ thường) và "HOST" (chữ hoa) là hoàn toàn khác nhau.
- `Lý thuyết Kênh ẩn`: Kỹ thuật này khai thác "lỗ hổng" đó. Nó tạo ra một kênh ẩn mà các máy chủ web (cấp ngữ nghĩa) bỏ qua, nhưng một bộ phân tích gói tin (cấp cú pháp) có thể đọc được.
    - `host: example.com (cú pháp 1)` -> Mã hóa cho bit 0.
    - `HOST: example.com (cú pháp 2)` -> Mã hóa cho bit 1.

**2. Kỹ thuật "Mã hóa Thứ tự" (Order-based Encoding)**

Phần thứ hai (giấu tin trong thứ tự headers) khai thác một "vùng xám" (ambiguity) khác của tiêu chuẩn HTTP.
- `Tiêu chuẩn HTTP (RFC)`: Không quy định một cách nghiêm ngặt về thứ tự của hầu hết các header. Một máy chủ hợp lệ sẽ chấp nhận cả hai trường hợp: `Host` đứng trước `User-Agent`, hoặc `User-Agent` đứng trước `Host`.
- `Lý thuyết Kênh ẩn`: Kẻ giấu tin lợi dụng sự linh hoạt này để mã hóa thông tin. Bên gửi và bên nhận thống nhất một quy tắc về thứ tự:
    - Host trước User-Agent -> Mã hóa cho bit 0.
    - User-Agent trước Host -> Mã hóa cho bit 1.
- Đặc điểm: Kỹ thuật này cực kỳ khó phát hiện vì cả hai biến thể đều hoàn toàn hợp lệ và phổ biến.

**3. Lý thuyết "Kênh ẩn Đa trường" (Multi-Header Channel)**

Phần thứ ba (kết hợp nhiều header) tương tự như bài lab kết hợp TTL + IP ID + WS, nhưng áp dụng ở tầng HTTP.
- `Nguyên tắc`: Thay vì chỉ dùng 1 bit cho mỗi gói tin (ví dụ: chỉ dùng thứ tự), kẻ giấu tin kết hợp nhiều trường header (ví dụ: `Host` và `Accept-Encoding`) để mã hóa nhiều bit trong cùng một gói tin.
- `Mục đích`: Tăng băng thông (bandwidth) của kênh ẩn, cho phép gửi thông điệp nhanh hơn và hiệu quả hơn.

**4. Lý thuyết "Kênh ẩn Kết hợp" (Combined Technique Channel)**

Phần thứ tư mở rộng hơn nữa: nó kết hợp các thành phần khác nhau của chính yêu cầu HTTP.
- `Nguyên tắc`: Kênh ẩn không chỉ giới hạn trong các trường header. Nó có thể được tạo ra bằng cách kết hợp:
    - `Phương thức HTTP (HTTP Method)`: GET (bit 0) vs. POST (bit 1).
    - `Trường Header`: User-Agent (ví dụ: dùng User-Agent A cho bit 0, User-Agent B cho bit 1).
- Ví dụ:
    - `GET + User-Agent A` -> Mã hóa cho chuỗi bit 00.
    - `POST + User-Agent B` -> Mã hóa cho chuỗi bit 11.
- `Hệ quả`: Điều này làm cho việc phát hiện trở nên phức tạp hơn rất nhiều, vì kẻ phòng thủ phải theo dõi mối tương quan giữa nhiều thành phần khác nhau của giao thức.

**5. Nguyên tắc "Thêm nhiễu" (Chaffing / Decoy Traffic)**

Phần cuối cùng (sử dụng gói tin giả) là một kỹ thuật lẩn tránh kinh điển mà chúng ta đã thấy ở bài lab trước, nhưng được áp dụng tinh vi hơn ở tầng HTTP.
- `Vấn đề`: Nếu tất cả các gói tin đều đi đến example.com và tuân theo một quy tắc kỳ lạ (ví dụ: chỉ dùng GET/POST), luồng truy cập này rất đáng ngờ.
- `Giải pháp`: Script `noise_traffic_pcap.py` cố ý tạo ra các gói tin mồi (decoy):
    - `Gói tin nhiễu`: Truy cập đến một máy chủ khác (noise.com), sử dụng các phương thức ngẫu nhiên (GET/PUT).
    - `Gói tin thật`: Truy cập đến máy chủ mục tiêu (example.com), tuân theo quy tắc mã hóa (GET=0, POST=1).
- `Mục đích`: "Làm loãng" (dilute) luồng tin ẩn trong một biển lưu lượng trông có vẻ bình thường, khiến cho bên phân tích khó lọc ra đâu là tín hiệu thật.
## Thực hành
Trên máy client, mở wireshark:

    wireshark &

Thực thi file `host_case_pcap.py`:

    sudo python3 host_case_pcap.py

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image0.png?raw=true)

Mở file `host_case_pcap` trên Wireshark:

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image1.png?raw=true)

Phân tích file pcap này, kỹ thuật giấu tin trong trường "Host" (viết hoa/viết thường) sẽ hoạt động như sau:

    Bit 0 = host: example.com, Bit 1 = HOST: example.com.

Sau khi tìm được giá trị bits của thông điệp, trên máy client, thực thi file decode_bits.py:

    python3 decode_bits.py

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image2.png?raw=true)

Ghi giá trị chuỗi bits và thông điệp giải mã tìm được vào file answer.txt:

    nano answer.txt

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image3.png?raw=true)

Trên máy client, thực thi file `header_order_pcap.py`:

    sudo python3 header_order_pcap.py

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image4.png?raw=true)

Trên Wireshark, mở file header_order.pcap:

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image5.png?raw=true)

Kỹ thuật giấu tin trong thứ tự headers hoạt động theo quy tắc sau:

    Bit 0 = Host xuất hiện trước User-Agent, Bit 1 = User-Agent trước Host.

Dựa vào đây, tìm ra giá trị bits của thông điệp ẩn, sử dụng file decode_bits.py để kiểm tra. Ghi kết quả vào file answer.txt.

Trên máy client, thực thi file multi_header_pcap.py:

    sudo python3 multi_header_pcap.py

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image6.png?raw=true)

Trên Wireshark, mở file multi_header_pcap:

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image7.png?raw=true)

Tiếp tục phân tích các gói tin, tìm ra giá trị bits của thông điệp được giấu. Kỹ thuật giấu tin này sẽ kết hợp nhiều header fields lại, (Ví dụ: Host header và Accept-Encoding header) để giấu tin. Sau khi tìm được giá trị bits của thông điệp, thực thi file decode_bits.py để tìm thông điệp gốc.

Ghi kết quả vào file answer.txt

Trên máy client, thực thi file combined_tech_pcap.py:

    sudo python3 combined_tech_pcap.py

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image8.png?raw=true)

Trên Wireshark, mở file combined_tech_pcap.py:

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image9.png?raw=true)

Kỹ thuật này sẽ kết hợp cả phương thức của HTTP request và HTTP headers để giấu tin (VD: GET protocol và User-Agent hay POST protocol và User-Agent...). Tiếp tục phân tích các gói tin, tìm ra giá trị bits của thông điệp được giấu.

Sau khi tìm được giá trị bits của thông điệp, thực thi file decode_bits.py để tìm thông điệp gốc. Ghi kết quả vào file answer.txt

Thực hiện giấu tin sử dụng kỹ thuật thêm nhiễu (gói tin giả). Với kỹ thuật này, ngoài kết hợp các header, phương thức HTTP để giấu tin, các gói tin gây nhiễu cũng được gửi đi nhằm phát hiện khó khăn hơn.

Ví dụ: gói gây nhiễu sẽ có Host header có giá trị là “noise.com”, method ngẫu nhiên GET/PUT. Trong khi đó gói có thông điệp ẩn giấu sẽ có Host header là example.com, method GET (bit 0) hoặc POST (bit 1), có thể kèm payload.

Trên máy client, thực thi file noise_traffic_pcap.py:

    sudo python3 noise_traffic_pcap.py

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image10.png?raw=true)

Trên Wireshark, mở file noise_traffic.pcap:

![img](https://github.com/DucThinh47/ptit-http-headers-steg-code/blob/main/images/image11.png?raw=true)

Tiếp tục phân tích các gói tin, tìm ra giá trị bits của thông điệp được giấu. Sau khi tìm được giá trị bits của thông điệp, thực thi file decode_bits.py để tìm thông điệp gốc. Ghi kết quả vào file answer.txt

Sau khi đã tìm được thông điệp từ 5 gói tin, ghép lại thành thông điệp có ý nghĩa ban đầu, ghi nội dung thông điệp vào file answer.txt.

