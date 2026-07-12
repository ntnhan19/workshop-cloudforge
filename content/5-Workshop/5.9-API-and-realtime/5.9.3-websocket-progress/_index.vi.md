---
title : "Thiết lập WebSocket"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.9.3. </b> "
---

Nếu HTTP API (RESTful) đóng vai trò là hệ xương sống vững chắc cho các luồng thao tác gửi/nhận dữ liệu tiêu chuẩn, thì giao thức **WebSocket** chính là "hệ thống thần kinh" (Nervous system) truyền tải các luồng tín hiệu thời gian thực (Real-time).

Thay vì bắt buộc Frontend phải liên tục gửi yêu cầu "hỏi thăm" máy chủ (cơ chế Polling) - một phương pháp vô cùng tốn kém tài nguyên mạng và tạo áp lực khổng lồ lên cơ sở dữ liệu, chúng ta sẽ thiết lập một luồng kết nối WebSocket dai dẳng (Persistent Connection). Cơ chế này cho phép máy chủ chủ động đẩy (Push) thông báo trực tiếp về trình duyệt của người dùng ngay khoảnh khắc AI Worker hoàn tất vòng đời phân tích.

#### Sức mạnh tiềm ẩn của Application Load Balancer (ALB)
Trong quá trình triển khai thực tế, việc thiết lập WebSocket qua API Gateway đòi hỏi kiến trúc phức tạp (chẳng hạn như cần Network Load Balancer - NLB để kết nối VPC Link, hoặc phải viết mã nguồn quản lý trạng thái `@connections`).

Tuy nhiên, mã nguồn Backend FastAPI của dự án đã được lập trình sẵn để quản lý kết nối WebSocket nguyên bản (Native WebSockets) thông qua thư viện `fastapi.WebSocket` và Redis PubSub. Điều tuyệt vời là **Application Load Balancer (ALB) mà chúng ta đã triển khai ở chương 5.7 hỗ trợ định tuyến giao thức WebSocket (ws:// và wss://) một cách hoàn toàn tự nhiên** trên cùng một cổng HTTP (Port 80) mà không cần cấu hình thêm bất cứ dòng nào!

#### Cơ chế đẩy thông báo (Stateful Push Notification)
Nhờ sự hỗ trợ nguyên bản của ALB, luồng dữ liệu thời gian thực hoạt động cực kỳ mượt mà và trực tiếp:
1. **Thiết lập kết nối:** Trình duyệt Frontend sẽ không gọi vào API Gateway, mà mở kết nối WebSocket đâm xuyên trực tiếp qua ALB: `ws://[ALB-DNS]/api/v1/ingest/ws/[job_id]`.
2. **Duy trì trạng thái (Stateful):** ALB chuyển tiếp nguyên vẹn phiên WebSocket này tới một Container ECS Fargate đang chạy FastAPI. Container này sẽ giữ kết nối mở.
3. **Phát sóng (Pub/Sub):** Khi AI Worker xử lý xong Video, nó báo cáo tiến độ vào Redis. Backend FastAPI đang "lắng nghe" Redis sẽ ngay lập tức chộp lấy thông điệp này.
4. **Đẩy kết quả:** Backend đẩy trực tiếp gói tin JSON chứa trạng thái `ANALYSIS_COMPLETED` qua đường truyền WebSocket đang mở thẳng về màn hình người dùng.

{{% notice tip %}}
**Lợi thế Kiến trúc:** Giải pháp sử dụng Native WebSockets qua ALB giúp đơn giản hóa cực độ hạ tầng AWS (loại bỏ hoàn toàn API Gateway WebSocket rườm rà), giảm thiểu độ trễ mạng (Network Hop), và tiết kiệm tối đa chi phí vận hành cho doanh nghiệp.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống giao tiếp thời gian thực đã sẵn sàng. Chúng ta sẽ tiến tới bài học cuối cùng của chương: **Kiểm thử luồng dữ liệu toàn vẹn (End-to-End)** để xác nhận sự phối hợp nhịp nhàng giữa HTTP API và WebSocket.
