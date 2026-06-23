# Bài 6: Bảng băm (Hash Tables) và Thuật toán tra cứu $O(1)$

Cho đến thời điểm hiện tại, rào cản vĩ đại nhất của các tổ hợp dữ liệu Mảng và Danh sách liên kết là sự chậm chạp của tính năng Tra cứu Tìm kiếm. Để kiểm tra sự tồn tại của một phần tử, hệ thống buộc phải quét tuyến tính từ đầu mảng đến cuối mảng ($O(N)$). Với bài toán Hệ thống quản lý thẻ Tín dụng của một ngân hàng, quá trình tuyến tính để rà soát hàng chục triệu mã thẻ là điều không tưởng.

Để hiện thực hóa tính năng Tìm kiếm với độ phân tán nháy mắt **$O(1)$**, Khoa học Máy tính đã ra mắt một phát minh mang tính cách mạng: **Bảng băm (Hash Tables / Dictionaries / Maps)**.

---

## 1. Cơ chế vận hành của Hàm Băm (Hash Functions)

Trung tâm sức mạnh của Bảng băm không nằm ở tổ chức cấu trúc, mà nằm ở hệ thống giải thuật **Hàm Băm (Hash Function)**.

**Hàm băm là gì?**
Đó là một bộ thuật toán chuyển hóa đặc biệt, cho phép tiếp nhận một tham số đầu vào (Chuỗi chuỗi ký tự, Chuỗi số ngẫu nhiên) và sản sinh ra một **Mã băm (Hash Code) duy nhất có độ dài cố định**. 

Nguyên tắc của Hàm băm lý tưởng:
1. **Tính Nhất quán (Deterministic):** Cùng một đầu vào `Alex` luôn luôn cho ra một kết quả băm giống nhau, ví dụ `105`. Bất kể chạy bao nhiêu ngàn lần.
2. **Tính Phân tán rải rác (Uniformity):** Đầu vào `Alex` và `Alez` (chỉ sai lệch 1 ký tự) phải sinh ra kết quả băm sai lệch hàng vạn đơn vị, tránh chụm cục bộ cụm dữ liệu.
3. **Hiệu suất (Efficiency):** Luồng tính toán phải được CPU biên dịch nhanh cực độ, vì mọi thông lượng ghi đọc vào kho dữ liệu đều phải đi ngang Hàm băm.

```mermaid
graph LR
    K[Mã thẻ Tín Dụng<br/>"5432-1111-XXXX"] -->|Truyền Dữ liệu| H(Hàm Băm hệ thống<br/>Hash Function)
    H -->|Xử lý O_1| I[Mã số Nguyên: 4]
    
    subgraph Bảng Băm gốc (Array Cơ sở)
        A0[Index 0]
        A1[Index 1]
        A2[Index 2]
        A3[Index 3]
        A4[Index 4: Lưu thông tin thẻ Alex]
    end
    
    I -.->|Trỏ trực tiếp O_1| A4
    
    style H fill:#d1ecf1,stroke:#17a2b8
    style I fill:#fff3cd,stroke:#ffc107
    style A4 fill:#d4edda,stroke:#28a745
```

### Kiến trúc phân tầng Bảng Băm
Bản chất cơ học vật lý của Bảng băm thực chất là một mảng Mảng tĩnh kích thước rộng (Array).
Khi muốn lưu trữ giá trị `("Alex", Tiền: $500)`:
1. Đẩy khóa chuỗi `"Alex"` vào thuật toán Hash, kết quả sinh ra hệ số nguyên lớn: `34125890`.
2. Hệ thống áp dụng phép chia lấy dư với kích thước giới hạn của Mảng (ví dụ mảng dài 10): `34125890 % 10 = 0`. Chỉ số vị trí được ấn định là Vị trí 0.
3. Bỏ dữ liệu `$500` vào ô nhớ mảng `Array[0]`. Thao tác tính toán này hoàn tất tại $O(1)$.

Trong thao tác tìm kiếm sau này, nếu gọi xuất `"Alex"`, cỗ máy lại lặp lại thuật toán băm ra số `0`. Rồi đi ngược về ổ cứng truy xuất giá trị `Array[0]` để trả về dữ liệu `$500`. Hiệu suất cực đại vì không tốn công rà soát dữ liệu.

---

## 2. Vấn đề Xung đột (Collision) và Cấu trúc giải quyết

Mọi thứ dường như hoàn hảo, ngoại trừ một giới hạn toán học: Số lượng chuỗi ký tự trên đời là vô hạn, trong khi kích thước mảng tĩnh là hữu hạn. Chắc chắn sẽ có hiện tượng hai chuỗi dữ liệu (như `"Alex"` và `"John"`) vô tình sau quá trình Băm lại cùng cho ra chung một chỉ số dư là `Index 0`. Tình trạng 2 khối dữ liệu tranh chấp không gian bộ nhớ này là **Xung đột mã băm (Hash Collision)**.

Để giải quyết Xung đột, hai thuật toán cơ sở được áp dụng:

1. **Phương pháp Rẽ nhánh (Separate Chaining):**
   Thay vì lưu giá trị tĩnh trong Mảng, mỗi ô của mảng giờ đây trở thành một khối trỏ định vị của Danh sách liên kết (Linked List). Khi hai giá trị xung đột tại ô nhớ `0`, dữ liệu thứ hai sẽ được tạo một Nút rời rạc và móc nối đuôi vào sau khối dữ liệu thứ nhất. 
   **Rủi ro:** Khi danh sách liên kết quá dài (Vượt 8 Node liên tục), thời gian tra cứu sẽ biến chất từ $O(1)$ suy thoái cực độ xuống $O(N)$. Ngôn ngữ Java 8+ tối ưu lại bằng cách nếu chuỗi đụng độ lớn hơn 8 đơn vị, cấu trúc tự dịch chuyển từ Linked List sang Tree cân bằng để hãm rủi ro xuống mốc giới hạn $O(\log N)$.

2. **Phương pháp Dò tuyến tính (Open Addressing / Linear Probing):**
   Nếu ô số `0` đã bị chiếm cứ, thuật toán ra lệnh dò tìm tuần tự xuống ô kế tiếp (Ô `1`, Ô `2`...) cho đến khi phát hiện khoảng không còn trống để chứa. Phương pháp này bảo toàn việc dùng Mảng khép kín nguyên vẹn, nhưng lại nảy sinh hiệu ứng Tụ đám (Clustering) cản trở hiệu năng nếu mảng chạm mức sử dụng trên 70%.

---

## 3. Tổng kết Cấu trúc Tuyến tính

Dòng Cấu trúc dữ liệu tuyến tính (Linear Structures) gồm Array, Linked List, Stack, Queue và Hash Table tạo nên vỏ bọc hạ tầng cho hầu hết mọi chức năng lưu trữ 1 chiều thông dụng nhất của lập trình. Ở phần sau, chúng ta sẽ xem xét cấu trúc lưu trữ 2 chiều phức tạp hơn để biểu diễn dữ liệu có sự gắn kết thứ bậc: Mạng lưới Phi tuyến tính.

---
**Navigation:**
[⬅️ Previous: Bài 5: Cấu trúc Trạng thái: Ngăn xếp (Stack) và Hàng đợi (Queue)](./05-stack-and-queue.md) | [Next: Bài 7: Phi Tuyến tính cơ sở: Cây (Trees) và Cây Nhị phân Tìm kiếm (BST) ➡️](./07-trees-and-bst.md)
