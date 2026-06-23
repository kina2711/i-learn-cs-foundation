# Bài 11: Mật mã học Cơ sở và Quá trình Bắt tay TLS/SSL (Handshake)

Luồng byte nhị phân của gRPC (Bài 10) dẫu có tối ưu về băng thông đến đâu, nó vẫn chạy qua một sợi cáp quang nằm dưới đáy biển Đại Tây Dương. Bất cứ ai cắm thiết bị nghe lén (Sniffer/Man-in-the-Middle) vào trạm tiếp sóng đều có thể sao chép và đọc rõ mồn một các giao dịch dữ liệu tài chính công ty bạn.

Để đường ống TCP trở thành một đường ống Bọc thép tàng hình, lớp vỏ bọc **TLS/SSL (Transport Layer Security)** ra đời (chữ 'S' trong HTTPS). Nền tảng của nó dựa trên 3 khối rubik của nền Mật mã học (Cryptography).

---

## 1. Ba Nền tảng Mật mã Lõi (Symmetric, Asymmetric, Hashing)

### A. Hàm Băm (Hashing) - Đảm bảo Tính Toàn vẹn (Integrity)
- **Cơ chế:** Dù bạn ném 1 chữ hay 1 file phim 100GB vào hàm Hash (Ví dụ: `SHA-256`), nó cũng nghiền nát và nhả ra một chuỗi chữ số vô nghĩa có độ dài cố định.
- **Tính chất Tuyệt đối:** Không thể từ chuỗi kết quả dịch ngược lại bản gốc. Nếu ai đó trộm sửa dù chỉ 1 dấu phẩy trong file 100GB, chuỗi Hash tạo ra sẽ thay đổi hoàn toàn. Dùng để check xem file truyền qua mạng có bị hacker chèn virus vào giữa chừng hay không.

### B. Mã hóa Đối xứng (Symmetric Encryption) - Kẻ ban phát Tốc độ
- **Cơ chế:** Khóa để Khóa Cửa (Mã hóa) cũng chính là chiếc Khóa dùng để Mở Cửa (Giải mã). (Ví dụ: Thuật toán `AES`).
- **Ưu điểm:** Tốc độ Mã hóa siêu thanh nhờ thiết kế toán học trên chip CPU. Data Engineer dùng AES để mã hóa ổ đĩa cứng hoặc truyền luồng file Terabyte.
- **Nhược điểm tử huyệt:** Làm thế nào Máy A gửi chìa khóa AES sang Máy B qua mạng Internet mà không sợ Hacker rình mò copy mất chiếc chìa đó trên đường đi?

### C. Mã hóa Bất đối xứng (Asymmetric Encryption) - Chìa khóa Trao tay
Được phát minh trong hệ mã khóa `RSA` hay Đường cong Elliptic (`ECC`). Nó tạo ra **2 chiếc chìa khóa khác biệt**: Khóa Công khai (Public Key) và Khóa Bí mật (Private Key).
- **Cơ chế:** Nếu dùng Public Key để khóa hòm, thì CHỈ CÓ Private Key mới mở được hòm (Bản thân Public Key cũng bất lực không thể tự mở thứ do chính nó khóa lại).
- **Ứng dụng:** Máy B giấu biệt Private Key sâu trong ổ đĩa không bao giờ cho ai biết. Máy B phô trương Public Key ném cho toàn bộ thế giới (Cả Hacker cũng nhặt được). Máy A lấy Public Key đó, nhốt cái Khóa AES (ở phần B) vào hòm, khóa cái cạch, rồi gửi hòm đó cho Máy B. Hacker cướp được hòm giữa đường cũng chịu chết vì không có Private Key. Máy B dùng Private Key mở hòm lấy chiếc Khóa AES ra. Hai bên bắt đầu chuyển sang nói chuyện siêu tốc độ bằng Khóa Đối xứng AES.
- **Nhược điểm:** Thuật toán toán học của Mã hóa Bất đối xứng (Tính số nguyên tố cực lớn) tốn rất nhiều CPU và CỰC KỲ CHẬM, không thể dùng nó để truyền luồng Data. Nó chỉ dùng để **Mã hóa Chiếc Chìa Khóa**.

---

## 2. Quá trình Bắt tay Mật mã TLS (TLS Handshake)

Để thiết lập chữ `S` trong HTTPS, Máy A và Máy B phải trải qua quá trình hợp nhất 3 công nghệ trên. Đây là quá trình "Nhức nhối" nhất làm đội độ trễ của Data Pipelines lên cao mỗi khi mở kết nối mới.

```mermaid
graph TD
    subgraph Quá trình Thiết lập Kênh TLS An Toàn (Sau khi Bắt tay TCP 3-bước)
        C(Client / Máy Spark) -->|1. Client Hello (Chào, tôi xài chuẩn AES-256 nhé)| S(Server / API Gateway)
        
        S -->|2. Server Hello (OK, Đây là Giấy Chứng nhận - Certificate chứa Public Key của tôi)| C
        
        C -.->|Client ngầm gọi Hàm Băm| CA{Check Chữ ký với Tổ chức CA <br> Đảm bảo Server không giả mạo}
        
        C -->|3. Client khóa 'Chìa khóa Bí Mật AES' bằng Public Key của Server, Gửi đi| S
        
        S -.->|Server dùng Private Key giấu kín để giải mã lấy 'Chìa khóa Bí mật AES'| S
        
        C <==>|4. Hai bên có chung Chìa Khóa Bí Mật AES. Bắt đầu truyền Data siêu tốc độ| S
    end
    
    style CA fill:#ffc107,stroke:#000,color:#000
```

**Chi phí TLS (Overhead):**
1. Một giao dịch thiết lập đường ống HTTPS/TLS vắt kiệt CPU của cả 2 phía để tính toán giải mã RSA/Elliptic.
2. Quá trình bay đi bay về của lời chào (Handshake) cướp đi thêm 2 vòng RTT (Round Trip Time). Nếu mạng xa, Server sẽ bị chậm đi hàng trăm mili-giây.

**Ứng dụng thực tế cho Data Engineering:** 
- Đừng bao giờ bắt các Node Spark/Kafka CỨ MỖI LẦN GỬI 1 MESSAGE lại phải Bắt tay tạo 1 ống TLS mới. Đó là thảm họa sập CPU.
- Các thư viện chuẩn đều ứng dụng kỹ thuật **Connection Pooling (Hồ Chứa Kết Nối)**: Một ống mạng ngầm TLS sau khi mở xong sẽ được giữ trạng thái (Keep-Alive) vĩnh viễn. Các Request/Bytes truyền sau đó cứ thế tống qua ống bảo mật này, tận dụng tốc độ Mã hóa Đối xứng (AES) mà không tốn công phải dạo đầu "Handshake" lại. 

Ở bài học cuối cùng, chúng ta sẽ mở rộng tư duy mật mã này áp dụng vào bài toán lớn nhất của phân tán hệ thống: Làm thế nào cấp Thẻ Chứng Minh Thư (Xác thực) cho 1000 Server để chúng có quyền truy cập chéo vào nhau?

---
**Navigation:**
[⬅️ Previous: Bài 10: Sự tiến hóa HTTP và Kiến trúc Data Pipeline với gRPC](./10-http-evolution-and-grpc.md) | [Next: Bài 12: Xác thực Bảo mật Mạng: JWT, OAuth2 và Kiến trúc Lõi Kerberos ➡️](./12-authentication-oauth2-and-kerberos.md)
