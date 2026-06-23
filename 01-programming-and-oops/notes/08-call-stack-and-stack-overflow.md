# Bài 8: Cơ chế Ngăn xếp (Call Stack) và Lỗi tràn Stack

**Stack (Ngăn xếp)** là một trong những cơ chế hạ tầng quan trọng nhất của mọi ngôn ngữ lập trình, đảm bảo việc các chương trình xử lý theo tuần tự và ghi nhớ ngữ cảnh trở về một cách ổn định, đặc biệt là khi triển khai các quy trình gọi lồng nhau (Nested calls) hoặc đệ quy (Recursion).

Bộ nhớ Stack hoạt động đúng theo cấu trúc dữ liệu cùng tên trong toán học tổ hợp: **LIFO (Last In, First Out – Vào Sau, Ra Trước)**. Vùng dữ liệu đưa vào cuối cùng sẽ là dữ liệu được giải phóng đầu tiên.

---

## 1. Khung Gọi Hàm (Stack Frame / Activation Record)

Khi chương trình đi vào một thân hàm bất kỳ, hệ điều hành tạo ra một cấu trúc bộ nhớ bao đóng gọi là **Stack Frame (Khung Ngăn Xếp)** để thiết lập môi trường hoạt động độc lập cho hàm đó. Khung này chứa các phần tử chức năng:
1. **Tham số hàm (Arguments):** Dữ liệu truyền từ bên ngoài vào hàm.
2. **Biến cục bộ (Local Variables):** Vùng nhớ ngắn hạn khai báo nội bộ (ví dụ: `int i = 0;`).
3. **Địa chỉ trả về (Return Address):** Lưu trữ chính xác vị trí con trỏ lệnh (Instruction Pointer) mà CPU cần quay lại thực thi sau khi hoàn thành hàm.

Quá trình đẩy (Push) một Frame mới đè lên đỉnh Stack diễn ra khi có lời gọi hàm. Khung ở vị trí đỉnh cấu thành **Ngữ cảnh Thực thi (Execution Context)** hiện thời.

Khi hàm thực thi kết thúc vòng đời của mình (gặp lệnh `return` hoặc ngoặc nhọn kết thúc `}`), Khung Ngăn Xếp của nó sẽ bị Gỡ (Pop) khỏi Stack, tự động giải phóng toàn bộ không gian biến cục bộ và định tuyến CPU phục hồi công việc ở địa chỉ trả về tương ứng của Khung bên dưới.

```mermaid
graph TD
    subgraph Kiến trúc Call Stack
        F3[Stack Frame: Hàm `sum(a, b)`<br/>Biến cục bộ, Địa chỉ trả về]
        F2[Stack Frame: Hàm `calculate()`<br/>Đang chờ `sum` tính toán]
        F1[Stack Frame: Hàm `main()`<br/>Điểm khởi tạo tiến trình]
    end
    
    F1 --- F2 --- F3
    
    style F3 fill:#28a745,color:#fff
```

### Tại sao Stack Memory lại có tốc độ xử lý "tuyệt đối"?
Hiệu năng đọc/ghi của bộ nhớ Stack bỏ xa Heap Memory là do thiết kế tuyến tính và quản lý bằng con trỏ phần cứng. Các thanh ghi quản lý Stack (Stack Pointer Register) trên CPU chỉ cần dịch chuyển tịnh tiến bằng các phép cộng trừ theo hằng số Byte cố định (offset) để nạp Frame hoặc hủy bỏ biến cục bộ. Không cần tra cứu liên kết, không bị phân mảnh (Fragmentation).

---

## 2. Phân tích Hiện tượng Stack Overflow

Bộ nhớ Stack có giới hạn vật lý rất hạn hẹp so với Heap (thường chỉ mặc định khoảng 1MB trên Windows hoặc 8MB trên kiến trúc Linux). Sự bảo thủ trong thiết lập này đến từ thực tế rằng Stack chỉ dành riêng cho cấu trúc điều hướng hàm, không được thiết kế làm nơi trữ liệu dung lượng lớn.

**Lỗi Tràn Ngăn Xếp (Stack Overflow Error)** là trạng thái bất thường khi phần mềm phát sinh quá trình đẩy (Push) liên tục các Frame mới làm kiệt quệ dung lượng khả dụng của không gian Stack Memory, đẩy thanh ghi con trỏ vượt ra khỏi ranh giới địa chỉ hợp lệ.

### Phân tích hệ thống: Các nguyên nhân phát sinh lỗi
1. **Đệ quy vô hạn (Infinite Recursion):** 
   Khi một hàm gọi chính nó mà không triển khai cấu trúc Điểm dừng biên (Base Case) hợp lệ. Hàm liên tục tạo ra hàng ngàn Stack Frame chất đống cho tới khi hệ thống sụp đổ (Crash).
   
2. **Khai báo mảng lớn định dạng cục bộ:**
   Việc khởi tạo tĩnh mảng kích thước khổng lồ bên trong thân hàm, ví dụ `double big_array[1000000];`, sẽ ra lệnh cho trình biên dịch chiếm dụng một khoảng không khổng lồ trên Stack Frame, gây cạn kiệt ranh giới ngay từ khung thứ nhất.

### Giải pháp kỹ thuật và Tối ưu (Tail Call Optimization)
Sự tồn tại của rủi ro trên yêu cầu các kỹ sư phải hiểu rõ phạm vi (Scope) của tài nguyên. 
- Mọi dữ liệu lớn (Array, Object) bắt buộc phải được điều hướng sang lưu trữ trên Heap thông qua phân bổ động, chỉ để lại một đường dẫn tham chiếu (Pointer) trên bộ nhớ Stack nội bộ. 
- Với đệ quy sâu, các thuật toán cần được thiết kế lại theo hướng Vòng lặp (Iterative approaches) hoặc khai thác các trình biên dịch hiện đại hỗ trợ tính năng **Tối ưu Hóa Gọi Đuôi (Tail Call Optimization - TCO)**. TCO tái sử dụng trực tiếp một Stack Frame hiện tại cho lời gọi kế tiếp, hạn chế hoàn toàn hiện tượng cạn kiệt bộ nhớ cho các hàm đệ quy chuỗi cơ bản.

---
**Navigation:**
[⬅️ Previous: Bài 7: Tổng quan Mô hình Bộ nhớ (Memory Layout) của Tiến trình](./07-memory-segments.md) | [Next: Bài 8: Ngăn xếp Gọi hàm (The Call Stack) ➡️](./08-the-call-stack.md)
