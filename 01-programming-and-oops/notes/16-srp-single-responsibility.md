# Bài 16: SRP - Nguyên lý Đơn trách nhiệm (Single Responsibility Principle)

Sau khi hoàn thiện 4 trụ cột cơ sở của OOP, quá trình ứng dụng chúng vào các phần mềm quy mô lớn gặp phải thách thức về tổ chức mã nguồn và quản trị biến đổi. Để giải quyết các vướng mắc cấu trúc, năm nguyên lý thiết kế **S-O-L-I-D** được công bố bởi Robert C. Martin (Uncle Bob). Nguyên lý đầu tiên là chữ S: **Nguyên lý Đơn trách nhiệm (Single Responsibility Principle - SRP)**.

Định nghĩa cốt lõi: *"Một lớp (Class) chỉ nên có duy nhất một lý do để thay đổi (A class should have one, and only one, reason to change)."*

---

## 1. Vấn đề kiến trúc: God Object (Đối tượng Toàn năng)

Trong lập trình, xu hướng thiết lập các class ôm đồm quá nhiều chức năng đa luồng là một lỗi thiết kế phổ biến. Những khối đối tượng quy mô hàng ngàn dòng mã, thực thi nhiều luồng nghiệp vụ khác biệt được gọi bằng thuật ngữ hệ thống là Anti-Pattern **"God Object"**.

Xét một cấu trúc báo cáo điển hình (Report):
```java
class ReportManager {
    // Trách nhiệm 1: Thao tác logic truy xuất dữ liệu
    public void generateData() {
        // ... truy vấn cơ sở dữ liệu ...
    }

    // Trách nhiệm 2: Thao tác định dạng hiển thị
    public void formatToHTML() {
        // ... tạo thẻ HTML ...
    }

    // Trách nhiệm 3: Thao tác lưu trữ tệp tin
    public void saveToFile(String path) {
        // ... xử lý file I/O ...
    }
}
```

Bất lợi từ cấu trúc tích hợp (God Object):
1. **Lệ thuộc chéo (High Coupling):** Mã nguồn xử lý cơ sở dữ liệu bị kết nối với thư viện xử lý tệp tin. Nếu thư viện HTML cần cập nhật phiên bản, toàn bộ Class chứa logic truy xuất tài chính cũng buộc phải tái biên dịch và rủi ro ảnh hưởng chéo.
2. **Xung đột Quản lý phiên bản (Merge Conflict):** Nếu Lập trình viên A tiến hành sửa module báo cáo đồ họa, trong khi Lập trình viên B thực hiện sửa tính năng SQL, khả năng cao việc hợp nhất (Merge) qua Git trên cùng một file sẽ thất bại.
3. **Thách thức Kiểm thử tự động (Unit Test):** Rất khó cấu hình môi trường khởi tạo để bao phủ các kịch bản biệt lập của một khối xử lý đa tính năng.

---

## 2. Giải pháp phân tách Trách nhiệm (Refactoring)

Theo nguyên lý SRP, hệ thống phải tuân thủ sự chia rẽ logic: Logic nghiệp vụ, Logic đồ họa và Logic tương tác tệp tin không bao giờ được cấu thành chung một đơn vị thực thi.

Quá trình chia rẽ cấu trúc (Refactoring):

```mermaid
graph TD
    subgraph God Object
        A[ReportManager<br/>(Hàng ngàn dòng code, làm mọi việc)]
    end
    
    subgraph SRP Architecture
        D[ReportGenerator<br/>(Nhiệm vụ: Truy xuất dữ liệu DB)]
        F[ReportFormatter<br/>(Nhiệm vụ: Chuyển đổi định dạng PDF/HTML)]
        S[ReportSaver<br/>(Nhiệm vụ: Lưu trữ I/O lên Ổ cứng)]
    end
    
    A -.->|Phân mảnh cấu trúc| D
    A -.->|Phân mảnh cấu trúc| F
    A -.->|Phân mảnh cấu trúc| S
    
    style A fill:#f8d7da,stroke:#dc3545
    style D fill:#d4edda,stroke:#28a745
    style F fill:#d4edda,stroke:#28a745
    style S fill:#d4edda,stroke:#28a745
```

Mã nguồn điều phối:
```java
// Chịu trách nhiệm hoàn toàn về Logic Lọc dữ liệu
class ReportGenerator {
    public ReportData generateData() { /* ... */ }
}

// Chịu trách nhiệm thuần túy về biến đổi giao diện
class ReportFormatter {
    public String formatToHTML(ReportData data) { /* ... */ }
}

// Xử lý biệt lập các rủi ro hệ thống File I/O
class ReportSaver {
    public void saveToFile(String content, String path) { /* ... */ }
}
```

---

## 3. Tính gắn kết (Cohesion) trong Đánh giá Hệ thống

Mục tiêu sâu xa của Nguyên lý Đơn trách nhiệm là cực đại hóa **Độ gắn kết nội bộ (High Cohesion)**. Thuật ngữ Cohesion đánh giá mức độ tương quan và phụ thuộc liên kết giữa các thuộc tính với các phương thức hoạt động trong phạm vi một Lớp. 

Một lớp được đánh giá tuân thủ độ Cohesion cao khi phần lớn các phương thức của nó tận dụng được hết toàn bộ biến thành viên để xử lý luồng, không để dư thừa các biến tệp đính kèm. 
- Ngược lại, nếu một nửa số hàm trong lớp chỉ sử dụng tập hợp biến A, và nửa số hàm còn lại hoàn toàn chỉ làm việc độc lập với tập hợp biến B, ta có thể kết luận chắc chắn cấu trúc lớp đang bao hàm hai Trách nhiệm tách biệt. Việc tiến hành quy trình chẻ nhỏ (Split class) tuân thủ SRP là một cấu hình cần thiết để cải tiến cấu trúc vòng đời dài hạn.

---
**Navigation:**
[⬅️ Previous: Bài 15: Tính Trừu tượng (Abstraction): Abstract Class vs Interface](./15-abstraction.md) | [Next: Bài 17: OCP - Nguyên lý Đóng/Mở (Open/Closed Principle) ➡️](./17-ocp-open-closed.md)
