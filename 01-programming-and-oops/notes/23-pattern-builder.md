# Bài 23: Builder Pattern và Rủi ro Telescoping Constructor

Tương tự như Factory, **Builder** là một Mẫu thiết kế thuộc nhóm Khởi tạo (Creational Pattern). Tuy nhiên, nếu Factory hướng đến mục đích cung cấp một giao thức trừu tượng để phân phối các đối tượng hoàn chỉnh, Builder lại chuyên dụng để xử lý việc **cấu trúc và xây dựng từng phần của một đối tượng phức tạp**.

---

## 1. Rủi ro Cấu trúc: Telescoping Constructor (Hàm tạo dạng viễn vọng)

Trong quá trình mô hình hóa dữ liệu (ví dụ: tạo lớp mô tả một Máy tính `Computer`), các kỹ sư đối mặt với sự mở rộng các tham số thuộc tính. Để phục vụ việc khởi tạo linh hoạt các tổ hợp khác nhau, lập trình viên thường cung cấp một chuỗi các phương thức Constructor nạp chồng, dần dần gia tăng số lượng tham số đầu vào.

```java
// Ví dụ về Telescoping Constructor Anti-Pattern
public Computer(String cpu, String ram) { ... }
public Computer(String cpu, String ram, boolean hasGPU) { ... }
public Computer(String cpu, String ram, boolean hasGPU, boolean hasSSD) { ... }
public Computer(String cpu, String ram, boolean hasGPU, boolean hasSSD, boolean hasWifi) { ... }
```

Hạn chế nghiêm trọng của mô hình này:
1. **Sự nhầm lẫn vị trí tham số:** Khi khởi tạo hệ thống phức tạp, luồng gọi hàm trở nên không thể duy trì độ tin cậy. Đoạn mã `new Computer("Intel", "16GB", false, false, true)` rất dễ bị nhầm lẫn thứ tự các giá trị `boolean` liên tiếp, gây lỗi sai lệch dữ liệu nội bộ nghiêm trọng ở cấp biên dịch.
2. **Hạn chế của Setter:** Một phương án thay thế là tạo Constructor trống và sử dụng các phương thức `set`. Tuy nhiên, phương pháp này sẽ khiến đối tượng tạm thời tồn tại ở trạng thái không toàn vẹn trước khi các hàm `set` được gọi đầy đủ, đồng thời làm phá vỡ cơ chế của tính Bất biến (Immutability) - vốn tối quan trọng trong quy trình đa luồng (Multi-threading).

---

## 2. Giải pháp Kiến trúc Builder Pattern

Mẫu Builder phân tách quy trình thiết lập trạng thái ra khỏi quá trình hình thành lớp nguyên thủy. Việc khởi tạo được kiểm soát theo từng pha chức năng và chỉ trả về thể hiện toàn vẹn của lớp khi quá trình xác nhận kết thúc.

Mô hình hiện đại của Builder tích hợp **Fluent Interface (Giao thức Chuỗi hàm / Method Chaining)**, đạt được bằng việc tái tham chiếu `return this` tại mỗi phương thức phân mảnh.

```mermaid
graph LR
    U[Client Code] -->|1. Cấp phát CPU| B(ComputerBuilder)
    B -->|2. Cấp phát GPU| B
    B -->|3. Cấp phát RAM| B
    B -->|4. Thực thi lệnh build()| P[Đóng gói: Object Computer]
    
    style B fill:#d1ecf1,stroke:#17a2b8
    style P fill:#d4edda,stroke:#28a745
```

### Triển khai cấu trúc mã nguồn:

```java
class ComputerBuilder {
    // Lưu trữ tạm thời trạng thái quá độ
    private String cpu = "Default_CPU";
    private String ram = "8GB";
    private boolean hasGPU = false;
    
    // Phương thức gán giá trị trả về tham chiếu đến bản thân thực thể
    public ComputerBuilder setCpu(String cpu) {
        this.cpu = cpu;
        return this; 
    }
    
    public ComputerBuilder setRam(String ram) {
        this.ram = ram;
        return this;
    }
    
    public ComputerBuilder enableGPU() {
        this.hasGPU = true;
        return this;
    }
    
    // Chốt cấu hình và khởi tạo đối tượng bất biến
    public Computer build() {
        return new Computer(this.cpu, this.ram, this.hasGPU); 
    }
}
```

Quá trình điều phối tại lớp giao diện (Client):
```java
// Cấu trúc mô tả theo chuỗi, hạn chế tuyệt đối sai sót giá trị
Computer workstation = new ComputerBuilder()
                        .setCpu("Intel i9")
                        .setRam("32GB")
                        .enableGPU()
                        .build();
```

---

## 3. Đánh giá Hệ thống và Đặc điểm Ngôn ngữ

Builder Pattern đặc biệt phổ biến trong thiết kế API hệ thống thư viện C++, Java, và C# nhằm đảm bảo mức an toàn bộ nhớ tĩnh. Khả năng thiết lập giá trị trễ của Builder giúp Lập trình viên xây dựng các khối khởi tạo động tùy thuộc vào điều kiện môi trường chạy (Runtime configurations) mà không cần can thiệp logic thiết lập trong lớp cơ sở.

> [!NOTE]
> **Định tuyến cấu trúc của Ngôn ngữ Thế hệ mới:**
> Ở các ngôn ngữ hiện đại hỗ trợ định dạng **Named Arguments** hoặc **Data Classes** (như Python, Kotlin, Swift), sự hiện diện của Builder đã mất dần ưu thế.
> ```python
> # Cú pháp Python triệt tiêu Telescoping Constructor
> my_pc = Computer(cpu="Intel", has_gpu=True, has_ssd=False)
> ```
> Cấu trúc này giải quyết tính minh bạch tham số một cách căn bản mà không yêu cầu Lập trình viên phải triển khai thiết kế Builder tĩnh như đối với hệ sinh thái Java cổ điển.

---
**Navigation:**
[⬅️ Previous: Bài 22: Nhóm Khởi tạo: Factory Method và Abstract Factory](./22-pattern-factory.md) | [Next: Bài 24: Nhóm Cấu trúc: Adapter và Facade Pattern ➡️](./24-pattern-adapter-facade.md)
