---
title : "Thiết lập Tên miền Route 53"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.11.2. </b> "
---

Sau khi giao diện tĩnh đã được triển khai lên hạ tầng AWS Amplify, hệ thống sẽ cấp phát một đường dẫn URL mặc định. Để đáp ứng tiêu chuẩn của một hệ thống Production thực thụ, nhóm dự án tiến hành cấu hình tên miền tùy chỉnh (Custom Domain) thông qua Amazon Route 53.

Việc thiết lập Custom Domain không chỉ nâng cao nhận diện thương hiệu mà còn là yêu cầu bắt buộc để thiết lập các quy tắc CORS (Cross-Origin Resource Sharing) chặt chẽ trên API Gateway và Amazon Cognito ở các bước sau.

#### Bước 1: Quản lý tên miền trên Amplify
Quá trình tích hợp diễn ra trực tiếp trên giao diện của AWS Amplify nhờ khả năng liên kết tự động với Route 53.

1. Truy cập vào ứng dụng Smart Media Analytics trên bảng điều khiển **AWS Amplify**.
2. Tại thanh điều hướng bên trái, tìm đến mục **Hosting** và chọn **Custom domains**.
3. Nhấn vào nút **Add domain** ở góc trên cùng bên phải.

#### Bước 2: Cấu hình ánh xạ Tên miền
Hệ thống Amplify sẽ tự động truy xuất danh sách các Hosted Zones hiện có trong Amazon Route 53 của tài khoản AWS.

1. **Chọn Domain:** Tại trường *Domain*, chọn tên miền (Ví dụ: `cloudforge.com`) đã được đăng ký hoặc quản lý bởi Route 53.
2. **Thiết lập Subdomain:** Nhấn vào **Configure domain** để thiết lập quy tắc định tuyến.
   - Trỏ trực tiếp root domain (ví dụ: `cloudforge.com`) hoặc một subdomain (ví dụ: `app.cloudforge.com`) về nhánh mã nguồn `main` vừa triển khai ở bài 5.11.1.
   - Cấu hình chuyển hướng (Redirect) tự động từ `www.cloudforge.com` sang root domain để đảm bảo luồng truy cập của người dùng không bị phân mảnh.
3. Nhấn **Save** để lưu cấu hình.

![Amplify Custom Domain](/images/5-Workshop/5.11-Frontend-deployment/5.11.2-setup-route53/5.11.2-amplify-domain.png)

#### Bước 3: Xác thực SSL/TLS tự động (ACM)
Khác với kiến trúc truyền thống yêu cầu người quản trị phải tự cấp phát và cấu hình chứng chỉ bảo mật, AWS Amplify xử lý toàn bộ quy trình này một cách hoàn toàn tự động.

- Ngay khi lưu cấu hình, Amplify sẽ gửi yêu cầu tới **AWS Certificate Manager (ACM)** để cấp phát chứng chỉ SSL/TLS miễn phí cho tên miền vừa thiết lập.
- Hệ thống Amplify cũng tự động chèn các bản ghi CNAME xác thực vào Hosted Zone của Route 53 mà không cần sự can thiệp thủ công.
- Trạng thái của Domain sẽ chuyển qua các bước: *Pending verification* -> *Provisioning* -> *Available*.

Khi trạng thái chuyển thành **Available**, ứng dụng Frontend đã chính thức được bảo vệ bởi kết nối mã hóa HTTPS và sẵn sàng phục vụ người dùng thông qua Custom Domain.

{{% notice info %}}
**Lưu ý Kỹ thuật:** Quá trình xác thực chứng chỉ SSL và cập nhật bản ghi DNS (Domain Name System) trên toàn cầu có thể mất từ 5 đến 30 phút tùy thuộc vào hạ tầng mạng. 
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống Backend và Frontend đã được định danh và bảo mật bằng SSL. Trong bước cuối cùng ở bài 5.11.3, nhóm dự án sẽ thiết lập cấu hình biến môi trường trên Frontend để kết nối trực tiếp với API Gateway và hệ thống xác thực Amazon Cognito.
