# Bài 15: Tính Trừu tượng (Abstraction): Abstract Class vs Interface

Tính Trừu tượng (Abstraction) đóng vai trò là khối phân tách chức năng cuối cùng hoàn thiện nền tảng Hướng đối tượng. Nó được mô tả bằng nguyên lý thiết kế: **Che khuất các logic thao tác nội bộ phức tạp, và chỉ cung cấp cho hệ thống ngoài những thông số/mô tả chức năng thiết yếu nhất để giao tiếp.**

Việc hiện thực hóa quy trình thiết kế trừu tượng trong hầu hết các ngôn ngữ định kiểu tĩnh (Statically typed) được phân bố dưới dạng hai thành phần cơ sở: **Abstract Class (Lớp trừu tượng)** và **Interface (Giao diện)**.

---

## 1. Abstract Class (Lớp Trừu tượng)

`Abstract Class` chia sẻ toàn bộ quyền năng của một Lớp thông thường, điểm thiết kế khác biệt duy nhất của nó là sự tồn tại của khái niệm chưa định nghĩa. Lớp trừu tượng cho phép khai báo định danh các phương thức trống gọi là `Abstract Method`, chỉ đề ra chữ ký cấu trúc mà từ chối cung cấp khối lệnh (Body logic). 

Nguyên tắc bắt buộc:
1. Do thiếu sự toàn vẹn của logic, không một máy ảo hay trình biên dịch nào cho phép khởi tạo thực thể trực tiếp (`new`) từ Abstract Class.
2. Các Lớp con dẫn xuất bắt buộc phải bổ sung khối lượng mã lệnh để hoàn thiện sự thiếu hụt này (Quy trình Ghi đè `Override`).

```java
// Class trừu tượng: Mô hình nền tảng chưa được xây dựng đầy đủ
abstract class GameEntity {
    public int x, y; // Thuộc tính trạng thái cụ thể

    // Phương thức có logic hoàn chỉnh
    public void moveTo(int x, int y) {
        this.x = x; this.y = y;
    }

    // Phương thức trừu tượng: Chỉ thiết kế chữ ký
    public abstract void draw(); 
}

// Class dẫn xuất phải hoàn thành cấu trúc bị bỏ lỡ
class Player extends GameEntity {
    @Override
    public void draw() {
        // Cung cấp logic đồ họa tương ứng
    }
}
```

*Sử dụng Abstract Class khi thiết lập quan hệ IS-A chặt chẽ nhưng cần chia sẻ các thuộc tính hoặc một phần tính toán cơ bản dùng chung.*

---

## 2. Interface (Giao diện)

Nếu Abstract Class đại diện cho Lớp cơ sở chưa hoàn thiện, thì **Interface** đẩy mức độ tính trừu tượng lên mức cực đại. Interface loại bỏ mọi biến lưu trữ trạng thái. Trong định dạng cốt lõi của OOP cổ điển, một Interface đóng vai trò là một **Bản hợp đồng (Contract)** thuần túy.

Bản hợp đồng này chỉ bao gồm danh sách các định danh phương thức (không có logic thực thi). Bất kỳ Lớp nào chấp nhận triển khai (implement) hợp đồng này sẽ chịu sự cưỡng chế cấu trúc của Trình biên dịch: Bắt buộc hoàn thiện mã lệnh cho toàn bộ danh sách chức năng đã được liệt kê.

```java
// Bản hợp đồng
interface IClickable {
    void onClick();
    void onHover();
}

// Bất kỳ class nào (Nút bấm, Hình ảnh, Thẻ) cũng có thể đăng ký thực thi hợp đồng
class ImageWidget implements IClickable {
    @Override
    public void onClick() { /* Logic click hình */ }
    
    @Override
    public void onHover() { /* Logic đổi hiệu ứng */ }
}
```

### Kiến trúc giải quyết đa kế thừa
Như đã trình bày tại Bài 13, sự xung đột Vấn đề Hình thoi (Diamond Problem) do Đa kế thừa bị nghiêm cấm đối với cấp độ Lớp (Class). Thế nhưng, vì Interface hoàn toàn trống rỗng và không hề chứa khối lệnh logic tĩnh, sự phân cực kế thừa không thể xảy ra. Do vậy, một Class được phép thực thi (implement) tự do và đồng thời đa dạng các Interface khác nhau.

```java
// Hợp lệ trong kiến trúc ngôn ngữ
class SmartPhone implements ICamera, IPhone, IInternet {
    // Hoàn thành bổ sung các cấu trúc bắt buộc
}
```

---

## 3. Nguyên tắc Ứng dụng: Abstract Class hay Interface?

Định vị sai chức năng giữa hai cấu trúc này là một vi phạm cơ sở trong quá trình phân tích thiết kế phần mềm:

1. **Abstract Class (Lớp trừu tượng)**
   - Áp dụng khi mô hình các phân lớp con cùng sở hữu **Bản chất chia sẻ (Core Identity)**. Ví dụ: `Cat` và `Dog` có phân bố hình học và đặc tính sinh học chung, vì vậy nên dẫn xuất từ nhánh trừu tượng `Animal` có chức năng định nghĩa các chỉ số chung về sự sống.
   - Thích hợp chia sẻ mã lệnh nền tảng (Base logic).

2. **Interface (Giao diện cấu trúc)**
   - Áp dụng khi muốn cung cấp năng lực (Capability/Behavior) có tính kết hợp xuyên qua các nhánh mô hình khác biệt không mang mối tương đồng huyết thống.
   - Ví dụ: Hành động Lưu trữ cơ sở dữ liệu (Saveable). Cấu hình dữ liệu trò chơi, Cấu hình người chơi và Cấu hình tệp đồ họa hoàn toàn dị biệt về mặt cấu trúc. Hệ thống thiết lập chung phương thức bảo vệ bằng cách ép buộc toàn bộ implement cấu trúc `ISerializable`.
   - Các hệ thống hiện đại mở rộng việc ưu tiên tối đa thiết kế cấu trúc Interface, giảm sự ràng buộc đối với kiến trúc nền của Abstract Class để duy trì Tính linh hoạt tuyệt đối (Coupling Reduction).

---
**Navigation:**
[⬅️ Previous: Bài 14: Tính Đa hình (Polymorphism) và Bảng phương thức ảo (V-Table)](./14-polymorphism-and-vtables.md) | [Next: Bài 16: SRP - Nguyên lý Đơn trách nhiệm (Single Responsibility Principle) ➡️](./16-srp-single-responsibility.md)
