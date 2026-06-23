# Bài 6: Toán tử Thao tác Bit (Bitwise Operations)

Ở mức độ thiết kế thuật toán cấp cao, chúng ta thường thao tác trên dữ liệu thông qua các phép toán số học truyền thống (cộng, trừ, nhân, chia) và phép so sánh logic (boolean). Tuy nhiên, các cấu trúc dữ liệu nguyên thủy được lưu trữ dưới dạng mảng bit nhị phân. Các tập lệnh của vi xử lý cung cấp những **Toán tử Thao tác Bit (Bitwise Operations)** cho phép xử lý trực tiếp trên các bit riêng lẻ của dữ liệu.

Nhờ việc tương tác trực tiếp với mạch logic số lượng cổng thấp, Bitwise có tốc độ thực thi nhanh nhất trong toàn bộ tập lệnh xử lý của CPU.

---

## 1. Cơ sở Logic của các Phép toán Bitwise

Các thao tác bitwise áp dụng độc lập trên từng bit song song ở cùng vị trí của hai toán hạng.

- **AND (`&`):** Kết quả là `1` chỉ khi cả hai bit đầu vào đều là `1`. Thường dùng để xóa (mask-off) một nhóm bit (giữ lại các bit cần thiết, ép các bit còn lại về 0).
- **OR (`|`):** Kết quả là `1` nếu một trong hai hoặc cả hai bit đầu vào là `1`. Thường dùng để bật (set) một bit lên 1 mà không ảnh hưởng tới các bit khác.
- **XOR (`^` - Exclusive OR):** Kết quả là `1` nếu hai bit đầu vào khác nhau. Nếu giống nhau (0-0 hoặc 1-1), kết quả trả về `0`. Kỹ thuật XOR rất hữu dụng để đảo trạng thái (toggle) một tập hợp bit nhất định.
- **NOT (`~`):** Phép toán một ngôi. Lật ngược toàn bộ trạng thái bit của toán hạng (0 thành 1, 1 thành 0). Đây là cơ sở của Số bù một (Bài 3).

---

## 2. Toán tử Dịch Bit (Bit Shifts)

Phép dịch bit di chuyển toàn bộ dải bit của dữ liệu sang trái hoặc sang phải theo một biên độ (offset) chỉ định.

### Dịch trái (Left Shift `<<`)
Đẩy tất cả các bit sang trái $n$ vị trí. Các bit tràn ở bên trái bị loại bỏ, các bit trống ở bên phải được lấp đầy bằng số `0`.
Ví dụ: `00000101` (số 5) dịch trái 1 vị trí (`5 << 1`) tạo ra kết quả `00001010` (số 10).
- **Phân tích toán học:** Dịch trái $n$ bit tương đương với phép nhân giá trị số học cho $2^n$. CPU xử lý phép dịch `<< 1` nhanh hơn rất nhiều so với phép nhân `* 2` bằng mạch số nhân (Multiplier).

### Dịch phải (Right Shift `>>` / `>>>`)
- **Arithmetic Right Shift (`>>`):** Dịch bit sang phải và lấp đầy các vị trí trống bằng giá trị của MSB (Bit Dấu) nhằm giữ nguyên dấu âm dương của số (Two's Complement). Việc dịch phải $n$ bit tương đương với phép chia cho $2^n$ (có làm tròn xuống).
- **Logical Right Shift (`>>>`):** Dịch phải nhưng lấp đầy vị trí trống hoàn toàn bằng số `0`, bất chấp dấu của giá trị. Áp dụng khi cần thao tác chuỗi bit như một mẫu dữ liệu (bit pattern) không mang tính số học nguyên âm (Unsigned).

---

## 3. Phân tích Ứng dụng Kỹ thuật Hệ thống

Kỹ thuật lập trình Bitwise thường xuyên được áp dụng trong các module cần tối ưu hiệu năng hoặc quản lý tài nguyên khắt khe.

### 1. Kỹ thuật Quản lý Trạng thái (Bitmasks / Flags)
Thay vì sử dụng mảng Boolean yêu cầu cấu trúc phức tạp và tiêu tốn nhiều bộ nhớ, một biến số nguyên `int32` duy nhất có thể được sử dụng làm một vector quản lý cùng lúc 32 trạng thái cờ logic (Flags). Cơ chế này phổ biến trong thiết kế Hệ điều hành (ví dụ: cấp quyền Đọc-Ghi-Thực thi tập tin).
- Để kiểm tra Cờ thứ $k$ có được bật hay không: `(Flags & (1 << k)) != 0`
- Để bật Cờ thứ $k$: `Flags = Flags | (1 << k)`

### 2. Tối ưu hóa thuật toán
- **Kiểm tra số chẵn lẻ:** Lệnh `(x % 2 == 0)` có thể được thay thế bằng lệnh `(x & 1 == 0)`. Việc kiểm tra bit LSB phản ánh trực tiếp tính chẵn lẻ, tránh chi phí thực thi tốn kém của phép chia lấy dư bằng mạch ALU.
- **Tính toán chỉ số Bộ đệm Vòng (Circular Buffer):** Với các mảng có dung lượng $N = 2^k$ (ví dụ: cơ chế HashMap nội bộ), phép toán lấy module mảng `(index % N)` thường được tối ưu bằng Bitwise AND: `(index & (N - 1))`.

Việc ứng dụng toán tử Bitwise là dấu hiệu phân biệt giữa khả năng thiết kế ở cấp độ luồng làm việc logic và khả năng nắm bắt vi kiến trúc xử lý của các hệ thống hiệu năng cao (High-Performance Computing).

---
**Navigation:**
[⬅️ Previous: Bài 5: Thứ tự Byte (Endianness) và Định dạng Dữ liệu Mạng](./05-endianness.md) | [Next: Bài 7: Tổng quan Mô hình Bộ nhớ (Memory Layout) của Tiến trình ➡️](./07-memory-segments.md)
