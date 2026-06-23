# Bài 12: Xác thực Bảo mật Mạng: JWT, OAuth2 và Kiến trúc Lõi Kerberos

Khi một người dùng, hoặc một Job Python (Airflow), muốn chọc vào cơ sở dữ liệu Hadoop để tải bảng Sales, tường lửa VPC hay ống mạng TLS (Bài 8, 11) chỉ cản kẻ bên ngoài, nhưng nó mù tịt trước câu hỏi: *"Kẻ đang đứng trong ống mạng này thực sự là ai? Có quyền SELECT bảng Sales hay không?"*.

Trong mạng nội bộ phân tán (hàng nghìn Microservices), việc kiểm soát Định danh (Authentication) và Quyền hạn (Authorization) đòi hỏi kiến trúc Cấp Chứng minh thư điện tử. Nổi bật nhất là kiến trúc Token **JWT (Web)** và Kiến trúc Pháo đài **Kerberos (Data Center)**.

---

## 1. Môi trường Web/Microservices: JWT và OAuth2

**Sự thất bại của Session (Session-based Auth):**
Xưa kia, khi User đăng nhập, Máy chủ Web A sinh ra một ID (Session ID) lưu vào RAM của nó. Lần sau User đưa ID đó lên, Máy A nhớ ra liền. Nhưng nếu mạng lưới Load Balancer (Bài 9) hất User sang Máy chủ Web B, Máy B sẽ ngơ ngác vì không lưu dữ liệu RAM đó, yêu cầu User đăng nhập lại.

**Giải pháp Stateless Không Lưu RAM: JSON Web Token (JWT)**
Kiến trúc Microservices vứt bỏ hoàn toàn RAM. 
- Khi User đăng nhập thành công, Máy chủ lấy một con dấu điện tử (Được mã hóa Hashing bí mật bằng Chìa khóa Server) đóng dấu Mộc đỏ lên tờ giấy chứng nhận: `"Tên: Alice, Quyền: Admin"`. Tờ giấy này được mã hóa Base64 thành một chuỗi siêu dài gọi là **JWT Token**.
- Server đưa JWT Token này nhét thẳng vào túi áo (Trình duyệt) của User và **quên sạch đi**.
- Ngày hôm sau, User mang Token này vào Máy chủ B. Máy B không cần tra cơ sở dữ liệu. Nó chỉ cần cầm Con dấu Mộc Hashing của công ty ra soi. Thấy chuẩn chữ ký điện tử mã hóa, nó cho qua ngay lập tức. Tính năng Không-Lưu-Trạng-Thái (Stateless) này giúp hệ thống Scale-out hàng vạn máy chủ cực nhẹ nhàng.

**OAuth 2.0 (Giao quyền không lộ mật khẩu):**
Nếu trang web Datacamp muốn đọc thư Gmail của bạn. Bạn không thể đưa User/Pass Google cho Datacamp được. 
Giao thức OAuth2 làm trung gian: Bạn sang Google xác nhận. Google sẽ cấp cho Datacamp một tờ vé mượn tạm thời (Access Token - thường là JWT) có thời hạn 1 giờ, chỉ được quyền Đọc Thư.

---

## 2. Lõi Bảo mật Big Data (Hadoop): Cấu trúc Kerberos

JWT rất tuyệt cho ứng dụng Web. Nhưng trong lõi mạng nội bộ của Data Center, nơi các cỗ máy (Server Kafka, Hadoop DataNode, Spark Worker) liên tục giao tiếp cấp thấp (gRPC/TCP) với nhau với vận tốc hàng triệu lượt/giây, cấu trúc JWT bị coi là không đủ mạnh và phân tán.
Hệ sinh thái Apache Hadoop và Kafka Enterprise chọn Kiến trúc pháo đài tập trung khét tiếng nhất lịch sử an ninh mạng: **Kerberos** (Tên con chó săn 3 đầu canh gác địa ngục).

**Kerberos không dùng Mã hóa Bất đối xứng (Chậm), nó xài 100% Mã hóa Đối xứng AES siêu tốc.**

### Cấu trúc Ba Đầu Của Địa Ngục
Trong mạng Kerberos, mọi máy tính là mù lòa, không ai tin ai, trừ một kẻ độc tài đứng giữa duy nhất: **Trung tâm Phân phối Chìa Khóa (KDC - Key Distribution Center)**. KDC giữ bản copy Chìa khóa Bí mật tĩnh (Master Key) của mọi Server trong công ty.

```mermaid
graph TD
    subgraph Kiến trúc Giao dịch Vé (Ticket) Kerberos
        C(Client <br> Cần truy cập Hadoop) -->|1. Xin chào, tôi là Client| KDC((Máy chủ KDC <br> Giữ mọi mật khẩu))
        
        KDC -.->|2. KDC kiểm tra Master Key <br> Trả về 'Vé xin việc' (TGT)| C
        
        C -->|3. Tôi muốn vào máy Server Hadoop. <br> Đây là vé TGT của tôi| KDC
        
        KDC -.->|4. KDC cấp 'Vé Vào Cửa Hadoop'| C
        
        C -->|5. Trình 'Vé Vào Cửa Hadoop'| S[(Máy chủ Hadoop)]
        
        S -.->|Máy chủ Hadoop tự lấy chìa khóa của nó mở vé. <br> Khớp 100%, cho truy cập| C
    end
    style KDC fill:#dc3545,stroke:#000,color:#fff
```

**Tại sao Kerberos bảo mật tới mức cực đoan?**
1. **Không bao giờ gửi Mật khẩu qua mạng:** Xuyên suốt chu trình, Client KHÔNG BAO GIỜ gửi chuỗi Password qua cáp mạng (kể cả khi đã bọc TLS). Nó gửi các "Dấu hiệu chứng minh mình có password" qua thuật toán mã hóa khóa đối xứng. Hacker rình mò cũng chỉ lấy được đống byte vô nghĩa.
2. **Xác thực 2 Chiều (Mutual Authentication):** Client sợ truy cập nhầm Server Fake (Hacker giả mạo Hadoop). Kerberos thiết kế chiếc Vé Vào Cửa theo dạng "Khóa 2 lớp". Server Fake không giữ cái chìa khóa thật của Hadoop được cấu hình tại KDC, nên nó sẽ bất lực không thể mở chiếc vé mà Client đưa cho nó. Cuộc đàm phán sụp đổ ngay lập tức.
3. **Vé Có Thời Hạn Nhất Định (Ticket Expiration):** Một chiếc vé TGT của Kerberos chỉ sống vài tiếng. Lỡ Hacker cướp được cũng không thể xài mãi mãi. 

**Nỗi đau của Data Engineer:** Việc cấu hình tệp `keytab` (Tệp chứa chìa khóa bí mật vật lý) của Kerberos trên cụm trăm máy chủ Kafka/HDFS là một trong những thao tác cực khổ và dễ Crash hệ thống bậc nhất. Tuy nhiên, nếu thiếu đi con chó 3 đầu Kerberos, kiến trúc Big Data nội bộ của bạn về cơ bản đang cởi truồng trước bất kỳ đoạn mã độc nội bộ nào (Ransomware) lọt qua được lớp bảo mật VPC.

---
*(Kết thúc Chặng 4 - Hoàn tất lộ trình Cốt lõi của Khoa học Máy Tính)*

---
**Navigation:**
[⬅️ Previous: Bài 11: Mật mã học Cơ sở và Quá trình Bắt tay TLS/SSL (Handshake)](./11-tls-ssl-and-symmetric-encryption.md)
