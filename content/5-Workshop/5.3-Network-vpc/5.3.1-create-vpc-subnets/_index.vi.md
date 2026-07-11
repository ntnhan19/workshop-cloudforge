---
title : "Tạo VPC & Subnets"
date : 2026-07-09
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

Để đảm bảo tính sẵn sàng cao (High Availability) và bảo mật tuyệt đối cho các Microservices, chúng ta sẽ tự thiết kế một Virtual Private Cloud (VPC) riêng. Chúng ta sẽ tuân thủ nguyên tắc tách biệt hoàn toàn các tài nguyên giao tiếp với bên ngoài (như Application Load Balancer) khỏi các dịch vụ xử lý nội bộ và cơ sở dữ liệu (ECS, RDS, Redis).

Thay vì phải tạo thủ công từng thành phần rời rạc, chúng ta sẽ sử dụng công cụ **VPC and more** để khởi tạo nhanh toàn bộ kiến trúc mạng đa tầng, trải dài trên 2 Availability Zones với định tuyến (routing) đã được cấu hình sẵn.

#### Các bước triển khai

1. Truy cập vào dịch vụ **VPC** trên AWS Console.
2. Bấm nút màu cam **Create VPC**.
3. Ở mục **Resources to create**, chọn **VPC and more**.
4. Thiết lập các thông số mạng như sau:
   * **Name tag auto-generation:** Nhập `cloudforge`.
   * **IPv4 CIDR block:** `10.0.0.0/16` (Cung cấp 65,536 địa chỉ IP nội bộ).
   * **Number of Availability Zones (AZs):** Chọn `2` (Ví dụ: `ap-southeast-1a` và `ap-southeast-1b` để dự phòng sự cố).
   * **Number of public subnets:** Chọn `2` (Dành cho Application Load Balancer và NAT Gateway).
   * **Number of private subnets:** Chọn `2` (Dành cho ECS Fargate Workers, RDS PostgreSQL, và ElastiCache).
   * **NAT gateways ($):** Bấm chọn nút **In 1 AZ** (hoặc **Zonal** tùy theo bản cập nhật giao diện của AWS). *(Lưu ý: Môi trường Production thực tế thường dùng 1 NAT cho mỗi AZ, nhưng với dự án này, dùng 1 NAT là đủ để tối ưu chi phí).*
   * **VPC endpoints:** Chọn **S3 Gateway**. *(Cực kỳ quan trọng: Tính năng này cho phép các ứng dụng AI chạy ngầm trong Private subnet có thể đọc/ghi video vào S3 trực tiếp qua mạng nội bộ AWS, bỏ qua hoàn toàn phí truyền tải dữ liệu đắt đỏ của NAT Gateway).*
   * **DNS options:** Đảm bảo cả hai tuỳ chọn *Enable DNS resolution* và *Enable DNS hostnames* đều được tick.

5. Nhìn sang nửa phải màn hình, kiểm tra bảng **Preview**. Hệ thống sẽ vẽ ra một sơ đồ mạng lưới cực kỳ đẹp mắt và logic về hạ tầng bạn sắp tạo.

   ![VPC Preview Map](/images/5-Workshop/5.3-Network-vpc/5.3.1-create-vpc-subnets/vpc_preview_map.png)

6. Cuộn xuống cuối trang và bấm **Create VPC**.
7. Chờ vài phút để AWS cấp phát toàn bộ tài nguyên (VPC, Subnets, Internet Gateway, NAT Gateway, Route Tables và S3 Endpoint). Khi hoàn tất, bạn sẽ thấy màn hình báo thành công.

   ![VPC Success](/images/5-Workshop/5.3-Network-vpc/5.3.1-create-vpc-subnets/vpc_success_creation.png)
