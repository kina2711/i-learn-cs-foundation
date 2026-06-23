# Bài 14: Tính Đa hình (Polymorphism) và Bảng phương thức ảo (V-Table)

**Đa hình (Polymorphism)** là trụ cột thứ ba của thiết kế Hướng đối tượng, cung cấp năng lực cho phép các đối tượng khác nhau phản ứng theo các phương thức vận hành chuyên biệt của riêng chúng khi cùng nhận một lời gọi hàm chung. Khả năng này hỗ trợ mở rộng mã nguồn tối đa (Scale) khi làm việc với tập dữ liệu các lớp đối tượng phân cấp.

Có hai cấp độ Đa hình tồn tại trong tiến trình xử lý biên dịch: Trạng thái tĩnh (Compile-time) và Trạng thái động (Runtime).

---

## 1. Đa hình Tĩnh (Compile-time): Phương pháp Nạp chồng (Method Overloading)

Đa hình tĩnh diễn ra hoàn toàn trong giai đoạn biên dịch. Cụ thể, một lớp (Class) được phép chứa nhiều phương thức trùng tên, miễn là cấu trúc danh sách tham số truyền vào (số lượng, kiểu dữ liệu) hoàn toàn khác biệt.

Khi lập trình viên phát sinh lời gọi hàm, Trình biên dịch phân tích chữ ký (Signature) của kiểu dữ liệu được cung cấp và tiến hành **Liên kết tĩnh (Static Binding)** để xác định dứt khoát hàm đích nào trong bộ nhớ sẽ được thực thi.

```java
class Calculator {
    // Phiên bản nạp chồng xử lý hai số nguyên
    public int add(int a, int b) { return a + b; }
    
    // Phiên bản nạp chồng xử lý ba số nguyên
    public int add(int a, int b, int c) { return a + b + c; }
    
    // Phiên bản nạp chồng xử lý dấu phẩy động
    public double add(double a, double b) { return a + b; }
}
```

Tính chất này giúp tái sử dụng cấu trúc đặt tên (Naming convention), giảm tải việc phân mảnh định danh (không cần thiết kế các hàm `addTwoInts`, `addThreeInts`).

---

## 2. Đa hình Động (Runtime): Phương pháp Ghi đè (Method Overriding)

Trong khi Đa hình tĩnh vận hành độc lập, Đa hình động là cốt lõi sức mạnh của OOP. Nó liên kết trực tiếp với mô hình Kế thừa, mô phỏng cơ chế một con trỏ kiểu Lớp cha có năng lực dẫn xuất xuống định dạng cụ thể của các Lớp con đa dạng vào thời điểm thực thi lệnh.

Cơ chế này đạt được bằng việc cung cấp các phiên bản logic riêng biệt, thực hiện ghi đè (Override) phương thức định dạng ban đầu của Lớp cha.

```java
class Animal {
    public void speak() { System.out.println("Animal makes a sound"); }
}

class Dog extends Animal {
    @Override
    public void speak() { System.out.println("Gâu Gâu!"); }
}

class Cat extends Animal {
    @Override
    public void speak() { System.out.println("Meo Meo!"); }
}
```

Khả năng phân phối động:
```java
// Upcasting: Biến khai báo kiểu Animal, nhưng khởi tạo một tập hợp các Lớp con đa dạng
Animal[] zoo = { new Dog(), new Cat(), new Animal() };

for (Animal a : zoo) {
    a.speak(); // Hệ thống tự xác định kiểu đối tượng thật và thực thi lời gọi hàm đúng đắn.
}
```
Việc điều hướng mã lệnh tại thời gian chạy lệnh (Runtime) này được gọi là **Liên kết động (Dynamic Dispatch)**. Tuy nhiên, bằng cách nào hệ điều hành và vi xử lý có thể nhận dạng được chính xác logic hàm con cần thực thi giữa muôn vàn Lớp con ẩn dưới định danh của Lớp cha?

---

## 3. Bản chất cơ học: Bảng phương thức Ảo (Virtual Method Table / V-Table)

Dưới góc nhìn kiến trúc hệ thống, để hỗ trợ khả năng Liên kết động, Trình biên dịch xây dựng tự động một cấu trúc chỉ mục đặc biệt được gọi là **V-Table (Bảng phương thức Ảo)**. Bảng này ánh xạ trực tiếp các con trỏ logic từ đối tượng tới địa chỉ vật lý lưu trữ tập lệnh hàm tại phân vùng Text Segment.

Mỗi khi một Lớp khai báo ít nhất một phương thức có thể bị ghi đè (từ khóa `virtual` trong C++, hoặc mặc định tự có trong Java), Trình biên dịch tạo ra một mảng tĩnh chứa các tham chiếu hàm riêng cho Lớp đó. 
Bất kỳ đối tượng Heap Memory nào được nạp bởi Lớp đó cũng sẽ chứa thêm một biến con trỏ hệ thống tàng hình, tên là **`vptr` (Virtual Pointer)**, dẫn hướng thẳng về mảng tĩnh V-Table của Lớp tương ứng.

```mermaid
graph LR
    subgraph Đối tượng trên Heap
        D[Object Dog<br/>+ vptr]
        C[Object Cat<br/>+ vptr]
    end
    
    subgraph Bảng V-Table lưu trong RAM
        VD[V-Table của Dog<br/>+ speak() -> Địa chỉ vùng hàm Gâu Gâu]
        VC[V-Table của Cat<br/>+ speak() -> Địa chỉ vùng hàm Meo Meo]
    end
    
    D -.->|Trỏ đến| VD
    C -.->|Trỏ đến| VC
```

**Quá trình độ trễ:**
Khi thực thi chỉ thị `a.speak()`, CPU không thể đi thẳng vào khối lệnh để tính toán ngay lập tức. Hệ thống bắt buộc phải giải quyết theo tiến trình hai bước:
1. Đọc dữ liệu biến con trỏ nội bộ `vptr` tại cấu trúc Object hiện hành để dò ra phân vùng V-Table của lớp tương thích.
2. Tra cứu (Lookup) trên cấu trúc mảng V-Table để rút trích địa chỉ ô nhớ chứa đoạn mã nguồn thực thi của hàm `speak()` cần thiết, sau đó mới dịch chuyển con trỏ truy cập tới vùng Segment đó.

Hệ quả của tiến trình hai bước này là cơ chế Liên kết động mang tới một mức phí tổn hiệu suất độ trễ nhỏ (Overhead). Trong thiết kế phần mềm với khung tham chiếu đồ họa nhịp độ cực cao (Realtime C++ Game Engines), lập trình viên sẽ hạn chế tối đa việc thiết kế Đa hình nếu không cần thiết, tránh phát sinh cơ chế V-Table gây tắc nghẽn luồng xử lý tại vi xử lý cấp thấp.

---
**Navigation:**
[⬅️ Previous: Bài 13: Tính Kế thừa (Inheritance), Vấn đề Hình thoi và Mô hình Composition](./13-inheritance-and-diamond-problem.md) | [Next: Bài 15: Tính Trừu tượng (Abstraction): Abstract Class vs Interface ➡️](./15-abstraction.md)
