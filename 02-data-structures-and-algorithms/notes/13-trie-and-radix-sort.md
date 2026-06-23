# Bài 13: Vượt qua Rào cản O(N log N) - Cây Tiền tố (Trie) và Sắp xếp Tuyến tính

Trong lý thuyết Khoa học Máy tính, một định lý kinh điển liên quan đến Thuật toán Sắp xếp là: **"Bất kỳ thuật toán sắp xếp dựa trên thao tác So sánh (Comparison-based Sorting) nào cũng không thể vượt qua giới hạn độ phức tạp thời gian tối thiểu $O(N \log N)$"**.

Tuy nhiên, nếu ta ngừng việc so sánh các phần tử với nhau và chỉ sử dụng các cấu trúc phân rã đặc trưng của biến (ví dụ: bóc tách chữ số hoặc ký tự), hệ thống có khả năng triệt tiêu giới hạn lý thuyết này, giảm thiểu độ phức tạp xuống phân khúc Cấp tiến bậc nhất: **Sắp xếp thời gian Tuyến tính $O(N)$**. 

Đại diện nổi bật cho chiến lược này là nhóm Non-comparison Sort (Radix Sort, Bucket Sort) và kiến trúc Cây Tiền tố (Trie).

---

## 1. Bucket Sort (Sắp xếp theo Thùng đệm)

Trong điều kiện tập dữ liệu được cung cấp dưới dạng các giá trị phân phối tương đối đồng đều trong một giới hạn nhất định (ví dụ: điểm sinh viên từ 0 đến 100), kỹ sư có thể tránh khỏi việc so sánh đôi.

**Quá trình thực thi cấu trúc:**
1. Khởi tạo một cụm mảng lưu trữ đóng vai trò là các "Thùng/Khay" (Buckets) ảo chia dải dung sai. (Ví dụ 10 Thùng: `0-9, 10-19...`).
2. Luồng quét đầu tiên (Duyệt mảng N phần tử): Dựa trên thuộc tính toán học của điểm số, thả phần tử vào chính xác Thùng chịu trách nhiệm biên độ đó. Thao tác rẽ nhánh $O(1)$ cho từng số lượng tử.
3. Kích hoạt thuật toán cơ sở (như Insertion Sort) để sắp xếp các nhóm hạt nhân nhỏ bên trong từng Thùng. Do kích thước dữ liệu trong Thùng rất loãng, chi phí sắp xếp cục bộ rất nhỏ.
4. Đẩy ngược tất cả dữ liệu từ các Thùng kết lại thành danh sách kết quả.

**Hiệu năng:** Độ trễ cấu trúc chạm đáy $O(N + K)$ (với $K$ là số lượng Thùng được cấp phát). Tuy nhiên nó phụ thuộc khắt khe vào hình thái dữ liệu phải tản mát đều, nếu dồn cụm vào một cực, phương án này suy thoái thành $O(N^2)$.

---

## 2. Radix Sort (Sắp xếp theo Cơ số)

Giải pháp Radix Sort triển khai việc phân dải giá trị thông minh hơn bằng cách tận dụng chữ số (Digit) ở từng vị trí (Cơ số/Radix).

**Cấu trúc xử lý (Ví dụ phương án LSD - Least Significant Digit):**
- Giả sử sắp xếp dãy số nguyên: `[170, 45, 75, 90, 802, 24, 2, 66]`
- Bước 1: Hệ thống duyệt mảng và gom nhóm toàn bộ con số dựa trên giá trị của **Chữ số hàng Đơn vị**. 
  - Đuôi `0`: 170, 90
  - Đuôi `2`: 802, 2
  - Đuôi `4`: 24
  - ... (Lắp ghép sơ cấp hoàn thành)
- Bước 2: Hệ thống tiếp tục gom nhóm dựa trên **Chữ số hàng Chục**.
- Bước 3: Hoàn thành bước duyệt dựa trên **Chữ số hàng Trăm**. 
(Lưu ý: Quá trình gom nhóm nội sinh buộc phải dùng một phương pháp cấu trúc đảm bảo Độ ổn định - Stable, như Counting Sort để không đảo lộn mảng trước đó).

**Hiệu suất Đột phá:** Tổng vòng lặp bằng số lượng tối đa của bộ số $D$ (Max digits). Khối lượng thuật toán: $O(D \times N)$. Nếu $D$ là hằng số hoặc rất nhỏ so với $N$, Radix Sort bứt phá trở thành thuật toán tuyến tính.

---

## 3. Kiến trúc Cây Tiền tố (Trie)

Rời xa nền tảng sắp xếp số nguyên, trong không gian xử lý chuỗi ký tự khổng lồ (Text processing) của bộ lưu trữ Công cụ Tìm kiếm, việc dò tìm từ khóa dựa trên BST hoặc Hash Table đôi khi không mang lại giá trị trích xuất từ vựng phù hợp (Autocomplete/Prefix Match). **Trie (Prefix Tree)** là cấu trúc được thiết kế tối giản hóa dữ liệu chữ cái.

**Bản chất Trie:** 
Là một Cây Phi tuyến, nhưng thay vì các Node mang dữ liệu trực tiếp, các Cạnh (Edges) nối hoặc thân các Nút đại diện cho 1 ký tự cụ thể. 

```mermaid
graph TD
    Root((Gốc rỗng)) --> A((A))
    Root --> C((C))
    A --> P1((P)) --> P2((P)) --> L1((L)) --> E1((E <br/> "apple"))
    A --> P1 --> P3((P))
    C --> A1((A)) --> T1((T <br/> "cat"))
    C --> A1 --> R1((R <br/> "car"))
```

**Tính ưu việt trong Môi trường Ký tự:**
- **Không gian lưu trữ:** Bằng cách đan xen chung tất cả các chuỗi có chung Tiền tố (Prefix) vào cùng một nhánh (ví dụ "cat" và "car" sẽ cùng rẽ chung đường ở cụm "ca"), cấu trúc này siêu tiết kiệm dung lượng RAM với mật độ từ vựng lớn.
- **Tốc độ Autocomplete hoàn hảo:** Để tìm xem ứng dụng có từ nào bắt đầu bằng `app` hay không, CPU chỉ mất chính xác số lượng truy xuất bằng độ dài ký tự của chuỗi đầu vào (3 thao tác) $O(L)$, tốc độ tiệm cận bằng không, độc lập tuyệt đối với việc trong cơ sở dữ liệu có $N = 100$ hay $N = 100$ triệu từ khóa.

Sức mạnh cấu trúc này làm nền tảng cốt lõi cho tính năng Suggesion Autocomplete khi người dùng gõ phím trên thanh tìm kiếm của Google hoặc ElasticSearch engine.

---
**Navigation:**
[⬅️ Previous: Bài 12: Kỹ thuật Phân chia để trị (Merge Sort và Quick Sort)](./12-divide-and-conquer-sorting.md) | [Next: Bài 14: Tư duy Cấu trúc Phân rã - Đệ quy (Recursion) và Quay lui (Backtracking) ➡️](./14-recursion-and-backtracking.md)
