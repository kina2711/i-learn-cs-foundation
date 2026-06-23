# Bài 7: Bộ đệm (Caching), Redis và Chiến lược Xử lý rủi ro Cache

Giới hạn lớn nhất của một hệ thống cơ sở dữ liệu truyền thống (Database) là sự giới hạn tốc độ truy xuất của Ổ cứng tĩnh (Disk I/O), ngay cả khi đã sử dụng Indexing (B-Tree). Trong một hệ thống lưu lượng khủng (High-traffic), việc ép Database phục vụ hàng triệu truy vấn Read giống hệt nhau mỗi giây (Ví dụ: Số lượng like của một bài post) sẽ đánh sập máy chủ ngay lập tức. 

Để bảo vệ Database, Data Engineer thiết lập một lớp áo giáp phòng thủ ở phía trước: **Bộ đệm (Cache System)**. 

---

## 1. Cơ chế Bộ đệm và CSDL Key-Value (Redis)

Cache là không gian lưu trữ dữ liệu hoàn toàn trên bộ nhớ trong **RAM (In-memory)**. Tốc độ đọc ghi của RAM vượt tốc độ ổ đĩa từ $10^4$ đến $10^5$ lần. Thay vì dùng cấu trúc Cây phức tạp để dò tìm dữ liệu, Cache System áp dụng thẳng cấu trúc thuật toán **Bảng băm (Hash Table)** để trích xuất dữ liệu với độ phức tạp thời gian cực hạn $O(1)$.

Đại diện tiêu biểu nhất của hệ sinh thái Cache là CSDL Key-Value: **Redis (Remote Dictionary Server)** và **Memcached**.

**Luồng dữ liệu Tiêu chuẩn (Cache-Aside Pattern):**
1. Ứng dụng client gửi luồng Read Request. Đầu tiên, nó tra cứu khóa (Key) vào hệ thống Cache (Redis).
2. Nếu dữ liệu có trên Cache (**Cache Hit**), trả kết quả ngay tức khắc, kết thúc luồng. (Thời gian ~ 1 mili-giây).
3. Nếu dữ liệu không tồn tại (**Cache Miss**), hệ thống buộc phải phá rào đi xuống truy vấn Database nặng nề ở dưới (Ví dụ MySQL).
4. Sau khi trích xuất kết quả từ MySQL, nó nạp ngược kết quả đó vào Redis để các user đến sau có thể lấy thẳng từ Cache.

### Chính sách Trục xuất Dữ liệu (Eviction Policies)
Vì tài nguyên RAM vật lý vô cùng đắt đỏ và hữu hạn (Ví dụ Server chỉ có 64GB RAM), Redis không thể lưu trữ vĩnh viễn mọi thứ. Khi không gian RAM tràn 100%, hệ thống tự động kích hoạt thuật toán trục xuất để dọn chỗ trống cho dữ liệu mới.
- **LRU (Least Recently Used):** Xóa các cặp Key-Value có mốc thời gian truy cập lâu nhất (Ít được dùng nhất gần đây).
- **LFU (Least Frequently Used):** Xóa các cặp Key-Value có tổng số lần được truy xuất ít nhất (Dựa vào tần suất).

---

## 2. Các rủi ro sụp đổ Cache (Cache Failure Patterns) và Cách phòng thủ

Lớp áo giáp Cache không hoàn hảo. Bất kì lỗ hổng nào trong thiết kế Cache cũng sẽ dội thẳng lưu lượng khổng lồ vào Database, gây chết dải chuyền (Cascading Failure). Kỹ sư Data Engineer phải đối mặt với 3 thảm họa kinh điển.

### A. Thảm họa Xuyên thấu Cache (Cache Penetration)
**Vấn đề:** Hacker cố tình spam liên tục hàng ngàn Request chứa các tham số không bao giờ tồn tại (Ví dụ: Tìm ID sản phẩm `-9999`). 
Vì là tham số ảo, Redis chắc chắn trả về `Cache Miss`. Ứng dụng đi xuống truy vấn MySQL, không tìm thấy, trả về Rỗng. Redis không lưu chuỗi Rỗng. Request tiếp theo lại tiếp tục đập thủng Redis và trút thẳng vào MySQL.
**Giải pháp kiến trúc:**
1. Lưu cả kết quả Null/Rỗng vào Redis với một vòng đời (TTL) cực ngắn (10 giây).
2. Sử dụng thuật toán Xác suất **Bloom Filter**: Một cấu trúc bitmap chặn trước cổng Redis. Nó có thể xác định cực nhanh $O(1)$ rằng: "Cái ID -9999 này 100% chưa từng tồn tại trong Database", từ đó ngắt kết nối ngay trước khi luồng chạy kịp chạm vào MySQL.

### B. Thảm họa Tuyết lở (Cache Avalanche)
**Vấn đề:** Data Engineer thiết lập cấu hình Vòng đời (TTL - Time to Live) cho 10.000 từ khóa giống hệt nhau là "Tồn tại trong 12 giờ". Vào thời khắc điểm 12:00:00, toàn bộ 10.000 từ khóa đồng loạt bốc hơi khỏi RAM cùng một micro-giây. Ngay mili-giây tiếp theo, hàng vạn khách hàng truy vấn vào, tạo ra hàng vạn `Cache Miss` đập sầm sập vào Database, đánh sập ngay lập tức toàn bộ hệ thống.
**Giải pháp kiến trúc:** 
Thêm một hệ số độ trễ Ngẫu nhiên (Random Jitter) vào giá trị TTL. Ví dụ: TTL = 12 giờ + Random(1 đến 10 phút). Sự phân rã thời gian tự nhiên này sẽ phá vỡ "đỉnh sóng" truy cập.

### C. Thảm họa Giẫm đạp (Cache Stampede / Cache Breakdown)
**Vấn đề:** Khác với Avalanche là sập nhiều khóa cùng lúc, Stampede là hiện tượng sập ở **Chỉ duy nhất 1 từ khóa (Hot Key)** (Ví dụ: Sự kiện Flash Sale iPhone lúc 0h). Khi đồng hồ điểm 0h, cache của giá Flash Sale vừa hết hạn. Ngay khoảnh khắc trống (Vài mili-giây) đó, 10.000 khách hàng F5 trang web. Vì cache trống, cả 10.000 tiến trình đồng loạt kết luận `Cache Miss` và kéo nhau cùng chạy thẳng vào MySQL để lấy đúng 1 giá trị.
**Giải pháp kiến trúc:**
1. **Khóa Mutex (Mutex Lock):** Phân tán hệ thống. Tiến trình nào đến trước trong micro-giây sẽ giành được khóa ảo, nó đi xuống DB lấy data. 9.999 tiến trình đến sau thấy cửa bị khóa, sẽ đứng chờ (Sleep) vài chục mili-giây, sau đó đọc thẳng từ Cache đã được nạp bởi tiến trình số 1.
2. **Khởi tạo ngầm (Background Warming):** Đừng để Hot Key tự hết hạn nhờ truy vấn của User. Lập trình một tiến trình Cron Job chạy ngầm định kỳ đi nạp lại dữ liệu cho Hot Key lên Redis 5 phút trước khi nó kịp hết hạn.

---
**Navigation:**
[⬅️ Previous: Bài 6: Phân mảnh ngang (Sharding) và Băm nhất quán (Consistent Hashing)](./06-partitioning-and-sharding.md) | [Next: Bài 8: Cơ sở dữ liệu Tài liệu (Document DB) và Cấu trúc BSON ➡️](./08-document-databases-and-mongodb.md)
