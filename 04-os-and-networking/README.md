# Part 4: Lõi Hệ điều hành và Mạng Máy tính (OS Kernel & Computer Networking)

Chào mừng bạn đến với **Module 4: Hệ điều hành & Mạng Máy tính**.

Nếu Module 2 (Thuật toán) và Module 3 (Cơ sở dữ liệu) giúp bạn định hình cách phần mềm tổ chức dữ liệu, thì Module 4 sẽ bóc tách lớp màn bí ẩn cuối cùng: **Cỗ máy tính và Sợi cáp mạng bên dưới phần mềm của bạn thực sự hoạt động như thế nào?**

Nội dung được thiết kế đặc thù cho kỹ sư dữ liệu (Data Engineer) và Backend. Thay vì tập trung vào quản trị mạng văn phòng hay thiết kế giao diện OS, giáo trình này mổ xẻ **Tối ưu hóa hiệu năng (Performance Optimization)**: Kỹ thuật Zero-copy của Kafka, cơ chế cách ly tài nguyên của Docker/Kubernetes, và quá trình gRPC/TCP chia nhỏ hàng Gigabyte dữ liệu để nén qua cáp quang.

---

## 📚 Mục lục Nội dung (Syllabus)

Toàn bộ tài liệu của Module này được lưu trữ trong thư mục `notes/`.

### 🖥️ Chương 1: Lõi Hệ điều hành & Hiệu suất I/O (OS Kernel & I/O Performance)
- [Bài 1: Ranh giới An ninh (Kernel vs User Space) và Chuyển đổi Ngữ cảnh (Context Switching)](./notes/01-kernel-user-space-and-context-switching.md)
- [Bài 2: Bộ nhớ Ảo, Page Cache, Swapping và Lỗi OOM (Out-Of-Memory)](./notes/02-memory-management-and-page-cache.md)
- [Bài 3: Bí mật tốc độ của Kafka: Kỹ thuật Zero-copy và Direct I/O](./notes/03-zero-copy-and-disk-io.md)
- [Bài 4: Process vs Thread, Python GIL và Cơ chế Async I/O (epoll)](./notes/04-process-thread-and-async-io.md)

### 🐳 Chương 2: Điện toán Đám mây & Ảo hóa (Containerization & Cloud Compute)
- [Bài 5: Lõi Linux: Namespaces, Cgroups và Bản chất của Docker](./notes/05-namespaces-cgroups-and-docker.md)
- [Bài 6: Giải phẫu Kiến trúc Kubernetes (K8s) cho Data Pipelines](./notes/06-kubernetes-architecture.md)

### 🌐 Chương 3: Nền tảng Mạng Phân tán (Networking in Distributed Systems)
- [Bài 7: Mô hình TCP/IP, 3-way Handshake và Thuật toán Kiểm soát Tắc nghẽn](./notes/07-tcp-ip-and-congestion-control.md)
- [Bài 8: Định danh & Định tuyến: IP, DNS, BGP và Mạng riêng ảo (VPC)](./notes/08-dns-bgp-and-vpc.md)
- [Bài 9: Kiến trúc Cân bằng Tải (Load Balancing Layer 4 vs Layer 7) và Reverse Proxy](./notes/09-load-balancing-and-reverse-proxy.md)

### 🔒 Chương 4: Giao thức Ứng dụng & Bảo mật Mạng (App Protocols & Security)
- [Bài 10: Sự tiến hóa HTTP/2, HTTP/3 (QUIC) và Giao thức gRPC (Protocol Buffers)](./notes/10-http-evolution-and-grpc.md)
- [Bài 11: Mật mã học, Hashing và Cơ chế Bắt tay TLS/SSL Handshake](./notes/11-tls-ssl-and-symmetric-encryption.md)
- [Bài 12: Bảo mật Mạng lưới: JWT, OAuth2 và Kiến trúc Kerberos (Hadoop Core)](./notes/12-authentication-oauth2-and-kerberos.md)

---
