# TravelSense v2 — Hệ thống lập kế hoạch du lịch thông minh

## Tổng quan và kịch bản project

**TravelSense v2** là một nền tảng lập kế hoạch du lịch thông minh được thiết kế để giải quyết vấn đề nan giải mà hầu hết du khách đều gặp phải: việc lập kế hoạch du lịch tốn rất nhiều thời gian, thường thiếu thông tin đáng tin cậy và khó có thể tối ưu hóa tuyến đường một cách hiệu quả. Thay vì tạo ra một công cụ AI hoàn toàn tự động hóa quy trình, TravelSense v2 hoạt động như một trợ lý đồng hành (co-planning assistant), nơi AI không thay thế quyết định của người dùng mà chủ động phát hiện những thông tin còn thiếu, đưa ra các gợi ý có căn cứ và minh bạch về nguồn gốc, đồng thời tối ưu hóa lịch trình theo các ràng buộc thực tế như giờ mở cửa, điều kiện thời tiết và nhịp độ phù hợp với từng nhóm du khách.

Hệ thống được xây dựng trên triết lý "nhập liệu tối thiểu nhưng thông minh", chỉ hỏi những thông tin thực sự cần thiết và sử dụng các câu hỏi bổ sung ngắn gọn có thể bỏ qua khi chưa cấp thiết. Khi người dùng cung cấp các dữ liệu cơ bản như điểm xuất phát, điểm đến, khung thời gian và ngân sách, hệ thống sẽ tạo ra một bản nháp lịch trình thông minh, tích hợp dữ liệu từ nhiều nguồn API miễn phí và trả phí để đảm bảo độ chính xác. Điểm đặc biệt của TravelSense v2 là khả năng chỉnh sửa cục bộ - khi người dùng thay đổi một phần lịch trình bằng cách kéo-thả, hệ thống chỉ tính toán lại những đoạn bị ảnh hưởng thay vì làm lại toàn bộ, đồng thời cung cấp so sánh rõ ràng giữa trước và sau thay đổi về mặt thời gian, chi phí và tính khả thi.

## Kịch bản sử dụng thực tế

**Tình huống mẫu**: Gia đình 4 người (2 người lớn, 2 trẻ em 8 và 12 tuổi) muốn đi du lịch Đà Nẵng - Hội An trong 4 ngày 3 đêm với ngân sách 20 triệu VNĐ, ưu tiên hoạt động phù hợp với trẻ em và muốn trải nghiệm cả văn hóa lẫn biển.

**Bước 1 - Nhập liệu thông minh**: Người dùng chỉ cần nhập thông tin cơ bản (điểm xuất phát: Hà Nội, điểm đến: Đà Nẵng-Hội An, thời gian: 4 ngày, 4 người, ngân sách: 20M, phong cách: gia đình có trẻ em). Hệ thống AI sẽ phân tích và đưa ra các câu hỏi bổ sung thông minh như "Bạn có muốn ưu tiên resort gần biển hay ở trung tâm phố cổ Hội An?" hoặc "Trẻ em có sợ độ cao không (để gợi ý cầu Vàng)?"

**Bước 2 - Tạo nháp thông minh**: Dựa trên thông tin và phân tích từ vector store về các chuyến đi tương tự, hệ thống tạo nháp lịch trình với các gợi ý như: Ngày 1 - Bay đến Đà Nẵng, check-in resort, trưa thưởng thức mì Quảng tại quán Bà Mua, chiều tắm biển Mỹ Khê; Ngày 2 - Sáng Bà Nà Hills (cầu Vàng), trưa buffet trên núi, chiều phố cổ Hội An, tối ăn cao lầu tại Thành; Ngày 3 - Sáng làng rau Trà Quế và nấu ăn cùng nông dân, chiều thả đèn lồng, tối chợ đêm và thử đặc sản bánh xèo; Ngày 4 - Sáng tham quan chùa Linh Ứng, trưa hải sản tại bãi biển, mua sắm và bay về. Mỗi gợi ý đều kèm nguồn tham khảo, giờ mở cửa, menu và giá cả, thời gian di chuyển và ước tính chi phí bao gồm cả ăn uống.

**Bước 3 - Tối ưu và tương tác**: Người dùng có thể kéo-thả để thay đổi thứ tự (ví dụ đổi Bà Nà Hills sang ngày 3), hệ thống sẽ tự động tính toán lại thời gian di chuyển, kiểm tra xung đột về giờ mở cửa và cập nhật chi phí. Nếu có vấn đề (như Bà Nà Hills đóng cửa vào thứ 3), hệ thống sẽ cảnh báo và đề xuất phương án thay thế. Bảng ngân sách được cập nhật real-time theo từng nhóm: lưu trú (40%), ăn uống (25%), vận chuyển (20%), vé tham quan (15%).

**Bước 4 - Hoàn thiện và xuất**: Sau khi hài lòng với lịch trình, người dùng có thể xuất thành PDF có bản đồ chi tiết, file ICS để đưa vào lịch điện thoại, hoặc link chia sẻ với gia đình. Hệ thống sẽ khóa phiên bản này để đảm bảo tính nhất quán khi booking, nhưng vẫn cho phép tạo bản sao để tiếp tục chỉnh sửa nếu cần.

## Danh sách sơ đồ

