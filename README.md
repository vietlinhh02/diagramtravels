# TravelSense v2 — Hệ thống lập kế hoạch du lịch thông minh

## Tổng quan và kịch bản project

**TravelSense v2** là một nền tảng lập kế hoạch du lịch thông minh được thiết kế để giải quyết vấn đề nan giải mà hầu hết du khách đều gặp phải: việc lập kế hoạch du lịch tốn rất nhiều thời gian, thường thiếu thông tin đáng tin cậy và khó có thể tối ưu hóa tuyến đường một cách hiệu quả. Thay vì tạo ra một công cụ AI hoàn toàn tự động hóa quy trình, TravelSense v2 hoạt động như một trợ lý đồng hành (co-planning assistant), nơi AI không thay thế quyết định của người dùng mà chủ động phát hiện những thông tin còn thiếu, đưa ra các gợi ý có căn cứ và minh bạch về nguồn gốc, đồng thời tối ưu hóa lịch trình theo các ràng buộc thực tế như giờ mở cửa, điều kiện thời tiết và nhịp độ phù hợp với từng nhóm du khách.

Hệ thống được xây dựng trên triết lý "nhập liệu tối thiểu nhưng thông minh", chỉ hỏi những thông tin thực sự cần thiết và sử dụng các câu hỏi bổ sung ngắn gọn có thể bỏ qua khi chưa cấp thiết. Khi người dùng cung cấp các dữ liệu cơ bản như điểm xuất phát, điểm đến, khung thời gian và ngân sách, hệ thống sẽ tạo ra một bản nháp lịch trình thông minh, tích hợp dữ liệu từ nhiều nguồn API miễn phí và trả phí để đảm bảo độ chính xác. Điểm đặc biệt của TravelSense v2 là khả năng chỉnh sửa cục bộ - khi người dùng thay đổi một phần lịch trình bằng cách kéo-thả, hệ thống chỉ tính toán lại những đoạn bị ảnh hưởng thay vì làm lại toàn bộ, đồng thời cung cấp so sánh rõ ràng giữa trước và sau thay đổi về mặt thời gian, chi phí và tính khả thi.

## Kịch bản sử dụng thực tế

### Kịch bản 1: Lập kế hoạch du lịch gia đình

**Tình huống mẫu**: Gia đình 4 người (2 người lớn, 2 trẻ em 8 và 12 tuổi) muốn đi du lịch Đà Nẵng - Hội An trong 4 ngày 3 đêm với ngân sách 20 triệu VNĐ, ưu tiên hoạt động phù hợp với trẻ em và muốn trải nghiệm cả văn hóa lẫn biển.

**Bước 1 - Input Phase (Nhập liệu thông minh)**: Người dùng nhập thông tin cơ bản (điểm xuất phát: Hà Nội, điểm đến: Đà Nẵng-Hội An, thời gian: 4 ngày, 4 người, ngân sách: 20M). Hệ thống Missing Info Detector phân tích user profile history và tạo priority question list với completeness score. AI đưa ra câu hỏi bổ sung theo mức độ ưu tiên: "Trẻ em có sợ độ cao không?" (cho cầu Vàng), "Ưu tiên resort gần biển hay trung tâm Hội An?", "Có ai bị dị ứng thực phẩm?". Người dùng có thể bỏ qua câu hỏi nào không quan trọng, hệ thống vẫn áp dụng smart defaults từ similar trips.

**Bước 2 - Data Enrichment (Làm giàu dữ liệu)**: Hệ thống chạy song song multiple processes: Vector Search tìm similar family trips đến Đà Nẵng-Hội An từ database; External APIs gọi đồng thời Places API (POI details, reviews), Weather API (dự báo 4 ngày), Maps API (distance matrix), Restaurant APIs (Zomato/local sources cho dining options), và Accommodation APIs (hotels/resorts có kids-friendly facilities). LLM Processing phân tích user persona từ tất cả dữ liệu này để generate personalized activity ideas và suggest hidden gems phù hợp với gia đình có trẻ em.

**Bước 3 - Draft Generation (Tạo nháp với constraint modeling)**: Hệ thống xây dựng comprehensive constraint matrix bao gồm time windows (giờ mở cửa POI, kids bedtime), budget limits per category, group preferences (accessibility, kids-friendly activities). Route Optimizer chạy Nearest Neighbor TSP kết hợp local 2-opt optimization, sau đó Time Feasibility Check xác minh tính khả thi. Draft itinerary được tạo với timeline chi tiết: Ngày 1 (bay 8:00, đến 10:30, check-in resort 12:00, mì Quảng Bà Mua 13:00-14:30, biển Mỹ Khê 15:00-17:30); Ngày 2 (Bà Nà Hills 8:00-15:00 với gap warning về thời gian di chuyển đến Hội An); kèm Gap Detection highlight những thông tin còn thiếu và source citation cho mỗi gợi ý.

