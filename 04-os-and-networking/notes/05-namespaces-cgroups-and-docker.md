# Bài 5: Cấu trúc Cách ly Lõi Linux (Namespaces) và Bản chất Container (Docker)

Nhiệm vụ quản lý hạ tầng Dữ liệu lớn (Big Data) đòi hỏi Data Engineer phải triển khai hàng trăm ứng dụng khác biệt trên cùng một cụm máy chủ. Máy chủ có Spark chạy Python 3.8, máy khác chạy Kafka dùng Java 11. Sự xung đột thư viện (Dependency Hell) giữa các ứng dụng trên cùng một Hệ điều hành vật lý là nguyên nhân gây sập hệ thống (Crash) phổ biến nhất.

Giải pháp ảo hóa ra đời để cách ly chúng. Máy ảo (Virtual Machines - VM) là thế hệ đầu tiên, nhưng **Containerization (Docker)** mới là đỉnh cao hiệu năng làm thay đổi hoàn toàn kiến trúc Đám mây hiện đại.

---

## 1. Ranh giới của Máy Ảo (VM - Virtual Machines)

VM, được cung cấp bởi các Hypervisor như VMware hoặc VirtualBox, hoạt động theo triết lý "Mô phỏng Toàn phần". 
Nó cắt một phần cứng (RAM, CPU, Disk) của máy chủ vật lý, sau đó cài đặt nguyên vẹn một Hệ Điều Hành Khách (Guest OS) khổng lồ chứa hàng tỷ dòng lệnh Kernel (Ví dụ Ubuntu 10GB) lên phần cứng ảo đó. 

**Cơn ác mộng Hiệu năng của VM:**
1. Khởi động 1 VM mất từ 1 đến vài phút (Phải chạy từ màn hình BIOS, nạp Kernel boot).
2. Tốn tài nguyên rác: Để chạy 1 phần mềm Redis 50MB, bạn phải cõng nguyên một cái Guest OS 2GB RAM chỉ để phục vụ nó. Nếu chạy 10 con Redis, bạn phải cài 10 cái Hệ điều hành OS dư thừa lên RAM.

---

## 2. Ảo ảnh của Docker Container

Docker không phải là một cỗ máy ảo. Nó không hề có Kernel hệ điều hành hay phần cứng mô phỏng.

**Bản chất Vật lý:** Container dưới góc nhìn của Hệ điều hành thực chất chỉ là MỘT TIẾN TRÌNH (Process) bình thường đang chạy trên máy tính, giống hệt một tệp Script Python hay C++. Tuy nhiên, nó là một "Tiến trình bị lừa dối". 
Linux cung cấp hai vũ khí tối thượng ở tầng Kernel (Ring 0) để tạo ra sự lừa dối hoàn hảo này: **Namespaces** và **Cgroups**.

### A. Namespaces (Ảo ảnh Môi trường)
Namespace là bộ định tuyến phân định ranh giới cách ly (Isolation).
Khi OS bọc một tiến trình bằng Namespace, tiến trình đó sẽ bị "nhốt trong một cái lồng kính ảo giác":
- **PID Namespace:** Ảo giác tiến trình. Docker tạo 10 tiến trình, nhưng bên trong Container, phần mềm tin rằng nó là chúa tể duy nhất trên máy có PID số 1.
- **Mount Namespace:** Ảo giác Tệp tin. Ứng dụng trong lồng kính không nhìn thấy thư mục `C:\` hay `/root` thật sự của OS chủ. Nó chỉ nhìn thấy một bộ thư mục mô phỏng của riêng nó.
- **Net Namespace:** Ảo giác Mạng. Tiến trình sở hữu một địa chỉ IP nội bộ, Card mạng và Port riêng không xung đột với máy chủ.

### B. Cgroups (Control Groups - Cai ngục Tài nguyên)
Nếu Namespace đánh lừa góc nhìn, Cgroups là chiếc khóa xiềng xích phần cứng. Cgroups kiểm soát mức trần tiêu thụ tài nguyên phần cứng cực kỳ khắt khe cho tiến trình Container:
- "Chỉ được dùng tối đa 20% chu kỳ CPU Core 1".
- "Không được phép xài vượt quá 500MB RAM. Vượt qua mức này, tiến trình lập tức bị OOM-Killer thủ tiêu ngay (Bài 2)".

---

## 3. Kiến trúc Đột phá Không Hệ điều hành

Chính nhờ việc không cần thiết lập Guest OS trung gian, Docker Container hoạt động với sức mạnh của một kiến trúc Vô song.

```mermaid
graph TD
    subgraph "Kiến trúc Máy ảo (VMware/KVM)"
        H1[Server Hardware] --> HY[Hypervisor]
        HY --> G1["Guest OS 1 (2GB RAM")] --> A1[App A]
        HY --> G2["Guest OS 2 (2GB RAM")] --> A2[App B]
    end
    
    subgraph "Kiến trúc Container (Docker)"
        H2[Server Hardware] --> L[Host OS Linux Kernel]
        L -->|Phân cách Cgroups/Namespaces| D1[Docker Eng]
        D1 -.-> CA["Container App A (Chạy thẳng trên Kernel Host")]
        D1 -.-> CB["Container App B (Chạy thẳng trên Kernel Host")]
    end
```

**Thành tựu Tốc độ:**
1. Container dùng chung lõi Kernel của hệ điều hành Host vật lý bên dưới. (Đây là lý do bạn không thể chạy Container Kernel Windows trực tiếp trên lõi Linux mà không qua lớp máy ảo ngầm định).
2. Nhờ xài chung Kernel, một Container không cần cài cắm OS. Khởi động 1 Container bản chất chỉ là bật 1 tiến trình (Process), tốn đúng **Vài chục mili-giây**.
3. Kích thước ổ đĩa của 1 Image Redis chỉ nặng vỏn vẹn 30MB (Chứa duy nhất hệ file library và mã thực thi, 0 MB rác hệ điều hành).

Với sức mạnh này, Data Engineer có thể đóng gói toàn bộ Data Pipeline gồm (Airflow, Spark, dbt) vào các Image cấu trúc bất biến (Immutable), ném lên bất kỳ máy chủ nào trên thế giới mà vẫn chắc chắn 100% chúng chạy y hệt nhau, không bao giờ bị dính lỗi xung đột môi trường. Sự nở rộ của hàng ngàn Container vi mô đòi hỏi một thuyền trưởng vĩ đại để điều phối: **Kubernetes**.

---
**Navigation:**
[⬅️ Previous: Bài 4: Song song hóa (Process vs Thread), Nút thắt GIL và Cơ chế Async I/O](./04-process-thread-and-async-io.md) | [Next: Bài 6: Giải phẫu Kiến trúc Kubernetes (K8s) cho Data Pipelines ➡️](./06-kubernetes-architecture.md)
