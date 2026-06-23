# Bài 12: Đóng gói (Encapsulation) - Nghệ thuật Che giấu

Tính Đóng gói (Encapsulation) là trụ cột đầu tiên của OOP. Các sách giáo khoa thường giải thích nó bằng một câu thần chú rất chán: *"Đóng gói là việc gom data và method vào một class, sau đó dùng `private` để giấu data đi và dùng `public` getter/setter để truy cập"*.

Hệ quả của cách dạy này là sinh viên tạo ra những cái class vô nghĩa như sau:
```java
class BankAccount {
    private int balance;

    public int getBalance() { return balance; }
    public void setBalance(int amount) { balance = amount; }
}
```
Code trên **không hề có tính Đóng gói**. Bạn giấu tiền (`private`) vào két sắt, nhưng lại để sẵn chìa khóa (`setBalance`) ngoài cửa cho bất kỳ ai cũng có thể đổi số tiền của bạn thành 0 đồng.

Vậy bản chất thực sự của Đóng gói trong Software Engineering là gì?

---

## 1. Tư duy Đóng gói: Giao tiếp qua Hợp đồng (API), không phải qua Dữ liệu

> [!TIP]
> **ELI5:** Hãy tưởng tượng Class của bạn là một **Cái Tivi**. Bạn là người dùng (User code).
> - Tính đóng gói có nghĩa là: Tivi giấu toàn bộ bảng mạch điện, dây nhợ lằng nhằng (`private data`) ở bên trong lớp vỏ nhựa.
> - Nó chỉ đưa ra cho bạn cái Remote (Điều khiển) với các nút bấm như `Tăng âm lượng`, `Chuyển kênh` (`public methods`).
> - Nếu không có đóng gói: Bạn phải thò tay vào trong ruột tivi, tự lấy dây điện A chập vào dây điện B để chuyển kênh. Rất dễ giật điện và làm cháy Tivi.

Trong lập trình, Đóng gói là việc **Bảo vệ Tính Toàn Vẹn Của Trạng Thái (State Integrity)**.
Trạng thái của một Object không được phép thay đổi một cách tùy tiện, mà phải thông qua những "cái nút trên Remote" có kèm theo bộ lọc kiểm tra (Validation).

Thay vì dùng `setBalance` vô tri, một `BankAccount` chuẩn OOP phải viết như sau:

```java
class BankAccount {
    private int balance = 0;

    // Getter chỉ để ĐỌC, không cho phép SỬA
    public int getBalance() { 
        return balance; 
    }

    // Các "Nút bấm trên Remote" để tương tác
    public void deposit(int amount) {
        if (amount <= 0) throw new Exception("Tiền gửi phải lớn hơn 0");
        this.balance += amount;
    }

    public void withdraw(int amount) {
        if (amount <= 0) throw new Exception("Tiền rút phải lớn hơn 0");
        if (amount > this.balance) throw new Exception("Số dư không đủ");
        this.balance -= amount;
    }
}
```

```mermaid
graph LR
    User[Code gọi Hàm]
    
    subgraph "BankAccount Object"
        API["Public API<br/>deposit(")<br/>withdraw()]
        Data[(Private Data<br/>balance = 100)]
    end
    
    User -- "Gọi hàm hợp lệ" --> API
    API -- "Kiểm tra Rule & Sửa Data" --> Data
    User -. "Cố gắng sửa trực tiếp (LỖI)" .-> Data
    
    style API fill:#d4edda,stroke:#28a745
    style Data fill:#dc3545,color:#fff
```

---

## 2. Tại sao phải cực khổ như vậy? (The "Why")

Lập trình viên tay ngang thường bực mình: *"Gõ `account.balance += 100` có phải nhanh không, đẻ ra hàm `deposit()` làm gì cho tốn dòng code?"*

Trong các dự án siêu lớn, code của bạn được dùng bởi 50 kỹ sư khác. Nếu bạn để `public balance`, một người nào đó ở team khác sẽ lỡ tay viết: `account.balance = -5000;`.
1 tháng sau, QA báo lỗi số dư khách hàng bị âm. Bạn sẽ phải `Ctrl + Shift + F` tìm kiếm chữ `balance` trong 1 triệu dòng code của dự án để xem chỗ nào đã trừ sai tiền. Một cơn ác mộng.

**Nhờ có Đóng gói:**
1. **Dễ dàng Debug:** Bạn chỉ cần đặt duy nhất 1 cái Breakpoint (Điểm dừng) ở trong hàm `withdraw()`. Bất kỳ ai trên thế giới muốn rút tiền đều phải đi qua cái cửa trạm thu phí này. Bạn tóm gọn lỗi trong 3 giây.
2. **Che giấu sự phức tạp (Information Hiding):** Ngày hôm nay, số dư lưu bằng `int`. Ngày mai, ngân hàng nâng cấp, yêu cầu lưu số dư bằng `BigDecimal` và phải mã hóa chuỗi (Encryption) vào Database. Vì bạn đã giấu biến `balance` thành `private`, bạn chỉ cần sửa nội bộ trong class `BankAccount`. 50 kỹ sư team khác xài hàm `deposit()` vẫn giữ nguyên code, không bị Break!

---

## 3. Các Mức độ Truy cập (Access Modifiers)

Mỗi ngôn ngữ (C++, Java, C#) đều trang bị vũ khí để bạn bảo vệ "ruột Tivi" của mình:

- **`private` (Tuyệt mật):** Không ai được đụng vào, trừ chính các hàm nằm bên trong bản thân Class đó. (Dùng cho 99% các biến Data).
- **`public` (Công khai):** Trưng bày ra cho cả thế giới xài. (Dùng cho các Hàm API giao tiếp).
- **`protected` (Của để dành cho con cái):** Người ngoài không được đụng, nhưng các Class con (kế thừa từ Class này) thì được xài. (Sẽ học ở bài Kế thừa).
- **`default` / `package-private`:** Chỉ những Class nằm chung một thư mục (Package) mới thấy nhau.

> [!WARNING]
> **Vấn đề với Python và JavaScript đời cũ:**
> Triết lý của Python là *"We are all consenting adults here"* (Chúng ta đều là người lớn cả). Python không có keyword `private`. 
> Nếu bạn muốn giấu một biến, bạn thêm dấu gạch dưới `_balance`. Nó chỉ mang tính chất **quy ước nhắc nhở**: *"Làm ơn đừng đụng vào cái biến này"*. Tuy nhiên, nếu bạn cố tình gõ `obj._balance = 100`, Python vẫn cho phép! 
> (Trong JS hiện đại, người ta đã đẻ ra dấu `#` ví dụ `#balance` để thực thi Hard-Private y hệt Java).

Tóm lại: Đóng gói không phải là dùng Getter/Setter một cách máy móc. Đóng gói là **Chỉ cho phép thế giới bên ngoài tương tác với Object thông qua các Hành vi (Behaviors / Methods) có kiểm soát.**

---
**Navigation:**
[⬅️ Previous: Bài 12: Tính Đóng gói (Encapsulation) và Cơ chế Truy cập](./12-encapsulation-and-properties.md) | [Next: Bài 13: Tính Kế thừa (Inheritance), Vấn đề Hình thoi và Mô hình Composition ➡️](./13-inheritance-and-diamond-problem.md)
