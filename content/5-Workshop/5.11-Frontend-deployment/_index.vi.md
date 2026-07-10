---
title : "Triển khai Frontend"
date : 2026-07-10
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

### Tổng quan Kiến trúc Triển khai Frontend

Sau khi hệ sinh thái xử lý dữ liệu (Backend) và trí tuệ nhân tạo đã vận hành trơn tru và bảo mật phía sau lớp mạng nội bộ VPC, nhóm dự án tiến hành bước tích hợp cuối cùng: Đưa giao diện người dùng (User Interface) của ứng dụng Smart Media Analytics lên môi trường Internet.

Để đáp ứng các tiêu chuẩn về tính sẵn sàng cao (High Availability), độ trễ thấp (Low Latency) và tự động hóa quy trình phân phối (CI/CD), nhóm dự án quyết định lựa chọn **AWS Amplify Hosting**.

#### Tại sao lại chọn AWS Amplify Hosting?
Khác với việc tự thiết lập hệ thống máy chủ tĩnh truyền thống (như cấu hình thủ công Amazon S3 kết hợp với CloudFront), AWS Amplify cung cấp một giải pháp quản lý toàn diện (Fully Managed) với các ưu điểm kiến trúc vượt trội:

- **Tự động hóa CI/CD:** Kết nối trực tiếp với kho lưu trữ mã nguồn (GitHub/GitLab/CodeCommit). Mỗi khi có nhánh mã mới được hợp nhất (Merge), Amplify tự động kích hoạt tiến trình Build và Deploy mà không cần công cụ bên thứ ba.
- **Mạng phân phối nội dung toàn cầu (CDN):** Được vận hành ngầm trên nền tảng hạ tầng CloudFront của AWS, đảm bảo tài nguyên tĩnh (HTML/CSS/JS) được phân phối với tốc độ tối đa ở mọi khu vực địa lý.
- **Đơn giản hóa định tuyến và bảo mật:** Hỗ trợ sẵn cấu hình Custom Domain (Tên miền tùy chỉnh) qua Route 53 và tự động quản lý chứng chỉ SSL/TLS miễn phí (thông qua AWS Certificate Manager).

#### Lộ trình triển khai Frontend:
1.  **Khởi tạo AWS Amplify:** Kết nối kho lưu trữ mã nguồn Frontend và thiết lập môi trường Build.
2.  **Cấu hình Domain (Route 53):** Ánh xạ tên miền tùy chỉnh vào ứng dụng Amplify để thay thế đường dẫn mặc định.
3.  **Tích hợp xác thực (Cognito):** Cấu hình biến môi trường và liên kết giao diện với hệ thống xác thực tập trung Amazon Cognito.

### Nội dung thực hành

- [Triển khai với AWS Amplify](5.11.1-deploy-amplify/)
- [Thiết lập Tên miền Route 53](5.11.2-setup-route53/)
- [Kết nối Cognito và Frontend](5.11.3-connect-cognito-frontend/)
