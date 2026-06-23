# Bài 4: Bất khả xâm phạm: Exactly-Once Semantics, 2PC và Tính Lũy đẳng

Trong bài 2 và 3, hệ thống Streaming của chúng ta đã tính đúng thời gian và không sợ sập RAM. Nhưng hệ thống tài chính Core Banking luôn trăn trở 1 cơn ác mộng cực độ: **Sự trùng lặp giao dịch (Duplication)**. 

Khi một giao dịch chuyển 5 triệu VNĐ trôi qua Flink và ghi vào Database. Nếu mạng chập chờn, Flink tưởng gói tin bị rớt, nó quyết định **gửi lại lần 2**. Hệ thống vô tình trừ 10 triệu của khách hàng. 

Bài toán chống đúp giao dịch đẻ ra 3 định mức cam kết mạng lưới (Delivery Semantics):
1. **At-most-once (Cao nhất 1 lần):** Gửi đại qua mạng, mất thì chịu. Nhanh nhưng thiếu tin cậy (Cơ chế UDP - Bài 6 Part 4).
2. **At-least-once (Ít nhất 1 lần):** Flink phải chờ chốt (ACK). Nếu không thấy ACK, Flink cứ kiên nhẫn Gửi đi gửi lại hoài đến khi nào lọt thì thôi. Chống mất dữ liệu tuyệt đối, nhưng **Chắc chắn sinh ra Trùng Lặp**.
3. **Exactly-once (Chính xác Duy nhất 1 lần):** Chén thánh của Data Engineering. Dù mạng rớt, server sập, ổ cứng hư, gói tin cũng chỉ được tính Tiền đúng duy nhất 1 lần.

---

## 1. Sự kết nối Vĩ đại: Flink Kafka Two-Phase Commit (2PC)

Đạt được Exactly-once bên trong nội bộ máy chủ Flink là nhờ thuật toán Checkpoint Barrier (Bài 2). Nhưng khi Flink vươn tay ra lưu dữ liệu vào mạng lưới Kafka đích (Sink), thảm họa xảy ra nếu Flink chết ngay khoảnh khắc đã ghi Kafka nhưng chưa kịp lưu Checkpoint RAM.

Để kết nối vĩnh cửu trạng thái giữa 2 hệ thống độc lập này, Flink và Kafka thiết lập thuật toán **Giao dịch Hai pha (Two-Phase Commit - 2PC)**.

**Quá trình 2PC thần thánh:**
1. **Pha 1 (Pre-commit):** Flink tính toán xong, tống Data sang cho Kafka. Tuy nhiên, nó ra lệnh cho Kafka: *"Ghi Data vào đĩa đi, nhưng khóa nó lại, giấu đi (Uncommitted), không cho tụi Consumer bên ngoài đọc được nha"*.
2. **Sự cố rủi ro:** Nếu Flink sập nguồn ngay lúc này. Vì Data bên Kafka đang bị giấu, không ai biết, Flink khởi động lại sẽ vứt bỏ luồng Data nháp đó và làm lại từ đầu. Không có thảm họa nào xảy ra.
3. **Pha 2 (Commit):** Nếu Flink chạy suôn sẻ, chốt sổ RAM (Checkpoint) thành công. Nó sẽ phát đi một tiếng rống (Commit Request) sang Kafka: *"Xong rồi, anh xác nhận mở khóa Data nháp đó đi, cho thiên hạ lấy về đọc!"*. Lúc này, dữ liệu chính thức đi vào lịch sử Kafka. Mọi thứ an toàn 100% nguyên khối (Atomic Transaction).

```mermaid
graph TD
    subgraph Thuật toán 2PC Exactly-Once (Flink to Kafka)
        F(Flink Worker) -.->|1. Viết dữ liệu giấu kín (Pre-commit)| K(Kafka Broker)
        F -->|2. Flink Checkpoint RAM thành công| S3[(S3 State)]
        F -->|3. Phát lệnh chốt sổ toàn mạng| K
        K -.->|Mở khóa, Consumer được phép đọc| C(Khách hàng)
    end
    
    style K fill:#28a745,stroke:#000,color:#fff
```
Sự đánh đổi: 2PC mang lại chén thánh Exactly-once cho mọi dòng dữ liệu tài chính, nhưng vì phải chạy đi chạy lại mở khóa, tốc độ độ trễ (Latency) của toàn mạng lưới bị giảm đi vài chục mili-giây so với At-least-once.

---

## 2. Phòng thủ Biên: Tính Lũy đẳng (Idempotence)

Thuật toán 2PC bao bọc an toàn Flink và Kafka. Nhưng lỡ như thằng App Mobile của khách hàng bấm nút "Thanh toán" 2 lần liên tục vì bực mình do mạng 3G chậm thì sao? App Mobile không chạy 2PC. Nó sẽ bắn 2 cục Data vào Kafka. Kafka và Flink sẽ nhận 2 cục và tính đúng Exactly-once cho... cả 2 cục.

Để chống rác từ ngọn, Lập trình viên Backend và Data Engineer phải áp dụng tuyệt kỹ thiết kế hệ thống tối thượng: **Tính Lũy đẳng (Idempotency)**.

**Lũy đẳng là gì?** Là một phép toán / hàm xử lý mà cho dù bạn có chạy 1 lần hay 1 triệu lần với cùng một dữ liệu đầu vào, kết quả hệ thống trả ra vẫn KHÔNG HỀ THAY ĐỔI.

- **Không Lũy đẳng:** Câu lệnh SQL `UPDATE bank SET tien = tien - 500`. Chạy câu lệnh này 3 lần do lỗi mạng gửi đúp, khách hàng mất 1500. Thảm họa.
- **Lũy đẳng Tuyệt đối:** App Mobile khi bấm Thanh Toán, nó tự sinh ra một mã định danh duy nhất (UUID = `XYZ_123`) in chết vào gói tin. Server cấu hình Database: `UPDATE bank SET tien = 5000 WHERE id = XYZ_123`. Nếu Flink hay App có gửi đúp gói tin `XYZ_123` 1 triệu lần, cái Database (như Cassandra/Redis - nhờ tính chất Idempotent Key) chỉ ghi đúng 1 lần đầu, 999.999 lần sau nó tóm được khóa chính UUID trùng lặp, nó tự động hủy lệnh bỏ vào thùng rác.

**Bài học System Design Thực tế:** 
Thay vì cố xây dựng một đường mạng dây nhôm hoàn hảo không bao giờ rớt gói tin (Điều không thể theo định lý CAP), Kỹ sư giỏi xây dựng hệ thống chấp nhận mạng lưới rách nát (At-least-once) nhưng bọc chúng bằng **Thiết kế Lũy đẳng (Idempotency Key)** ở khâu nạp dữ liệu cuối cùng (Upsert trên Database lõi). Nó mang lại tốc độ cực đại của At-least-once và tính toàn vẹn 100% của Exactly-once.

---
**Navigation:**
[⬅️ Previous: Bài 3: Xử lý Thời gian thực: Event Time, Watermarks và Windowing](./03-event-time-watermarks-and-windowing.md) | [Next: Bài 5: Data Lakehouse: Mang tính Nhất quán ACID lên Lưu trữ Đám mây ➡️](./05-lakehouse-and-acid-on-cloud.md)
