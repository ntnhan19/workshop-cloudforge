---
title : "Tích hợp ALB nội bộ"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.9.2. </b> "
---

Sau khi đã khởi tạo xong cổng API Gateway công cộng, thách thức tiếp theo là làm sao để chuyển tiếp (Forward) các yêu cầu từ cổng này đâm xuyên qua ranh giới mạng riêng để đến trúng đích **Application Load Balancer (ALB)** của Backend đang nằm trong Private Subnets. 

Để giải quyết bài toán này mà vẫn đảm bảo tính bảo mật nghiêm ngặt, AWS cung cấp một giải pháp kết nối chuyên dụng gọi là **VPC Link**. Trong phân đoạn này, chúng ta sẽ thiết lập VPC Link và cấu hình định tuyến tích hợp cho HTTP API.

#### Khái niệm VPC Link trong HTTP API
Đối với dòng HTTP API, VPC Link hoạt động như một "cây cầu mạng ảo" kết nối trực tiếp tài nguyên của API Gateway (vốn nằm ngoài VPC) với một điểm cuối bên trong mạng Private của bạn. Cơ chế này mang lại các lợi điểm:
- **Bảo mật tuyệt đối:** Lưu lượng từ API Gateway đi vào ALB hoàn toàn thông qua đường truyền nội bộ của AWS, không cần mở public ALB ra Internet.
- **Tách biệt hạ tầng:** Khách hàng chỉ nhìn thấy Endpoint của API Gateway, toàn bộ cấu trúc định tuyến ALB và ECS phía sau được ẩn giấu hoàn toàn.

#### Bước 1: Khởi tạo VPC Link
1. Tại thanh điều hướng bên trái của dịch vụ API Gateway, nhấp chọn mục **VPC links**.
2. Nhấp chuột vào nút màu cam **Create** ở góc phải màn hình.
3. Cấu hình các thông số cho VPC Link như sau:
   - **Choose a VPC link version:** Tích chọn **VPC link V2** (Bắt buộc chọn V2 vì đây là phiên bản hỗ trợ giao thức HTTP API mới).
   - **Name:** Nhập tên `cloudforge-vpc-link`.
   - **VPC:** Chọn đúng VPC của dự án (`cloudforge-vpc`).
   - **Subnets:** Tích chọn 2 **Private Subnets** ở hai Availability Zones khác nhau (`cloudforge-subnet-private1-ap-southeast-1a` và `cloudforge-subnet-private2-ap-southeast-1b`) để đảm bảo tính sẵn sàng cao.
   - **Security groups:** Tích chọn **`default`** (hoặc `cloudforge-ecs-app-sg` đều được, để đảm bảo VPC Link có quyền Outbound đẩy luồng dữ liệu tới ALB).
4. Nhấn **Create** và đợi khoảng 1-2 phút để trạng thái chuyển sang **Available**.

![Create VPC Link](/images/5-Workshop/5.9-API-and-realtime/5.9.2-create-alb/create_vpc_link.png)

#### Bước 2: Cấu hình định tuyến (Routes & Integration) cho API
Khi đã có cây cầu VPC Link, chúng ta cần cấu hình cho API biết khi nào thì cần sử dụng cây cầu này để đẩy dữ liệu vào Backend.

1. Quay lại trang chi tiết của API `CloudForge-Media-API`. Nhìn menu bên trái, chọn mục **Routes**.
2. Nhấn **Create** để tạo một tuyến đường mới. Ở ô **Path**, nhập đường dẫn bắt tất cả (catch-all) là `/{proxy+}` và chọn phương thức là **ANY** (Điều này giúp API hứng toàn bộ các requests như `/api/v1/auth`, `/api/v1/videos`... và chuyển tiếp nguyên vẹn sang Backend). Nhấn **Create**.
3. Sau khi Route được tạo, nhấp chọn vào dòng chữ `ANY /{proxy+}` vừa sinh ra. Nhìn sang khung bên phải, bấm nút **Attach integration** (Gắn kết nối tích hợp).
4. Bấm **Create integration** và điền thông tin kết nối vào ALB:
   - **Integration type:** Chọn **Private resource** (Tài nguyên riêng tư).
   - **Integration method:** Chọn **ANY**.
   - **Target service:** Chọn **VPC link**.
   - **VPC link:** Chọn đúng cái `cloudforge-vpc-link` vừa tạo ở Bước 1.
   - **Target Type:** Chọn **Application Load Balancer**.
   - **Load balancer:** Chọn đúng ALB của dịch vụ backend (`cloudforge-backend-alb`).
   - **Listener:** Chọn listener cổng `80` (HTTP) hoặc `443` (HTTPS) của ALB tùy thuộc vào cấu hình Backend.
5. Cuộn xuống dưới cùng và nhấn **Create**.

![Attach ALB Integration](/images/5-Workshop/5.9-API-and-realtime/5.9.2-create-alb/attach_alb_integration.png)

{{% notice tip %}}
**Cốt lõi Vận hành:** Kể từ thời điểm này, luồng giao tiếp chuẩn (REST API HTTP) đã thông suốt. Bất kỳ yêu cầu nào gửi tới `https://[API-Gateway-URL]/api/v1/...` sẽ được tự động đâm xuyên qua VPC Link, đi qua ALB và phân phối thẳng tới các Container Backend chạy trong ECS Fargate.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống đã thông suốt luồng gọi API chuẩn. Tuy nhiên, để đáp ứng tính năng thông báo trạng thái phân tích video tức thì, chúng ta cần thiết lập thêm một kênh kết nối hai chiều song song. Hãy cùng bước sang bài tiếp theo để xây dựng hạ tầng **API Gateway WebSocket**!
