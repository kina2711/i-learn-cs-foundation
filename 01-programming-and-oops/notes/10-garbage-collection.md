# Bài 10: Quản lý Bộ nhớ tự động (Garbage Collection)

Trước hệ quả nghiêm trọng của việc Rò rỉ bộ nhớ (Memory Leak) ở ngôn ngữ C/C++, nơi các kỹ sư phải tự tay thiết kế và tính toán thủ công từng lệnh khởi tạo `malloc` và giải phóng `free`, các kiến trúc phần mềm hiện đại (Java, Python, C#, Go) đã dịch chuyển theo hướng trừu tượng hóa toàn bộ trách nhiệm Quản lý Bộ nhớ Động cho trình thực thi (Runtime Environment).

Cơ chế đó được chuẩn hóa với thuật ngữ **Garbage Collection (Bộ Thu Gom Rác - GC)**.

---

## 1. Cơ chế Hoạt động Cốt lõi của GC

Bản chất thuật toán của mọi GC đều xoay quanh một nhiệm vụ duy nhất: Xác định đâu là "Rác" và Thu hồi không gian bộ nhớ của nó, đưa các khối dữ liệu trả lại cho Pool trống của Heap Memory.

Thuật ngữ "Rác" (Garbage) mô tả các đối tượng dữ liệu được khởi tạo trên Heap nhưng hiện không còn bất kỳ liên kết tham chiếu (Reference Pointer) đang kích hoạt nào từ cấu trúc hoạt động của ứng dụng (Root Pointers từ Stack hoặc Global Context) trỏ tới chúng. Chúng bị cô lập hoàn toàn và trở nên vô giá trị đối với hệ thống tính toán.

Có hai trường phái thuật toán chính trong thiết kế bộ thu gom rác:

### Trường phái 1: Thuật toán Đếm Tham Chiếu (Reference Counting)
*(Ứng dụng nổi bật: Python, PHP, Objective-C/Swift với mô hình ARC).*

Hệ thống thiết lập một biến số nguyên ngầm định đính kèm cùng với cấu trúc của mỗi Object. Bộ mã hóa sẽ phân tích vòng đời tham chiếu:
- Bất kỳ khi nào một con trỏ được khởi tạo tham chiếu tới Object, bộ đếm tăng lên 1 (VD: `ref_count = 1`).
- Khi biến con trỏ đi khỏi ranh giới hoạt động của hàm (Out of Scope), bộ đếm giảm đi 1.
- Nếu `ref_count` chạm mức `0`, hệ thống xử lý thu hồi tức thời khối bộ nhớ đó trong cùng một chu kỳ nhịp, đảm bảo hiệu suất phản hồi nhanh.

**Hạn chế hệ thống:** Thuật toán thất bại khi phải đối mặt với **Tham chiếu vòng (Circular Reference)**. Khi đối tượng A giữ tham chiếu tới B, và B giữ liên kết tới A, dù cả cụm cấu trúc mất liên kết với Root, `ref_count` của cả hai vĩnh viễn giới hạn mức `1`. Python phải giải quyết lỗi này bằng việc bổ sung định kỳ một bộ thu gom dò vòng lặp.

### Trường phái 2: Thuật toán Truy vết (Tracing GC / Mark-and-Sweep)
*(Ứng dụng nổi bật: Java Virtual Machine (JVM), C# .NET, Go, V8 JavaScript).*

Thuật toán này không tính toán liên tục, thay vào đó thực hiện xử lý đồng bộ theo chu kỳ (Cycle):
1. **Pha Đánh Dấu (Marking):** GC đình chỉ quy trình tạm thời, duyệt toàn bộ sơ đồ cây các biến gốc (Root set variables) đang tồn tại ở Stack và Data Segment. Mọi Object được quét theo chuỗi liên kết và đánh dấu cờ `Reachable` (Có thể với tới). 
2. **Pha Quét (Sweeping):** Dò lại phân vùng Heap. Những object nào không có trạng thái `Reachable` sẽ bị đưa vào quy trình thu hồi không gian bộ nhớ.

```mermaid
graph TD
    subgraph GC Roots (Stack Memory)
        R1[Main Thread Pointer]
        R2[Static Variable]
    end
    
    subgraph Heap Memory
        O1[Object A<br>Mark: Đang sử dụng]
        O2[Object B<br>Mark: Đang sử dụng]
        O3[Object C<br>Mark: Unreachable]
        O4[Object D<br>Mark: Unreachable]
    end
    
    R1 -.-> O1
    O1 -.-> O2
    R2 -.-> O2
    
    O3 -. "Lỗi Tham chiếu vòng" .-> O4
    O4 -.-> O3
    
    style O3 fill:#f8d7da,stroke:#dc3545
    style O4 fill:#f8d7da,stroke:#dc3545
```

---

## 2. Phân tích chi phí: Hiệu ứng Stop-The-World

Sự ra đời của GC giải phóng sức sáng tạo và ngăn chặn những lỗi kiến trúc nghiêm trọng. Tuy nhiên, nó đánh đổi bằng một giới hạn thiết kế cố hữu ảnh hưởng đến các hệ thống hiệu năng tương tác thời gian thực (Real-time).

Để tránh việc dữ liệu bị dịch chuyển và tham chiếu bị ngắt giữa chừng, ở giai đoạn Tracing GC hoạt động, tiến trình ảo sẽ bắt buộc toàn bộ Thread đang xử lý nghiệp vụ chính phải đình trệ trong vài mili-giây hoặc hơn. Thuật ngữ này được biết đến là **Sự kiện Stop-The-World (STW)**.

Trong các nền tảng có lượng dữ liệu cực lớn, độ trễ STW có thể gây tụt khung hình đồ họa (Frame drop trong Game) hoặc tắc nghẽn luồng xử lý giao dịch máy chủ (Server Latency Spikes). Vì rào cản này, các ngôn ngữ cấp thấp ưu tiên tối ưu tài nguyên tuyệt đối (C/C++, Rust) thường từ chối việc vận hành cùng Tracing GC ngầm định.

**Hướng đi nâng cao:**
Các kỹ sư phần mềm hệ thống liên tục nâng cấp kiến trúc GC qua nhiều thế hệ nhằm tối ưu hóa chi phí thời gian dọn rác, ví dụ cấu trúc **Generational GC** (Thu gom theo Hệ sinh thái phân mảnh: Vùng Eden/Young cho object ngắn hạn và Vùng Tenured cho cấu trúc dài hạn, nhằm thu nhỏ phạm vi phân vùng quét trong hệ sinh thái Java). Việc hiểu sâu sắc kiến trúc GC cũng như tối giản chu trình tái sử dụng (Object Pools) là mấu chốt để duy trì ổn định đối với các hệ thống mở rộng theo chiều dọc (Vertical Scaling).

---
**Navigation:**
[⬅️ Previous: Bài 9: Vùng nhớ Động và Phân mảnh bộ nhớ (The Heap)](./09-the-heap-memory.md) | [Next: Bài 11: Bản chất của Class và Cơ chế hoạt động của con trỏ "this" ➡️](./11-classes-and-hidden-pointers.md)
