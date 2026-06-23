# Bài 8: Cơ sở dữ liệu Tài liệu (Document DB) và Cấu trúc BSON

Hệ quản trị CSDL Quan hệ (RDBMS) như MySQL được xây dựng dựa trên nguyên lý **Chuẩn hóa chặt chẽ (Normalization)**. Dữ liệu bị băm nhỏ và ép vào khuôn khổ các Bảng (Tables) cố định. 
Tuy nhiên, trong một số nghiệp vụ lập trình hiện đại (Log hệ thống biến đổi, Catalog hàng hóa điện tử phong phú đa dạng), việc một thuộc tính có thể biến thiên linh hoạt ở từng dòng là rất cần thiết. Để đáp ứng luồng lưu trữ phi định dạng này, hệ sinh thái NoSQL kiến tạo ra mô hình **Document Database**, tiêu biểu nhất là **MongoDB**.

---

## 1. Bản chất Kiến trúc: Document (Tài liệu)

Trong MongoDB, Không có khái niệm Table, Row, hay Column. Thay vào đó, dữ liệu được cấu trúc dưới dạng:
- **Collection** (Tương đương Table): Tập hợp nhóm các tài liệu.
- **Document** (Tương đương Row): Một bản ghi riêng rẽ.

Một Document là một cấu trúc dữ liệu khép kín, được lưu trữ dưới định dạng tương tự **JSON (JavaScript Object Notation)**. Đặc tính quan trọng nhất của nó là sự cho phép **Schema-less (Linh hoạt lược đồ)**.

Cùng nằm trong một Collection `users`, hai Document hoàn toàn có thể có cấu trúc khác biệt sâu sắc:
```json
// Document 1
{
  "_id": 1,
  "name": "Alice",
  "contact": "alice@gmail.com"
}

// Document 2 (Cùng Collection)
{
  "_id": 2,
  "name": "Bob",
  "addresses": [
     {"type": "Home", "city": "NY"},
     {"type": "Work", "city": "LA"}
  ]
}
```

**Sức mạnh của Dữ liệu Lồng ghép (Nested/Embedded Data):**
Trong RDBMS, để lấy thông tin của Bob và địa chỉ, CPU phải thực thi phép nối (JOIN) chậm chạp giữa bảng `users` và bảng `addresses`. 
Trong Document DB, do đặc tính nhúng lồng thẳng mảng địa chỉ vào bên trong Document gốc, CPU chỉ tốn đúng 1 thao tác I/O đọc ổ đĩa để kéo trọn vẹn toàn bộ hồ sơ khách hàng. Độ trễ Read Latency được giảm xuống cực đại.

---

## 2. Nền tảng Kỹ thuật Hệ thống: Định dạng BSON

Mặc dù biểu diễn cú pháp cho lập trình viên dưới dạng văn bản JSON, dưới góc nhìn của ổ cứng (Disk Layout), lưu trữ JSON thô sẽ tốn không gian nghiêm trọng vì bản chất chuỗi ký tự (String) của chúng. Do vậy, MongoDB chuyển đổi toàn bộ cấu trúc thành **BSON (Binary JSON)**.

**Sức mạnh của BSON so với JSON truyền thống:**
1. **Kiểu dữ liệu mở rộng:** Hỗ trợ chuẩn nguyên thủy cấp máy như Số nguyên 32-bit, Số thực 64-bit float, Timestamp, đối tượng ngày tháng (Date) thay vì việc phải dùng Text để mã hóa chúng trong JSON.
2. **Khả năng Định vị bộ nhớ (Memory Offsets):** Mỗi chuỗi phần tử trong BSON đều được bắt đầu bằng một biến ghi lại kích thước dung lượng Byte của phần tử đó. Lợi thế? Khi hệ thống đọc BSON và tìm kiếm biến `"city"`, nó không cần phải phân tích đọc lướt qua từng chữ cái văn bản tốn kém sức lực CPU; nó chỉ cần dùng thông số kích thước để "nhảy cóc" (Skip) con trỏ bộ nhớ tới thẳng vị trí của biến cần tìm. Tốc độ quét tăng lên cấp số nhân.

---

## 3. Ranh giới và Nút thắt Kiến trúc

Sự linh hoạt và lưu trữ lồng ghép đổi lại bằng những đánh đổi khốc liệt trong kiến trúc. Mặc dù các kỹ sư có xu hướng lạm dụng Document DB làm Data Lake lưu trữ mọi thứ, nó vẫn có những giới hạn phần cứng nghiêm trọng.

- **Vấn đề Dung lượng Ổ cứng (Storage Bloat):** Ở cơ sở dữ liệu có quan hệ (SQL), tên cột được định nghĩa một lần duy nhất trên Metadata hệ thống. Ngược lại, trong MongoDB, tên của thuộc tính (Ví dụ chữ `"addresses"`) bị lặp lại vật lý ở MỌI DÒNG document. Nếu bảng có 1 tỷ dòng, cái chữ `"addresses"` cũng bị lưu ổ cứng đúng 1 tỷ lần. Dẫn đến kích cỡ tệp Data Files bị phình to khủng khiếp.
- **Rủi ro Dữ liệu mồ côi (Anomalies):** Do sự vắng bóng của Khóa ngoại ràng buộc (Foreign Key), nếu hệ thống có Document A lưu thông tin Hóa đơn và liên kết thủ công với Document B (Tài khoản User). Nếu User bị xóa, Hóa đơn vẫn nằm đó trên Database nhưng tham chiếu User biến thành rác vô giá trị.
- **Nguy cơ Phân mảnh Đĩa (Disk Fragmentation):** Do cấu trúc BSON trên ổ đĩa là liên tục (Contiguous). Nếu kích thước của Document 2 bất ngờ bị phình to (do Bob nhập thêm hàng trăm địa chỉ mới), hệ thống sẽ không có đủ dung lượng rãnh ghi nối tiếp. DBMS sẽ phải xé tài liệu ra và di dời chuyển nhà (Relocate) Document 2 sang một Sector đĩa trống khác. Quá trình di dời dữ liệu nội sinh này cực kì hao tổn tài nguyên và sinh ra hàng loạt rãnh đĩa trống rác rưởi (Fragmentation).

Sự hiểu biết sâu sắc này buộc Data Engineer phải quy hoạch: Document DB rất tuyệt vời trong các kho Log theo thời gian thực (Event Logs) hoặc Danh mục hàng hóa E-commerce, nhưng tuyệt đối không dùng trong hệ thống tài chính kế toán lõi.

---
**Navigation:**
[⬅️ Previous: Bài 7: Bộ đệm (Caching), Redis và Chiến lược Xử lý rủi ro Cache](./07-key-value-stores-and-caching.md) | [Next: Bài 9: Cơ sở dữ liệu Cột Rộng (Wide-Column) và Thuật toán LSM-Tree ➡️](./09-wide-column-and-lsm-trees.md)
