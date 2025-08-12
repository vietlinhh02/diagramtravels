# 07 — Chiến lược giữ chân người dùng TravelSense

## Vấn đề cốt lõi
Người dùng không đi du lịch thường xuyên. Làm sao để TravelSense trở thành một ứng dụng được sử dụng đều đặn, thay vì chỉ khi có kế hoạch đi xa?

## Giải pháp: Mở rộng giá trị cốt lõi từ "Lập kế hoạch" sang "Khám phá & Trải nghiệm"

TravelSense cần trở thành một người bạn đồng hành cho mọi nhu cầu khám phá, từ chuyến du lịch lớn trong năm cho đến buổi tối cuối tuần trong thành phố của chính mình.

### 1. Trải nghiệm khám phá tại địa phương (Local Discovery)

Đây là hướng đi quan trọng nhất để tăng tần suất sử dụng.

*   **Tính năng:**
    *   **"Weekend Planner":** Mỗi thứ Sáu, AI tự động gửi một gợi ý kế hoạch nhỏ cho cuối tuần dựa trên thời tiết, các sự kiện đang diễn ra và sở thích của người dùng. Ví dụ: "Cuối tuần này trời đẹp, bạn có muốn thử quán cafe mới mở ở Quận 1 và sau đó đi xem triển lãm nghệ thuật gần đó không?"
    *   **"Food Tour Tự tạo":** Cho phép người dùng chọn một chủ đề (ví dụ: "món bún", "cafe trứng", "đồ ăn vặt") và AI sẽ tạo ra một tour ẩm thực nhỏ quanh một khu vực, tối ưu quãng đường di chuyển.
    *   **Tìm kiếm dựa trên "Mood":** Thay vì tìm kiếm cụ thể, người dùng có thể nhập: "Tìm một nơi yên tĩnh để đọc sách" hoặc "Gợi ý một nơi sôi động cho tối nay".
*   **Triển khai kỹ thuật:**
    *   Tận dụng **Restaurant Service** và **Places API** đã có trong kiến trúc.
    *   Mở rộng **AI Orchestrator** để có các prompt template chuyên cho gợi ý tại địa phương (ví dụ: `local_discovery_prompt`).
    *   Sử dụng **Background Jobs** để gửi thông báo "Weekend Planner" hàng tuần.

### 2. Xây dựng nội dung và cộng đồng

Biến ứng dụng từ một công cụ thành một nơi để tìm cảm hứng và chia sẻ.

*   **Tính năng:**
    *   **"Travel Log" cá nhân:** Sau mỗi chuyến đi (dù là đi xa hay đi cafe cuối tuần), ứng dụng tự động tạo một "nhật ký" nhỏ với bản đồ, hình ảnh (nếu người dùng cho phép truy cập) và các địa điểm đã ghé thăm. Người dùng có thể thêm ghi chú và chia sẻ.
    *   **"Bộ sưu tập" (Collections):** Người dùng có thể tạo các bộ sưu tập công khai hoặc riêng tư như "Những quán cafe nhất định phải thử ở Hà Nội" hoặc "Checklist các bãi biển đẹp ở Việt Nam".
    *   **Tích hợp Social:** Cho phép người dùng theo dõi các "Travel Planner" hoặc "Food Blogger" uy tín ngay trên app để xem các lịch trình và bộ sưu tập của họ.
*   **Triển khai kỹ thuật:**
    *   Mở rộng **User Service** và **Trip Service** để lưu trữ "Travel Logs" và "Collections".
    *   Sử dụng **File Storage (S3/R2)** để lưu ảnh.
    *   Thêm quan hệ "follow" trong database (PostgreSQL).

### 3. Gamification (Trò chơi hóa)

Tạo ra các yếu tố vui vẻ để khuyến khích người dùng quay lại.

*   **Tính năng:**
    *   **Huy hiệu (Badges):** Trao huy hiệu khi người dùng đạt được các cột mốc. Ví dụ: "Nhà Thám Hiểm" (ghé thăm 5 thành phố), "Chuyên gia Ẩm thực" (thử 10 nhà hàng được gợi ý), "Người lập kế hoạch cuối tuần" (tạo 3 kế hoạch cuối tuần).
    *   **Thử thách (Challenges):** Đưa ra các thử thách hàng tháng như "Khám phá 3 quán ăn mới trong tháng này".
    *   **Hệ thống điểm thưởng:** Tích điểm khi hoàn thành lịch trình, viết review, chia sẻ... Điểm này có thể đổi lấy các voucher nhỏ từ đối tác (ví dụ: giảm giá Grab, voucher cafe).
*   **Triển khai kỹ thuật:**
    *   Tạo một **Gamification Service** mới hoặc tích hợp logic này vào **User Service**.
    *   Thiết kế schema cho Badges, Challenges và Points trong PostgreSQL.

### 4. Công cụ hỗ trợ sau chuyến đi

Giá trị của ứng dụng không kết thúc khi chuyến đi kết thúc.

*   **Tính năng:**
    *   **Quản lý chi tiêu:** Cho phép người dùng nhập chi tiêu thực tế sau chuyến đi để so sánh với kế hoạch và quản lý tài chính tốt hơn cho các chuyến đi sau.
    *   **AI tự động tạo Album ảnh:** Sau chuyến đi, AI có thể phân tích ảnh của người dùng (với sự cho phép) để nhóm chúng theo ngày, địa điểm và tạo thành một album kể chuyện.
*   **Triển khai kỹ thuật:**
    *   Bổ sung tính năng vào **Trip Service** để lưu chi tiêu thực tế.
    *   Yêu cầu một service xử lý ảnh (có thể là một background job) để phân tích và nhóm ảnh.

---

Bằng cách triển khai các chiến lược này, TravelSense sẽ trở thành một phần không thể thiếu trong cuộc sống hàng ngày của người dùng, giúp họ khám phá thế giới xung quanh một cách thú vị, từ đó giải quyết được bài toán giữ chân người dùng một cách hiệu quả.