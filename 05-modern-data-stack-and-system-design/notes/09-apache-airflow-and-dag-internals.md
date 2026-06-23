# Bài 9: Apache Airflow Dưới Góc Độ Kernel: Scheduler, Executor và Giao tiếp XCom

Một Data Pipeline phức tạp có thể bao gồm 10 bước nối tiếp nhau: *(1) Tải File CSV từ S3 -> (2) Ép Spark dọn rác -> (3) Đổ vào Iceberg -> (4) Gửi Slack báo cáo sếp*. 
Điều gì xảy ra nếu Bước 2 chạy được 50% thì máy Spark sập? Bạn không thể tự mở máy lên gõ lệnh chạy lại Bước 2 lúc 3h sáng.
Chúng ta cần một nhạc trưởng điều phối toàn bộ các công cụ độc lập này: **Data Orchestrator**, và ngôi vương hiện tại thuộc về **Apache Airflow**.

---

## 1. Bản chất của DAG và Giới hạn Lập lịch (Scheduler)

Airflow bắt buộc Data Engineer phải thiết kế mọi Pipeline dưới cấu trúc **DAG (Directed Acyclic Graph - Đồ thị có hướng Không tuần hoàn)**. 
- *Có hướng:* Bước 1 xong mới tới Bước 2.
- *Không tuần hoàn:* Bước 3 không bao giờ được quay ngược lại gọi Bước 1 (Tránh lặp vô tận Deadlock).

**Lõi Scheduler (Trái tim nhịp đập của Airflow):**
Scheduler là một vòng lặp vô tận (Heartbeat loop) chạy trên 1 tiến trình Python liên tục quét qua thư mục code DAG của bạn. Nó tra vào Database PostgreSQL trung tâm (Metadata DB) để kiểm tra xem đã đến giờ chưa.
- Lập trình viên thiết lập: Chạy lúc 12h đêm hàng ngày. 
- Khi đồng hồ điểm 12h, Scheduler tạo ra một cái vé gọi là **DagRun**. Sau đó nó xé nhỏ các Bước ra thành **TaskInstance** và ném vào một Hàng đợi (Message Queue) thường là Redis hoặc RabbitMQ (Bài 14 Part 3).

---

## 2. Kiến trúc Thực thi (Executors) và Sự Cồng kềnh Của Worker

Airflow Scheduler không tự chạy Code Python xử lý Data. Nó ném việc (Task) cho các **Executors (Bộ thực thi)** xử lý. Có 2 loại Executor thống trị trong các hệ thống lớn:

### A. Celery Executor (Kiến trúc Cố định)
- Dùng Celery Worker (những máy chủ luôn luôn bật 24/7 đứng chờ chực). 
- **Ưu điểm:** Nhanh, Task được quăng vào là chạy ngay.
- **Tử huyệt:** Tốn tiền. Nếu từ sáng đến chiều không có DAG nào chạy, 50 con máy Worker này vẫn phải mở điện ăn không ngồi rồi tiêu tốn hàng ngàn Đô-la tiền Cloud.

### B. Kubernetes Executor (Kiến trúc Co dãn Tuyệt đối)
- Bứt phá bằng cách lợi dụng K8s (Bài 6 Part 4). Airflow không duy trì máy chủ Worker cố định nào cả. 
- Khi Scheduler rặn ra 1 Task, nó gửi lệnh cho Kubernetes API. K8s lập tức đẻ ra 1 cái **Pod (Container) mới tinh** chỉ để chạy duy nhất cái Task đó. Task vừa chạy báo Cập nhật DB xong, K8s LẬP TỨC khai tử, bắn nát cái Pod đó để giải phóng 100% RAM và CPU. Hệ thống co dãn theo thời gian thực (Zero-scaling).

---

## 3. Lỗ hổng Trạng thái: Bí mật về XCom và Thiết kế Task Độc Lập

Khác với Flink lưu State ở RocksDB. Các Task trong Airflow chạy hoàn toàn độc lập (Stateless). Task A (chạy trên máy 1) Tải File xong, nó muốn truyền chữ "File Tên Là X" sang cho Task B (chạy trên máy 2) xử lý tiếp.
Vì 2 máy bị cách ly, Task A không thể truyền biến Python (Variable) sang Task B được.

**XCom (Cross-Communication) - Vị cứu tinh đầy Rủi ro:**
Airflow sinh ra XCom. Khi Task A chạy xong, nó ném chữ "File Tên Là X" vào một cái rổ (Lưu cứng vào cơ sở dữ liệu PostgreSQL của Airflow). Task B bật lên, thò tay vào PostgreSQL đọc cái rổ đó ra để xài.

**Thảm họa Thiết kế Hệ thống của Data Engineer Nghiệp dư:**
Nhiều kỹ sư tiện tay, thay vì ném chữ "Tên File", họ bắt Task A kéo luôn cái file DataFrame 1GB ném thẳng vào XCom để chuyền cho Task B.
- Kết quả: Cái File 1GB đó bị tống thẳng vào cấu trúc B-Tree nhỏ xíu của PostgreSQL Database lõi (Metadata DB). PostgreSQL nghẹt thở, sập ngay lập tức. Toàn bộ cụm máy chủ Airflow của công ty (đang quản lý hàng ngàn DAG khác) chết chùm (Cascading Failure).

> [!WARNING] 
> **Nguyên tắc Vàng thiết kế DAG:** Airflow là nhạc trưởng, KHÔNG PHẢI công nhân bê vác. XCom chỉ được lưu Trạng thái (Tên file, ID, Status dài vài Byte). Mọi khối lượng Dữ liệu lớn (Megabyte, Gigabyte) BẮT BUỘC phải được lưu ở S3/HDFS. Các Task chỉ được phép nói chuyện với nhau bằng việc truyền Tên File, chứ tuyệt đối không truyền Nội dung File.

---
**Navigation:**
[⬅️ Previous: Bài 8: Não bộ Spark SQL: Tối ưu hóa Chi phí CBO và Broadcast Hash Join](./08-cost-based-optimizer-cbo.md) | [Next: Bài 10: Quản trị Phân tán: Data Lineage, Catalog và Mô hình Lưới (Data Mesh) ➡️](./10-data-lineage-and-observability.md)
