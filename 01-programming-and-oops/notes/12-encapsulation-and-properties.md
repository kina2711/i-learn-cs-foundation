# Bài 12: Tính Đóng gói (Encapsulation) và Cơ chế Truy cập

Tính Đóng gói (Encapsulation) là trụ cột đầu tiên trong bốn nguyên lý cốt lõi của Lập trình Hướng đối tượng (OOP). Nó đề cập đến việc **gom nhóm dữ liệu (thuộc tính) và các phương thức hoạt động trên dữ liệu đó vào cùng một đơn vị (Class)**, đồng thời **hạn chế quyền truy cập trực tiếp từ bên ngoài vào trạng thái nội bộ của đối tượng**.

Mục tiêu của đóng gói không phải là bảo mật (Security) theo nghĩa mã hóa, mà là bảo vệ tính toàn vẹn của dữ liệu (Data Integrity) trước những tương tác không lường trước từ các đoạn mã khác trong hệ thống.

---

## 1. Hạn chế của Truy cập Trực tiếp (Public State)

Xét cấu trúc dữ liệu cơ bản không áp dụng đóng gói:
```java
class BankAccount {
    public double balance; // Trạng thái phơi bày (Public state)
}

// Bất kỳ class nào cũng có thể can thiệp
BankAccount acc = new BankAccount();
acc.balance = -5000; // Trạng thái phi logic (Logic Invariant Violation)
```

Việc để lộ các thuộc tính cấu trúc trực tiếp ra ngoài tạo ra những rủi ro kiến trúc:
1. **Vi phạm ràng buộc nghiệp vụ (Business Invariants):** Lập trình viên có thể gán giá trị âm cho số dư tài khoản mà không kích hoạt quy trình kiểm tra hợp lệ nào.
2. **Khó khăn trong tái cấu trúc (Refactoring):** Nếu hệ thống cần đổi đơn vị tiền tệ từ VNĐ sang USD, kỹ sư phải tìm và sửa đổi tất cả các luồng truy cập trực tiếp biến `balance` rải rác khắp dự án, gây ảnh hưởng lan truyền (Ripple effect).

---

## 2. Cơ chế Access Modifiers và Getter/Setter

Giải pháp kiến trúc của Đóng gói là che giấu dữ liệu bằng các bộ từ khóa điều khiển quyền truy cập (Access Modifiers: `private`, `protected`, `public`) và thiết lập các cổng giao tiếp chuẩn hóa thông qua phương thức kiểm soát (Getter/Setter).

```java
class BankAccount {
    // 1. Dữ liệu bị che giấu tuyệt đối
    private double balance; 

    // 2. Cổng truy cập được kiểm soát chặt chẽ
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Số tiền nạp phải lớn hơn 0");
        }
        this.balance += amount; // Cập nhật trạng thái an toàn
    }

    // 3. Cổng đọc dữ liệu (Chỉ đọc - Readonly)
    public double getBalance() {
        return this.balance;
    }
}
```

Bằng cách thiết lập bức tường `private` và cung cấp bộ phương thức giao tiếp trung gian (API nội bộ):
- Class có quyền kiểm soát tuyệt đối tính hợp lệ của dữ liệu trước khi thay đổi trạng thái gốc.
- Tính mềm dẻo của hệ thống được duy trì. Quá trình tính toán, quy đổi đơn vị có thể được ngầm thay đổi bên trong logic của hàm `getBalance()` mà các thành phần đối tượng gọi bên ngoài (Client code) không cần cập nhật và biên dịch lại.

---

## 3. Cải tiến kiến trúc: Properties (Thuộc tính)

Mặc dù mô hình Getter/Setter giải quyết triệt để rủi ro cấu trúc, nhưng nó làm mã nguồn trở nên cồng kềnh (Boilerplate code). Kỹ sư phải khai báo hàng loạt hàm `get/set` cho cả những biến chỉ yêu cầu gán cơ bản.

Các ngôn ngữ hiện đại thế hệ sau (như C#, Kotlin, Swift) đưa ra giải pháp trung hòa mang tên **Properties (Thuộc tính)**.

Cơ chế Property cung cấp cú pháp gán/đọc ngắn gọn của biến `public`, nhưng ngầm định biên dịch chúng thành các phương thức trung gian, cho phép kỹ sư cài cắm logic kiểm tra khi cần thiết mà không phá vỡ giao diện lập trình.

```csharp
// Ngôn ngữ C#
class BankAccount {
    private double _balance; // Biến nền (Backing field)

    // Khai báo Property
    public double Balance {
        get { return _balance; }
        set { 
            if (value < 0) throw new Exception("Invalid");
            _balance = value; 
        }
    }
}

// Cú pháp thao tác giống hệt biến public, nhưng thực chất đang gọi hàm
BankAccount acc = new BankAccount();
acc.Balance = 1000; // Gọi block 'set'
Console.WriteLine(acc.Balance); // Gọi block 'get'
```

Tóm lại, tính Đóng gói đảm bảo nguyên tắc: Một lớp (Class) phải độc lập quản trị và chịu trách nhiệm cho các trạng thái nội tại của chính nó. Không có thực thể ngoại vi nào được phép thay đổi trực tiếp luồng dữ liệu mà bỏ qua các quy tắc nghiệp vụ nội bộ.

---
**Navigation:**
[⬅️ Previous: Bài 11: Bản chất của Class và Cơ chế hoạt động của con trỏ "this"](./11-classes-and-hidden-pointers.md) | [Next: Bài 12: Đóng gói (Encapsulation) - Nghệ thuật Che giấu ➡️](./12-encapsulation.md)
