# Bài 5: Nhân bản Dữ liệu (Replication) và Thuật toán Đồng thuận Quorum

Khi xây dựng một kiến trúc Hệ thống Phân tán, để đạt được độ Sẵn sàng (Availability) và Chịu lỗi (Fault Tolerance), chiến lược cơ sở là sao chép dữ liệu từ máy chủ này sang nhiều máy chủ khác. Cơ chế này được gọi là **Replication**.

Replication giải quyết hai bài toán kinh điển: Phân tán tải đọc (Scale-out Read) và Phòng ngừa mất mát khi ổ đĩa cháy nổ. Để quản trị quá trình luân chuyển dữ liệu ghi (Write), có các cấu trúc liên kết phổ biến sau.

---

## 1. Single-Leader Replication (Master-Slave)

Đây là mô hình tiêu chuẩn và được áp dụng rộng rãi nhất trên các nền tảng Relational Database (PostgreSQL, MySQL).

**Luồng thiết kế:**
1. Trong một cụm mạng lưới, hệ thống bỏ phiếu bầu định danh ra duy nhất một Node làm **Leader (Master)**. Các Node còn lại đóng vai trò là **Follower (Slave / Replica)**.
2. Mọi tác vụ Ghi (Write / Update / Delete) từ ứng dụng bắt buộc phải đi thẳng vào Leader.
3. Sau khi Leader ghi cục bộ, nó đóng gói Transaction đó thành một luồng dữ liệu (Data Stream / WAL Log) và phát tán tới toàn bộ các Follower.
4. Mọi tác vụ Đọc (Read / Select) có thể được định tuyến tới bất kỳ Node nào.

**Ưu điểm & Nhược điểm:**
- Điểm mạnh lớn nhất là Tính Đơn Giản: Không bao giờ có xung đột (Conflict) dữ liệu ghi, vì mọi thay đổi do 1 người quyết định.
- Yếu huyệt Rủi ro: Leader trở thành Điểm nghẽn Cổ chai (Bottleneck) về hiệu suất Ghi. Khủng khiếp hơn, nếu phần cứng của Leader bị sập (Single point of failure), toàn bộ hệ thống ngay lập tức bị liệt chức năng Ghi cho đến khi hệ thống mạng chạy lại thuật toán tự động Bầu cử (Failover) để cử 1 Follower lên thay thế. Quá trình này có thể gây Down-time vài chục giây.

---

## 2. Multi-Leader và Leaderless Replication

Để xử lý việc giới hạn thông lượng Ghi của 1 Leader, các hãng công nghệ thiết kế mô hình **Multi-Leader** (Ví dụ: Các Datacenter đặt ở 3 châu lục, mỗi Datacenter có 1 Leader riêng xử lý User khu vực đó, sau đó 3 Leader đồng bộ chéo cho nhau). 

Triệt để hơn nữa là kiến trúc **Leaderless (Không có Lãnh đạo)**, tiêu biểu là nền tảng Amazon Dynamo và Apache Cassandra.
Ở mô hình này, mọi Node đều ngang hàng (Peer-to-Peer). Client có thể phóng lệnh `UPDATE` tới bất kì Node nào.

**Thảm họa Xung đột (Conflict Resolution):**
Việc cho phép nhiều luồng Write cùng lúc sinh ra sự cố. Client A báo với Node 1: "X = 5". Cùng mili-giây đó, Client B báo với Node 2: "X = 10". 
Do độ trễ mạng, khi Node 1 và Node 2 gửi bản đồng bộ cho nhau, hệ thống không biết phiên bản nào là đúng. Cấu trúc yêu cầu các thuật toán dung giải cực kì phức tạp như Vectơ Đồng hồ (Vector Clocks) hay LWW (Last Write Wins) dựa trên nhãn thời gian NTP - thường dẫn tới rủi ro mất mát dữ liệu do đồng hồ máy chủ bị lệch mili-giây.

---

## 3. Quorum Consensus (Đồng thuận Số đông)

Dù ở kiến trúc Leader hay Leaderless, khi ghi dữ liệu ra nhiều máy, bài toán PACELC (Bài 4) tái xuất hiện: DB có nên chờ tất cả các máy ghi xong (Sync) hay không chờ ai cả (Async)?
- **Đồng bộ toàn phần (Full Sync):** Ghi 1 bản, chờ 5 máy xác nhận. Hệ thống an toàn tuyệt đối nhưng quá chậm. Nếu 1 máy hỏng card mạng, toàn bộ giao dịch tê liệt.
- **Bất đồng bộ (Async):** Chỉ ghi máy 1, không cần chờ ai. Rất nhanh, nhưng nếu máy 1 cháy ổ cứng, dữ liệu đó mất vĩnh viễn chưa kịp truyền đi.

Lời giải toán học trung dung hoàn mỹ cho vấn đề này là **Quorum (Số đông Quá bán)**.

Hệ thống đặt ra 3 tham số mạng:
- $N$: Tổng số lượng máy chủ chứa bản sao (Replicas).
- $W$: Số lượng máy chủ bắt buộc phải ghi thành công trước khi hệ thống báo `OK` cho giao dịch Write.
- $R$: Số lượng máy chủ bắt buộc phải liên hệ cùng lúc để lấy dữ liệu khi thực hiện giao dịch Read.

**Định lý Quorum:** Nếu hệ thống thiết lập cơ cấu sao cho **$W + R > N$**, thì chắc chắn 100% trong số $R$ máy trả lời cho phiên Read, sẽ có ít nhất 1 máy chứa dữ liệu cập nhật mới nhất từ phiên Write.

```mermaid
graph TD
    subgraph Thuật toán Quorum W=2, R=2 trên 3 Replicas (N=3)
        C1[Client GHI: X=10] -->|Thành công| N1((Node 1 <br/> Lưu X=10))
        C1 -->|Thành công| N2((Node 2 <br/> Lưu X=10))
        C1 -.->|Thất bại/Trễ| N3((Node 3 <br/> Kẹt ở X=5))
        
        C2[Client ĐỌC X] -->|R=2| N2
        C2 -->|R=2| N3
        
        N2 -.->|Trả về v.2| P[Phân tích Version <br/> X=10]
        N3 -.->|Trả về v.1| P
    end
```
*Phân tích sơ đồ:* Client Đọc lấy ngẫu nhiên 2 máy (Node 2 và 3). Kể cả dính phải Node 3 bị trễ mạng (X=5), nó vẫn nhận được kết quả X=10 từ Node 2. Giao thức Quorum tự động so sánh timestamp và loại bỏ kết quả cũ, trả về thông tin Nhất quán (Consistency) tuyệt đối cho người dùng.

Bằng cách hạ $W$ và $R$ xuống thấp hơn $N$, hệ thống phân tán đạt năng lực Cân bằng Tải cực tốt, chấp nhận vài node chết nhưng vẫn bảo đảm tính Toàn vẹn Logic toán học của kho dữ liệu.

---
**Navigation:**
[⬅️ Previous: Bài 4: Định lý CAP, PACELC và Thiết kế Đánh đổi Hệ thống Phân tán](./04-cap-theorem-and-pacelc.md) | [Next: Bài 6: Phân mảnh ngang (Sharding) và Băm nhất quán (Consistent Hashing) ➡️](./06-partitioning-and-sharding.md)
