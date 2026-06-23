# Bài 27: Decorator Pattern và Thiết kế Mở rộng Đối tượng động

Khép lại lộ trình Hệ thống Cơ sở của Lập trình Hướng đối tượng, chúng ta nghiên cứu **Decorator (Mẫu Trang trí)** - một Mẫu thiết kế Cấu trúc cung cấp cơ chế gán thêm (Attach) các tính năng mới vào một thể hiện đối tượng (Object Instance) một cách linh động ngay tại thời điểm thực thi (Runtime).

Thiết kế Decorator được xem là giải pháp cốt lõi để loại bỏ kiến trúc **Bùng nổ Tổ hợp Kế thừa (Class Explosion)**.

---

## 1. Bài toán Mở rộng Cấu trúc Tính năng

Việc lạm dụng Kế thừa đa tầng để mô phỏng sự pha trộn của thuộc tính đối tượng dễ dẫn đến sự gia tăng số lượng lớp (Class) theo dạng cấp số nhân.

Giả sử hệ thống xử lý giao dịch thương mại cần mô tả sản phẩm `BaseCoffee`. Khi bổ sung các phần mở rộng chức năng tính cước độc lập (Thêm sữa, thêm đường, thêm đá), nếu phân tầng bằng phương pháp tĩnh (Kế thừa), kiến trúc sư phải tạo ra vô số các tổ hợp con:
- `CoffeeWithMilk`
- `CoffeeWithSugar`
- `CoffeeWithMilkAndSugar`
- `CoffeeWithIce`
- `CoffeeWithMilkAndIce`...

Sự phân tán của chuỗi hàng chục cấu trúc lớp vô nghĩa này đẩy dự án vào vòng lặp thiết kế cồng kềnh, vi phạm nghiêm trọng tính tổ chức logic hệ thống.

---

## 2. Decorator: Sử dụng Tổ hợp (Composition) thay thế Kế thừa

Để phá vỡ giới hạn này, cấu trúc của Decorator Pattern tuân thủ triết lý bọc lớp (Wrapper). Thay vì tạo ra một thiết kế tĩnh kết hợp, ta nhúng tham chiếu của đối tượng cơ sở (Core Object) vào một thiết kế bọc bên ngoài. Lớp bọc (Decorator) và thực thể nằm ở trung tâm đều tuân thủ chung một chuẩn Giao thức (Interface).

Cơ chế tổ chức hoạt động giống như một củ hành tây nhiều lớp vỏ: Lớp vỏ ngoài cùng tiếp nhận luồng xử lý từ người dùng, sửa đổi kết quả rồi đẩy tuần tự vào lớp bên trong cho tới trung tâm.

```mermaid
graph TD
    subgraph Vỏ bọc mở rộng 2 (SugarDecorator)
        subgraph Vỏ bọc mở rộng 1 (MilkDecorator)
            subgraph Thực thể cơ sở (BaseCoffee)
                C[Cà Phê Gốc<br/>Giá trị: 10]
            end
            M[Thêm Sữa<br/>Tiếp nhận luồng lõi + 5]
        end
        B[Thêm Đường<br/>Tiếp nhận luồng lõi + 2]
    end
    
    style C fill:#fff3cd,stroke:#ffc107
    style M fill:#d4edda,stroke:#28a745
    style B fill:#d1ecf1,stroke:#17a2b8
```

### Cấu trúc mã nguồn triển khai

```java
// 1. Chuẩn giao thức hợp nhất cho Lõi và Vỏ bọc
interface Beverage {
    int getCost();
}

// 2. Thực thể Lõi (Lớp cơ sở đóng gói xử lý lõi)
class BaseCoffee implements Beverage {
    public int getCost() { return 10; }
}

// 3. Khung thiết kế Lớp vỏ Decorator (Base Decorator)
// *Sống còn: Nó phải VỪA tuân thủ chuẩn Interface, VỪA lưu giữ tham chiếu đến Interface (Đệ quy cấu trúc)*
abstract class BeverageDecorator implements Beverage {
    protected Beverage wrapper; // Lõi bên trong
    
    public BeverageDecorator(Beverage inner) {
        this.wrapper = inner; 
    }
}

// 4. Các cấu trúc mở rộng chức năng cụ thể
class MilkDecorator extends BeverageDecorator {
    public MilkDecorator(Beverage inner) { super(inner); }
    
    public int getCost() {
        // Tái sử dụng giá trị lõi, can thiệp thêm tham số bổ sung
        return this.wrapper.getCost() + 5; 
    }
}

class SugarDecorator extends BeverageDecorator {
    public SugarDecorator(Beverage inner) { super(inner); }
    
    public int getCost() {
        return this.wrapper.getCost() + 2; 
    }
}
```

Quá trình lắp ráp chuỗi cấu trúc mềm dẻo ở Runtime:
```java
// Khởi tạo gốc
Beverage myOrder = new BaseCoffee();

// Lồng ghép thêm cấu trúc Decorator để nâng cấp đối tượng
myOrder = new MilkDecorator(myOrder); // Bọc lớp 1
myOrder = new SugarDecorator(myOrder); // Bọc lớp 2

System.out.println("Kết quả tổng hợp: " + myOrder.getCost()); // Output: 17
```

Bằng phương án này, để tạo ra hàng vạn tổ hợp kết hợp bất kỳ giữa Cà phê, Sữa, Đường, Trân châu,... dự án chỉ phải quản trị chính xác 4 cấu trúc class tuyến tính. Việc mở rộng đối tượng không cần sự hỗ trợ của đa cấu hình (if-else).

---

## 3. Hệ thống Tiêu chuẩn áp dụng Decorator

Việc vận dụng Decorator cực kỳ mạnh mẽ trong các kiến trúc phát triển Thư viện vào ra (I/O Streams). 
Đoạn mã đọc tệp tin quen thuộc của Java:
```java
InputStream in = new BufferedInputStream(new FileInputStream("data.txt"));
```
Chính là một minh chứng hệ thống Decorator. `FileInputStream` đóng vai trò là thực thể cơ sở, tương tác tín hiệu tệp tin với cấu trúc Byte vật lý, trong khi `BufferedInputStream` cung cấp lớp vỏ bọc tăng tốc (đệm dữ liệu vào RAM) để giảm thời gian tìm kiếm từ đĩa cứng I/O. Nhà phát triển có thể thiết lập thêm một lớp `ZipInputStream` lồng bên ngoài để kích hoạt giải nén dữ liệu thời gian thực.

Trong hệ sinh thái phát triển ứng dụng web hiện đại (Python, TypeScript), tính năng bọc hàm thông qua toán tử đánh dấu `@` (Annotations / Decorators) chính là hiện thân thực tế hóa kiến trúc này nhằm cung cấp khả năng tiền kiểm tra (như bảo vệ định danh Login cho Route API) bằng cách nhúng các lớp trung gian (Middlewares) quanh thực thể chức năng lõi.

---
**Navigation:**
[⬅️ Previous: Bài 26: Observer Pattern và Kiến trúc Hướng Sự kiện (Event-Driven)](./26-pattern-observer.md)
