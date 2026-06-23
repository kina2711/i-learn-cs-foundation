# Bài 11: Phân tích Thuật toán Sắp xếp Cơ bản (Basic Sorting Algorithms)

Yêu cầu tiên quyết cho một cấu trúc tìm kiếm nhị phân ổn định (Bài 10) là tập dữ liệu bắt buộc phải được sắp xếp theo đúng trật tự. Bài toán Sắp xếp (Sorting) trở thành một lĩnh vực định hình sự phát triển của hệ thống giải thuật máy tính. 

Trong tiến trình lịch sử, các hệ thống bắt đầu với 3 phương pháp sắp xếp Sơ cấp, tất cả đều được định hình chung bởi cấu trúc khối lượng tính toán phức tạp ở mức độ cao $O(N^2)$.

---

## 1. Bubble Sort (Sắp xếp Nổi bọt)

**Nguyên lý vận hành:** Giải thuật tiến hành duyệt tuyến tính qua danh sách lặp đi lặp lại. Ở mỗi bước lặp, nó so sánh hai phần tử liền kề nhau; nếu phát hiện phần tử đứng trước lớn hơn phần tử đứng sau, nó kích hoạt thao tác Đảo vị trí (Swap). 
Hành vi này đẩy các giá trị lớn "nổi bọt" dần về phía đuôi danh sách.

```java
for (int i = 0; i < n - 1; i++) {
    for (int j = 0; j < n - i - 1; j++) {
        if (arr[j] > arr[j + 1]) { // Nếu nghịch thế
            swap(arr, j, j + 1);   // Trao đổi
        }
    }
}
```

**Phân tích kỹ thuật:**
- Khối lượng thời gian: Lớp ngoài lặp $N$ lần, lớp trong lặp $N$ lần. Số lượng thao tác là $N \times N$, hệ quả là Big O $O(N^2)$ ở kịch bản rủi ro nhất (Worst-case) lẫn Trung bình (Average-case).
- Ứng dụng: Chỉ dùng cho giáo dục, không có tính ứng dụng trong kiến trúc lưu trữ phần mềm do tốn chu kỳ đảo giá trị liên tục trong RAM.

---

## 2. Selection Sort (Sắp xếp Chọn)

**Nguyên lý vận hành:** Giải thuật duy trì hai khu vực khái niệm song song: Phần mảng đã được sắp xếp (Đứng đầu mảng) và Phần chưa phân tích. Mỗi vòng lặp, hệ thống thực hiện một chu trình quyét toàn diện vùng chưa phân tích để **Tìm giá trị Nhỏ nhất (Min)**. Khi xác nhận hoàn tất, nó đảo trị giá này vào đầu vùng chưa sắp xếp, từ đó mở rộng vùng sắp xếp lên một ô nhớ.

```java
for (int i = 0; i < n - 1; i++) {
    int minIndex = i;
    for (int j = i + 1; j < n; j++) {
        if (arr[j] < arr[minIndex]) {
            minIndex = j; // Xác định lại index nhỏ nhất
        }
    }
    swap(arr, i, minIndex); // Chỉ Swap 1 lần mỗi vòng
}
```

**Phân tích kỹ thuật:**
- Khối lượng thời gian: Vẫn phải thực thi vòng quét đôi nên vẫn duy trì mức tồi tệ $O(N^2)$.
- Ưu điểm hơn Bubble: Số lần can thiệp ghi đè (Write Memory) vào mảng được tối ưu hóa tối đa $O(N)$ lần. Điều này hữu dụng ở các thiết bị như EEPROM/Flash cũ có tuổi thọ ghi/xóa thấp, nhưng không mang lại khác biệt lớn cho môi trường RAM hiện đại.

---

## 3. Insertion Sort (Sắp xếp Chèn)

**Nguyên lý vận hành:** Hoạt động giống như thao tác sắp xếp các lá bài trên tay. Hệ thống duyệt từng giá trị, coi toàn bộ vùng bên trái giá trị hiện tại là Vùng Đã Sắp Xếp. Nó lấy giá trị hiện tại và trượt ngược về phía bên trái, dịch chuyển mọi giá trị lớn hơn sang phải cho đến khi tìm thấy khoảng trống chèn hợp lý.

```java
for (int i = 1; i < n; i++) {
    int key = arr[i];
    int j = i - 1;
    // Dịch các Nút lớn hơn qua Phải để giành không gian
    while (j >= 0 && arr[j] > key) {
        arr[j + 1] = arr[j];
        j = j - 1;
    }
    arr[j + 1] = key; // Chèn thẳng
}
```

**Phân tích kỹ thuật và Khái niệm Độ Ổn định (Stability):**
1. **Khối lượng thời gian:** Mặc định $O(N^2)$. Tệ hơn trong trường hợp mảng bị nghịch chiều hoàn toàn.
2. **Kịch bản Lạc quan (Best-case):** Nếu mảng đầu vào **Đã được sắp xếp tương đối (Nearly sorted)**, cấu trúc `while` sẽ bị triệt tiêu vì điều kiện đánh giá `arr[j] > key` trả về sai ngay lập tức. Thuật toán này có năng lực xử lý một luồng dữ liệu chuẩn với độ trễ chỉ còn $O(N)$.
3. **Độ ổn định hệ thống (Stability):** Đây là thuộc tính kiến trúc quan trọng. Một thuật toán sắp xếp được gọi là Ổn định (Stable) nếu hai phần tử có trị giá toán học bằng nhau (Ví dụ: `A[5]` và `A[10]` cùng giá trị `15`) sẽ luôn duy trì thứ tự gốc (5 đứng trước 10) sau quá trình sắp xếp. 
   - **Insertion Sort và Bubble Sort là Thuật toán Ổn định (Stable).**
   - **Selection Sort là Thuật toán Không Ổn định (Unstable)** vì tính chất lấy điểm chèn tùy tiện của hàm Min có nguy cơ đảo lộn thứ tự bản ghi gốc.

> [!NOTE]
> Thuật toán Insertion Sort vì tính ưu việt đối với Cấu trúc Dữ liệu đã định dạng tốt, nên nó được các nhà thiết kế ngôn ngữ ứng dụng làm module lõi xử lý "cuối cùng" khi tích hợp vào các thuật toán Phân chia để trị như Timsort (Công nghệ lõi trong phương thức `sort()` của Python và Java).

---
**Navigation:**
[⬅️ Previous: Bài 10: Tìm kiếm Nhị phân (Binary Search)](./10-binary-search.md) | [Next: Bài 12: Kỹ thuật Phân chia để trị (Merge Sort và Quick Sort) ➡️](./12-divide-and-conquer-sorting.md)
