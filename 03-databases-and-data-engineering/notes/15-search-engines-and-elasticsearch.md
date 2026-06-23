# Bài 15: Công cụ Tìm kiếm (Search Engine) và Chỉ mục Ngược (Inverted Index)

Bài toán cuối cùng của chặng 3 thuộc về lĩnh vực Trích xuất Dữ liệu Ký tự (Text Retrieval).
B-Tree tuyệt vời cho các con số ID, Hash Table hoàn hảo cho kết quả 1-1, nhưng khi đối diện với khung tìm kiếm Google Search: "Tìm tất cả các bài viết chứa cụm từ *Lỗi phần cứng máy chủ* trên toàn bộ diễn đàn", RDBMS và NoSQL tiêu chuẩn sụp đổ thảm hại do phải sử dụng cấu trúc chuỗi tìm kiếm càn quét `LIKE '%lỗi phần cứng%'` ($O(N)$ Disk Scan).

Sự ra đời của Elasticsearch và nền tảng Apache Lucene đã xác lập riêng một chuẩn mực kiến trúc thứ 3: **Search Engine**, sử dụng thuật toán trái tim **Inverted Index (Chỉ mục Ngược)**.

---

## 1. Cấu trúc Thuật toán Chỉ mục Ngược (Inverted Index)

Để tìm kiếm văn bản trong mili-giây, Elasticsearch bẻ khóa toàn bộ chuỗi ký tự theo phương pháp lật ngược tư duy.

- Trong CSDL Quan hệ, chúng ta trỏ ID Văn bản đến Nội dung (Văn bản số 1 $\rightarrow$ Nội dung: "Hôm nay trời đẹp"). 
- Chỉ mục ngược (Inverted Index) lật hệ quy chiếu: Trỏ **Từng từ vựng (Term)** trở lại danh sách các **ID Văn bản** chứa nó.

Quá trình luồng xử lý đằng sau (Under-the-hood Indexing Phase):
Giả sử ta nhập 2 đoạn Document log vào hệ thống:
> Doc 1: "Máy chủ bị lỗi ổ cứng"
> Doc 2: "Máy tính xách tay bị lỗi RAM"

Hệ thống phân tích cú pháp (Tokenizer) xé vụn văn bản thành các từ và lưu vào một cấu trúc Bảng Băm/Trie:
- "Máy" $\rightarrow$ [Doc 1, Doc 2]
- "lỗi" $\rightarrow$ [Doc 1, Doc 2]
- "chủ" $\rightarrow$ [Doc 1]
- "ổ cứng" $\rightarrow$ [Doc 1]
- "RAM" $\rightarrow$ [Doc 2]

Khi người dùng gõ tìm kiếm chữ "lỗi", hệ thống nhảy cực nhanh vào Bảng Băm $O(1)$, lấy thẳng vào từ khóa "lỗi" và kéo ra tức khắc một mảng `[Doc 1, Doc 2]`. CPU không mất bất kỳ một tia năng lượng nào để đi quét file vật lý.

---

## 2. Sắp xếp Xếp hạng (Ranking) và Thuật toán TF-IDF

Tìm ra hàng triệu bài báo chứa chữ "ổ cứng" là chưa đủ, Google Search hoặc Elasticsearch cần trả về kết quả theo độ Liên quan (Relevance) để xếp bài nào lên trang 1. 

Công thức thống kê kinh điển của mọi Search Engine là **TF-IDF (Term Frequency - Inverse Document Frequency)**:

1. **TF (Tần suất xuất hiện từ cục bộ):** Nếu một đoạn log dài 100 chữ mà chứa chữ "ổ cứng" tới 10 lần, đoạn log này cực kỳ liên quan đến chủ đề phần cứng. Điểm TF cao.
2. **IDF (Tần suất từ trên toàn cục):** Nhưng nếu User tìm chữ "máy", và chữ "máy" xuất hiện ở hàng tỷ văn bản trên internet (Vì nó là một từ rất phổ thông). Hệ thống sẽ hạ nhục điểm số trọng lượng của chữ "máy" (Phạt điểm). Ngược lại, nếu từ khóa là chữ "Tụ điện", rất hiếm, thì bất kì trang nào chứa chữ "Tụ điện" sẽ được đẩy điểm vọt lên cực đại.

Sự giao thoa của $TF \times IDF$ sinh ra điểm số (Score). Các tệp Doc sẽ được sắp xếp trả về màn hình cho người dùng dựa trên mức điểm cực kì chuẩn xác này.

---

## 3. Đánh đổi Hệ thống (The Trade-offs)

Lý do tại sao các hệ thống OLTP không dùng thẳng Elasticsearch làm bộ nhớ chính? Sự mạnh mẽ của tìm kiếm đổi bằng cái giá rất chát về Tài nguyên hệ thống.

1. **Sưng phồng Không gian đĩa (Storage Bloat):** Quá trình Tokenizer để nhả ra Inverted Index khiến một văn bản gốc 1MB có thể phình to tệp Chỉ mục lên tới 2MB (Gấp đôi kích thước gốc). Hệ thống Search ngốn RAM và Ổ cứng khốc liệt.
2. **Ghi cực chậm (Write Latency):** Mỗi khi Data Engineer thêm 1 dòng log mới, CPU của Elasticsearch phải đi xé đoạn log đó ra thành hàng trăm từ rời rạc, sau đó chọc vào hàng trăm rãnh của Chỉ mục Ngược để cập nhật (`Doc 3`) vào từng dãy Array một. Sự phân tích từ vựng này khiến tốc độ Ghi của Elasticsearch thấp hơn rất nhiều so với kiến trúc LSM-Tree của Cassandra (Bài 9). 

Do vậy, trong mô hình Big Data, Data Engineer luôn kết hợp cặp đôi hoàn hảo: Đặt một Data Warehouse/Lake làm hồ lưu trữ gốc an toàn (Single Source of Truth), sau đó bật Data Pipeline bơm luồng dữ liệu sang cho Elasticsearch chuyên biệt để phục vụ tìm kiếm.

---
*(Kết thúc Chặng 3 - Hành trình xuyên suốt cấu trúc hạ tầng Dữ liệu Máy tính)*

---
**Navigation:**
[⬅️ Previous: Bài 14: Kiến trúc Hướng Sự kiện (Event-Driven) và Apache Kafka](./14-message-queues-and-kafka.md)
