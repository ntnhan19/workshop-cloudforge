---
title : "API & Real-time"
date : 2026-07-10
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### Tổng quan Giao tiếp Cấp phát & Thời gian thực (API & Real-time)

Trong các kiến trúc xử lý dữ liệu bất đồng bộ (Asynchronous) dựa trên hàng đợi, một thách thức kinh điển luôn được đặt ra: Làm thế nào để Frontend (ứng dụng trình duyệt của người dùng) biết được chính xác khi nào một tác vụ nền nặng nề (như phân tích AI) đã hoàn tất? Việc để Frontend liên tục gửi yêu cầu "hỏi thăm" máy chủ (Polling) sẽ bào mòn tài nguyên mạng và làm quá tải hệ thống một cách lãng phí.

Chương này sẽ giải quyết triệt để bài toán giao tiếp thời gian thực (Real-time) và quản lý giao diện lập trình (API Management) nhằm mang lại trải nghiệm người dùng liền mạch nhất.

#### Lợi ích cốt lõi của Kiến trúc API & Real-time:
- **Trải nghiệm mượt mà (Seamless UX):** Cơ chế WebSocket cho phép máy chủ chủ động đẩy (Push) thông báo trực tiếp về trình duyệt người dùng ngay khi AI Worker xử lý xong, với độ trễ gần như bằng không.
- **Bảo vệ hệ thống (Throttling & Security):** API Gateway đóng vai trò như một "người gác cổng", che giấu toàn bộ hạ tầng nội bộ bên dưới (ALB, ECS), đồng thời cung cấp năng lực giới hạn lưu lượng để chống lại các cuộc tấn công DDoS hoặc lạm dụng API.
- **Tách biệt phân lớp (Decoupling):** Frontend chỉ cần giao tiếp với một điểm cuối (Endpoint) duy nhất của API Gateway, không cần quan tâm đến sự phức tạp của mạng lưới Microservices phía sau.

#### Lộ trình triển khai phân lớp API & Real-time:
1. **Amazon API Gateway (HTTP API):** Khởi tạo cổng giao tiếp công cộng, hứng lưu lượng từ Internet.
2. **Application Load Balancer (ALB) Integration:** Cấu hình liên kết mạng riêng (VPC Link) để định tuyến an toàn lưu lượng từ API Gateway vào hệ thống Backend nằm sâu trong Private Subnet.
3. **Amazon API Gateway (WebSocket):** Thiết lập kết nối mạng hai chiều dai dẳng (Persistent Connection) chuyên phục vụ luồng thông báo tiến độ (Progress Tracking).
4. **Kiểm thử Luồng Real-time:** Xác thực toàn bộ chu trình đi từ lúc tải Video lên $\rightarrow$ Xử lý AI $\rightarrow$ Đẩy thông báo qua WebSocket về Frontend.

Với chương này, Smart Media Analytics không chỉ thông minh ở lớp học máy mà còn cực kỳ tinh tế và tối ưu trong cách tương tác với người dùng cuối!

### Nội dung thực hành

- [Tạo API Gateway](5.9.1-create-api-gateway/)
- [Tích hợp ALB](5.9.2-create-alb/)
- [Thiết lập WebSocket](5.9.3-websocket-progress/)
- [Kiểm thử Real-time Flow](5.9.4-test-realtime-flow/)
