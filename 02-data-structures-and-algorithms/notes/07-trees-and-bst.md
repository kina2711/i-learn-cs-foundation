# Bài 7: Phi Tuyến tính cơ sở: Cây (Trees) và Cây Nhị phân Tìm kiếm (BST)

Các tập dữ liệu trong thực tế không chỉ nằm ngang hàng nối tiếp nhau. Sơ đồ tổ chức nhân sự của một tập đoàn, cấu trúc thư mục trên Hệ điều hành, hay cấu trúc thẻ HTML (DOM) của trình duyệt web là minh chứng rõ ràng cho dữ liệu mang tính **Thứ bậc (Hierarchical)**.

Cấu trúc **Cây (Tree)** được tạo ra để phản chiếu mối quan hệ Thứ bậc này.

---

## 1. Cấu trúc Cây Cơ bản

Cây là một đồ thị đặc biệt gồm các đối tượng rời rạc phân cấp. Điểm bắt đầu của hệ thống gọi là **Nút Gốc (Root Node)**. Root tham chiếu xuống các **Nút Con (Child Nodes)**, và các Child Nodes này lại tiếp tục sở hữu Nút con, hình thành nên các **Nút Lá (Leaf Nodes)** nằm ở tận cùng các nhánh.

Trong Khoa học Máy tính, một định dạng đặc thù của Tree được sử dụng cực kì rộng rãi là **Cây Nhị phân (Binary Tree)** - giới hạn ràng buộc mỗi Nút tối đa chỉ được lưu trữ tham chiếu đến 2 Nút con (Left Child và Right Child). Mặc dù đơn giản, Binary Tree vẫn chưa mang lại giá trị nào đáng kể cho thao tác tìm kiếm cho đến khi một quy tắc toán học đặc biệt được thêm vào.

---

## 2. Binary Search Tree (BST) - Sức mạnh Phân đôi

**Cây Nhị phân Tìm kiếm (BST)** thêm vào quy tắc sắp xếp giá trị một cách nghiêm ngặt trong nội bộ cấu trúc:
**Tất cả các Nút nằm bên nhánh Trái phải có giá trị nhỏ hơn Nút Cha, và tất cả Nút nằm bên nhánh Phải phải có giá trị lớn hơn Nút Cha.**

Quy tắc này biến quá trình dò tìm số trên BST hoạt động hệt như giải pháp Tìm kiếm Nhị phân.
Mỗi khi hệ thống đứng tại một điểm Nút, nó tự động so sánh: Nếu giá trị cần tìm nhỏ hơn Nút hiện tại, nó dứt khoát bỏ qua toàn bộ phần nhánh cây khổng lồ bên Phải và chỉ tiếp tục đệ quy quét theo nhánh cây bên Trái. 

Sự phân cực luồng đi này thiết lập mức giới hạn tìm kiếm ở con số $O(\log N)$.

```mermaid
graph TD
    subgraph BST Lý tưởng - O(log N)
        R((50)) --> L1((20))
        R --> R1((70))
        L1 --> L2((10))
        L1 --> R2((30))
        R1 --> L3((60))
        R1 --> R3((80))
    end
```
*Tra cứu giá trị 60: Bắt đầu từ 50 -> Rẽ Phải đến 70 -> Rẽ Trái tới 60. Chỉ mất 3 bước cho khối 7 Nút.*

---

## 3. Thao tác và Suy thoái Cấu trúc (Degradation)

- **Tìm kiếm (Search):** Do tính chất phân nhánh nhị phân, số lượng vòng rẽ bằng đúng chiều cao của cây ($h$). Tốc độ tìm kiếm chuẩn: $O(h)$.
- **Chèn (Insert) và Xóa (Delete):** Các thao tác này yêu cầu định tuyến xuống dưới đáy để tìm vị trí hợp lệ hoặc thay thế tham chiếu node, do đó tốc độ phụ thuộc chặt chẽ vào chiều cao $h$.

**Vấn đề Tối kỵ của BST: Sự Suy thoái (Degradation)**
Công thức cấu trúc $O(\log N)$ chỉ chính xác với một Cây Cân bằng (Balanced). Tuy nhiên, BST cơ sở không sở hữu cơ chế bảo vệ phân nhánh ngang.
Giả sử ta nhập một chuỗi số theo thứ tự tuần tự tăng dần: `10, 20, 30, 40, 50`. 

```mermaid
graph TD
    subgraph BST Bị Suy thoái - O(N)
        10 --> 20 --> 30 --> 40 --> 50
    end
```
Tất cả các Nút lớn hơn nên đều dồn hết về một nhánh Phải duy nhất, biến Cấu trúc Cây biến dạng khuyết tật và trở lại thành một Linked List tuyến tính tồi tệ ($O(N)$). Đây là rủi ro kinh điển khiến BST nguyên bản hiếm khi được dùng trực tiếp ở máy chủ Production.

Tại phần tiếp theo (Bài 8), chúng ta sẽ phân tích cách các nhà thiết kế thuật toán khắc phục giới hạn dị tật này để ứng dụng trong cơ sở dữ liệu.

---
**Navigation:**
[⬅️ Previous: Bài 6: Bảng băm (Hash Tables) và Thuật toán tra cứu $O(1)$](./06-hash-tables.md) | [Next: Bài 8: Cây Tự cân bằng (Self-Balancing Trees) và Quản trị Rủi ro O(N) ➡️](./08-balanced-trees.md)
