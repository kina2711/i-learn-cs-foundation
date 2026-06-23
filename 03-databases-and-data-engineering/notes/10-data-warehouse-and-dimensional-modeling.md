# Bài 10: Kiến trúc Data Warehouse và Mô hình hóa Đa chiều (Dimensional Modeling)

Trong môi trường doanh nghiệp hiện đại, dữ liệu giao dịch thường nằm rải rác ở hàng chục hệ thống OLTP riêng biệt (MySQL chứa tài khoản user, MongoDB chứa log giao dịch, PostgreSQL chứa hóa đơn). Nếu muốn thống kê một báo cáo BI: *"Tổng doanh thu tuần trước của những người dùng trên 30 tuổi mua đồ điện tử"*, việc chạy SQL JOIN nọc thẳng vào các Database vận hành là một hành động tự sát cấu trúc, dẫn đến treo cứng hệ thống bán hàng trực tiếp (Production).

Kiến trúc **Kho Dữ liệu (Data Warehouse)** được sinh ra để khắc phục. Nó là một cơ sở dữ liệu OLAP (Bài 1, Part 3) trung tâm, lưu trữ dữ liệu lịch sử khổng lồ phục vụ mục đích Duy Nhất là Phân tích (Analytics).

---

## 1. Thiết kế Hệ thống Kho: Tại sao Chuẩn hóa (Normalization) sụp đổ?

Trong thế giới OLTP, mọi kỹ sư phần mềm phải thiết kế bảng theo Quy tắc Chuẩn hóa (1NF, 2NF, 3NF). Mục tiêu của chuẩn hóa là xé vụn các thực thể thành nhiều bảng nhỏ nối với nhau bằng Foreign Key để tránh trùng lặp dữ liệu (Dễ sửa chữa UPDATE).

Tuy nhiên, trong Data Warehouse (Môi trường OLAP), dữ liệu là tĩnh (Read-only) và không bao giờ sửa chữa lại quá khứ. Nếu giữ mô hình 3NF với hàng trăm bảng bị xé vụn, một câu lệnh SQL báo cáo (Report) đơn giản sẽ phải sử dụng cấu trúc `JOIN` qua 15 bảng khác nhau. Phép nối JOIN là thao tác nặng nề và đắt đỏ bậc nhất đối với CPU và RAM.

Lý thuyết Cơ sở Dữ liệu đảo ngược phương pháp tiếp cận tại Data Warehouse: **Khử Chuẩn hóa (Denormalization)** và áp dụng **Dimensional Modeling (Mô hình hóa Đa chiều)**.

---

## 2. Dimensional Modeling: Kiến trúc Lược đồ Hình Sao (Star Schema)

Được giới thiệu bởi Ralph Kimball, Dimensional Modeling sắp xếp toàn bộ dữ liệu rối rắm thành 2 loại Bảng duy nhất: **Bảng Sự kiện (Fact Tables)** và **Bảng Chiều phân tích (Dimension Tables)**.

### A. Fact Tables (Trái tim của hệ thống)
Bảng Fact chứa các đại lượng đo lường thực tế sinh ra từ giao dịch nghiệp vụ. Nó thường vô cùng hẹp ngang (ít cột) nhưng cực kỳ dài (hàng chục tỷ dòng).
Đặc tính kỹ thuật:
- Cột khóa: Chỉ chứa một loạt các ID làm cầu nối sang bảng Dimension.
- Cột Metrics: Đo lường con số. (Số tiền thanh toán, Khối lượng hàng, Giảm giá).
- *Ví dụ:* Bảng `Fact_Sales` (Gồm: Mốc_Thời_gian_ID, Sản_phẩm_ID, Khách_hàng_ID, Tổng_tiền, Số_lượng).

### B. Dimension Tables (Bộ lọc góc nhìn)
Bảng Dimension chứa các thuộc tính ngữ cảnh (Text, Vị trí, Phân khúc) dùng để chú thích cho bảng Fact, phục vụ câu lệnh `WHERE` và `GROUP BY`. Bảng này rất rộng (hàng trăm cột) nhưng số lượng dòng rất nhỏ.
Đặc tính kỹ thuật:
- Cột thông tin: Đã được Khử Chuẩn Hóa. Ví dụ Bảng `Dim_Customer` gộp luôn cả thông tin Phường, Quận, Thành phố, Tỉnh vào chung một bảng (Chấp nhận trùng lặp chữ "Hà Nội" hàng ngàn lần để đổi lấy tốc độ truy xuất không cần JOIN).
- *Ví dụ:* `Dim_Product` (Chứa tên SP, Nhãn hiệu, Ngành hàng, Kích thước).

```mermaid
graph TD
    subgraph Kiến trúc Star Schema
        F[(Fact_Sales<br> 50 Tỷ dòng)] 
        
        D1[Dim_Date<br> (Ngày, Tháng, Quý)] -->|Date_ID| F
        D2[Dim_Product<br> (SKU, Brand, Category)] -->|Product_ID| F
        D3[Dim_Customer<br> (Tuổi, Tỉnh/Thành)] -->|Customer_ID| F
        D4[Dim_Store<br> (Chi nhánh, Kênh bán)] -->|Store_ID| F
    end
    
    style F fill:#dc3545,color:#fff
    style D1 fill:#28a745,color:#fff
```

**Sức mạnh Hiệu suất của Star Schema:**
Vì kiến trúc hình sao chỉ sâu đúng 1 tầng, mọi câu lệnh truy vấn phân tích chỉ cần thực hiện 1 phép JOIN duy nhất từ bảng Trung tâm lan ra các bảng Vệ tinh, quét trên cấu trúc Columnar Storage (Lưu trữ theo cột) để nén dữ liệu. Độ trễ phân tích từ vài giờ có thể rớt xuống chỉ còn vài giây.

---

## 3. Snowflake Schema (Lược đồ Bông Tuyết)

Khi các bảng Dimension quá cồng kềnh dẫn đến kích thước lưu trữ ổ đĩa phình to không thể kiểm soát (Lãng phí quá nhiều Space), cấu trúc Star Schema được bẻ nhánh thêm một tầng.

Bảng `Dim_Customer` sẽ được chuẩn hóa nhẹ lại, tách nhánh "Thành phố/Tỉnh" ra thành bảng `Dim_Geography`. Mô hình này tạo thành hình hoa thị sâu 2-3 tầng gọi là Snowflake.
*Trade-off (Sự đánh đổi):* Snowflake Schema tối ưu không gian ổ cứng (Storage), nhưng làm chậm Tốc độ Truy vấn (Do CPU phải thực hiện thêm 1 tầng JOIN). Trong thực tế lưu trữ đám mây hiện đại (Cloud Storage rất rẻ), Data Engineer thường ưu tiên 100% Star Schema để tối ưu hóa sức mạnh Tính toán (Compute Power).

---
**Navigation:**
[⬅️ Previous: Bài 9: Cơ sở dữ liệu Cột Rộng (Wide-Column) và Thuật toán LSM-Tree](./09-wide-column-and-lsm-trees.md) | [Next: Bài 11: Data Lake, Hệ thống phân tán và Định dạng File Columnar ➡️](./11-data-lake-and-file-formats.md)
