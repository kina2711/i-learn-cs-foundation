# Bài 9: Kiến trúc Cân bằng Tải (Load Balancing) và Reverse Proxy

Khi người dùng hay hàng triệu thiết bị IoT đồng loạt bắn dữ liệu sự kiện (Event Streaming) về máy chủ công ty, 1 máy chủ Kafka hay 1 máy chủ Web API sẽ lập tức quá tải CPU và sập nguồn. Chúng ta cần dàn trải 10 máy chủ, nhưng phía người dùng chỉ muốn ghi nhớ **đúng 1 địa chỉ IP duy nhất**.

Đó là lúc hệ thống **Cân bằng tải (Load Balancer)** xuất hiện. Đứng chắn giữa thế giới Internet bão táp và cụm máy chủ nội bộ mong manh, Load Balancer đóng vai trò như một Tổng đài viên chia việc (Traffic Distributor) và một Tấm khiên bảo vệ (Reverse Proxy).

---

## 1. Cân bằng tải Layer 4 (Transport Layer)

Layer 4 Load Balancer hoạt động ở tầng Giao vận (Transport Layer) của mô hình OSI, can thiệp vào giai đoạn bắt tay mạng TCP (Bài 7).
Công cụ tiêu biểu: **HAProxy, AWS Network Load Balancer (NLB)**.

**Đặc tính hoạt động:**
- Nó cực kì "ngu ngốc" và mù tịt về nội dung bức thư. Nó không biết người dùng đang tải ảnh, xem video, hay gửi file JSON.
- Nó chỉ nhìn vào bì thư: Địa chỉ IP Nguồn, IP Đích, và Cổng (Port).
- Khi Client mở kết nối TCP đến Load Balancer, Load Balancer lập tức sửa địa chỉ IP Đích (NAT) và ném nguyên kiện gói tin TCP sang cho Máy chủ Backend bên trong. 

**Hiệu năng:** Vì không cần phải mở bưu kiện ra xem mã hóa, Layer 4 cực kỳ nhẹ. Nó tốn ít CPU và có thể phân phát hàng triệu gói TCP mỗi giây (Triệu Request/s). Thích hợp nhất cho các luồng dữ liệu Streaming nặng nề như kết nối ngầm gRPC hoặc Kafka TCP.

---

## 2. Cân bằng tải Layer 7 (Application Layer)

Layer 7 Load Balancer hoạt động ở tầng Ứng dụng (Application Layer), thọc sâu vào tận cùng giao thức của thông điệp (như HTTP, HTTPS, WebSocket).
Công cụ tiêu biểu: **Nginx, AWS Application Load Balancer (ALB)**.

**Đặc tính hoạt động (Lợi hại và Nặng nề):**
- Nó bắt buộc phải mở bưu kiện mã hóa ra. Nó phải thực hiện quá trình bắt tay TCP, sau đó lại thực hiện giải mã TLS/SSL (Bài 11) để đọc toàn bộ đoạn Header và Body của gói tin HTTP.
- Bù lại, sự tinh vi của nó tạo ra quyền năng **Định tuyến Thông minh (Smart Routing)**.

```mermaid
graph TD
    subgraph Kiến trúc Layer 7 Load Balancing (Reverse Proxy)
        Client((Mobile App)) -->|Gửi POST /api/v1/payments| LB[Nginx Layer 7 LB]
        Client2((Trình duyệt)) -->|Gửi GET /images/logo.png| LB
        
        LB -.->|Đọc URL: Thấy chữ /payments <br> Chuyển sang Cụm API Thanh Toán| API[Server Thanh Toán 1 & 2]
        LB -.->|Đọc URL: Thấy chữ /images <br> Chuyển sang Cụm Static Storage| CDN[Server Hình ảnh 1 & 2]
    end
```

**Đánh đổi của Layer 7:** Máy chủ Load Balancer Layer 7 tốn cực nhiều CPU và RAM vì nó phải tự thân cắt dán (parsing) văn bản HTTP và giải mã bảo mật SSL. Tốc độ Request/s của nó thấp hơn Layer 4 rất nhiều. Nhưng đối với kiến trúc Microservices hiện đại, tính năng đọc hiểu đường dẫn URL để chia việc (Path-based Routing) là không thể thay thế.

---

## 3. Bản chất của Reverse Proxy và Các thuật toán Chia việc

Hầu hết Load Balancer Layer 7 đều kiêm luôn chức năng **Reverse Proxy (Đại diện ranh giới ngược)**.
Proxy thông thường (Forward Proxy) đại diện cho máy tính của bạn đi ra Internet (như VPN). Ngược lại, **Reverse Proxy đại diện cho Máy chủ nội bộ để đối mặt với Internet**.
- **Che giấu cấu trúc mạng:** Internet không bao giờ biết được số lượng và IP nội bộ của các máy chủ Spark hay API giấu bên trong VPC (Bài 8). Chúng chỉ nhìn thấy IP mặt tiền của Nginx Reverse Proxy.
- **SSL Termination (Đỡ đạn Bảo mật):** Thay vì bắt 10 máy chủ nội bộ tự đi giải mã mật khẩu SSL của hàng vạn user (Tốn CPU kinh khủng), người ta dồn 100% công việc giải mã này cho Reverse Proxy đứng chặn cửa làm. Khi bức thư vào đến mạng nội bộ an toàn, nó được truyền đi dưới dạng không mã hóa (Plaintext), cứu rỗi hiệu năng CPU cho các máy chủ tính toán.

### 3 Thuật toán Phân mảnh Kết nối Cơ bản:
Để chọn máy chủ nhận việc, Load Balancer dùng các thuật toán:
1. **Round Robin (Xoay vòng):** Lượt 1 cho Máy A, lượt 2 Máy B, lượt 3 Máy C. Đều đặn, nhưng không biết máy nào đang rảnh hay bận.
2. **Least Connections (Giao cho người rảnh nhất):** Đếm số lượng kết nối TCP đang mở của các Máy. Máy nào đang hầu hạ ít kết nối nhất sẽ bị quăng thêm việc.
3. **IP Hash (Trói buộc người dùng):** Nó dùng hàm băm (Hash) lấy địa chỉ IP của Client làm Khóa. Nếu Client mang IP 103.5.2.1, thuật toán luôn chỉ định vĩnh viễn Client này vào Máy B. Ứng dụng để giữ giỏ hàng / Phiên đăng nhập bộ nhớ đệm (Session Sticky) không bị phân mảnh. (Mặc dù vậy, với Redis thì Session Sticky không còn quá khắt khe - Bài 7 Part 3).

---
**Navigation:**
[⬅️ Previous: Bài 8: Mạng Nội bộ Đám mây: IP, DNS, BGP và Kiến trúc VPC](./08-dns-bgp-and-vpc.md) | [Next: Bài 10: Sự tiến hóa HTTP và Kiến trúc Data Pipeline với gRPC ➡️](./10-http-evolution-and-grpc.md)