**Bước 4 - User Interaction (Tương tác real-time)**: Interface cho phép drag-drop POI với live recalculation. Khi user chọn resort gần biển thay vì trung tâm, hệ thống instantly update travel times, detect potential conflicts (ví dụ: từ resort đến phố cổ Hội An chiều nay có thể tắc đường), và trigger Alternative Suggestions. Budget breakdown được update real-time theo từng thay đổi với detailed cost estimates từ accommodation pricing APIs. WebSocket connection đảm bảo UI reflects changes immediately với visual indicators cho feasible/conflict states.

**Bước 5 - Optimization (Tối ưu cuối cùng)**: Multi-objective optimization engine chạy Genetic Algorithm cho global search kết hợp Simulated Annealing cho local optimization, với primary objective là feasibility và secondary objective là user satisfaction score. LLM Structured Validation (GPT-4/Claude Sonnet) double-check mọi constraints và suggest alternatives nếu phát hiện issues. Final itinerary được tạo với optimized route, detailed timeline, backup options cho mỗi activity và comprehensive cost breakdown.

**Bước 6 - Export & Version Management**: Hệ thống tạo snapshot của current state và calculate diff từ previous versions. User có thể export multiple formats: PDF với embedded maps và QR codes cho bookings, ICS với calendar events cho từng thành viên gia đình, JSON cho integration với travel apps, và shareable web link với real-time updates. Version control cho phép rollback hoặc tạo variations, với change history tracking để hiểu user preferences cho future trips.

### Kịch bản 2: Khám phá tại địa phương (Local Discovery)

**Tình huống**: Anh Minh ở Hà Nội muốn tìm điều gì đó thú vị để làm vào cuối tuần này, nhưng không có ý tưởng cụ thể.

**Bước 1 - Weekend Planner tự động**: Vào thứ Sáu, TravelSense gửi thông báo: "Cuối tuần này trời mát mẻ và có triển lãm nghệ thuật mới tại Vincom Center. Bạn có muốn thử một food tour mini quanh Hàng Bông kết hợp xem triển lãm không?"

**Bước 2 - Tùy chỉnh theo mood**: Anh Minh nhập "Tìm một nơi yên tĩnh để đọc sách và uống cafe", hệ thống gợi ý 3 quán cafe ít người biết với không gian riêng tư, kèm menu đồ uống và đánh giá về độ ồn.

**Bước 3 - Tạo Food Tour tự động**: Chọn chủ đề "bún phở Hà Nội", AI tạo ra tuyến đường đi qua 4 quán bún phở nổi tiếng trong bán kính 2km, tối ưu hóa quãng đường và thời gian, ước tính tổng chi phí 150K cho cả tour.

### Kịch bản 3: Xây dựng Travel Log và chia sẻ kinh nghiệm

**Tình huống**: Chị Lan vừa hoàn thành chuyến đi Sapa và muốn lưu lại kỷ niệm đồng thời chia sẻ với bạn bè.

**Bước 1 - Auto Travel Log**: Sau khi chuyến đi kết thúc, TravelSense tự động tạo nhật ký với bản đồ route đã đi, các địa điểm check-in và timeline chi tiết. Chị Lan có thể thêm ảnh và ghi chú cá nhân.

**Bước 2 - Tạo Collection**: Chị Lan tạo bộ sưu tập "Trekking cho người mới bắt đầu" với các địa điểm, tips và gear list từ kinh nghiệm thực tế, chia sẻ công khai để giúp các traveler khác.

**Bước 3 - Social Integration**: Theo dõi travel planner uy tín, xem các lịch trình mới và được notify khi có bộ sưu tập mới phù hợp với sở thích.

### Kịch bản 4: Gamification và thử thách

**Tình huống**: Nam, sinh viên 22 tuổi, thích khám phá những nơi mới nhưng ngân sách hạn chế.

**Bước 1 - Thử thách hàng tháng**: Nhận thử thách "Khám phá 5 quán ăn dưới 50K trong tháng này", mỗi quán được AI gợi ý dựa trên location và review rating.

**Bước 2 - Tích điểm và huy hiệu**: Hoàn thành lịch trình cuối tuần tại Hà Nội cũ nhận 50 điểm, viết review chi tiết nhận thêm 25 điểm. Đạt 500 điểm trong tháng được huy hiệu "Local Explorer" và voucher Grab 20K.

**Bước 3 - Thăng hạng**: Sau 6 tháng tích cực, Nam đạt level "Hanoi Expert" và được mời tham gia beta test các tính năng mới, đồng thời có thể tạo và chia sẻ các mini-guide riêng.

## Danh sách sơ đồ

- [01 — Kiến trúc tổng quan](./01_kien_truc_tong_quan.md)
- [02 — Luồng dữ liệu chi tiết](./02_luong_du_lieu_chi_tiet.md)
- [03 — AI Orchestration](./03_ai_orchestration.md)
- [04 — Deployment & Infrastructure](./04_deployment_infrastructure.md)
- [05 — API miễn phí cho project](./05_free_apis.md)
- [06 — Tích hợp Ăn uống & Nhà hàng](./06_dining_integration.md)
- [07 — Chiến lược giữ chân người dùng](./07_user_retention_strategy.md)

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
