---
title : "Kiểm tra NAT Gateway & Định tuyến"
date : 2026-07-09
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

Trong bước trước, công cụ **VPC and more** đã tự động cấu hình và khởi tạo một NAT Gateway cho hệ thống của hệ thống. NAT (Network Address Translation) Gateway là một thành phần tối quan trọng trong kiến trúc Multi-tier: Nó đóng vai trò làm "người trung gian", cho phép các tài nguyên nằm trong vùng biệt lập **Private Subnet** (như ECS Fargate Tasks) có thể chủ động đi ra ngoài internet để tải Docker Image hoặc gọi API của các dịch vụ ngoài, nhưng đồng thời chặn đứng hoàn toàn mọi nỗ lực kết nối bất hợp pháp từ Internet đi ngược vào trong lớp lõi.

Trong phần này, tiến hành tiến hành kiểm tra trạng thái hoạt động của NAT Gateway và phân tích sâu các Bảng định tuyến (Route Tables) để hiểu rõ luồng giao thông mạng được kiểm soát như thế nào.

#### 1. Kiểm tra NAT Gateway & Elastic IP
1. Tại thanh điều hướng bên trái của dịch vụ VPC, chọn mục **NAT gateways**.
2. Kiểm tra và đảm bảo trạng thái (**State**) của NAT Gateway mang tên `cloudforge-nat-...` đang hiển thị màu xanh **Available** (Sẵn sàng).
3. Quan sát mục **Elastic IP address**: Đây là địa chỉ IP tĩnh công cộng duy nhất mà AWS cấp phát riêng cho người dùng. Kể từ bây giờ, mọi luồng dữ liệu từ tất cả các máy chủ Private Subnet khi đi ra thế giới bên ngoài sẽ đều "ngụy trang" dưới địa chỉ IP công cộng này.

   ![NAT Gateway Verification](/images/5-Workshop/5.3-Network-vpc/5.3.2-create-nat-gateway/nat_gateway_verify.png)

#### 2. Phân tích Route Tables & Subnet Associations
Route Table (Bảng định tuyến) chứa các quy tắc mạng quyết định gói tin từ một Subnet sẽ được điều hướng đi đâu. Hệ thống tự động đã tạo ra các bảng định tuyến chuyên biệt:

##### A. Đối với Public Route Table (Tên chứa `cloudforge-rtb-public`)
1. Chọn bảng định tuyến công cộng này và bấm sang tab **Routes**. Sẽ thấy một luật định tuyến:
   * **Destination:** `0.0.0.0/0` (Toàn bộ lưu lượng Internet).
   * **Target:** Trỏ về `igw-XXXX` (Internet Gateway). Đây chính là "chìa khóa" biến các Subnet gắn liền với nó trở thành Public Subnet.
2. Chuyển sang tab **Subnet associations** → Mục **Explicit subnet associations**: Xác nhận rằng 2 Subnet Public (`cloudforge-subnet-public-...`) đã được gắn chặt vào đây.

   ![Public Route Table](/images/5-Workshop/5.3-Network-vpc/5.3.2-create-nat-gateway/public_route_table.png)

##### B. Đối với Private Route Tables (Tên chứa `cloudforge-rtb-private`)
1. Chọn bảng định tuyến riêng tư này và bấm sang tab **Routes**. Quan sát luật định tuyến:
   * **Destination:** `0.0.0.0/0` (Mọi traffic đi ra ngoài).
   * **Target:** Trỏ thẳng về `nat-XXXX` (Cục NAT Gateway vừa kiểm tra ở Mục 1). Việc này giúp dữ liệu nội bộ được bảo vệ an toàn khi ra Internet.
2. Chuyển sang tab **Subnet associations**: Xác nhận rằng các Subnet Private (nơi sẽ chứa ECS và RDS) đã được cô lập vào bảng này, tách biệt hoàn toàn khỏi Internet Gateway.

   ![Private Route Table](/images/5-Workshop/5.3-Network-vpc/5.3.2-create-nat-gateway/private_route_table.png)

***

**Bước tiếp theo:** Sau khi đã thông suốt luồng mạng ra Internet, ở bài **5.3.3**, tiến hành kiểm tra một thành phần tối ưu chi phí cực kỳ quan trọng: [**S3 Gateway Endpoint**](../5.3.3-create-s3-gateway-endpoint/) để đảm bảo quá trình truyền tải dữ liệu lớn (Video) không bị tính phí "oan" qua NAT Gateway.
