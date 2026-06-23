# Bài 25: Strategy Pattern và Cơ chế Tách rời Thuật toán

Chuyển sang nhóm Hành vi (Behavioral Patterns), tiêu điểm của nhóm này giải quyết bài toán giao tiếp sự kiện và ủy thác tính toán giữa các thực thể hệ thống. Trọng tâm cốt lõi của việc trừu tượng hóa các luồng lệnh rẽ nhánh là **Strategy Pattern (Mẫu Chiến lược)**.

---

## 1. Vấn đề: Tích hợp Đa thuật toán (Algorithm Accumulation)

Trong kiến trúc phát triển ứng dụng, có các lớp cấu trúc yêu cầu hỗ trợ đa dạng phương pháp tính toán hoặc thuật toán xử lý dữ liệu chung. 

Xét lớp `Navigator` của công cụ bản đồ số, cung cấp các tính năng lập tuyến đường tương tác:
```java
class Navigator {
    public void buildRoute(String start, String end, String travelMode) {
        if (travelMode.equals("Car")) {
            // Xử lý đồ thị đường cao tốc, tối ưu hóa điểm trạm thu phí...
        } 
        else if (travelMode.equals("Pedestrian")) {
            // Tính toán cầu vượt, đường bộ hành...
        }
        else if (travelMode.equals("Transit")) {
            // Xử lý điểm dừng xe buýt, tích hợp lịch trình giao thông...
        }
    }
}
```

Kiến trúc này phát sinh sai lầm định dạng:
1. **Vi phạm Nguyên lý OCP (Bài 17):** Khả năng mở rộng bắt buộc các nhà lập trình phải đính kèm liên tục các khối `else-if` khổng lồ vào duy nhất một tập tin điều phối.
2. **Quá tải Cấu trúc (Monolithic Class):** Việc duy trì đồng thời hàng chục thuật toán (từ hàng chục dòng đến hàng nghìn dòng mã) trong một không gian duy nhất khiến khả năng kiểm tra đơn vị (Unit Testing) và gỡ lỗi (Debugging) chịu rủi ro bùng nổ độ phức tạp.

---

## 2. Giải pháp: Kiến trúc Strategy

Nguyên lý Strategy đề xuất tái kiến trúc bằng việc: **Trừu tượng hóa từng khối thuật toán (Chiến lược) thành một tệp đối tượng riêng biệt (implements chung một Interface). Lớp khởi tạo trung tâm không còn đảm nhận phân tích logic, thay vào đó, nó thiết lập một kênh giao tiếp ủy thác (Delegation) để cho phép cắm ghép và chuyển đổi động các thuật toán ở quá trình thực thi (Runtime).**

```mermaid
graph TD
    subgraph "Lớp Định vị (Context)"
        N["Navigator<br/>Dữ liệu tham số: Toạ độ"]
    end
    
    subgraph "Giao diện Chiến lược (Strategy)"
        I((IRouteStrategy<br/>+ execute()))
    end
    
    subgraph "Các Thuật toán Cụ thể (Concrete Strategies)"
        C[CarStrategy]
        W[WalkingStrategy]
        P[TransitStrategy]
    end
    
    N -->|Sở hữu Liên kết| I
    C -.->|Triển khai| I
    W -.->|Triển khai| I
    P -.->|Triển khai| I
    
    style N fill:#d1ecf1,stroke:#17a2b8
    style I fill:#007bff,color:#fff
    style C fill:#d4edda,stroke:#28a745
    style W fill:#d4edda,stroke:#28a745
    style P fill:#d4edda,stroke:#28a745
```

Mã nguồn chuyển đổi:
```java
// 1. Phân tầng Giao diện Chiến lược
interface IRouteStrategy {
    void execute(String start, String end);
}

// 2. Chuyển phân khối xử lý thuật toán vào các Module độc lập
class CarStrategy implements IRouteStrategy {
    public void execute(String start, String end) { /* Thuật toán xe ô tô */ }
}
class WalkingStrategy implements IRouteStrategy {
    public void execute(String start, String end) { /* Thuật toán đi bộ */ }
}

// 3. Lớp Giao tiếp Ngữ cảnh (Context)
class Navigator {
    private IRouteStrategy strategy; // Con trỏ uỷ thác logic

    // Cơ chế thiết lập cho phép chuyển đổi động (Runtime Change)
    public void setStrategy(IRouteStrategy newStrategy) {
        this.strategy = newStrategy;
    }

    // Luồng thực thi trừu tượng, ẩn giấu cấu trúc rẽ nhánh
    public void calculateRoute(String start, String end) {
        this.strategy.execute(start, end);
    }
}
```

Kiểm thử hệ thống:
```java
Navigator nav = new Navigator();

// Phân hệ Car
nav.setStrategy(new CarStrategy());
nav.calculateRoute("Point A", "Point B"); 

// Khi điều kiện thay đổi, có thể nạp thuật toán khác lên cùng Context
nav.setStrategy(new WalkingStrategy());
nav.calculateRoute("Point A", "Point B"); 
```

---

## 3. Phân tích Ứng dụng Kỹ thuật

Strategy là phương thức mạnh mẽ định hình khả năng tháo lắp linh hoạt các module logic (Plug-and-play). Các hệ thống nổi tiếng đều lấy Strategy làm trụ cột thiết kế:
1. **API Phân loại hệ thống (Sorting Algorithms):** Phương thức `Collections.sort(list, comparator)` trong Java là tiêu chuẩn Strategy. Mảng (Context) cung cấp cơ chế hoán đổi đối tượng, nhưng nguyên tắc phán xét tiêu chí đối tượng ưu tiên (`comparator`) sẽ do một Strategy được cắm ghép từ bên ngoài xử lý ủy thác.
2. **Cổng giao dịch E-commerce:** Quá trình thanh toán của giỏ hàng có thể sử dụng hàng chục phương pháp (PayPal, Visa, Crypto). Việc chuyển trạng thái các phương pháp này do Strategy đảm nhận độc lập, tăng khả năng liên kết mở rộng mà không tác động luồng dữ liệu chính của giỏ hàng.

Tóm lại, Strategy Pattern chuyển đổi Hành vi định tuyến của mã nguồn từ Cấu trúc tĩnh (Static if-else) sang Kiến trúc tham chiếu động (Polymorphic Delegation), đóng góp vào việc thực thi Nguyên lý Đảo ngược Phụ thuộc (DIP).

---
**Navigation:**
[⬅️ Previous: Bài 24: Nhóm Cấu trúc: Adapter và Facade Pattern](./24-pattern-adapter-facade.md) | [Next: Bài 26: Observer Pattern và Kiến trúc Hướng Sự kiện (Event-Driven) ➡️](./26-pattern-observer.md)
