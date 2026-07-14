---
title : "VPC & Networking"
date : 2026-07-09 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Nền móng Mạng nội bộ (VPC)

Trong phần này, tiến hành xây dựng một Virtual Private Cloud (VPC) bảo mật và có tính sẵn sàng cao để lưu trữ toàn bộ kiến trúc của hệ thống Smart Media Analytics. Thay vì sử dụng mạng mặc định, việc tự thiết kế một VPC giúp chúng ta kiểm soát hoàn toàn luồng giao thông mạng (traffic flow), cô lập các dịch vụ nhạy cảm và tối ưu hóa chi phí đường truyền.

Tiến hành triển khai một kiến trúc mạng đa tầng (multi-tier network) bao gồm:
- **Public Subnets**: Nơi đặt Application Load Balancer để tiếp nhận các yêu cầu từ người dùng (Frontend).
- **Private Subnets**: Nơi đặt các dịch vụ xử lý nội bộ (ECS Fargate Workers), cơ sở dữ liệu (RDS PostgreSQL) và bộ đệm (ElastiCache Redis) nhằm đảm bảo chúng không bao giờ bị truy cập trực tiếp từ Internet.
- **NAT Gateway**: Đóng vai trò là cầu nối giúp các dịch vụ trong Private Subnet có thể tải các bản vá lỗi hoặc kéo các model AI từ bên ngoài một cách an toàn.
- **S3 Gateway Endpoint**: Một "đường cao tốc" nội bộ giúp ECS Fargate lưu trữ và truy xuất video từ Amazon S3 mà không bị tính phí truyền tải dữ liệu qua NAT Gateway.

Bằng cách sử dụng công cụ **VPC and more** tích hợp sẵn trên AWS Console, toàn bộ các thành phần này cùng với hệ thống định tuyến (Route Tables) sẽ được khởi tạo tự động một cách chuẩn xác, giúp tiết kiệm thời gian đáng kể.

#### Nội dung

- [Tạo VPC & Subnets](5.3.1-create-vpc-subnets/)
- [Kiểm tra NAT Gateway & Định tuyến](5.3.2-create-nat-gateway/)
- [Phân tích S3 Gateway Endpoint](5.3.3-create-s3-gateway-endpoint/)
- [Cấu hình Security Groups](5.3.4-security-groups/)
