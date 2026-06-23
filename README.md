# 🚀 Hành trình Khoa học Máy tính & Kiến trúc Dữ liệu

Chào mừng bạn đến với **Lộ trình Khoa học Máy tính Toàn diện**, được thiết kế đặc biệt dành riêng cho định hướng **Kỹ sư Dữ liệu (Data Engineer) & Kiến trúc sư Đám mây (Cloud Architect)**. 

Dự án này là một thư viện tri thức gồm **83 Chuyên đề** đi từ những byte bộ nhớ nhỏ nhất của hệ điều hành, cho đến cấu trúc thuật toán lõi, và vươn tới kiến trúc phân tán. Toàn bộ nội dung được biên soạn không chỉ để "biết code", mà để hiểu sâu sắc **Cơ chế dưới mui xe (Under-the-hood)** của mọi hệ thống.

---

## 🧭 Cấu trúc Lộ trình (5 Chặng Cốt Lõi)

Lộ trình được chia làm 5 Module nối tiếp nhau, mỗi Module là một mảnh ghép không thể tách rời:

### 🔹 [Part 1: Tư duy Máy tính & Lập trình Hướng đối tượng](./01-programming-and-oops/README.md)
*Nền tảng giao tiếp với máy tính ở cấp độ thấp.*
- Quản lý bộ nhớ Stack & Heap, Vòng đời Garbage Collection.
- Bản chất của Con trỏ (Pointers), Tham chiếu (References).
- Thiết kế Hệ thống với Lập trình Hướng đối tượng (OOP) và Design Patterns.
- **Tổng số: 27 Bài học.**

### 🔹 [Part 2: Cấu trúc Dữ liệu & Thuật toán (DSA)](./02-data-structures-and-algorithms/README.md)
*Nghệ thuật Đánh đổi (Trade-offs) giữa Thời gian CPU và Không gian RAM.*
- Phân tích độ phức tạp Big O ($O(1)$, $O(\log N)$, $O(N)$...).
- Giải phẫu Mảng (Array), Danh sách liên kết (Linked List), Ngăn xếp (Stack).
- Sức mạnh của Bảng băm (Hash Tables), Cây nhị phân (Binary Trees) và Đồ thị (Graphs).
- **Tổng số: 17 Bài học.**

### 🔹 [Part 3: Cơ sở Dữ liệu & Xử lý Dữ liệu Lớn (Databases & Big Data)](./03-databases-and-data-engineering/README.md)
*Trái tim của Data Engineering.*
- Cấu trúc vật lý ổ cứng: Row-oriented vs Columnar, B-Tree vs LSM-Tree.
- Hệ thống Phân tán: Định lý CAP, Cơ chế Nhân bản (Replication), Băm nhất quán (Consistent Hashing).
- Kỷ nguyên kho chứa: Data Warehouse (Star Schema) vs Data Lake (Parquet).
- Xử lý Mảng/Khối: Apache Spark (In-Memory) và Apache Kafka (Message Queue).
- **Tổng số: 15 Bài học.**

### 🔹 [Part 4: Hệ Điều Hành & Hệ thống Mạng (OS & Networking)](./04-os-and-networking/README.md)
*Môi trường thực thi và những Nút thắt Vật lý.*
- Lõi Linux: Context Switching, OOM-Killer, Page Cache và Ma thuật Zero-copy của Kafka.
- Container: Bản chất dối trá của Docker (Namespaces, Cgroups) và Kubernetes Control Plane.
- Giao thức Mạng: Tắc nghẽn TCP/IP, Cấu trúc VPC, Load Balancing (Layer 4 vs Layer 7).
- Bảo mật Nâng cao: Sự tiến hóa HTTP sang gRPC, Bắt tay TLS/SSL và Chó 3 đầu Kerberos.
- **Tổng số: 12 Bài học.**

### 🔹 [Part 5: Kiến trúc Dữ liệu Hiện đại & System Design](./05-modern-data-stack-and-system-design/README.md)
*Hội tụ kiến thức thành siêu hệ thống xử lý thời gian thực.*
- Streaming vs Batch: Nút thắt Lambda/Kappa, Apache Flink Checkpointing và Exactly-once.
- Modern Lakehouse: Apache Iceberg (Time Travel, Metadata Tree) phá bỏ giới hạn File Listing S3.
- Truy vấn CPU: Volcanic Iterator vs Sức mạnh Tập lệnh CPU SIMD (Vectorization).
- Airflow DAGs và Quản trị Lưới dữ liệu (Data Mesh).
- Thực chiến System Design: Bộ đếm 1 triệu View/giây của Youtube và Thuật toán Geohash giá cước Uber.
- **Tổng số: 12 Bài học.**

---

## 🎯 Triết lý Học tập
> *"Đừng học cách sử dụng công cụ. Hãy học cách công cụ được tạo ra."*

Tất cả các bài viết trong Repo này đều tuân thủ nguyên tắc:
1. **Khách quan và Khoa học:** Loại bỏ sự màu mè, tập trung vào bản chất I/O, RAM, CPU và Mạng.
2. **Sơ đồ Trực quan:** 100% sử dụng biểu đồ `Mermaid` để minh họa dòng chảy kiến trúc phân tán.
3. **Thực tiễn Data Engineering:** Mọi khái niệm thuật toán đều được ánh xạ thẳng vào các nút thắt thường gặp trong thiết kế Data Pipeline và ETL.

Chúc bạn có một hành trình khai mở tư duy thành công!