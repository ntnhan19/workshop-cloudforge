---
title : "Triển khai với AWS Amplify"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.11.1. </b> "
---

Bài viết này ghi chép lại quy trình kỹ thuật để khởi tạo và triển khai liên tục (CI/CD) bộ mã nguồn giao diện (Frontend) của ứng dụng Smart Media Analytics thông qua dịch vụ AWS Amplify Hosting.

#### Bước 1: Khởi tạo ứng dụng Amplify
Quá trình triển khai bắt đầu bằng việc thiết lập một ứng dụng Web tĩnh (Web App) trên nền tảng AWS Amplify.

1. Truy cập dịch vụ **AWS Amplify** từ AWS Management Console.
2. Tại bảng điều khiển Amplify, cuộn xuống phần **Amplify Hosting** và chọn **Host your web app**.
3. Hệ thống sẽ yêu cầu kết nối với nền tảng lưu trữ mã nguồn (Version Control System). Nhóm dự án chọn mục **GitHub** và nhấn **Continue**.
4. Thực hiện xác thực (Authorize) cấp quyền cho AWS Amplify đọc dữ liệu từ tài khoản GitHub tương ứng.

![Amplify Connect GitHub](/images/5-Workshop/5.11-Frontend-deployment/5.11.1-deploy-amplify/5.11.1-amplify-github.png)

#### Bước 2: Thiết lập luồng triển khai (Build Settings)
Sau khi kết nối thành công, AWS Amplify yêu cầu chỉ định kho lưu trữ cụ thể và nhánh mã (Branch) sẽ được sử dụng làm nguồn cấp dữ liệu.

1. **Chỉ định Repository:** Tại phần *Repository and branch*, chọn đúng kho lưu trữ chứa mã nguồn Frontend (Ví dụ: `cloudforge-frontend`) và chọn nhánh **`main`** làm nhánh triển khai Production. Nhấn Next.
2. **Cấu hình Build:** Tại phần *Build settings*, Amplify tự động nhận diện framework để đưa ra file cấu hình `amplify.yml`. Nhóm dự án điều chỉnh phần `preBuild` để thêm các biến môi trường kết nối tới hệ thống Backend (Các biến này sẽ được thiết lập chi tiết ở bài 5.11.3).
3. **Ủy quyền tự động:** Đánh dấu vào tùy chọn *Allow AWS Amplify to automatically deploy all files hosted in your project root directory* để đảm bảo CI/CD diễn ra suôn sẻ. Nhấn Next.

#### Bước 3: Xem lại và Triển khai (Review & Deploy)
1. Ở màn hình Review, kiểm tra lại toàn bộ các thông số liên kết Repository và thiết lập Build.
2. Nhấn **Save and deploy**. 

*Trạng thái hệ thống: Lúc này, Amplify sẽ bắt đầu tiến trình tự động hóa gồm 4 giai đoạn:*

- **Provision:** Cấp phát môi trường container ảo để chuẩn bị biên dịch.
- **Build:** Tải mã nguồn từ GitHub, tải các thư viện (`npm install`) và biên dịch ứng dụng thành các tài nguyên tĩnh (`npm run build`).
- **Deploy:** Đẩy các tài nguyên tĩnh đã biên dịch lên hệ thống CDN (CloudFront + S3) ẩn của AWS.
- **Verify:** Kiểm tra trạng thái hoạt động của ứng dụng.

![Amplify Build Pipeline](/images/5-Workshop/5.11-Frontend-deployment/5.11.1-deploy-amplify/5.11.1-amplify-pipeline.png)

Khi cả 4 bước chuyển sang màu xanh (Success), AWS Amplify sẽ cung cấp một đường dẫn URL mặc định có định dạng `https://[branch-name].[app-id].amplifyapp.com`. 

{{% notice tip %}}
**Đánh giá Kiến trúc:** Đường dẫn mặc định của Amplify hoàn toàn có thể sử dụng được cho môi trường Staging/Testing. Tuy nhiên, đối với một ứng dụng Production chuyên nghiệp, nhóm dự án bắt buộc phải thay thế đường dẫn này bằng một tên miền tùy chỉnh (Custom Domain) để nâng cao nhận diện thương hiệu và thiết lập các chính sách bảo mật CORS nghiêm ngặt hơn.
{{% /notice %}}

***

**Bước tiếp theo:** Mã nguồn Frontend đã được biên dịch và phân phối thành công. Nhóm dự án sẽ tiến hành gắn tên miền tùy chỉnh cho ứng dụng thông qua dịch vụ Amazon Route 53 ở bài 5.11.2.
