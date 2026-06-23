# Bài 9: Vùng nhớ Động và Phân mảnh bộ nhớ (The Heap)

Chúng ta đã biết Stack cực kỳ nhanh và tự động dọn dẹp. Vậy tại sao ngôn ngữ lập trình không nhét toàn bộ mọi thứ vào Stack cho khỏe?
Câu trả lời là: **Stack bị OS giới hạn cực nhỏ (1MB - 8MB)** và **Thời gian sống của nó quá ngắn (Hàm kết thúc là mất sạch)**.

Giả sử bạn làm một Game, bạn load tấm bản đồ nặng 1 Gigabyte vào RAM, và bạn muốn bản đồ này TỒN TẠI XUYÊN SUỐT qua hàng trăm hàm khác nhau. Nếu nhét bản đồ vào Stack, game crash ngay lập tức vì `StackOverflow`.

Đây là lúc **Heap (Đống)** xuất hiện.

---

## 1. Heap là gì? Sự hoang dã của bộ nhớ

> [!TIP]
> **ELI5:** Hãy tưởng tượng **Stack** như một tủ đồ có nhiều ngăn kéo nhỏ (có OS xếp sẵn gọn gàng). Còn **Heap** là một cái bãi đất trống khổng lồ (bằng toàn bộ số RAM còn lại của máy). Bạn muốn đặt một căn nhà bự cỡ nào lên bãi đất đó cũng được, miễn là đất còn đủ chỗ.

- **Stack** do Hệ điều hành (OS) quản lý nghiêm ngặt.
- **Heap** là vùng tự do do **Lập trình viên (qua ngôn ngữ lập trình)** trực tiếp thầu và cấp phát.

Trong C/C++, bạn xin đất bằng hàm `malloc(1_Gigabyte)`. OS sẽ tìm trên bãi đất trống một khu vực liền kề vừa đúng 1GB và giao cho bạn.
Trong Java/C#/Python, bạn xin đất bằng từ khóa `new Object()`.

```mermaid
graph TD
    subgraph RAM
        S["Stack Memory<br>Giới hạn: 8MB<br>Lưu: int x, char y, Địa chỉ con trỏ"]
        H["Heap Memory<br>Giới hạn: Tính bằng GBs<br>Lưu: Object, Mảng siêu lớn, Strings"]
    end
    
    S -.->|Pointer (Con trỏ chứa địa chỉ)| H
    style S fill:#ffc107
    style H fill:#28a745
```

**Mối liên hệ giữa Stack và Heap:**
Khi bạn viết trong Java: `User u = new User();`
- `new User()` tạo ra một Object khổng lồ (chứa Name, Email, Avatar) nằm tít ngoài **Heap**.
- Chữ `u` thực chất chỉ là một **Con trỏ (Pointer)** lưu số địa chỉ nhà của Object đó (ví dụ: `0x5F3A`), và con trỏ bé xíu này được lưu trong **Stack**.
- Khi hàm kết thúc, con trỏ `u` trong Stack biến mất (đứt dây diều). Nhưng Object ngoài Heap **vẫn nằm nguyên đó** chờ người đến dọn (Garbage Collector).

---

## 2. Vấn đề cốt tử 1: Memory Leak (Rò rỉ bộ nhớ)

Cái giá của sự tự do là trách nhiệm. Trong C/C++, khi bạn xin đất bằng `malloc()`, OS tin tưởng bạn 100%. Nếu dùng xong bạn quên không trả lại đất bằng lệnh `free()`, mảnh đất đó sẽ bị đánh dấu là "Đang có người xài" mãi mãi.

**Rò rỉ (Leak)** xảy ra khi bạn tạo ra quá nhiều rác trên Heap mà quên dọn.
Ví dụ: Một con Bot chạy ngầm, mỗi giây tải về 1 file cấu hình 1MB (bỏ vào Heap), nhưng quên xóa file cũ.
Sau 1 tiếng đồng hồ, 3.6 GB RAM đã bị cắn sạch. OS không chịu nổi nữa, nó "giết chết" con Bot với thông báo `OOM (Out Of Memory)`.

