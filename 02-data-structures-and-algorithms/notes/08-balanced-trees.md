# Bài 8: Cây Tự cân bằng (Self-Balancing Trees) và Quản trị Rủi ro O(N)

Để giải quyết vấn đề cấu trúc Cây Nhị phân Tìm kiếm (BST) bị thoái hóa dạng chuỗi thành danh sách liên kết dẫn tới hiệu suất sụt giảm về $O(N)$ (Bài 7), các nhà toán học đã kiến tạo ra một giải pháp bù đắp: **Thuật toán tự động tái cơ cấu chiều cao (Self-Balancing Trees)**. 

Trọng tâm của tính năng tự cân bằng xoay quanh hai biến thể quan trọng nhất: **AVL Tree** và **Red-Black Tree (Cây Đỏ Đen)**.

---

## 1. Cơ chế Hoạt động: Tree Rotations (Xoay Nút)

Cốt lõi của việc kiểm soát Cây cân bằng nằm ở khả năng **Tái xoay cấu trúc (Rotations)**. Thuật toán giám sát liên tục sau mỗi thao tác Chèn (`insert`) hoặc Xóa (`delete`). Nếu phát hiện một nhánh cây dài quá mức cho phép, hệ thống sẽ thực hiện các thao tác Xoay Trái (Left Rotation) hoặc Xoay Phải (Right Rotation) để nâng các Node ở phía dưới lên thành Root mới, san bằng phân cấp của cả hai nhánh.

Toàn bộ thao tác hoán đổi con trỏ để quay Nút chỉ tiêu tốn một hằng số $O(1)$, vì bản chất nó chỉ điều chỉnh lại đường link giữa các đối tượng trong Heap.

```mermaid
graph LR
    subgraph Trạng thái Lệch (Unbalanced)
        A((A)) -->|Phải| B((B))
        B -->|Phải| C((C))
    end
    
    subgraph Xoay Trái (Left Rotation)
        B2((B)) -->|Trái| A2((A))
        B2 -->|Phải| C2((C))
    end
    
    A -.->|Xoay Trái quanh A| B2
```

---

## 2. AVL Tree - Giới hạn chênh lệch tuyệt đối

**Nguyên tắc cấu trúc:** AVL Tree ràng buộc một hệ luật cân bằng siêu ngặt nghèo: *Sự chênh lệch chiều cao giữa Nhánh Trái và Nhánh Phải tại bất kỳ Node nào cũng không được phép vượt quá 1 tầng.*

**Đánh giá Hiệu năng:**
Vì luôn duy trì trạng thái hoàn hảo đối xứng nhất có thể, thời gian Tìm kiếm (`Search`) trên AVL Tree luôn chắc chắn đạt tốc độ tối đa $O(\log N)$. 

Tuy nhiên, giới hạn này cũng là một con dao hai lưỡi. Việc duy trì quy tắc chênh lệch 1 khiến AVL Tree bị mắc kẹt với hàng loạt các thao tác `Rotation` cực kì dày đặc mỗi khi hệ thống có tác vụ Ghi/Xóa dữ liệu (Write operations). Trong các hệ thống giao dịch tốc độ cao với tầng chèn dữ liệu cường độ lớn, chi phí cân bằng sẽ kéo tụt hiệu năng phần cứng đáng kể.

---

## 3. Red-Black Tree - Cân bằng Thống kê tương đối

Thay vì cầu toàn khắt khe như AVL, **Cây Đỏ Đen (Red-Black Tree)** định hình khái niệm cân bằng Tương đối. 

**Nguyên tắc cấu trúc:** Mỗi Nút được gán thêm 1 Bit thông tin cấu hình màu sắc (Đỏ hoặc Đen). Các quy tắc phối màu đảm bảo rằng *Đường dẫn dài nhất từ Root đến chiếc lá xa nhất không bao giờ được phép dài quá gấp đôi đường dẫn ngắn nhất.* 

**Đánh giá Hiệu năng:**
Sự nới lỏng giới hạn (chấp nhận chênh lệch tầng gấp đôi) cho phép Red-Black Tree giảm thiểu các thao tác `Rotation` không cần thiết. Khi một Node mới được chèn, hệ thống thường giải quyết sự cố lệch nhẹ bằng cách **Đổi Màu Node** (Thao tác O(1) chỉ sửa đổi 1 Bit) thay vì Xoay cấu trúc. 
- Tìm kiếm (Read): $O(\log N)$ - Chậm hơn AVL một chút vì độ dài đường dẫn có thể lớn hơn.
- Chèn/Xóa (Write): Tối đa duy trì cực tốt tại $O(\log N)$, ít xoay hơn AVL đáng kể.

### Ứng dụng Thực tiễn trong Ngôn ngữ cốt lõi

Do sự vượt trội trong khả năng trung hòa rủi ro đọc và ghi dữ liệu, Cây Đỏ Đen được chọn làm nền tảng cấu trúc (Under-the-hood) của hầu hết các đối tượng Collection hệ thống nổi tiếng:
- Kiểu `TreeMap` và `TreeSet` trong hệ sinh thái Java Collections Framework.
- Kiểu `std::map` và `std::set` trong Thư viện Chuẩn C++ (STL).

### So sánh mở rộng với Bảng băm (Hash Tables)

Một bảng Hash Table có tốc độ tra cứu đỉnh cao $O(1)$ (Tham khảo Bài 6), vậy tại sao ta vẫn cần cấu trúc Cây $O(\log N)$ để lưu trữ đối tượng dạng Key-Value? 

Câu trả lời nằm ở Thuộc tính Trật Tự (Ordering): Bảng băm trải rác ngẫu nhiên các khóa, khiến việc xuất dữ liệu theo đúng danh sách thứ tự tăng/giảm dần là bất khả thi nếu không tiến hành sắp xếp (Sort) lại toàn bộ kho mảng $O(N \log N)$. Trong khi đó, BST/Red-Black Tree bảo lưu kết cấu thứ tự toán học vững chãi, việc xuất mảng theo trật tự tự nhiên (In-order Traversal) chạy trơn tru với chi phí ổn định $O(N)$.

---
**Navigation:**
[⬅️ Previous: Bài 7: Phi Tuyến tính cơ sở: Cây (Trees) và Cây Nhị phân Tìm kiếm (BST)](./07-trees-and-bst.md) | [Next: Bài 9: Phi Tuyến tính Bậc cao: Đồ thị (Graphs) ➡️](./09-graphs.md)
