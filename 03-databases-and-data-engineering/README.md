# Part 3: Cơ sở Dữ liệu và Kiến trúc Hệ thống Dữ liệu (Databases & Data Engineering Core)

Chào mừng bạn đến với **Module 3: Cơ sở Dữ liệu & Data Engineering**.

Tiếp nối các khái niệm về Cấu trúc Dữ liệu từ Module 2, Module này đi sâu vào cách thức các hệ thống máy tính lưu trữ, truy xuất và xử lý khối lượng dữ liệu khổng lồ (Big Data) trong thực tế. Nội dung được thiết kế đặc thù theo định hướng **Data Engineer (Kỹ sư Dữ liệu)**, không chỉ tập trung vào hệ quản trị cơ sở dữ liệu truyền thống (OLTP) mà còn phân tích chuyên sâu các công nghệ lưu trữ phân tán, đường ống dữ liệu (Data Pipelines) và hệ thống xử lý song song.

Các bài học tuân thủ nguyên tắc khách quan, khoa học, đi sát vào kiến trúc phần cứng (Under-the-hood) của các engine hệ thống.

---

## 📚 Mục lục Nội dung (Syllabus)

Toàn bộ tài liệu của Module này được lưu trữ trong thư mục `notes/`.

### 🗄️ Chương 1: Kiến trúc Lưu trữ Cơ sở Hệ thống (Storage Architecture & OLTP)
- [Bài 1: Phân tầng Lưu trữ, OLTP vs OLAP và Kiến trúc Dòng/Cột](./notes/01-oltp-vs-olap-and-storage-engines.md)
- [Bài 2: Kiến trúc Indexing Cốt lõi: B-Tree và Hash Index](./notes/02-database-indexing-and-btrees.md)
- [Bài 3: Giao dịch ACID và Nhật ký Ghi trước (WAL)](./notes/03-wal-and-acid-transactions.md)

### 🌐 Chương 2: Cấu trúc Hệ thống Phân tán (Distributed Systems Core)
- [Bài 4: Định lý CAP, PACELC và Thiết kế Đánh đổi Hệ thống](./notes/04-cap-theorem-and-pacelc.md)
- [Bài 5: Nhân bản Dữ liệu (Replication) và Đồng thuận Quorum](./notes/05-replication-and-consensus.md)
- [Bài 6: Phân mảnh ngang (Sharding) và Băm nhất quán (Consistent Hashing)](./notes/06-partitioning-and-sharding.md)

### ⚡ Chương 3: Hệ sinh thái NoSQL & Tối ưu Dữ liệu (NoSQL & Caching)
- [Bài 7: Bộ đệm (Caching), Redis và Chiến lược xử lý rủi ro Cache](./notes/07-key-value-stores-and-caching.md)
- [Bài 8: Cơ sở dữ liệu Tài liệu (Document DB) và Cấu trúc BSON](./notes/08-document-databases-and-mongodb.md)
- [Bài 9: Cơ sở dữ liệu Cột Rộng (Cassandra) và LSM-Tree](./notes/09-wide-column-and-lsm-trees.md)

### 🏗️ Chương 4: Kiến trúc Kho Dữ liệu Lớn (Data Warehousing & Data Lakes)
- [Bài 10: Kiến trúc Data Warehouse và Mô hình hóa Đa chiều (Dimensional Modeling)](./notes/10-data-warehouse-and-dimensional-modeling.md)
- [Bài 11: Data Lake, Hệ thống phân tán HDFS/S3 và Định dạng Parquet/ORC](./notes/11-data-lake-and-file-formats.md)
- [Bài 12: Thiết kế Đường ống Dữ liệu: Lựa chọn ETL vs ELT và Change Data Capture (CDC)](./notes/12-etl-vs-elt-and-cdc.md)

### 🌊 Chương 5: Xử lý Dữ liệu Lớn & Tìm kiếm (Big Data Processing & Search)
- [Bài 13: Mô hình MapReduce và Nguyên lý Xử lý In-Memory của Apache Spark](./notes/13-mapreduce-and-spark.md)
- [Bài 14: Kiến trúc Hướng Sự kiện (Event-Driven) và Apache Kafka](./notes/14-message-queues-and-kafka.md)
- [Bài 15: Công cụ Tìm kiếm (Elasticsearch) và Chỉ mục Ngược (Inverted Index)](./notes/15-search-engines-and-elasticsearch.md)

---
