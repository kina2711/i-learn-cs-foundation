# Bài 1: Khái niệm Độ phức tạp Thời gian và Không gian (Time & Space Complexity)

Trong kỹ thuật phần mềm, một bài toán có thể được giải quyết bằng nhiều phương pháp thuật toán khác nhau. Tuy nhiên, trong môi trường hệ thống thực tế (đặc biệt đối với các hệ thống quy mô lớn như cơ sở dữ liệu hay máy chủ truy vấn), sự khác biệt giữa một thuật toán tốt và một thuật toán kém có thể là ranh giới giữa việc xử lý yêu cầu trong 10 mili-giây và việc làm treo toàn bộ máy chủ (Server Crash) do cạn kiệt tài nguyên.

Để đánh giá và so sánh sự hiệu quả của các thuật toán một cách khách quan, bất kể thông số kỹ thuật của phần cứng (CPU, RAM) hay ngôn ngữ lập trình, Khoa học Máy tính sử dụng hai chỉ số đo lường cơ bản: **Độ phức tạp Thời gian (Time Complexity)** và **Độ phức tạp Không gian (Space Complexity)**.

---

## 1. Bản chất của việc Đo lường

Tại sao chúng ta không đo lường thời gian thực thi thuật toán bằng số giây (Ví dụ: 0.5 giây, 2 giây)?

Câu trả lời nằm ở tính không nhất quán của môi trường vật lý. Nếu chạy cùng một thuật toán sắp xếp 1 triệu phần tử:
- Trên một máy tính cá nhân cấu hình thấp, nó có thể mất 10 giây.
- Trên hệ thống Siêu máy tính hoặc một cụm AWS EC2, nó có thể chỉ mất 0.1 giây.
- Cùng một máy tính, thời gian chạy vào ban đêm có thể khác ban ngày do sự điều tiết tài nguyên của Hệ điều hành (OS Scheduling).

Do đó, Khoa học Máy tính cần một đại lượng đo lường mang tính **Tiệm cận toán học (Asymptotic Analysis)**. Đại lượng này không phản ánh số giây đồng hồ đếm được, mà phản ánh **Tốc độ tăng trưởng của thời gian xử lý (hoặc dung lượng bộ nhớ) tỷ lệ thuận như thế nào khi khối lượng dữ liệu đầu vào ($N$) tiến tới vô cùng lớn.**

---

## 2. Độ phức tạp Thời gian (Time Complexity)

Độ phức tạp Thời gian biểu thị **số lượng thao tác tính toán cơ sở (Basic Operations)** mà một thuật toán cần thực thi tương ứng với kích thước dữ liệu đầu vào.

Các thao tác tính toán cơ sở thường được Vi xử lý (CPU) hoàn thành trong một hoặc một vài chu kỳ xung nhịp (Clock cycles), bao gồm:
- Phép toán số học (Cộng, Trừ, Nhân, Chia).
- Gán giá trị biến.
- Đánh giá điều kiện logic (So sánh lớn bé, IF).
- Khởi tạo con trỏ và truy xuất mảng dựa trên chỉ mục (Index).

**Mô hình phân tích:**
Giả sử ta có thuật toán in ra màn hình toàn bộ các số từ mảng $A$ có độ dài là $N$.
```java
void printArray(int[] A) {
    for (int i = 0; i < A.length; i++) { // Vòng lặp chạy N lần
        System.out.println(A[i]);        // Thao tác in (coi như cơ sở)
    }
}
```
Khi $N = 10$, hệ thống cần 10 thao tác in. Khi $N = 10,000,000$, hệ thống cần 10,000,000 thao tác. Ta có thể xác định định lượng toán học của thuật toán này là Thời gian thực thi tăng trưởng trực tiếp theo tỷ lệ tuyến tính với $N$. 

