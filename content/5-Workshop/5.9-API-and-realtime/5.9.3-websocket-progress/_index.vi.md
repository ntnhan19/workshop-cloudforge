---
title : "Thiết lập WebSocket"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.9.3. </b> "
---

Nếu HTTP API (RESTful) đóng vai trò là hệ xương sống vững chắc cho các luồng thao tác gửi/nhận dữ liệu tiêu chuẩn, thì giao thức **WebSocket** chính là "hệ thống thần kinh" (Nervous system) truyền tải các luồng tín hiệu thời gian thực (Real-time).

Thay vì bắt buộc Frontend phải liên tục gửi yêu cầu "hỏi thăm" máy chủ (cơ chế Polling) - một phương pháp vô cùng tốn kém tài nguyên mạng và tạo áp lực khổng lồ lên cơ sở dữ liệu, chúng ta sẽ thiết lập một luồng kết nối WebSocket dai dẳng (Persistent Connection). Cơ chế này cho phép máy chủ chủ động đẩy (Push) thông báo trực tiếp về trình duyệt của người dùng ngay khoảnh khắc AI Worker hoàn tất vòng đời phân tích.

#### Cấu trúc của WebSocket API
Khác với giao thức HTTP truyền thống (Stateless), một kết nối WebSocket duy trì trạng thái kết nối hai chiều (Bi-directional). AWS API Gateway quản lý WebSocket thông qua các tuyến đường (Routes) cốt lõi:
- **`$connect`**: Kích hoạt khi trình duyệt mở kết nối. Đây là lúc kết nối được thiết lập và giữ chân người dùng trên "đường truyền" thời gian thực.
- **`$disconnect`**: Kích hoạt khi người dùng đóng trình duyệt hoặc mất kết nối, giúp hệ thống dọn dẹp các phiên làm việc cũ.

#### Khởi tạo và Cấu hình Tối ưu
Trong quá trình triển khai thực tế, dự án đối mặt với một đặc điểm kiến trúc của AWS: WebSocket API yêu cầu Network Load Balancer (NLB) nếu muốn tích hợp sâu vào mạng nội bộ qua VPC Link. Để tối ưu hóa chi phí hạ tầng và đơn giản hóa luồng vận hành cho Workshop mà vẫn đảm bảo tính năng Real-time, dự án lựa chọn giải pháp **Mock Integration**.

1. Truy cập dịch vụ **API Gateway**, chọn **Build** tại khối **WebSocket API**.
2. **Thiết lập cơ bản:**
   - **API name:** `CloudForge-Media-WS`.
   - **Route selection expression:** `$request.body.action` (Dùng để định tuyến các thông điệp từ người dùng dựa trên trường "action" trong gói tin JSON).
3. **Cấu hình Tuyến đường (Routes):** Thêm hai route mặc định là `$connect` và `$disconnect`.
4. **Thiết lập Tích hợp (Integrations):**
   - Đối với cả hai route, chọn **Integration type** là **Mock**. 
   - Việc sử dụng Mock Integration đóng vai trò như một "trạm trung chuyển" nhanh, lập tức trả về phản hồi thành công (HTTP 200) để trình duyệt thiết lập kết nối ngay lập tức mà không cần chờ đợi xử lý phức tạp từ phía sau.
5. **Triển khai:** Tạo Stage tên là `production` và tiến hành **Create and deploy**.

![WebSocket Routes](/images/5-Workshop/5.9-API-and-realtime/5.9.3-websocket-progress/websocket_routes.png)

#### Cơ chế đẩy thông báo (Push Notification)
Điểm tinh tế của kiến trúc này nằm ở chỗ: Mặc dù cổng vào là "Mock", nhưng luồng dữ liệu thời gian thực vẫn hoạt động hoàn hảo:
- Khi AI Worker xử lý xong, thông tin được ghi vào Database. 
- Backend API sẽ sử dụng quyền IAM để gọi trực tiếp vào API Gateway thông qua Endpoint quản lý kết nối (**@connections API**). 
- API Gateway dựa trên mã kết nối (`connectionId`) đang được giữ mở để đẩy gói tin JSON kết quả thẳng xuống màn hình người dùng.

{{% notice tip %}}
**Lợi thế Kiến trúc:** Giải pháp này đảm bảo trải nghiệm người dùng (UX) đạt mức tối đa với thông báo tức thì, trong khi vẫn giữ cho hạ tầng AWS cực kỳ gọn nhẹ, không phát sinh chi phí cho các bộ cân bằng tải bổ sung không cần thiết.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống giao tiếp thời gian thực đã sẵn sàng. Chúng ta sẽ tiến tới bài học cuối cùng của chương: **Kiểm thử luồng dữ liệu toàn vẹn (End-to-End)** để xác nhận sự phối hợp nhịp nhàng giữa HTTP API và WebSocket.
