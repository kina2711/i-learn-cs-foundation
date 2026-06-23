# Part 1: Lập trình Cơ sở và Hướng Đối Tượng (Programming Foundations & OOPs)

Chào mừng bạn đến với **Module 1: Nền tảng Lập trình & Hướng đối tượng**. 
Tài liệu này không tập trung vào cú pháp ngôn ngữ cơ bản, mà hướng đến việc phân tích chuyên sâu các kiến trúc cốt lõi của Khoa học Máy tính. Nội dung trải dài từ nguyên lý xử lý dữ liệu ở cấp độ phần cứng (Hardware level) cho đến các mô hình thiết kế phần mềm trừu tượng (Software Architecture).

Mục tiêu của học phần là cung cấp một góc nhìn hệ thống, khách quan và mang tính kỹ thuật cao, giúp người học xây dựng nền tảng vững chắc về lý thuyết điện toán.

---

## 📚 Mục lục Nội dung (Syllabus)

Toàn bộ tài liệu của Module này được lưu trữ trong thư mục `notes/`. Dưới đây là danh sách 27 chuyên đề kỹ thuật:

### 🏛️ Chương 1: Cơ sở Biểu diễn Dữ liệu (Bits, Bytes & Encoding)
Phân tích nguyên lý mã hóa và xử lý tín hiệu điện toán ở mức độ thấp.
- [Bài 1: Hệ nhị phân (Binary), Hệ thập lục phân (Hexadecimal) và Đơn vị lưu trữ](./notes/01-binary-and-hex.md)
- [Bài 2: Bảng mã ASCII, Unicode và Cơ chế của UTF-8](./notes/02-character-encodings.md)
- [Bài 3: Biểu diễn số âm và Số bù hai (Two's Complement)](./notes/03-negative-numbers-and-two-s-complement.md)
- [Bài 4: Số thực dấu phẩy động (Floating-Point) và Chuẩn IEEE 754](./notes/04-floating-point-numbers.md)
- [Bài 5: Thứ tự Byte (Endianness) và Định dạng Dữ liệu Mạng](./notes/05-endianness.md)
- [Bài 6: Toán tử Thao tác Bit (Bitwise Operations)](./notes/06-bitwise-operations.md)

### 🧩 Chương 2: Cấu trúc Bộ nhớ Tiến trình (Memory Layout)
Phân tích cơ chế quản trị và phân bổ không gian RAM của hệ điều hành.
- [Bài 7: Tổng quan Mô hình Bộ nhớ (Memory Layout) của Tiến trình](./notes/07-memory-segments.md)
- [Bài 8: Cơ chế Ngăn xếp (Call Stack) và Lỗi tràn Stack](./notes/08-call-stack-and-stack-overflow.md)
- [Bài 9: Cấp phát động, Bộ nhớ Heap và Con trỏ](./notes/09-heap-memory-and-pointers.md)
- [Bài 10: Quản lý Bộ nhớ tự động (Garbage Collection)](./notes/10-garbage-collection.md)

### 🏛️ Chương 3: Phân tích Lập trình Hướng đối tượng (Deep Dive into OOP)
Đánh giá bản chất kỹ thuật của các cơ chế Hướng đối tượng.
- [Bài 11: Bản chất của Class và Cơ chế hoạt động của con trỏ "this"](./notes/11-classes-and-hidden-pointers.md)
- [Bài 12: Tính Đóng gói (Encapsulation) và Cơ chế Truy cập](./notes/12-encapsulation-and-properties.md)
- [Bài 13: Tính Kế thừa (Inheritance), Vấn đề Hình thoi và Mô hình Composition](./notes/13-inheritance-and-diamond-problem.md)
- [Bài 14: Tính Đa hình (Polymorphism) và Bảng phương thức ảo (V-Table)](./notes/14-polymorphism-and-vtables.md)
- [Bài 15: Tính Trừu tượng (Abstraction): Abstract Class vs Interface](./notes/15-abstraction.md)

### 📐 Chương 4: Các Nguyên lý Thiết kế (S-O-L-I-D Principles)
Các chuẩn mực thiết kế và phân tách module nhằm tối ưu khả năng mở rộng hệ thống.
- [Bài 16: SRP - Nguyên lý Đơn trách nhiệm (Single Responsibility Principle)](./notes/16-srp-single-responsibility.md)
- [Bài 17: OCP - Nguyên lý Đóng/Mở (Open/Closed Principle)](./notes/17-ocp-open-closed.md)
- [Bài 18: LSP - Nguyên lý Thay thế Liskov (Liskov Substitution Principle)](./notes/18-lsp-liskov-substitution.md)
- [Bài 19: ISP - Nguyên lý Giao diện Phân tách (Interface Segregation Principle)](./notes/19-isp-interface-segregation.md)
- [Bài 20: DIP - Nguyên lý Đảo ngược Phụ thuộc (Dependency Inversion Principle)](./notes/20-dip-dependency-inversion.md)

### 🎭 Chương 5: Các Mẫu Thiết kế Cốt lõi (Design Patterns)
Các chiến lược giải quyết vấn đề cấu trúc phần mềm dựa trên chuẩn Gang of Four (GoF).
- [Bài 21: Singleton Pattern và Rủi ro Đa luồng (Multi-threading)](./notes/21-pattern-singleton.md)
- [Bài 22: Nhóm Khởi tạo: Factory Method và Abstract Factory](./notes/22-pattern-factory.md)
- [Bài 23: Builder Pattern và Rủi ro Telescoping Constructor](./notes/23-pattern-builder.md)
- [Bài 24: Nhóm Cấu trúc: Adapter và Facade Pattern](./notes/24-pattern-adapter-facade.md)
- [Bài 25: Strategy Pattern và Cơ chế Tách rời Thuật toán](./notes/25-pattern-strategy.md)
- [Bài 26: Observer Pattern và Kiến trúc Hướng Sự kiện (Event-Driven)](./notes/26-pattern-observer.md)
- [Bài 27: Decorator Pattern và Thiết kế Mở rộng Đối tượng động](./notes/27-pattern-decorator.md)

---
