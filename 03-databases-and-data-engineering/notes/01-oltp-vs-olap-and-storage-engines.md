# Bài 1: Phân tầng Lưu trữ, OLTP vs OLAP và Kiến trúc Dòng/Cột

Hệ quản trị cơ sở dữ liệu (Database Management System - DBMS) là một hệ thống phần mềm trung gian đóng vai trò cầu nối giữa ứng dụng máy tính và hệ thống lưu trữ vật lý của Hệ điều hành. Chức năng cốt lõi của nó là đảm bảo dữ liệu được ghi chép, truy xuất và bảo vệ một cách toàn vẹn.

Để định hình kiến trúc của một hệ thống dữ liệu, các kỹ sư cần xác định rõ giới hạn của phần cứng lưu trữ và đặc thù của luồng truy vấn nghiệp vụ.

---

## 1. Phân cấp Lưu trữ (Storage Hierarchy) và Nút thắt I/O

Mọi cấu trúc dữ liệu và thuật toán trong DBMS đều được thiết kế dựa trên một sự thật vật lý: **Sự chênh lệch tốc độ khổng lồ giữa Bộ nhớ trong (RAM) và Bộ nhớ ngoài (Disk).**

- **RAM (In-memory):** Tốc độ truy xuất cỡ nano-giây (ns). Dữ liệu biến mất khi sập nguồn (Volatile).
- **Disk (SSD/HDD):** Tốc độ truy xuất cỡ mili-giây (ms). Dữ liệu bền vững (Non-volatile). Chênh lệch tốc độ truy xuất giữa Disk và RAM lên tới hàng trăm ngàn lần.

Khái niệm **Disk I/O (Input/Output)** dùng để chỉ thao tác đọc/ghi dữ liệu từ ổ cứng. Vì chi phí thời gian của thao tác Disk I/O vô cùng đắt đỏ, mục tiêu kiến trúc tối thượng của mọi Engine Database là: **Giảm thiểu tối đa số lần thực thi Disk I/O.**

Để làm được điều này, Database không đọc từng byte dữ liệu từ đĩa. Nó giao tiếp với ổ cứng theo từng khối lớn gọi là **Page** hoặc **Block** (thường có kích thước 4KB, 8KB hoặc 16KB). Một lần gọi I/O sẽ bốc toàn bộ 1 Page đưa lên RAM để xử lý. Các kỹ thuật Indexing (Bài 2) đều nhằm mục đích đưa dữ liệu có tính liên kết vào chung một Page để CPU chỉ cần gọi I/O một lần duy nhất.

---

## 2. Hai luồng Hệ thống Cốt lõi: OLTP và OLAP

Phân tích yêu cầu xử lý dữ liệu của doanh nghiệp, Khoa học Máy tính chia hệ thống cơ sở dữ liệu thành hai trường phái đối lập hoàn toàn về mặt kiến trúc:

### A. Hệ thống OLTP (Online Transaction Processing)
Hệ thống xử lý giao dịch trực tuyến, đại diện cho phần mềm vận hành hàng ngày (Ứng dụng Ngân hàng, Web E-commerce).
- **Đặc trưng:** Xử lý hàng vạn giao dịch nhỏ trong tích tắc.
- **Tác vụ:** Chèn (Insert), Cập nhật (Update), Xóa (Delete) liên tục.
- **Truy vấn:** Tìm kiếm chi tiết một vài bản ghi cụ thể (Ví dụ: Lấy chi tiết đơn hàng số 10245).
- **Kiến trúc phù hợp:** Lưu trữ dựa trên Dòng (Row-oriented). Cơ sở dữ liệu đại diện: PostgreSQL, MySQL, Oracle.

