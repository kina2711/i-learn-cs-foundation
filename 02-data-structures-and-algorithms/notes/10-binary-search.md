# Bài 10: Tìm kiếm Nhị phân (Binary Search)

Các hoạt động tìm kiếm (Searching) chiếm tới 70% khối lượng tải xử lý của các hệ thống truy vấn và cơ sở dữ liệu trên thế giới. Trong Khoa học Máy tính, một trong những thuật toán cơ sở đóng vai trò là xương sống cho tính năng tìm kiếm tốc độ cao là **Tìm kiếm Nhị phân (Binary Search)**.

Đây là phương thức nền tảng giúp khai phá cấp độ phức tạp $O(\log N)$, chia rẽ sức mạnh phần cứng giữa khối lượng xử lý Tuyến tính và khối lượng xử lý Logarit.

---

## 1. Điều kiện tiên quyết: Tính có Trật tự (Sorted Property)

Bản thân thuật toán Tìm kiếm Nhị phân không thể áp dụng trên một dãy dữ liệu thô (Raw Data). **Điều kiện ràng buộc duy nhất và bắt buộc để Binary Search hoạt động là Tập dữ liệu đầu vào phải được sắp xếp theo một trật tự xác định (Tăng dần hoặc Giảm dần)**.

Giả sử ta được yêu cầu tìm số `37` trong mảng dữ liệu có $N = 1,000,000$ bản ghi.
- Nếu mảng không có trật tự, thuật toán mặc định của CPU là **Tìm kiếm Tuyến tính (Linear Search)**: Duyệt từng số từ 0 đến 1,000,000. Hiệu suất $O(N)$.
- Nếu mảng đã có trật tự tăng dần, ta có thể kích hoạt Binary Search.

---

## 2. Giải thuật Phân đoạn (Divide Strategy)

Binary Search vận hành dựa trên nguyên tắc không ngừng khoanh vùng không gian tìm kiếm, loại bỏ một nửa khối lượng dữ liệu ở mỗi chu kỳ lặp.

**Mô hình luồng lệnh:**
1. Khởi tạo hai con trỏ ranh giới: `Left = 0` (Bắt đầu mảng) và `Right = N - 1` (Kết thúc mảng).
2. Vi xử lý tính toán vị trí Nút trung tâm: `Mid = (Left + Right) / 2`.
3. Trích xuất giá trị tại `Mid` và tiến hành so sánh logic với giá trị cần tìm `Target` (Ví dụ là số 37).
   - *Khả năng A:* Nếu `Array[Mid] == Target`, tìm kiếm kết thúc với kết quả hoàn hảo ở vị trí `Mid`.
   - *Khả năng B:* Nếu `Array[Mid] < Target` (Ví dụ giá trị giữa bằng 20), do mảng tăng dần, kỹ sư khẳng định chắc chắn 100% số `37` không thể nằm từ `Left` đến `Mid`. Khoanh vùng tìm kiếm thay đổi: `Left = Mid + 1`. Không gian mảng bên trái đã bị loại bỏ hoàn toàn.
   - *Khả năng C:* Nếu `Array[Mid] > Target`, loại bỏ nhánh phải: `Right = Mid - 1`.
4. Chu kỳ tìm kiếm lặp lại quá trình tính `Mid` trên phân đoạn mới bị thu hẹp cho đến khi `Left` vượt quá `Right` (Định nghĩa là Dữ liệu không tồn tại).

```mermaid
graph TD
    subgraph Thuật toán Loại trừ Binary Search
        M[Đầu vào 1 triệu bản ghi] -->|Lần lặp 1| M1[500,000 bản ghi]
        M1 -->|Lần lặp 2| M2[250,000 bản ghi]
        M2 -->|Lần lặp 3| M3[125,000 bản ghi]
        M3 -.->|Chỉ 20 lần lặp tối đa| M4[Tìm ra đúng bản ghi]
    end
    
    style M fill:#d1ecf1,stroke:#17a2b8
    style M4 fill:#28a745,color:#fff
```

**Tính toán độ chênh lệch Hiệu suất:**
Với 1 triệu bản ghi, Linear Search yêu cầu cấp phát $1,000,000$ lệnh so sánh. 
Với Binary Search, CPU thực thi chu kỳ: $1,000,000 \rightarrow 500,000 \rightarrow 250,000 \rightarrow \dots \rightarrow 1$. Hàm giảm trừ này tính theo tỷ lệ $\log_2(1,000,000) \approx 20$. 
Hệ điều hành đã tiết kiệm được $999,980$ chu kỳ tính toán, biểu diễn sức mạnh khủng khiếp của hàm Logarit.

---

## 3. Rủi ro về Số nguyên Tối đa (Integer Overflow)

Mặc dù giải thuật có vẻ toàn diện, phiên bản cài đặt trên thực tế từng xuất hiện một Lỗi Kiến trúc (Bug) kinh điển tổn tại suốt 20 năm trong bộ thư viện lõi của Java (Và nhiều ngôn ngữ tĩnh khác như C).

Công thức tìm điểm trung tâm gốc:
```java
int mid = (left + right) / 2;
```

**Sự cố giới hạn bộ nhớ:**
Khởi tạo biến `int` thường được gán cho kích thước giới hạn vật lý 32-bits (Maximum $\approx 2.14$ tỷ). 
Giả sử ta có một mảng khối lượng lớn với $N = 1.5$ tỷ. Tại những chu kỳ tìm kiếm cuối cùng ở nửa cuối danh sách, giả sử `left = 1.2` tỷ và `right = 1.4` tỷ. 
Phép cộng cơ học `left + right = 2.6` tỷ (Vượt ngưỡng 2.14). Hệ điều hành sẽ xử lý tràn số (Overflow), biến 2.6 tỷ trở thành một số âm giả mạo (`-xxx`). Khi chia 2, biến `mid` nhận một số âm, dẫn tới việc chỉ định `Array[-xxx]` gây Sập bộ nhớ (Segmentation Fault).

**Giải pháp an toàn cấu trúc:**
Để khắc phục, các nhà thiết kế thuật toán phải tiến hành bù trừ khoảng cách thay vì cộng trực tiếp, tuân thủ công thức an toàn tiêu chuẩn toàn cầu hiện tại:
```java
int mid = left + (right - left) / 2;
```
Bằng cách phân giải phép chia trước khi kết hợp độ dời, thuật toán đảm bảo việc cấp phát bộ đệm số nguyên không bao giờ vượt qua cấu trúc mảng gốc. Nguyên lý này được tích hợp vào toàn bộ cấu hình lõi của các thư viện API tìm kiếm trên toàn thế giới hiện tại.

---
**Navigation:**
[⬅️ Previous: Bài 9: Phi Tuyến tính Bậc cao: Đồ thị (Graphs)](./09-graphs.md) | [Next: Bài 11: Phân tích Thuật toán Sắp xếp Cơ bản (Basic Sorting Algorithms) ➡️](./11-basic-sorting.md)
