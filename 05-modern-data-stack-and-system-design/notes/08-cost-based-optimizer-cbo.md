# Bài 8: Não bộ Spark SQL: Tối ưu hóa Chi phí CBO và Broadcast Hash Join

Ngay cả khi bạn xài định dạng nén Parquet/Iceberg xịn nhất, CPU dùng lệnh SIMD nhanh nhất, bạn vẫn có thể làm sập nguyên một cụm 100 máy chủ (Cluster) khổng lồ chỉ bằng một câu lệnh `SQL JOIN` ngờ nghệch.

Bởi vì bản chất của Phân tán (Distributed) là sự băm nhỏ. Bảng Khách Hàng (Customer) nằm rải rác ở 50 máy. Bảng Hóa Đơn (Order) nằm rải rác ở 50 máy khác. Để JOIN chúng lại với nhau tìm ra *"Tên Khách Hàng cho từng Hóa Đơn"*, Máy tính buộc phải trao đổi chéo dữ liệu qua cáp quang mạng LAN.

Sự xáo trộn vĩ đại này gọi là **Shuffle (Bài 13, Part 3)**. Shuffle ngốn Network Bandwidth (Băng thông mạng), gây tràn RAM và cháy Ổ cứng lưu tạm. Kẻ đứng ra cứu rỗi hệ thống khỏi thảm họa Shuffle chính là **Cost-Based Optimizer (CBO - Trình tối ưu hóa dựa trên Chi phí)**.

---

## 1. Não bộ của Query Engine: Khác biệt giữa RBO và CBO

Khi bạn gõ lệnh SQL ném cho Spark SQL hoặc Trino, hệ thống không mù quáng chạy luôn. Câu lệnh dạng chữ Text của bạn sẽ được một Trình Biên Dịch (như Catalyst Optimizer của Spark) bóc tách thành một Sơ đồ DAG toán học (Logical Plan).

**A. Rule-Based Optimizer (RBO - Tối ưu hóa theo Luật):**
RBO rà soát sơ đồ DAG và áp dụng những Định lý Vật lý cố định:
- *Ví dụ Luật:* Nếu câu lệnh là `SELECT * FROM tbl WHERE age > 20`, RBO lập tức bứng cái màng lọc `Filter(age>20)` ép nó chạy DƯỚI CÙNG sát đĩa cứng (Predicate Pushdown). Màng lọc này sẽ chém chết 80% dữ liệu rác trước khi nhồi lên bước JOIN, cứu hệ thống khỏi cảnh ngộp dữ liệu. 
RBO rất tốt, nhưng nó bị MÙ. Nó không biết cái Bảng tbl to bằng ngôi nhà hay nhỏ bằng quả cam.

**B. Cost-Based Optimizer (CBO - Sự Thông Thái Thống Kê):**
CBO tinh vi hơn. Trước khi lên kế hoạch, nó lật **Bảng Thống kê (Statistics)** do Data Engineer đã chạy trước đó (như lệnh `ANALYZE TABLE`). Nó biết rõ: "À, Bảng Customer chỉ có 1 vạn dòng (Nặng 1MB). Bảng Order có tới 100 tỷ dòng (Nặng 10TB)".
Dựa vào con số Kích thước Data và Độ phân mảnh (Cardinality), CBO tính toán ra 5 phương án JOIN khác nhau, giả lập điểm số tiêu hao (Cost) của CPU/Mạng cho từng phương án, và tự động chọn phương án nhẹ nhất để thực thi ngầm.

---

## 2. Nghệ thuật Diệt Shuffle: Broadcast Hash Join

Hãy xem cách CBO cứu vớt câu lệnh kinh điển:
`SELECT c.Name, o.Total FROM Order o JOIN Customer c ON o.id = c.id`

**Cách 1 (Cách ngu ngốc nhất - Shuffle Sort-Merge Join):**
Không có CBO. Hệ thống ép 50 Máy chủ cầm bảng Order và 50 Máy chủ cầm bảng Customer Gửi Chéo dữ liệu (Shuffle) băm qua băm lại qua mạng cho nhau để xếp hai cái ID bằng nhau về chung 1 cỗ máy, sau đó nối lại. 10TB dữ liệu vứt vào cáp quang 1 Gbps, cáp quang đứt bóng và hệ thống sập.

**Cách 2 (Siêu việt do CBO chỉ định - Broadcast Hash Join):**
CBO nhìn vào cuốn sổ, thấy Bảng Customer nhẹ hều có 1MB. Nó đập bàn ra lệnh: **"Cấm Shuffle Bảng Order!"**.
Thay vào đó, CBO bốc nguyên cục Customer 1MB đó, Photocopy ra thành 100 bản sao. Gửi (Broadcast) 100 bản sao này quăng thẳng vào RAM nội bộ của 100 cái Máy chủ Thợ cày đang cầm Bảng Order.

```mermaid
graph TD
    subgraph Cơ chế Broadcast Hash Join (Không Shuffle)
        O[Bảng Order 10TB nằm rải rác]
        C[Bảng Customer Nhỏ 1MB]
        
        C -.->|Master Photocopy Broadcast 1MB| N1(Worker Node 1 <br> Đang cầm 100GB Order)
        C -.->|Master Photocopy Broadcast 1MB| N2(Worker Node 2 <br> Đang cầm 100GB Order)
        
        N1 -->|Worker tự JOIN CỤC BỘ ngay trên RAM mình <br> Lập tức nhả kết quả| R1[Kết quả 1]
        N2 -->|Không cần xin data chéo qua mạng| R2[Kết quả 2]
    end
```

**Kết quả ma thuật:** Toàn bộ 100 con máy Worker (Thợ cày) đều đã nhét gọn bảng Customer 1MB trong túi quần (Build Hash Table in RAM). Bọn nó chỉ cần lấy Bảng Order khổng lồ của riêng nó, rà soát với bảng Customer trong túi, và nhả ra kết quả JOIN ngay tắp lự. 
Không một tệp tin khổng lồ nào phải bò qua đường cáp mạng. Tốc độ JOIN một cái bảng 10TB với một bảng 1MB hoàn thành chỉ trong vài chục giây. 

Hiểu thấu bản chất của Broadcast Join, Data Engineer luôn biết cách tối ưu hóa Lược đồ đa chiều (Star Schema - Bài 10 Part 3) bằng cách biến mọi bảng Vệ tinh (Dimension) thành kích thước cực nhỏ để lừa hệ thống CBO kích hoạt cơ chế Broadcast thần thánh này trên Data Warehouse.

---
**Navigation:**
[⬅️ Previous: Bài 7: Truy vấn Siêu thanh: Lệnh SIMD và Kiến trúc Vectorized Execution](./07-vectorized-query-execution-and-simd.md) | [Next: Bài 9: Apache Airflow Dưới Góc Độ Kernel: Scheduler, Executor và Giao tiếp XCom ➡️](./09-apache-airflow-and-dag-internals.md)
