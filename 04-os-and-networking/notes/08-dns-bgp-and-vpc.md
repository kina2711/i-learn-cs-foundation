# Bài 8: Mạng Nội bộ Đám mây: IP, DNS, BGP và Kiến trúc VPC

Khi Data Engineer thiết lập một cụm máy chủ gồm 1 Node Master (Airflow), 3 Node Kafka và 10 Node Spark trên AWS hoặc Google Cloud. Việc đầu tiên không phải là cài phần mềm, mà là xây dựng một hệ thống mạng vô hình để các máy tính này có thể nói chuyện với nhau một cách an toàn.

Kiến trúc này được xây dựng trên tầng Mạng lưới (Network Layer - Layer 3), nơi ngự trị của **Địa chỉ IP**, **DNS** và không gian cách ly **VPC (Virtual Private Cloud)**.

---

## 1. Định dạng IP và Subnetting (Chia Mạng Con)

Mọi máy tính trên mạng đều phải có một địa chỉ IP (Ví dụ IPv4: `192.168.1.15`). Địa chỉ IP bản chất là một dải 32 bit nhị phân.
Để hệ thống định tuyến (Router) không bị điên khi quản lý 4 tỷ địa chỉ máy tính rải rác, cấu trúc IP được chia làm 2 phần: **Phần Mạng (Network ID)** và **Phần Máy khách (Host ID)**. 

Công cụ dùng để chặt IP làm 2 đoạn chính là **Subnet Mask**.
- Ví dụ phổ thông: IP `192.168.1.15` với Subnet Mask `255.255.255.0` (Ký hiệu ngắn gọn là `/24`).
- Điều này có nghĩa: 3 cụm đầu `192.168.1` là Tên Đường (Network ID). Cụm cuối `15` là Số Nhà (Host ID).
- Một mạng `/24` cung cấp giới hạn đúng $2^{8} - 2 = 254$ số nhà máy tính khác nhau.

### Kỹ thuật Subnetting cho Big Data
Nếu bạn triển khai 1000 Pods Kubernetes (Bài 6), một dải `/24` sẽ sập vì hết IP. Data Engineer phải chia dải mạng cực lớn, ví dụ `/16` (Kéo dài từ `10.0.0.0` đến `10.0.255.255`), cung cấp 65.534 không gian IP thoải mái cho hàng vạn Container.

---

## 2. DNS và Giao thức BGP (Định tuyến Toàn cầu)

Nếu máy chủ của bạn cần móc nối ra ngoài lấy dữ liệu chứng khoán từ `api.nasdaq.com`, máy tính không hiểu chữ "nasdaq". 

**A. Cơ chế Phân giải Tên miền (DNS):**
Hệ điều hành sẽ đi hỏi một máy chủ danh bạ (DNS Server) bằng cổng UDP siêu nhanh: "Số IP của thằng nasdaq là bao nhiêu?". Máy chủ sẽ lục tìm trong cơ sở dữ liệu và trả về cấu trúc `198.51.100.4`. Máy chủ Kafka của bạn lúc này mới chính thức bắn kết nối gói tin TCP (Bài 7) tới con số IP đó.

**B. Giao thức BGP (Border Gateway Protocol):**
Khi gói tin rời khỏi máy chủ của bạn để bay đến nước Mỹ (IP 198.51.100.4), nó không bay theo một đường thẳng. Nó đi qua hàng chục trạm gác biên giới của các nhà cung cấp cáp quang (ISP như VNPT, Viettel, AT&T).
Giao thức **BGP** là bộ luật giao thông định tuyến toàn cầu. Các Router lớn trên thế giới nói chuyện với nhau bằng BGP để đàm phán ra đường đi ngắn nhất hoặc rẻ nhất. Bất cứ khi nào BGP cấu hình sai (Lỗi con người), lập tức một nửa lượng traffic toàn cầu có thể bị hút vào một hố đen, làm sập toàn bộ Facebook hay Google trong vài giờ.

---

## 3. Kiến trúc VPC (Virtual Private Cloud) - Ranh giới An ninh Đám mây

Trong kỷ nguyên Đám mây, hàng chục ngàn công ty đang thuê máy chủ dùng chung trên cùng một siêu Datacenter của Amazon/Google. Để công ty A không nhìn trộm luồng dữ liệu giao dịch của công ty B, nền tảng điện toán đám mây sinh ra **VPC (Đám mây Riêng Ảo)**.

VPC là một khu công nghiệp ảo, rào bằng hàng rào kẽm gai điện tử (Software-Defined Networking). Bất cứ Data Pipeline nào cũng phải tuân thủ chuẩn kiến trúc VPC đa tầng:

```mermaid
graph TD
    subgraph VPC của Công ty (Dải IP: 10.0.0.0/16)
        IGW((Internet Gateway)) --> FW[Firewall / Security Group]
        
        subgraph Public Subnet (Có thể ra/vào Internet)
            FW --> LB[Load Balancer]
            FW --> BS[Bastion Host / SSH Jump Server]
        end
        
        subgraph Private Subnet (Kín bưng - Cô lập với Thế giới)
            LB --> App[Kafka Brokers & Spark Workers]
            BS -.-> App
            App --> DB[(Database / Data Warehouse)]
        end
        
        App --> NAT((NAT Gateway)) --> IGW
    end
```

**Nguyên tắc Vàng trong Thiết kế Mạng Data Engineering:**
1. **Private Subnet (Lớp giấu kín):** Tuyệt đối MỌI hệ thống lõi lưu trữ dữ liệu (Database, Kafka, Data Warehouse) phải nằm trong mạng Private Subnet. Không một ai trên Internet có thể ping hay kết nối thẳng tới chúng.
2. **NAT Gateway:** Khi Server Spark nằm trong Private Subnet muốn tải bản update thư viện Python từ Internet, nó bị kẹt vì không có cổng kết nối ra ngoài. Nó phải mượn đường chui qua máy ảo **NAT Gateway** nằm ở Public Subnet. NAT sẽ giấu IP thật của máy Spark, đại diện nó thò mặt ra Internet xin dữ liệu mang về.
3. **Security Groups:** Là những tường lửa (Firewall) tí hon áp chặt vào từng máy chủ. Nó cấu hình quy tắc chặn chốt: "Chỉ cho phép IP của Load Balancer gọi vào Cổng 80 của ứng dụng. Cấm tất cả các dải IP khác". 

Sự am hiểu sâu sắc về kiến trúc mạng VPC sẽ giúp kỹ sư tránh được những thảm họa bảo mật tàn khốc, như việc vô tình phơi bày cơ sở dữ liệu Elasticsearch ra Internet không có mật khẩu (Lỗ hổng khét tiếng nhất trong 10 năm qua của giới dữ liệu lớn).

---
**Navigation:**
[⬅️ Previous: Bài 7: Tầng Giao vận TCP/IP, Cửa sổ trượt và Kiểm soát Tắc nghẽn](./07-tcp-ip-and-congestion-control.md) | [Next: Bài 9: Kiến trúc Cân bằng Tải (Load Balancing) và Reverse Proxy ➡️](./09-load-balancing-and-reverse-proxy.md)
