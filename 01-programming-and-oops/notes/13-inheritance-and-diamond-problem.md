# Bài 13: Tính Kế thừa (Inheritance), Vấn đề Hình thoi và Mô hình Composition

Trụ cột thứ hai của OOP là **Tính Kế thừa (Inheritance)**. Đây là cơ chế cho phép một Lớp con (Sub-class / Derived Class) tái sử dụng, kế thừa cấu trúc dữ liệu và logic phương thức từ một Lớp cha (Super-class / Base Class).

Về mặt logic mô hình hóa, Kế thừa thường đại diện cho mối quan hệ **IS-A (Là một)**. Ví dụ: Chó (Dog) *là một* Động vật (Animal). Do đó, cấu trúc `Dog` có quyền thừa hưởng phương thức `eat()` và `sleep()` từ `Animal` mà không cần viết lại mã nguồn.

Tuy nhiên, việc sử dụng Kế thừa cũng đi kèm những thách thức kiến trúc và giới hạn cấu trúc hệ thống.

---

## 1. Giới hạn Kế thừa và Vấn đề Hình thoi (The Diamond Problem)

Ở cấp độ thiết kế cơ bản, ngôn ngữ như C++ cho phép **Đa kế thừa (Multiple Inheritance)**, tức một Lớp con có thể thừa hưởng trực tiếp từ hai hoặc nhiều Lớp cha song song. Sự linh hoạt này dẫn đến một xung đột kỹ thuật nổi tiếng: **The Diamond Problem (Vấn đề Hình thoi)**.

Xét mô hình cấu trúc sau:
1. Có lớp gốc `A` định nghĩa một phương thức `print()`.
2. Lớp `B` và `C` cùng kế thừa từ `A` và cùng thực hiện ghi đè (override) phương thức `print()` theo logic riêng của chúng.
3. Lớp `D` thực hiện Đa kế thừa đồng thời từ `B` và `C`.

```mermaid
graph TD
    A[Lớp A<br/>+ print()] -->|Kế thừa| B[Lớp B<br/>+ print() riêng]
    A -->|Kế thừa| C[Lớp C<br/>+ print() riêng]
    B -->|Đa kế thừa| D[Lớp D]
    C -->|Đa kế thừa| D
    
    style D fill:#f8d7da,stroke:#dc3545
```

**Sự cố kỹ thuật:** Khi đối tượng của lớp `D` gọi phương thức `print()`, Trình biên dịch không thể quyết định dứt khoát rằng Lớp `D` sẽ ưu tiên sử dụng bản gốc của phương thức từ nhánh `B` hay nhánh `C`. Sự phân cực này gây lỗi định tuyến ở thời điểm biên dịch (Compile-time).

Do bản chất rủi ro của vấn đề Hình thoi, hầu hết các ngôn ngữ thế hệ sau (như Java, C#) đã kiến trúc lại toàn bộ hệ thống bằng việc **Nghiêm cấm hoàn toàn Đa Kế Thừa đối với Class (Lớp)**. Mọi lớp chỉ được sở hữu một và chỉ một Lớp cha duy nhất (Single Inheritance).

---

## 2. Dịch chuyển kiến trúc: Composition over Inheritance

Qua thời gian vận hành trong các hệ thống quy mô lớn, kiến trúc sư phần mềm nhận thấy Kế thừa mang theo nhược điểm tiềm ẩn nghiêm trọng: **Sự phụ thuộc cấu trúc cứng nhắc (Tight Coupling)**. 
Bất kỳ sự thay đổi logic nào ở Lớp cha gốc cũng có nguy cơ làm vỡ liên kết và luồng chạy của toàn bộ các Lớp con bên dưới nhánh phân cấp. Hơn nữa, việc phân cấp hệ thống quá sâu (cây kế thừa dài 5-6 tầng) sẽ cản trở hiệu suất truy xuất phương thức và gây phức tạp trong quá trình Unit Test.

Để giải quyết tình trạng này, một nguyên lý thiết kế tiên tiến đã trở thành tiêu chuẩn chung của thiết kế hướng đối tượng: **"Composition over Inheritance" (Ưu tiên Thành phần hơn Kế thừa)**.

### Định nghĩa mô hình Composition
Thay vì triển khai mối quan hệ tĩnh **IS-A** (Kế thừa), mô hình Composition áp dụng sự cấu thành thông qua mối quan hệ **HAS-A (Sở hữu / Chứa đựng)**. Lớp tổng thể không kế thừa từ lớp linh kiện, mà nó lưu trữ các phiên bản đối tượng của lớp linh kiện bên trong biến cấu trúc của mình, và tái sử dụng bằng cách ủy thác thực thi (Delegation).

```java
// Thiết kế cũ: Kế thừa (Kém linh hoạt)
class Engine {
    public void start() {}
}
class Car extends Engine { // Kế thừa không hợp lý: Car KHÔNG PHẢI là Engine.
    // ...
}

// Thiết kế hiện đại: Composition (Linh hoạt, Loose Coupling)
class Engine {
    public void start() {}
}
class Car {
    private Engine engine; // Car SỞ HỮU Engine

    public Car(Engine engine) {
        this.engine = engine; // Truyền đối tượng vào thông qua hàm khởi tạo
    }

    public void drive() {
        this.engine.start(); // Ủy thác (Delegate) công việc cho Engine
        System.out.println("Car is moving...");
    }
}
```

**Ưu điểm của Composition:** 
Vì `Engine` không gắn cứng vào cấu trúc biên dịch của `Car`, ta hoàn toàn có thể thay đổi, khởi tạo đa dạng (Động cơ điện, Động cơ xăng) và cắm ngược (Inject) linh kiện vào mô hình `Car` ở thời điểm thực thi (Runtime). Cách tiếp cận này tuân thủ các nguyên lý thiết kế vững chắc, tạo tiền đề để chúng ta tìm hiểu Nguyên lý SOLID (Chương 4).

---
**Navigation:**
[⬅️ Previous: Bài 12: Đóng gói (Encapsulation) - Nghệ thuật Che giấu](./12-encapsulation.md) | [Next: Bài 14: Tính Đa hình (Polymorphism) và Bảng phương thức ảo (V-Table) ➡️](./14-polymorphism-and-vtables.md)
