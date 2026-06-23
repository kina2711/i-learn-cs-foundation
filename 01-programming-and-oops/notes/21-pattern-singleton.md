# Bài 21: Singleton Pattern và Rủi ro Đa luồng (Multi-threading)

**Design Patterns (Mẫu Thiết Kế)** là các bộ giải pháp kiến trúc đã được tối ưu hóa nhằm giải quyết các vấn đề thiết kế tái diễn trong công nghệ phần mềm, được chuẩn hóa bởi nhóm Gang of Four (GoF) vào năm 1994. 

Mẫu thiết kế khởi tạo đầu tiên và cơ bản nhất là **Singleton**.

---

## 1. Định nghĩa và Mục đích của Singleton

Trong kiến trúc phần mềm, có những đối tượng (Objects) mang tính chất hạ tầng và yêu cầu chia sẻ trạng thái chung trên toàn bộ hệ thống. Các ví dụ điển hình bao gồm:
- Bộ quản lý kết nối cơ sở dữ liệu (Database Connection Pool).
- Hệ thống ghi nhật ký ứng dụng (Logger Service).
- Bộ đệm cấu hình hệ thống (Configuration Cache).

Việc khởi tạo nhiều bản sao của các đối tượng này không chỉ gây rò rỉ bộ nhớ mà còn sinh ra xung đột trạng thái (ví dụ: nhiều tiến trình cùng tranh chấp ghi dữ liệu vào một tệp nhật ký). 

Mục tiêu của Singleton Pattern là: **Đảm bảo một lớp (Class) chỉ có duy nhất một phiên bản đối tượng (Instance) tồn tại trong suốt vòng đời của ứng dụng, và cung cấp một điểm truy cập toàn cục tới phiên bản đó.**

### Cấu trúc cơ bản của Singleton:
Để ngăn cản các luồng mã bên ngoài khởi tạo đối tượng tự do, Singleton áp dụng hai nguyên tắc kỹ thuật:
1. Đặt phương thức khởi tạo (Constructor) ở trạng thái `private`.
2. Duy trì một biến tham chiếu tĩnh (static) lưu trữ phiên bản đối tượng và một phương thức tĩnh `getInstance()` để cấp phát.

```java
class Logger {
    // 1. Biến tĩnh lưu trữ phiên bản duy nhất
    private static Logger instance;

    // 2. Ngăn chặn khởi tạo từ bên ngoài
    private Logger() { }

    // 3. Phương thức cấp phát (Áp dụng Lazy Initialization)
    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger(); // Chỉ phân bổ bộ nhớ ở lần gọi đầu tiên
        }
        return instance; 
    }

    public void log(String message) { /* Xử lý ghi log */ }
}
```

---

## 2. Rủi ro Phân bổ trong Môi trường Đa luồng (Multi-threading)

Cấu trúc trên vận hành ổn định trong mô hình đơn luồng (Single-thread). Tuy nhiên, khi hệ thống triển khai xử lý đa luồng (ví dụ: máy chủ Web xử lý đồng thời hàng ngàn yêu cầu), một lỗi điều kiện tranh chấp (Race Condition) nghiêm trọng sẽ xuất hiện.

Giả sử Luồng A và Luồng B cùng gọi hàm `getInstance()` tại cùng một chu kỳ thời gian:
- Luồng A kiểm tra `if (instance == null)`, kết quả là `true`, chuẩn bị phân bổ đối tượng.
- Trước khi Luồng A kịp ghi nhận giá trị, vi xử lý chuyển ngữ cảnh sang Luồng B. Luồng B cũng kiểm tra `instance == null` và vẫn nhận kết quả `true`.
- Hệ quả: Cả hai luồng sẽ bỏ qua chốt chặn và tiến hành khởi tạo hai đối tượng độc lập, phá vỡ hoàn toàn nguyên tắc duy nhất của Singleton.

### Giải pháp 1: Khóa phương thức (Synchronized Method)
Cách tiếp cận truyền thống là áp dụng khóa tương hỗ (Mutual Exclusion).

```java
public static synchronized Logger getInstance() {
    if (instance == null) { instance = new Logger(); }
    return instance;
}
```
**Hạn chế:** Hiệu suất hệ thống suy giảm nghiêm trọng. Việc đồng bộ hóa chỉ mang ý nghĩa ở lần khởi tạo đầu tiên. Sau khi đối tượng đã tồn tại, các luồng truy cập cấu trúc chỉ để lấy tham chiếu đều phải tuần tự chờ đợi nhau mở khóa, gây ra hiện tượng thắt cổ chai (Bottleneck) lớn trong môi trường luồng cao.

### Giải pháp 2: Double-Checked Locking (Kiểm tra kép)
Kiến trúc này tối ưu hóa độ trễ bằng cách chỉ áp dụng khóa hệ thống khi đối tượng thực sự chưa tồn tại.

```java
class Logger {
    // Từ khóa volatile đảm bảo khả năng hiển thị bộ nhớ đa luồng, 
    // ngăn ngừa lỗi Out-of-order execution của CPU.
    private static volatile Logger instance;

    private Logger() {}

    public static Logger getInstance() {
        if (instance == null) { // Kiểm tra lớp 1: Bỏ qua khóa nếu đối tượng đã tồn tại
            synchronized (Logger.class) { // Thiết lập khóa
                if (instance == null) { // Kiểm tra lớp 2: Đảm bảo an toàn khi luồng vừa bước vào
                    instance = new Logger();
                }
            }
        }
        return instance;
    }
}
```
Mô hình Double-Checked Locking giải quyết bài toán hiệu năng và tính toàn vẹn bộ nhớ trong thiết kế Singleton của Java/C++.

---

## 3. Phân tích Hiện trạng Kiến trúc (Anti-pattern)

Trong các thiết kế phần mềm hiện đại, Singleton gốc thường được xem xét lại bởi hai nhược điểm nền tảng:
1. **Bản chất Biến toàn cục (Global State):** Singleton che đậy việc thiết lập các trạng thái có thể bị truy xuất và sửa đổi từ mọi nơi trong hệ thống, làm suy giảm tính toàn vẹn luồng (Encapsulation).
2. **Khó khăn trong Kiểm thử (Unit Testing):** Trạng thái tĩnh của đối tượng Singleton duy trì qua các vòng kiểm thử độc lập, làm sai lệch kết quả đánh giá phần mềm do dư chấn dữ liệu từ các ca kiểm thử trước.

**Kiến trúc thay thế:** Xu hướng hiện hành áp dụng nguyên lý **Đảo ngược Phụ thuộc (Dependency Injection - DIP)**. Các Container quản lý vòng đời (như Spring Framework) sẽ chịu trách nhiệm khởi tạo một đối tượng duy nhất (Singleton scope) và tiêm trực tiếp vào các mô-đun yêu cầu thông qua Constructor, giúp các lớp hoàn toàn giữ được thiết kế cô lập và thân thiện với kiểm thử.

---
**Navigation:**
[⬅️ Previous: Bài 20: DIP - Nguyên lý Đảo ngược Phụ thuộc (Dependency Inversion Principle)](./20-dip-dependency-inversion.md) | [Next: Bài 22: Nhóm Khởi tạo: Factory Method và Abstract Factory ➡️](./22-pattern-factory.md)
