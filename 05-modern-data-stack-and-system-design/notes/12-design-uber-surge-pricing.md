# Bài 12: Case Study - Thiết kế Hệ thống Giá cước Tăng vọt (Uber Surge Pricing)

**Đề bài:** Thiết kế kiến trúc hậu đài (Backend) cho tính năng "Tăng giá mùa Mưa bão" (Surge Pricing) của Uber.
**Bản chất Dữ liệu:** Hàng triệu tài xế di chuyển không ngừng, điện thoại gửi Tọa độ (GPS) của họ lên Máy chủ 4 giây/lần (Khối lượng Write khổng lồ). Cùng lúc đó, khi khách hàng mở App đặt xe, thuật toán phải tra xem *Khu vực A bán kính 2km* quanh khách hàng có bao nhiêu xe đang rảnh, bao nhiêu khách đang chờ, để nhân hệ số giá (1.5x) và tính tiền ngay lập tức (Khối lượng Read phân tích chéo Cực đoan).

Đây là bài toán **Heavy Write & Heavy Read** phức tạp nhất của kỷ nguyên Internet Dữ liệu Thời gian thực.

---

## 1. Nút thắt Phân vùng Không gian (Spatial Partitioning)

Nếu bạn dùng tọa độ `(Kinh độ, Vĩ độ)` lưu vào Cassandra và gõ Query: 
`SELECT count(*) FROM drivers WHERE lat BETWEEN 10.0 AND 10.5 AND long BETWEEN 106.0 AND 106.5`. 
Cassandra sẽ phải càn quét (Full Scan) hàng triệu điểm GPS trên đĩa cứng để lọc, mất vài chục giây. Khách hàng đợi tính giá xe và gỡ App chuyển qua xài Grab ngay lập tức.

**Giải pháp Định hình: Thuật toán Geohash**
Chúng ta không lưu Kinh độ Vĩ độ. Chúng ta dùng thư viện băm chia nhỏ bề mặt Trái Đất thành những ô vuông bàn cờ (Grids). 
Ví dụ khu vực "Trung tâm Quận 1" sẽ bị thuật toán nén thành một chuỗi Hash duy nhất là `w21z`. 
Khi Điện thoại tài xế bắn GPS `(10.7, 106.7)`, điện thoại App tự dịch nó ra chữ `w21z`. 
Từ một dữ liệu số thực vô cực, chúng ta bẻ cong bài toán về bài toán So Sánh Chữ Cực Đơn Giản: Lấy tất cả tài xế có mã ô là chữ `w21z` (Phép $O(1)$ Hash Table).

---

## 2. Kiến trúc Data Pipeline Thời gian thực (Real-time Analytics)

Vì giá xe Surge Pricing được thay đổi 1 phút/lần. Hệ thống phải nạp dữ liệu vào các Bảng tạm Cache liên tục.

```mermaid
graph TD
    subgraph Kiến trúc Uber Surge Pricing (Real-time Stream)
        GPS(App Tài xế: Gửi Geohash w21z <br> 4 Giây / Lần) --> API[Web Socket Layer]
        KH(App Khách Hàng: Yêu cầu Đặt Xe <br> Mã Geohash w21z) --> API
        
        API --> K[Kafka: Streaming Bus]
        
        K --> F[Flink: Tính Tổng Cung Cầu trong 1 Phút]
        F -.->|Xe rảnh = 10 <br> Khách xin = 100| CALC[Thuật toán Tính Giá AI]
        CALC -.->|Lệnh: Giá = 3.5X| R[(Redis Cache Cluster)]
        
        KH -->|App hỏi: Giá quận 1 nhiêu?| R
        R -->|Trả lời trong 5 mili-giây: 3.5X| KH
    end
```

**Sự vận hành của Hệ thống Lưới:**
1. Hàng triệu tài xế liên tục bắn cục Tọa độ (Ping) vào Kafka. Kafka gánh trọn vẹn sát thương Ghi dữ liệu (Write-Heavy) với công nghệ Zero-copy để bảo vệ các cụm cơ sở dữ liệu yếu đuối ở phía sau.
2. Máy chủ Flink đóng vai trò như những Vòi nước Phân vùng (Partitioning). Flink áp dụng **Sliding Window (Cửa sổ 1 phút)**. 
3. Flink có hàng trăm cụm. Flink thông minh ném tất cả các sự kiện có gắn tem chữ `w21z` về TẬP TRUNG cho 1 máy Worker Flink DUY NHẤT để tính toán cục bộ. (Worker này ôm trọn mọi dữ liệu cung-cầu của Quận 1, nó tính toán tỷ lệ chênh lệch trên RAM siêu tốc).
4. Tính xong, Flink nhả đáp án hệ số giá vào Cụm Redis In-Memory.

---

## 3. Kiến trúc Đọc (Read) - Bẻ gãy Cơn bão Dữ liệu

Khi trời mưa bão lúc 5h chiều, 10.000 khách hàng ở Quận 1 móc điện thoại mở App Uber cùng lúc. 10.000 cái Điện thoại sẽ đồng thanh đập vào máy chủ hỏi xin giá. 

Nếu 10.000 câu truy vấn (Query) đó bay thẳng vào Cơ sở dữ liệu chính, hệ thống sẽ gây ra **Cache Stampede (Thảm họa Dẫm đạp Bộ đệm - Bài 7 Part 3)**.

**Phòng ngự Biên (Edge Caching):**
Giá của Khu vực Quận 1 (1 phút mới được Flink thay đổi 1 lần). Vậy tại sao phải lóc cóc chạy xuống tận cùng Redis/Database để xin giá liên tục?
- Load Balancer / Reverse Proxy Nginx ở tầng mạng ngoài cùng (Bài 9 Part 4) sẽ dùng bộ đệm Local RAM của riêng nó để lưu cái giá "3.5x".
- Bất cứ khách hàng nào chọc vào Nginx xin giá của vùng `w21z` sẽ được Nginx trả về ngay lập tức đoạn JSON `{price_surge: 3.5}` mà không tốn công gọi xuống Data Pipeline bên trong.

Sự trọn vẹn của thiết kế hệ thống là nắm thóp được vòng đời dữ liệu. Tại gốc (Kafka), ta chống đỡ Write; Tại thân (Flink), ta gộp khối lượng để xử lý tính toán trên RAM; và tại ngọn (Nginx/Redis), ta giăng bẫy Cache để phục vụ truy vấn Read tốc độ ánh sáng. Những mảng ghép này chính là sự thăng hoa rực rỡ nhất của hành trình Kỹ thuật Phần mềm (Software Engineering).

---
*(Kết thúc Chặng 5 - Khép lại Hành trình Trở thành Data Architect Đỉnh cao)*

---
**Navigation:**
[⬅️ Previous: Bài 11: Case Study - Thiết kế Bộ đếm Lượt Xem Youtube (Heavy-Write)](./11-design-youtube-view-counter.md)
