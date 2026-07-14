---
title : "Security Setup"
date : 2026-07-10
weight : 5 
chapter : false
pre : " <b> 5.5. </b> "
---

### Tổng quan Thiết lập Bảo mật (Security Setup)

Bảo mật không phải là một "tính năng" thêm vào sau, mà là nền tảng bắt buộc của mọi kiến trúc Cloud vững chắc. Đối với hệ thống Smart Media Analytics, nơi xử lý các dữ liệu truyền thông và tích hợp AI, cần thiết lập một chiến lược phòng thủ nhiều lớp (Defense in Depth).

Trong chương này, tiến hành hiện thực hóa các tiêu chuẩn bảo mật khắt khe nhất thông qua 3 trụ cột chính:

1. **Amazon Cognito (Identity & Access):** Xây dựng hệ thống định danh người dùng an toàn. Dịch vụ này sẽ gánh vác toàn bộ vòng đời xác thực (đăng ký, đăng nhập, quên mật khẩu) và cấp phát token chuẩn OAuth 2.0 / JWT mà chúng ta không cần phải tự xây dựng từ đầu.
2. **AWS Secrets Manager (Data Protection):** Hiện thực hóa cảnh báo bảo mật ở chương trước. Toàn bộ thông tin nhạy cảm như `DB_PASSWORD` hay API Keys sẽ được mã hóa và quản lý tập trung tại đây, loại bỏ hoàn toàn rủi ro lộ lọt từ mã nguồn hoặc biến môi trường (Plaintext).
3. **AWS WAF (Edge Protection):** Triển khai "tấm khiên" Web Application Firewall ở vòng ngoài cùng nhằm đánh chặn các luồng truy cập độc hại, tự động ngăn chặn các lỗ hổng phổ biến như SQL Injection, Cross-Site Scripting (XSS) và các cuộc tấn công DDoS.

Việc hoàn thành chương này sẽ nâng tầm kiến trúc dự án của hệ thống từ mức độ "chạy thử nghiệm" (Proof of Concept) lên chuẩn "sẵn sàng thực chiến" (Production-ready) với độ tin cậy cao nhất.

### Nội dung thực hành

- [Thiết lập Cognito User Pool](5.5.1-cognito-user-pool/)
- [Quản lý cấu hình với Secrets Manager](5.5.2-secrets-manager/)
- [Cấu hình WAF Rules bảo vệ API](5.5.3-waf-rules/)
