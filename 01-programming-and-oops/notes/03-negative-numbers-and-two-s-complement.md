# Bài 3: Biểu diễn số âm và Số bù hai (Two's Complement)

Ở Bài 1, chúng ta đã hiểu cách biểu diễn các số nguyên không âm (Unsigned Integer) thông qua hệ nhị phân. Tuy nhiên, dữ liệu thực tế đòi hỏi khả năng xử lý các giá trị âm (Signed Integer). Việc biểu thị dấu âm/dương trong một chuỗi bit, đồng thời tối ưu hóa khối xử lý tính toán (ALU) trong kiến trúc CPU là một bài toán kỹ thuật phức tạp đã trải qua nhiều thế hệ tối ưu.

---

## 1. Phương pháp Bit Dấu (Sign-Magnitude)

Giải pháp trực quan đầu tiên là dành riêng **Bit có trọng số lớn nhất (Most Significant Bit - MSB)** ở ngoài cùng bên trái để lưu trữ trạng thái dấu:
- `0`: Số dương.
- `1`: Số âm.
Các bit còn lại biểu diễn độ lớn (Magnitude) của số đó.

Giả sử chúng ta sử dụng kiến trúc 8-bit:
- Số `+5` = `00000101`
- Số `-5` = `10000101`

**Vấn đề của Sign-Magnitude:**
1. **Dư thừa số 0:** Phương pháp này sinh ra hai biểu diễn khác nhau cho số Không: `+0` (`00000000`) và `-0` (`10000000`). Điều này gây lãng phí một trạng thái tổ hợp và tạo thêm độ trễ khi so sánh logic kiểm tra (`if x == 0`).
2. **Tính toán phức tạp:** Nếu áp dụng phép cộng nhị phân thông thường cho `+5` và `-5`, kết quả thu được sẽ là `10001010` (tương đương `-10`), hoàn toàn sai lệch về mặt toán học. Để mạch điện tử thực hiện được phép tính, CPU phải được thiết kế với các bộ logic kiểm tra dấu và trừ riêng biệt, gây cồng kềnh cho phần cứng.

---

## 2. Số bù một (One's Complement)

Phương pháp này cải tiến bằng cách đảo ngược toàn bộ giá trị bit (0 thành 1, 1 thành 0) để tạo ra số âm tương ứng.
- Số `+5` = `00000101`
- Số `-5` (Bù một) = `11111010`

Phương pháp bù một cho phép thực hiện phép cộng tự nhiên hơn, tuy nhiên bài toán **"Hai số 0"** vẫn chưa được giải quyết (`00000000` và `11111111`). Sự phức tạp phần cứng vẫn tồn tại khi phải thiết kế xử lý tràn bit nhớ (end-around carry).

---

## 3. Chuẩn mực hiện đại: Số bù hai (Two's Complement)

Hầu hết mọi bộ vi xử lý và ngôn ngữ lập trình hiện đại đều sử dụng **Số bù hai** để biểu diễn số nguyên có dấu. Phương pháp này thanh lịch về mặt toán học và giải quyết triệt để vấn đề thiết kế ALU.

### Cơ chế tính Số bù hai
Để tìm dạng nhị phân biểu diễn số âm $-X$, ta thực hiện hai bước trên giá trị nhị phân của số dương $X$:
1. **Đảo ngược toàn bộ các bit (Bù một).**
2. **Cộng thêm 1 vào kết quả.**

Ví dụ: Biểu diễn số `-5` trong hệ 8-bit.
- Bắt đầu từ `+5`: `00000101`
- BƯỚC 1 (Đảo bit): `11111010`
- BƯỚC 2 (Cộng 1): `11111011`
$\rightarrow$ `11111011` chính là biểu diễn nhị phân của `-5`.

### Ưu điểm kiến trúc của Số bù hai

1. **Duy nhất một số 0:** Nếu bạn tìm số bù hai của `00000000`, kết quả sau khi đảo bit và cộng 1 vẫn trả về `00000000` (bit nhớ thừa bị loại bỏ do giới hạn thanh ghi 8-bit). 
2. **Đồng nhất hóa phép Cộng và Trừ:** CPU không cần sở hữu mạch trừ chuyên dụng. Phép trừ $A - B$ được máy tính biên dịch tự động thành $A + (-B)$. Bộ cộng đơn nguyên (Adder) trong ALU có thể xử lý cả số có dấu và không dấu bằng cùng một mạch vật lý.

```mermaid
graph TD
    subgraph Phép cộng: 5 + (-5) = 0
        A[00000101] -->|Cộng| R[1 00000000]
        B[11111011] -->|Cộng| R
    end
    
    R -.-> T[Tràn bit số 9 bị loại bỏ ở kiến trúc 8-bit]
    T -.-> F(Kết quả: 00000000)
```

---

## 4. Phân tích thực tiễn: Lỗi Tràn số (Integer Overflow)

Trong kiến trúc Số bù hai 8-bit, MSB vẫn phản ánh tính âm dương, nhưng dải biểu diễn bị dịch chuyển.
- Giá trị dương cực đại: `01111111` ($+127$)
- Giá trị âm cực đại: `10000000` ($-128$)

Một lỗi hệ thống phổ biến xảy ra khi giá trị vượt quá khả năng biểu diễn của thanh ghi. Nếu bạn cộng 1 vào giá trị dương cực đại (`+127`), do nguyên lý nhị phân, các bit bị lật thành `10000000` (đây lại là biểu diễn của `-128`).
Quá trình giá trị đột ngột lật từ số dương cực đại sang số âm cực đại được gọi là **Tràn số (Integer Overflow)**. Việc phân tích kỹ dải dữ liệu để lựa chọn kiểu dữ liệu phù hợp (ví dụ: `int32` vs `int64` trong cơ sở dữ liệu) là nguyên tắc cốt lõi để đảm bảo độ tin cậy của phần mềm.

---
**Navigation:**
[⬅️ Previous: Bài 2: Bảng mã ASCII, Unicode và Cơ chế của UTF-8](./02-character-encodings.md) | [Next: Bài 3: Máy tính biểu diễn số âm như thế nào? (Negative Numbers & Two's Complement) ➡️](./03-negative-numbers.md)
