# Bài 5: Cấu trúc Trạng thái: Ngăn xếp (Stack) và Hàng đợi (Queue)

Mảng và Danh sách liên kết cung cấp nền tảng vật lý cho không gian lưu trữ. Dựa trên các nền tảng vật lý này, Khoa học Máy tính định nghĩa nên các Cấu trúc dữ liệu trừu tượng (Abstract Data Types - ADT) thiết lập các tập quy tắc nghiêm ngặt về chu trình tiếp nạp dữ liệu.

Hai cấu trúc quan trọng nhất đảm nhận vai trò định tuyến dòng chảy luồng thông tin hệ thống là **Ngăn xếp (Stack)** và **Hàng đợi (Queue)**.

---

## 1. Ngăn xếp (Stack) - Nguyên lý LIFO

Stack tuân thủ triết lý hoạt động **LIFO (Last-In, First-Out: Vào Cuối, Ra Đầu)**. Mọi thao tác thêm dữ liệu mới (Push) hay lấy dữ liệu ra (Pop) đều bị giới hạn chỉ diễn ra tại một đầu duy nhất (gọi là Đỉnh - Top).

Về mặt độ phức tạp hiệu năng, bất kể thao tác `Push(x)`, `Pop()`, hay `Peek()` đều được hệ thống thực thi theo thời gian hằng số $O(1)$. 
Mảng (Array) hay Danh sách liên kết (Linked List) đều có khả năng thiết lập (implement) cơ chế Stack. Tuy nhiên, trong C/C++ tĩnh, Array implementation có nguy cơ bị lỗi tràn (Stack Overflow) nếu đầy.

### Ứng dụng Kiến trúc của Stack

Stack không chỉ là công cụ tính toán toán học, mà nó còn là nền móng phần cứng của bộ vi xử lý và hệ điều hành:

1. **Bộ nhớ Execution Stack (Call Stack):** Như đã phân tích tại Bài 8 (Part 1), luồng thực thi của ngôn ngữ bậc cao phụ thuộc hoàn toàn vào cấu trúc LIFO để nhớ điểm neo quay về của các hàm lồng nhau.
2. **Phân tích Cú pháp (Parsing & Balancing):** Trình biên dịch sử dụng Stack để đối chiếu đóng mở ngoặc `()` hay `{}`, rà soát cấu trúc thẻ HTML/XML. Mỗi khi mở thẻ, đẩy vào Stack. Khi gặp thẻ đóng, lấy ra từ đỉnh Stack để đối soát tính hợp lệ. Khác biệt thông số dẫn đến Lỗi cú pháp (Syntax Error).
3. **Mô hình Undo/Redo:** Trạng thái của các ứng dụng Text Editor được ghi lại dưới dạng cấu trúc Stack. 

---

## 2. Hàng đợi (Queue) - Nguyên lý FIFO

Queue tuân thủ định dạng luồng thông tin thực tế hơn: **FIFO (First-In, First-Out: Vào Trước, Ra Trước)**. Các phần tử xếp thành hàng: thao tác thêm dữ liệu (Enqueue) nằm ở phía Đuôi (Rear/Tail), thao tác xuất dữ liệu (Dequeue) diễn ra ở phía Đầu (Front/Head).

Cơ chế này phức tạp hơn Stack khi mô phỏng bằng Mảng cấu trúc. Việc lấy dữ liệu tại phần Đầu của mảng yêu cầu di dời chuỗi dữ liệu (Chi phí $O(N)$), vì vậy khi áp dụng Mảng làm bộ đệm Queue, Lập trình viên phải kiến tạo **Hàng đợi Vòng (Circular Queue)** để bảo toàn cấu trúc vòng lặp $O(1)$. Hoặc đơn giản là dùng Linked List.

### Ứng dụng Kiến trúc của Queue

Queue áp dụng trong trường hợp hệ thống mất kiểm soát năng lực xử lý bất đối xứng (Asynchronous Systems):
1. **Luồng sự kiện Hệ điều hành (OS Scheduling):** Quản trị danh sách các tiến trình ngầm định xin quyền truy cập CPU.
2. **Hệ thống hàng đợi Thông điệp (Message Brokers):** Các nền tảng như RabbitMQ, Apache Kafka (nền tảng của hệ thống Microservices) sử dụng cấu trúc Queue khổng lồ trong RAM để xếp hàng luồng gửi mail, nén video... tránh làm sụp đổ máy chủ phân quyền trong các chiến dịch tải lượng lớn (Traffic Spikes).
3. **Mô phỏng Giao tiếp Thiết bị (I/O Buffers):** Bàn phím hay luồng in tài liệu lưu tập tin thao tác trong cấu trúc Queue.

---

## 3. Biến thể Mở rộng: Hàng đợi Ưu tiên (Priority Queue)

Có những mô hình thông tin yêu cầu bỏ qua quy luật thời gian nạp và xử lý tập trung vào Tính Ưu tiên (Priority) của tài nguyên. Khi hệ điều hành Windows bị quá tải, nếu một tín hiệu gián đoạn (Interrupt Signal) từ cấu trúc phần cứng gửi đến, nó không phải xếp hàng ở phía sau Queue mà được đặc cách xử lý vượt cấp.

**Cấu trúc Priority Queue:**
- Dữ liệu luôn có khóa ưu tiên đi kèm (Ví dụ: Cấp độ 1, Cấp độ 5).
- Thao tác Lấy dữ liệu (Dequeue) không rút dữ liệu cũ nhất, mà rút dữ liệu có đặc quyền Ưu tiên cao nhất.
- Hàng đợi ưu tiên không dựa trên cấu trúc Mảng tĩnh thuần túy (việc liên tục chèn giữa sẽ tốn $O(N)$), mà được cắm ghép logic bởi nền tảng cấu trúc **Cây Heap (Max-Heap hoặc Min-Heap)**. Nền tảng này đảm bảo quá trình chèn hoặc loại bỏ được duy trì với mức trễ thuật toán rất nhỏ là $O(\log N)$. Tính chất của cấu trúc Cây Phi tuyến tính này sẽ được tìm hiểu sâu hơn trong Chương 3.

---
**Navigation:**
[⬅️ Previous: Bài 4: Cấu trúc Bộ nhớ Mảng (Arrays) và Danh sách liên kết (Linked Lists)](./04-arrays-and-linked-lists.md) | [Next: Bài 6: Bảng băm (Hash Tables) và Thuật toán tra cứu $O(1)$ ➡️](./06-hash-tables.md)
