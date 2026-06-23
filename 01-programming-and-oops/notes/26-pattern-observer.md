# Bài 26: Observer Pattern và Kiến trúc Hướng Sự kiện (Event-Driven)

Tiếp nối các Mẫu thiết kế Hành vi, **Observer (Quan sát viên)** là cơ sở lý thuyết cho việc thiết lập cơ chế đồng bộ trạng thái tự động giữa các đối tượng phân tán. Pattern này đặt nền móng cho các mô hình xuất bản - đăng ký (Pub/Sub) và kiến trúc Hướng sự kiện (Event-Driven Architecture) chi phối toàn bộ hệ thống Kafka, Redis Pub/Sub, hay cơ chế Reactivity của Frontend (ReactJS/Vue).

Bài toán kiến trúc: **Làm thế nào để khi một phân hệ cốt lõi thay đổi trạng thái, toàn bộ các đối tượng liên đới được cập nhật đồng bộ mà không cần phụ thuộc vào cơ chế kiểm tra trạng thái liên tục (Polling)?**

---

## 1. Rủi ro của mô hình Hỏi vòng (Polling)

Nếu các tiến trình con phụ thuộc vào chu kỳ lấy thông tin (ví dụ: vòng lặp Timer mỗi 5 giây) từ cụm lưu trữ để phát hiện biến đổi, kiến trúc sẽ đối mặt với sự trì trệ băng thông nghiêm trọng và mức tiêu tốn chu kỳ CPU lớn dành cho các lời gọi hàm không mang lại kết quả dữ liệu rỗng.

Thay vì cơ chế **Pull** (Các luồng bên dưới liên tục đòi dữ liệu từ hệ thống cấp trên), kiến trúc tối ưu phải là cơ chế **Push** (Hệ thống cấp trên lưu trữ danh sách các điểm cuối phụ thuộc, và chủ động ra tín hiệu khi có trạng thái mới cấu hình thành công).

---

## 2. Kiến trúc Observer

Mô hình thiết lập hai thực thể cấu trúc:
1. **Subject (Bản thể Trung tâm / Nhà xuất bản):** Đối tượng tạo ra các sự kiện trạng thái biến đổi. Subject quản lý một mảng tham chiếu (List) chứa mọi thực thể đã đăng ký tiếp nhận thông tin từ nó.
2. **Observer (Quan sát viên / Khán giả):** Các đối tượng cung cấp một giao thức hàm công khai để Subject có thể gọi hàm đẩy tín hiệu (Dispatch) một cách đồng bộ. 

```mermaid
graph TD
    subgraph Cấu trúc Subject
        S["DataStore<br/>- observersList: List<IObserver>"]
    end
    
    subgraph Kiến trúc Quan sát viên
        O1["CacheSyncService<br/>+ update()"]
        O2["LoggingService<br/>+ update()"]
        O3["EmailNotification<br/>+ update()"]
    end
    
    O1 -.->|1. subscribe()| S
    O2 -.->|1. subscribe()| S
    O3 -.->|1. subscribe()| S
    
    S -->|2. Event Dispatching: Duyệt List và gửi update()| O1 & O2 & O3
    
    style S fill:#dc3545,color:#fff
    style O1 fill:#007bff,color:#fff
    style O2 fill:#007bff,color:#fff
    style O3 fill:#007bff,color:#fff
```

### Triển khai hệ thống:

```java
// Hợp đồng quy chuẩn nhận tín hiệu
interface IObserver {
    void update(String eventData);
}

// 1. Phân hệ Quan sát (Người theo dõi)
class LoggerService implements IObserver {
    public void update(String eventData) {
        System.out.println("Ghi log sự kiện thay đổi: " + eventData);
    }
}

// 2. Phân hệ Subject (Bộ Quản trị Nguồn)
class DataManager {
    // Mảng lưu trữ động tập hợp các thực thể cần thông báo
    private List<IObserver> observers = new ArrayList<>();

    public void subscribe(IObserver o) {
        observers.add(o);
    }

    public void unsubscribe(IObserver o) {
        observers.remove(o);
    }

    // Cơ chế Phát động Sự kiện (Event Trigger)
    public void saveData(String newData) {
        // Thực thi nghiệp vụ lưu trữ chính...
        // Phát lệnh đồng bộ (Notify)
        for (IObserver o : observers) {
            o.update(newData); // Kích hoạt Push Mechanism
        }
    }
}
```

---

## 3. Phân tích Độ Gắn kết (Loose Coupling)

Observer là thiết kế chuẩn xác để đạt được sự phân ly độc lập (Loose Coupling) trong hệ thống phân nhánh rộng.
Trong đoạn mã trên, phân hệ `DataManager` hoạt động mà **không cần nhận diện lớp hệ thống cụ thể** của nhóm đang xử lý tin nhắn. Nó chỉ tương tác với cấu trúc giao diện `IObserver`. 

Nếu một kiến trúc sư cần triển khai việc cập nhật bộ nhớ tạm của CSDL Redis sau mỗi lần CSDL gốc lưu trữ dữ liệu, kỹ sư chỉ việc cung cấp một module `CacheUpdater implements IObserver` mới và đính kèm (subscribe) nó vào mảng quản trị của `DataManager`. Tiến trình gốc `DataManager` vẫn nguyên vẹn cấu trúc mã, tuân thủ Nguyên tắc Đóng/Mở (OCP).

Việc ứng dụng Observer loại bỏ rủi ro tạo ra các luồng kết nối tĩnh dày đặc, cung cấp khả năng tích hợp mô đun mềm dẻo. Trong các hệ sinh thái lớn, cấu trúc Observer nội bộ có thể được tách biệt ra thành một tiến trình trung gian thứ ba độc lập, gọi là **Message Queue (Hàng đợi Sự kiện)** như RabbitMQ hoặc Kafka, nhằm điều tiết tín hiệu vượt qua ranh giới giữa các cụm phân tán siêu nhỏ (Microservices).

---
**Navigation:**
[⬅️ Previous: Bài 25: Strategy Pattern và Cơ chế Tách rời Thuật toán](./25-pattern-strategy.md) | [Next: Bài 27: Decorator Pattern và Thiết kế Mở rộng Đối tượng động ➡️](./27-pattern-decorator.md)