Sự khốc liệt của C/C++ là lý do Java ra đời năm 1995 với tuyên ngôn: "Để máy ảo (JVM) tự đi dọn rác, lập trình viên chỉ cần tập trung viết logic".

---

## 3. Vấn đề cốt tử 2: Memory Fragmentation (Phân mảnh bộ nhớ)

Nhưng dù cho có GC hay tự dọn dẹp (C++), Heap vẫn phải đối mặt với một vấn đề mang tính chất vật lý: **Phân mảnh**.

Giả sử bãi đất (Heap) của bạn có 100MB trống.
1. Bạn xin 3 cục bộ nhớ (A, B, C), mỗi cục 30MB. Tổng xài 90MB. Còn trống 10MB cuối. (Heap: `[A(30)][B(30)][C(30)][Trống 10]`).
2. Cục B xài xong, bạn dọn dẹp cục B. (Heap: `[A(30)][Trống 30][C(30)][Trống 10]`).
3. Tổng dung lượng trống trên Heap bây giờ là: `30 + 10 = 40MB`.
4. Bây giờ, bạn xin OS cấp cho một mảng bự **35MB**.

**Kết quả: OS báo lỗi Out of Memory!**
Tại sao? Tổng trống là 40MB, tôi chỉ xin 35MB cơ mà?
**Giải thích:** Bộ nhớ cấp phát cho một Mảng (Array/Object) yêu cầu phải là **Bộ nhớ Liền kề (Contiguous Memory)**. Bạn có 1 khoảng trống 30MB và 1 khoảng trống 10MB bị kẹp giữa cục A và C. Không có khoảng trống nào đủ to nguyên khối 35MB để nhét vào. 

Đây gọi là hiện tượng **Phân mảnh ngoại (External Fragmentation)**.

```mermaid
block-beta
  columns 4
  A["Đã xài (30MB)"]:1
  Empty1["Trống (30MB)"]:1
  C["Đã xài (30MB)"]:1
  Empty2["Trống (10MB)"]:1
  
  style A fill:#dc3545,color:#fff
  style C fill:#dc3545,color:#fff
  style Empty1 fill:#d4edda,stroke-dasharray: 5 5
  style Empty2 fill:#d4edda,stroke-dasharray: 5 5
```
*Bạn không thể nhét khối 35MB vào bất kỳ chỗ nào ở trên!*

---

## 🛠️ Góc nhìn Kỹ sư: Đánh đổi hiệu năng

Vì Heap bị phân mảnh tơi tả, khi bạn xin cấp phát bộ nhớ (gọi `new`), Hệ điều hành phải chạy các thuật toán như *First-Fit, Best-Fit, Worst-Fit* để dò tìm từng ngóc ngách xem có chỗ trống nào vừa vặn không. 
Việc này **tốn rất nhiều thời gian CPU** so với việc chỉ việc cộng trừ một nấc con trỏ ở Stack.

Do đó, Luật bất thành văn trong System Design và Game C++:
**Object Pooling (Hồ chứa):** Thay vì liên tục `new` và `delete` hàng ngàn viên đạn trong game (gây phân mảnh cực nặng và giật lag Frame-rate do cấp phát bộ nhớ), hãy khởi tạo một mảng 1000 viên đạn một lần duy nhất lúc Load Game. Viên nào bắn xong thì đem giấu đi (ẩn), khi cần bắn tiếp thì lôi ra xài lại thay vì tạo mới. Kỹ thuật này giúp giải cứu Heap Memory!

---
**Navigation:**
[⬅️ Previous: Bài 9: Cấp phát động, Bộ nhớ Heap và Con trỏ](./09-heap-memory-and-pointers.md) | [Next: Bài 10: Quản lý Bộ nhớ tự động (Garbage Collection) ➡️](./10-garbage-collection.md)
