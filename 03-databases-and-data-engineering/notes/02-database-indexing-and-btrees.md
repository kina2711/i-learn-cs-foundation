# Bài 2: Kiến trúc Indexing Cốt lõi: B-Tree và Hash Index

Nhằm phá vỡ giới hạn truy xuất chậm chạp của cấu trúc ổ đĩa tĩnh (Disk I/O), Hệ quản trị Cơ sở dữ liệu triển khai một cấu trúc dữ liệu không gian phụ trợ (Auxiliary Space) gọi là **Chỉ mục (Index)**. 

Bản chất của việc tạo Chỉ mục trên một cột dữ liệu chính là việc CSDL khởi tạo và duy trì song song một cấu trúc Cây hoặc Bảng băm rút gọn của riêng cột đó, lưu trữ các con trỏ vật lý trỏ trực tiếp đến địa chỉ dòng dữ liệu gốc trên ổ cứng.

---

## 1. Nút thắt Của Cây Nhị phân Tìm kiếm (BST) trên Đĩa

Trong quá trình thiết kế, giới nghiên cứu máy tính đã thử nghiệm đặt Cây Nhị phân Cân bằng (Red-Black Tree) vào thiết kế CSDL (Tham khảo Bài 8, Part 2). Mặc dù chúng có độ phức tạp thuật toán cực mượt là $O(\log N)$ trên bộ nhớ RAM, chúng sụp đổ hoàn toàn trên hệ thống Disk.

**Tại sao BST thất bại?**
Hệ điều hành tương tác với ổ cứng qua từng Page cố định (ví dụ 8KB). 
Một Node của BST chỉ chứa được chính xác 1 Khóa (Key) và 2 con trỏ nhánh (Trái, Phải) - tiêu tốn khoảng 16 Bytes. 
Nếu lưu 1 Node này vào một Page 8KB, nó sẽ làm lãng phí 99.9% dung lượng truy xuất I/O. Hơn nữa, vì mỗi Node chỉ chia đôi mảng, cây BST 1 triệu bản ghi có chiều sâu lên tới 20 tầng. Truy xuất đến tầng đáy đòi hỏi hệ thống phải phát lệnh nhảy từ Page đĩa này sang Page đĩa khác 20 lần (20 Random Disk I/O) - tạo ra độ trễ khổng lồ (vài trăm mili-giây).

---

## 2. Giải pháp Kiến trúc: B-Tree và B+Tree

**B-Tree (Balanced Tree)** được thiết kế để bù đắp điểm mù của BST thông qua một chiến lược: **Tăng cường dung lượng trên mỗi Node để san bằng cấu trúc (N-ary Tree).**

Thay vì một Node chứa 1 Key, hệ thống nhồi nhét vào Node đó hàng trăm Keys và phân ra hàng trăm con trỏ nhánh (Fanout), với điều kiện: **Kích thước một Node phải vừa vặn hoàn hảo bằng đúng kích thước của 1 Page trên Đĩa.**

```mermaid
graph TD
    subgraph "B+Tree Structure (Cấu trúc phổ biến nhất trong RDBMS)"
        Root[""Root Page<br/>[100, 500, 1000"]"]
        L1[""Branch Page 1<br/>[10, 50, 80"]"]
        L2[""Branch Page 2<br/>[200, 350, 450"]"]
        L3[""Branch Page 3<br/>[600, 800"]"]
        
        Leaf1[""Leaf Page 1<br/>[10->Disk Address, 50->..., 80->..."]"]
        Leaf2[""Leaf Page 2<br/>[100->..., 200->..., 350->..."]"]
        Leaf3[""Leaf Page 3<br/>[450->..., 500->..., 600->..."]"]
        
        Root -->|< 100| L1
        Root -->|100 - 499| L2
        Root -->|500 - 999| L3
        
        L1 --> Leaf1
        L2 --> Leaf2
        L3 --> Leaf3
        
        Leaf1 -.->|Linked List kết nối ngang| Leaf2 -.->|Linked List kết nối ngang| Leaf3
    end
```

### Ưu điểm kỹ thuật của B+Tree
Trong định dạng cải tiến B+Tree (Được áp dụng bởi InnoDB của MySQL hay PostgreSQL):
1. Các Node trung gian (Root, Branch) chỉ làm nhiệm vụ điều hướng thuần túy, không chứa dữ liệu thật. Do đó, một Page 8KB có thể nén được tới hơn 500 con trỏ điều hướng.
2. Với sức chứa 500 con trỏ, Cây B+Tree sâu 3 tầng có thể bao phủ: $500 \times 500 \times 500 = 125,000,000$ bản ghi. CPU **chỉ cần thực thi tối đa 3 lần Disk I/O** để tra cứu ra bản ghi.
3. Toàn bộ Node Lá (Leaf Node - đáy cùng) được nối với nhau bằng Cấu trúc Danh sách Liên kết Kép (Doubly Linked List). Cấu trúc này làm nên sức mạnh tuyệt đối của B+Tree khi xử lý các truy vấn mảng phạm vi (Range Query: `SELECT * WHERE id BETWEEN 100 AND 500`). CPU tìm xuống Lá 100, và duyệt ngang RAM liên tục đến Lá 500 mà không cần tìm kiếm lại từ đầu rễ.

---

## 3. Hash Index (Chỉ mục Băm)

Khác với kiến trúc phân nhánh của B-Tree, **Hash Index** áp dụng thuật toán Bảng băm (Hash Table) vào cơ sở dữ liệu.

Cột khóa (Key) sẽ được nạp vào Hash Function (Bài 6, Part 2) để phát sinh trực tiếp kết quả là địa chỉ con trỏ của dòng dữ liệu vật lý. Tốc độ tra cứu là hoàn hảo $O(1)$. 

**Ranh giới Kỹ thuật:**
- **Ưu điểm:** Khả năng truy xuất ngẫu nhiên khớp chính xác (Exact-match Query) vô đối: `SELECT * FROM users WHERE email = 'admin@sys.com'`. Hash Index chiếm ưu thế tại các CSDL In-memory (Redis).
- **Hạn chế Nghiêm trọng:** Vì mã Hash xáo trộn hoàn toàn chuỗi toán học, các phần tử `30` và `31` hoàn toàn không còn nằm cạnh nhau trên không gian hàm. Hash Index sụp đổ khi nhận được các tập lệnh truy xuất giới hạn (Range Query) như `WHERE age > 18` hoặc `ORDER BY age`. Hệ thống bắt buộc phải bỏ qua Hash Index và thực hiện Full Table Scan (Duyệt càn $O(N)$ toàn bảng).

Do tính chất toàn năng trong cả hai nhiệm vụ: Tìm kiếm chính xác (Point Lookup) và Quét dải tuyến tính (Range Query), Cấu trúc **B+Tree** được các kiến trúc sư công nhận là tiêu chuẩn Vàng mặc định của toàn bộ ngành Cơ sở dữ liệu truyền thống hiện nay.

---
**Navigation:**
[⬅️ Previous: Bài 1: Phân tầng Lưu trữ, OLTP vs OLAP và Kiến trúc Dòng/Cột](./01-oltp-vs-olap-and-storage-engines.md) | [Next: Bài 3: Giao dịch ACID và Nhật ký Ghi trước (WAL) ➡️](./03-wal-and-acid-transactions.md)