### B. Hệ thống OLAP (Online Analytical Processing)
Hệ thống phân tích trực tuyến, phục vụ cho bộ phận Data Engineering và Business Intelligence (Báo cáo tổng hợp, AI).
- **Đặc trưng:** Thực thi vài chục giao dịch mỗi ngày, nhưng mỗi giao dịch quét qua hàng Terabyte dữ liệu.
- **Tác vụ:** Rất ít cập nhật/xóa, chủ yếu là chèn dữ liệu khối lớn (Batch Load).
- **Truy vấn:** Tính toán tổng hợp khối lượng lớn (Ví dụ: Tính tổng doanh thu tháng 10 của toàn bộ cửa hàng).
- **Kiến trúc phù hợp:** Lưu trữ dựa trên Cột (Columnar). Cơ sở dữ liệu đại diện: Amazon Redshift, Google BigQuery, ClickHouse.

---

## 3. Kiến trúc Vật lý: Row-oriented vs Columnar

Sự khác biệt vật lý của hai hệ thống OLTP và OLAP bắt nguồn trực tiếp từ cách Database sắp xếp dữ liệu (layout) trên các Sector của Ổ cứng.

Xét bảng dữ liệu Giao dịch: `[ID, User, Amount, Date]`

### Lưu trữ theo Dòng (Row-oriented Storage)
Toàn bộ thông tin của một bản ghi (Row) được xếp nối tiếp nhau thành một chuỗi byte liên tục trên đĩa.
`Disk Layout: [1, "Alice", $50, "10-01"] | [2, "Bob", $200, "10-01"] | [3, "Charlie", $30, "10-02"]`

- **Tối ưu cho OLTP:** Khi khách hàng Alice đăng nhập và cần xem chi tiết giỏ hàng, Database chỉ cần thực thi một lệnh Disk I/O quét khối Page chứa đúng dòng thứ 1, lấy trọn vẹn mọi thuộc tính `ID`, `User`, `Amount`, `Date` đưa lên RAM. Tốc độ ghi (Insert) cũng siêu nhanh vì chỉ cần nối chuỗi vào cuối tập tin.
- **Nút thắt phân tích:** Nếu Data Engineer gọi hàm `SUM(Amount)`. Hệ thống buộc phải quét toàn bộ các Page trên đĩa để trích xuất chỉ duy nhất trường `Amount`, kéo theo việc load dư thừa một lượng khổng lồ rác dữ liệu (`User`, `Date`) lên RAM, gây tắc nghẽn I/O (I/O Bottleneck).

### Lưu trữ theo Cột (Columnar Storage)
Hệ thống bóc tách bảng, phân lô theo các cột độc lập và lưu trữ chúng vào các tệp vật lý riêng biệt.
`Disk Layout (Tệp Amount): [$50, $200, $30]`
`Disk Layout (Tệp User): ["Alice", "Bob", "Charlie"]`

- **Tối ưu cho OLAP:** Khi Data Engineer gọi hàm `SUM(Amount)`, hệ thống bỏ qua toàn bộ các tệp khác, chỉ điều khiển đầu đọc I/O tới tệp Amount, hút một khối Page chứa hàng ngàn con số định dạng đồng nhất lên RAM tính toán. Khối lượng Disk I/O giảm thiểu đến mức tối đa.
- **Năng lực Nén dữ liệu (Data Compression):** Một Page trên ổ đĩa chứa toàn bộ ký tự `$` lặp lại cho phép áp dụng các thuật toán nén như Run-Length Encoding (RLE) để tối ưu không gian đĩa một cách hoàn hảo. Hệ thống Columnar thường nén kích thước Database nhỏ đi 5-10 lần so với định dạng Row.
- **Nút thắt Giao dịch:** Nếu phải chèn (Insert) một bản ghi mới, Database buộc phải định vị và thực hiện thao tác ghi song song trên tất cả các tập tin cột khác nhau. Chi phí khóa hệ thống và độ trễ ổ đĩa khiến cấu trúc Cột thất bại thảm hại trong vai trò OLTP.

Hiểu rõ hình thái lưu trữ là cơ sở bắt buộc để Data Engineer định tuyến các hệ thống tích hợp liên nền tảng (Data Pipelines) mà chúng ta sẽ tìm hiểu ở Chương 4.

---
**Navigation:**
[Next: Bài 2: Kiến trúc Indexing Cốt lõi: B-Tree và Hash Index ➡️](./02-database-indexing-and-btrees.md)
