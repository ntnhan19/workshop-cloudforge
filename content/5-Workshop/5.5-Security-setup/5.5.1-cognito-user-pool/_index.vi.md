---
title : "Thiết lập Cognito User Pool"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

Trong kiến trúc của Smart Media Analytics, chúng ta sẽ không tự xây dựng hệ thống quản lý tài khoản từ đầu (vì rất tốn thời gian và rủi ro bảo mật cao). Thay vào đó, chúng ta sẽ ủy quyền toàn bộ việc định danh và xác thực (Authentication) cho **Amazon Cognito**.

Hệ thống Backend (hoặc Frontend) sẽ giao tiếp với Cognito để đăng ký, đăng nhập và nhận về các token chuẩn JWT (JSON Web Token) để xác thực các yêu cầu API.

#### 1. Khởi tạo Cognito User Pool (Giao diện mới)
Truy cập vào **Amazon Cognito Console**, đảm bảo bạn đang ở đúng Region `ap-southeast-1` (Singapore) và làm theo giao diện thiết lập nhanh (Set up resources for your application):

**Bước 1: Define your application**
- **Application type:** Chọn **Single-page application (SPA)**. *(Việc chọn SPA sẽ giúp Cognito tự động cấu hình một Public Client không có Client Secret, rất phù hợp cho kiến trúc Frontend React/Vue gọi API).*
- **Name your application:** Nhập `cloudforge-app`.

**Bước 2: Configure options**
- **Options for sign-in identifiers:** Tích chọn **Email** (Người dùng sẽ dùng Email để đăng nhập).
- **Self-registration:** Tích chọn **Enable self-registration** (Cho phép người dùng tự đăng ký).
- **Required attributes for sign-up:** Click vào ô và chọn thuộc tính **name** (Yêu cầu người dùng phải nhập tên khi đăng ký).

**Bước 3: Add a return URL (Optional)**
- **Return URL:** Mặc dù ghi là tùy chọn, nhưng AWS khuyến nghị nhập. Bạn có thể nhập một địa chỉ tạm thời để phục vụ việc test ở Localhost: `https://localhost`

Cuối cùng, cuộn xuống dưới cùng và bấm nút **Create user directory** để AWS tự động thiết lập toàn bộ tài nguyên.

#### 2. Lấy thông tin tích hợp (App Client ID)
Sau khi tạo thành công, hệ thống sẽ đưa bạn vào trang quản lý của thư mục người dùng vừa tạo. Bạn cần ghi chú lại 2 thông số cực kỳ quan trọng để chuẩn bị nạp vào biến môi trường (`.env`) cho ứng dụng:

1. **User Pool ID:** Nằm ngay trên cùng của trang tổng quan (Có dạng `ap-southeast-1_xxxxxxxxx`).
2. **App Client ID:** Chuyển sang tab **App integration** (Tích hợp ứng dụng), cuộn xuống phần *App clients and analytics*, copy chuỗi ID của `cloudforge-app` (Ví dụ: `3abc123xyz...`).

![Cognito App Client](/images/5-Workshop/5.5-Security-setup/5.5.1-cognito-user-pool/cognito_app_client.png)

***

**Bước tiếp theo:** Hệ thống định danh người dùng đã hoàn tất. Chúng ta sẽ tiến tới phần **5.5.2: Quản lý cấu hình với Secrets Manager** để thiết lập kho lưu trữ siêu bảo mật, giải quyết bài toán cất giấu `DB_PASSWORD` một cách triệt để!
