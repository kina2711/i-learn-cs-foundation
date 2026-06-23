# Bài 4: Số thực dấu phẩy động (Floating-Point) và Chuẩn IEEE 754

Trong toán học, tập hợp số thực chứa đựng các giá trị thập phân (như `3.14`) hoặc các số phân số với độ chính xác vô hạn (như `1/3`). Tuy nhiên, bộ nhớ máy tính là hữu hạn, việc biểu diễn số thực đòi hỏi một cơ chế mã hóa đặc biệt nhằm cân bằng giữa hai yếu tố: **Phạm vi biểu diễn (Range)** và **Độ phân giải/Độ chính xác (Precision)**.

Tiêu chuẩn mã hóa thống trị toàn cầu hiện nay cho vấn đề này là **IEEE 754**.

---

## 1. Cơ chế biểu diễn Dấu phẩy động

Khái niệm "Dấu phẩy động" (Floating-Point) lấy cảm hứng từ ký hiệu khoa học (Scientific Notation). Thay vì lưu trữ cố định dấu thập phân ở một vị trí cụ thể trong dãy bit (Fixed-Point), dấu thập phân được phép "trôi nổi" dựa trên hệ số nhân lũy thừa.

Trong hệ thập phân, số $123.45$ có thể được viết dưới định dạng chuẩn hóa:
$1.2345 \times 10^2$

Chuẩn IEEE 754 áp dụng cơ chế tương tự nhưng trên hệ cơ số 2:
**$(-1)^S \times M \times 2^E$**

### Cấu trúc chuỗi bit chuẩn IEEE 754 (Kiểu Float 32-bit)

Một biến `float` (Độ chính xác đơn - Single Precision) có độ dài 32 bit, được chia thành 3 phân vùng (Segments) độc lập:

1. **Bit dấu (Sign Bit - S): 1 bit**
   - Nằm ở vị trí ngoài cùng bên trái. Xác định giá trị là dương (`0`) hay âm (`1`).
2. **Số mũ (Exponent - E): 8 bit**
   - Xác định phạm vi độ lớn của số (cực kỳ nhỏ đến cực kỳ lớn). Chuẩn này sử dụng cơ chế *Bù số mũ (Exponent Bias)* để lưu trữ cả số mũ âm và dương mà không cần dùng đến Two's Complement. Đối với 32-bit, Bias cố định là $127$.
3. **Phần định trị (Mantissa / Fraction - M): 23 bit**
   - Xác định độ chính xác các chữ số có nghĩa của giá trị. Vì ở hệ nhị phân chuẩn hóa, bit đầu tiên luôn là `1` (giống số $1.xxx$ ở hệ thập phân), chuẩn IEEE 754 thiết kế để "bỏ ẩn" số 1 này ở ngoài phần cứng nhằm tiết kiệm 1 bit lưu trữ.

```mermaid
graph LR
    subgraph Biểu diễn Float 32-bit (IEEE 754)
        S[1 bit<br/>Sign] --- E[8 bit<br/>Exponent] --- M[23 bit<br/>Mantissa]
    end
    
    style S fill:#dc3545,color:#fff
    style E fill:#007bff,color:#fff
    style M fill:#28a745,color:#fff
```

Với cấu trúc này, máy tính có thể phân bổ linh hoạt: nếu cần biểu diễn số cực nhỏ, Số mũ sẽ gánh vác việc dịch chuyển dấu phẩy. Nếu cần độ phân giải cao, Phần định trị sẽ làm nhiệm vụ ghi nhớ cấu trúc thập phân. Do số Byte của `float` là cố định (4 Bytes), chúng ta không thể cùng lúc đạt được cả phạm vi vô tận và độ chính xác tuyệt đối.

---

## 2. Giới hạn toán học: Hiện tượng suy giảm độ chính xác

Sử dụng cơ số 2 để mô phỏng các phân số của cơ số 10 gây ra hệ quả sai số tích lũy nghiêm trọng.

Trong hệ thập phân, phân số $1/3$ không thể biểu diễn chính xác dưới dạng độ dài hữu hạn ($0.3333\dots$).
Tương tự, trong hệ nhị phân, các phân số như $0.1$ hoặc $0.2$ (vốn rất tròn trịa trong hệ thập phân) lại trở thành những dãy bit lặp vô hạn tuần hoàn. Vì biến `float` (hoặc `double` 64-bit) có giới hạn độ dài bit Mantissa, phần đuôi vô hạn đó bắt buộc phải bị cắt xén (Rounding) để lưu vào RAM.

Đó là nguyên nhân căn bản giải thích cho hiện tượng tính toán kinh điển trong khoa học máy tính:
```python
>>> 0.1 + 0.2
0.30000000000000004
```
Vì $0.1$ và $0.2$ khi đưa vào bộ nhớ đã bị cắt xén nên tổng của chúng sẽ chệch một sai số nhỏ (Epsilon) so với giá trị $0.3$ nguyên bản.

---

## 3. Phân tích hệ thống và Ứng dụng thực tiễn

Hệ quả của chuẩn IEEE 754 buộc các kỹ sư phần mềm phải thiết lập các quy chuẩn nghiêm ngặt trong một số nghiệp vụ nhất định:

1. **Tuyệt đối không dùng toán tử `==` để so sánh hai số thực:**
   Vì sai số tiềm ẩn, hệ thống không nên kiểm tra `if (a == b)`. Thuật toán đúng đắn là kiểm tra trị tuyệt đối của hiệu số so với một ngưỡng dung sai (Epsilon) rất nhỏ: `if (abs(a - b) < 0.000001)`.
   
2. **Xử lý Dữ liệu Tài chính (Financial Systems):**
   Trong các nền tảng kế toán, ngân hàng hay thương mại điện tử, việc sử dụng `float` hoặc `double` để lưu trữ biến động tiền tệ là một **lỗi thiết kế (Design Flaw)** nghiêm trọng, có thể làm thất thoát tiền do sai số làm tròn khi xử lý khối lượng lớn giao dịch. 
   **Giải pháp:** Các hệ thống này bắt buộc sử dụng cơ chế số học thập phân (ví dụ: `BigDecimal` trong Java, `decimal` trong Python), hoặc quy đổi đơn vị về số nguyên nhỏ nhất (lưu trữ giá trị `$10.50` dưới định dạng kiểu nguyên `1050` cents).

---
**Navigation:**
[⬅️ Previous: Bài 4: Tại sao 0.1 + 0.2 != 0.3? (Floating Point Math & IEEE 754)](./04-floating-point-math.md) | [Next: Bài 5: Thứ tự Byte (Endianness) và Định dạng Dữ liệu Mạng ➡️](./05-endianness.md)
