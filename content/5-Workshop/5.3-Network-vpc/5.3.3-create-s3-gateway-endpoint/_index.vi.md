---
title : "Phân tích S3 Gateway Endpoint"
date : 2026-07-09
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

Trong các hệ thống xử lý Media / AI Pipeline, việc các Worker nằm trong Private Subnet hoặc các máy chủ ở Public Subnet phải liên tục đọc/ghi các file dung lượng lớn (như video, ảnh chất lượng cao) vào Amazon S3 là chuyện diễn ra liên tục. 

Nếu dữ liệu từ Private Subnet đi theo đường định tuyến mặc định (`Private Subnet` → `NAT Gateway` → `Internet` → `S3`), AWS sẽ tính phí xử lý dữ liệu của NAT Gateway (khoảng $0.045/GB). Với dung lượng video lớn của dự án, chi phí này sẽ tăng phi mã. Để giải quyết bài toán tối ưu chi phí này, công cụ **VPC and more** đã tự động thiết lập sẵn một **S3 Gateway Endpoint** để bẻ lái gói tin đi theo đường nội bộ.

#### 1. Kiểm tra trạng thái Endpoint
1. Tại thanh điều hướng bên trái của dịch vụ VPC, truy cập vào mục **Endpoints** (nằm ngay dưới mục *Managed prefix lists*).
2. Kiểm tra danh sách và chọn Endpoint có tên chứa mã dịch vụ S3 của region Singapore: `com.amazonaws.ap-southeast-1.s3`.
3. Đảm bảo trạng thái tại cột **Status** đang hiển thị màu xanh **Available** (Sẵn sàng hoạt động).

   ![S3 Endpoint Verification](/images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_verify.png)

#### 2. Phân tích Cấu hình Định tuyến trên các Route Tables
Bí mật của đường đi tắt nằm ở việc AWS tự động chèn một luật định tuyến đặc biệt vào cả hai nhóm bảng định tuyến (Public và Private) của dự án, giúp toàn bộ lưu lượng hướng tới S3 được giữ lại trong mạng băng thông nội bộ của AWS thay vì đi vòng ra Internet.

##### A. Đối với Private Route Tables (Tối ưu chi phí NAT)
1. Chọn một bảng định tuyến riêng tư (ví dụ: `cloudforge-rtb-private1`).
2. Tại tab **Routes**, quan sát luật định tuyến song song với NAT Gateway:
   - **Destination:** `pl-6fa54006` (Dải IP Prefix List đại diện cho toàn bộ dịch vụ S3 tại Region Singapore).
   - **Target:** Trỏ thẳng về ID của S3 Endpoint (`vpce-xxxxxx`).
   - **Ý nghĩa:** Mọi thao tác tải/xuất video từ ứng dụng AI Worker (Private) lên S3 sẽ đi trực tiếp qua cổng này, **bỏ qua hoàn toàn NAT Gateway**, giúp tốc độ đạt mức tối đa và chi phí xử lý dữ liệu qua NAT bằng **0 USD**.

   ![S3 Endpoint Routes Private](/images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_routes_private.png)

##### B. Đối với Public Route Table (Tối ưu băng thông công cộng)
1. Chuyển sang chọn bảng định tuyến công cộng `cloudforge-rtb-public`.
2. Tại tab **Routes**, bạn cũng sẽ thấy luật trỏ dải IP `pl-6fa54006` về `vpce-xxxxxx` xuất hiện độc lập với Internet Gateway (`igw-xxxx`).
   - **Ý nghĩa:** Ngay cả các tài nguyên ở vùng Public (như Load Balancer hoặc Bastion Host nếu có) khi tương tác với S3 cũng sẽ đi qua Endpoint nội bộ này, giảm tải cho Internet Gateway và tăng tính bảo mật cho luồng dữ liệu.

   ![S3 Endpoint Routes Public](/images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_routes_public.png)

***

**Bước tiếp theo:** Sau khi hệ thống mạng, định tuyến, và đường tắt tối ưu chi phí đã được xác minh hoạt động hoàn hảo, chúng ta sẽ tiến tới phần **5.3.4: Cấu hình Security Groups** để thiết lập các lớp khóa tường lửa bảo vệ cho từng dịch vụ cụ thể.