- [01 — Kiến trúc tổng quan](./01_kien_truc_tong_quan.md)
- [02 — Luồng dữ liệu chi tiết](./02_luong_du_lieu_chi_tiet.md)
- [03 — AI Orchestration](./03_ai_orchestration.md)
- [04 — Deployment & Infrastructure](./04_deployment_infrastructure.md)
- [05 — API miễn phí cho project](./05_free_apis.md)
- [06 — Tích hợp Ăn uống & Nhà hàng](./06_dining_integration.md)

## Công nghệ sử dụng

### Frontend
- **Next.js/React**: Giao diện tương tác
- **TypeScript**: Type safety
- **Tailwind CSS**: Styling hiện đại

### Backend
- **Node.js/FastAPI**: API server
- **PostgreSQL**: Dữ liệu giao dịch
- **Redis**: Cache và session
- **Vector Store**: Tìm kiếm ngữ nghĩa

### AI & Machine Learning
- **Dual LLM Strategy**: 
  - LLM rẻ (GPT-3.5/Claude Haiku): Chat, ideation
  - LLM cấu trúc (GPT-4/Claude Sonnet): Validation, structured output
- **Vector embeddings**: Tìm kiếm địa điểm theo ngữ nghĩa

### Tích hợp
- **Maps API**: Google Maps/OpenStreetMap
- **Places API**: Foursquare/Google Places
- **Weather API**: OpenWeatherMap
- **Accommodation**: Booking.com API/Airbnb

## Giá trị thương mại và định vị thị trường

**TravelSense v2** hướng đến 3 phân khúc thị trường chính với tiềm năng kinh doanh rõ ràng. **Thị trường B2C** là mục tiêu chính với các cá nhân và gia đình muốn lập kế hoạch du lịch tự túc nhưng thiếu kinh nghiệm hoặc thời gian research kỹ lưỡng. **Thị trường B2B** bao gồm các travel agencies và tour operators nhỏ vừa muốn nâng cao chất lượng dịch vụ tư vấn mà không cần thuê nhiều nhân viên chuyên môn. **Thị trường B2B2C** hướng đến việc tích hợp vào các platform du lịch hiện có như booking sites hoặc travel communities để tăng thêm value-add services.

**Mô hình kinh doanh** áp dụng chiến lược freemium bền vững: người dùng được tạo miễn phí 3 kế hoạch mỗi tháng với đầy đủ tính năng cơ bản, gói Premium ($9.99/tháng) cung cấp unlimited plans cùng advanced features như group collaboration và detailed analytics, đồng thời tạo doanh thu từ commission khi người dùng booking accommodation hoặc activities thông qua platform. Ước tính với 10,000 active users, conversion rate 15% sang Premium và 25% booking rate, doanh thu hàng tháng có thể đạt $25,000.

**Lợi thế cạnh tranh** của TravelSense v2 nằm ở cách tiếp cận co-planning độc đáo - thay vì tạo ra một black box AI hoàn toàn tự động, hệ thống minh bạch trong việc truy xuất nguồn gốc thông tin, cho phép người dùng kiểm soát và điều chỉnh theo ý muốn. Khả năng real-time optimization với tính toán ràng buộc thực tế (giờ mở cửa, thời tiết, giao thông) và tính năng local optimization (chỉnh sửa cục bộ không ảnh hưởng toàn bộ lịch trình) tạo ra trải nghiệm user-friendly mà các competitor hiện tại chưa có được.

## Chiến lược triển khai và đánh giá thành công

**Lộ trình phát triển** được thiết kế theo 4 giai đoạn linh hoạt nhưng có nhịp điệu rõ ràng. **Giai đoạn 1 (2-3 tuần)** tập trung xây dựng MVP core với hệ thống nhập liệu thông minh có khả năng phát hiện thiếu dữ liệu, tạo nháp lịch trình cơ bản và tích hợp 2-3 API miễn phí quan trọng nhất (OpenWeatherMap cho thời tiết, Foursquare cho POI, OpenStreetMap cho bản đồ). **Giai đoạn 2 (3-4 tuần)** nâng cấp khả năng tối ưu tuyến theo các ràng buộc thực tế, phát triển tính năng chỉnh sửa drag-drop với tính toán cục bộ và hoàn thiện bảng ngân sách chi tiết theo từng nhóm chi tiêu. **Giai đoạn 3 (2-3 tuần)** hoàn thiện tính năng xuất PDF/ICS/JSON, hệ thống chia sẻ và quản lý phiên bản, đồng thời tiến hành user testing với người dùng thật để thu thập feedback và tinh chỉnh. **Giai đoạn 4** là giai đoạn mở rộng liên tục với performance optimization, phát triển premium features và tích hợp đối tác để tạo doanh thu.

**Tiêu chí đánh giá thành công** được phân thành 3 nhóm chính. **Về mặt kỹ thuật**, hệ thống cần đạt response time dưới 2 giây cho việc tạo nháp lịch trình, duy trì 95% uptime và cache hit rate trên 80% để tối ưu chi phí API. **Về mặt kinh doanh**, mục tiêu là đạt plan completion rate trên 70% (người dùng hoàn thành và xuất lịch trình), user satisfaction score trên 4.0/5 điểm và tăng trưởng monthly active users trên 20% mỗi tháng. **Về chất lượng AI**, hệ thống cần đảm bảo source citation accuracy trên 95% (mọi thông tin đều có nguồn đáng tin cậy), constraint violation rate dưới 5% (lịch trình đề xuất khả thi) và tỉ lệ người dùng chấp nhận AI suggestions trên 60%, chứng tỏ AI thực sự hữu ích chứ không chỉ là gimmick công nghệ.
