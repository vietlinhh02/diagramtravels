# TravelSense v2 — Sơ đồ và giải thích 

## Tổng quan dự án

**TravelSense v2** là nền tảng lập kế hoạch du lịch thông minh sử dụng AI, hoạt động như một trợ lý đồng hành (co-planning) thay vì thay thế hoàn toàn quyết định của người dùng.

### Vấn đề giải quyết
- **Hiện tại**: Lập kế hoạch du lịch tốn thời gian, thiếu thông tin, khó tối ưu tuyến đường
- **Giải pháp**: AI chủ động phát hiện thiếu thông tin, gợi ý có căn cứ, tối ưu theo ràng buộc thực tế

### Đặc điểm chính
1. **Nhập liệu tối thiểu**: Chỉ hỏi thông tin cần thiết, hỏi bù thông minh
2. **Tối ưu thực tế**: Tôn trọng giờ mở cửa, thời tiết, nhịp độ nhóm
3. **Minh bạch nguồn**: Mọi gợi ý đều có dẫn chiếu và thời điểm kiểm tra
4. **Chỉnh sửa linh hoạt**: Kéo-thả, so sánh trước-sau, tính toán cục bộ
5. **Ngân sách rõ ràng**: Theo nhóm chi tiêu, biên độ dao động, cảnh báo vượt ngưỡng

## Danh sách sơ đồ

- [01 — Kiến trúc tổng quan](./01_kien_truc_tong_quan.md)
- [02 — Luồng dữ liệu chi tiết](./02_luong_du_lieu_chi_tiet.md)
- [03 — AI Orchestration](./03_ai_orchestration.md)
- [04 — Deployment & Infrastructure](./04_deployment_infrastructure.md)
- [05 — API miễn phí cho project](./05_free_apis.md)

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

## Giá trị thương mại

### Target Market
- **B2C**: Cá nhân/gia đình lập kế hoạch du lịch
- **B2B**: Travel agencies, tour operators
- **B2B2C**: Tích hợp vào platform du lịch existing

### Revenue Model
- Freemium: 3 kế hoạch/tháng miễn phí
- Premium: Unlimited + advanced features
- Commission: Từ booking accommodation/activities

### Competitive Advantage
- **Co-planning approach**: Không thay thế mà hỗ trợ quyết định
- **Real-time optimization**: Tính toán ràng buộc thực tế
- **Transparent sources**: Truy xuất nguồn gốc thông tin
- **Local optimization**: Chỉnh sửa cục bộ không ảnh hưởng toàn bộ

## Roadmap triển khai

### Phase 1 (2-3 tuần): MVP Core
- Nhập liệu thông minh + phát hiện thiếu dữ liệu
- Nháp lịch trình cơ bản
- Tích hợp 2-3 API miễn phí

### Phase 2 (3-4 tuần): Optimization
- Tối ưu tuyến theo ràng buộc
- Chỉnh sửa drag-drop
- Bảng ngân sách chi tiết

### Phase 3 (2-3 tuần): Export & Sharing
- Xuất PDF/ICS/JSON
- Chia sẻ và phiên bản
- User testing và feedback

### Phase 4 (ongoing): Scale & Business
- Performance optimization
- Premium features
- Partnership integrations

## Metrics thành công

### Technical
- Response time < 2s cho draft generation
- 95% uptime
- Cache hit rate > 80%

### Business
- Plan completion rate > 70%
- User satisfaction score > 4.0/5
- Monthly active users growth > 20%

### AI Quality
- Source citation accuracy > 95%
- Constraint violation rate < 5%
- User accepts AI suggestions > 60%
