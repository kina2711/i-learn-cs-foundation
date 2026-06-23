# Part 5: Kiến trúc Dữ liệu Hiện đại & Thiết kế Hệ thống (Modern Data Stack & System Design)

Chào mừng bạn đến với **Module 5**, giới hạn cuối cùng trong quá trình đào tạo Tư duy Kiến trúc sư Dữ liệu (Data Architect).

Nếu 4 Chặng trước cung cấp cho bạn từng mảnh ghép phần cứng và lý thuyết riêng lẻ (như TCP, Zero-copy, RAM, B-Tree, K8s), thì Chặng 5 này mang nhiệm vụ lắp ráp tất cả những mảnh ghép đó thành một cỗ máy hoàn chỉnh đang được vận hành tại các tập đoàn công nghệ lớn. 

Module này đi sâu vào **Cơ chế dưới mui xe (Under-the-hood)** của những công nghệ Big Data hiện đại nhất hiện nay (Flink, Iceberg, Vectorized Processing) và cách vận dụng chúng để thiết kế kiến trúc hệ thống xử lý hàng vạn giao dịch mỗi giây (System Design).

---

## 📚 Mục lục Nội dung (Syllabus)

Toàn bộ tài liệu của Module này được lưu trữ trong thư mục `notes/`.

### ⚡ Chương 1: Kiến trúc Tính toán Thời gian Thực (Streaming & Real-time Compute)
- [Bài 1: Giới hạn của Batch Processing và Sự tiến hóa của Kiến trúc Lambda/Kappa](./notes/01-batch-vs-streaming-and-lambda-architecture.md)
- [Bài 2: Trái tim Apache Flink: Checkpointing (Chandy-Lamport) và RocksDB](./notes/02-apache-flink-and-stateful-processing.md)
- [Bài 3: Xử lý Thời gian thực: Event Time, Watermarks và Thuật toán Windowing](./notes/03-event-time-watermarks-and-windowing.md)
- [Bài 4: Bất khả xâm phạm: Exactly-Once Semantics, 2PC và Tính Lũy đẳng (Idempotence)](./notes/04-exactly-once-semantics-and-idempotence.md)

### 🧊 Chương 2: Kiến trúc Hồ Dữ liệu Hiện đại (Modern Data Lakehouse)
- [Bài 5: Data Lakehouse: Đưa tính Nhất quán ACID lên nền tảng Lưu trữ Đám mây (S3)](./notes/05-lakehouse-and-acid-on-cloud.md)
- [Bài 6: Giải phẫu Apache Iceberg: Cấu trúc Metadata, Hidden Partitioning và Time Travel](./notes/06-apache-iceberg-architecture.md)
- [Bài 7: Truy vấn Siêu thanh: Trình lặp Volcanic và Sức mạnh Lệnh CPU SIMD (Vectorization)](./notes/07-vectorized-query-execution-and-simd.md)
- [Bài 8: Não bộ của Spark/Trino: CBO (Tối ưu hóa Chi phí) và Broadcast Hash Join](./notes/08-cost-based-optimizer-cbo.md)

### 🕸️ Chương 3: Điều phối và Quản trị Dữ liệu (Orchestration & Governance)
- [Bài 9: Apache Airflow Dưới Góc Độ Kernel: Scheduler, Executor và Giao tiếp XCom](./notes/09-apache-airflow-and-dag-internals.md)
- [Bài 10: Giám sát Mạng nhện Dữ liệu: Data Lineage, Data Catalog và Data Mesh](./notes/10-data-lineage-and-observability.md)

### 🏗️ Chương 4: Thiết kế Hệ thống Thực tiễn (hệ thống lớn System Design Case Studies)
- [Bài 11: Case Study 1 - Thiết kế Bộ đếm View Youtube (Kiến trúc Heavy-Write và Sharding)](./notes/11-design-youtube-view-counter.md)
- [Bài 12: Case Study 2 - Thiết kế Thuật toán Giá cước Uber (Geohash, Pub/Sub và Streaming Cache)](./notes/12-design-uber-surge-pricing.md)

---