Bằng cách loại bỏ cấu hình phần cứng, Kỹ sư có thể khẳng định chắc chắn rằng thuật toán $O(\log N)$ luôn vượt trội hơn thuật toán $O(N^2)$ khi khối lượng dữ liệu đủ lớn.

---

## 3. Độ phức tạp Không gian (Space Complexity)

Độ phức tạp Không gian đánh giá **lượng bộ nhớ bổ sung (Extra Memory / Auxiliary Space)** mà thuật toán cần phân bổ từ RAM (Heap hoặc Stack) trong quá trình thực thi, tính theo hàm tỷ lệ với lượng dữ liệu đầu vào $N$.

Phân biệt thuật ngữ:
- **Kích thước dữ liệu gốc (Input Size):** Nếu bài toán yêu cầu xử lý mảng 1GB, thì mảng 1GB này là không gian dữ liệu bắt buộc (Tồn tại sẵn độc lập với thuật toán).
- **Bộ nhớ phụ trợ (Auxiliary Space):** Là lượng dung lượng thuật toán **chiếm dụng thêm tạm thời** để hỗ trợ quá trình giải bài toán (Ví dụ: tạo thêm một mảng copy, cấp phát biến tạm, hoặc kích thước Ngăn xếp gọi hàm khi sử dụng Đệ quy).

Khi nói về Độ phức tạp không gian của một thuật toán, chúng ta thường đề cập tới khối lượng Bộ nhớ phụ trợ này.

**Ví dụ phân tích:**
```java
// Thuật toán Không gian hằng số O(1)
int sumArray(int[] A) {
    int total = 0; // Bộ nhớ phụ trợ duy nhất là biến total (4 bytes)
    for (int num : A) {
        total += num;
    }
    return total;
}
```
Bất kể mảng $A$ dài 10 phần tử hay 1 tỷ phần tử, bộ nhớ RAM yêu cầu thêm để chạy hàm này vẫn chỉ là 4 bytes cho biến `total`. Do đó, thuật toán này có Độ phức tạp Không gian tuyệt vời.

---

## 4. Sự Đánh đổi (Time-Space Trade-off)

Trong thiết kế hệ thống phần mềm, một định luật kiến trúc phổ biến là **Sự đánh đổi giữa Thời gian và Không gian (Time-Space Trade-off)**. Rất hiếm khi một thuật toán tối ưu hóa được cả hai đại lượng xuống mức tối thiểu cùng lúc.

- **Dùng RAM để đổi lấy Tốc độ CPU:** Đây là nguyên lý hình thành nên bộ nhớ Đệm (Cache). Ta tốn thêm một khoảng lớn bộ nhớ RAM (Space) để lưu trữ kết quả của một phép tính phức tạp, nhằm giúp CPU không phải tính toán lại (Giảm Time) ở các vòng lặp tiếp theo. Cơ sở dữ liệu sử dụng cấu trúc Indexing (B-Trees) cũng tiêu tốn dung lượng ổ đĩa lớn để đổi lấy tốc độ truy vấn nháy mắt.
- **Dùng CPU để tiết kiệm RAM:** Khi các thiết bị nhúng hoặc vi mạch IoT bị giới hạn cực đoan về RAM, hệ thống buộc phải nén dữ liệu. Khi cần truy cập, CPU phải mất thêm hàng ngàn xung nhịp (Time) để giải nén lại đoạn dữ liệu đó nhằm lấy ra một thuộc tính.

Vai trò của một kỹ sư không phải luôn chọn thuật toán có Thời gian nhanh nhất, mà là phân tích năng lực nền tảng (Platform capabilities) hiện có và đưa ra cấu hình Thuật toán trung hòa tối ưu nhất. Ở bài học tiếp theo, chúng ta sẽ mã hóa các cấp độ thời gian này dưới dạng Ký pháp Big O.

---
**Navigation:**
[Next: Bài 2: Phân tích Ký pháp Big O (Big O Notation) ➡️](./02-big-o-notation.md)
