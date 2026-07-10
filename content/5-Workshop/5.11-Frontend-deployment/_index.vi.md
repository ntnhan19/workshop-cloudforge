---
title : "Triển khai Frontend"
date : 2026-07-10
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

### Tổng quan Triển khai Frontend

Hệ thống Backend vững chắc cần một giao diện người dùng (UI) tương xứng. Trong chương này, chúng ta sẽ đưa bộ mã nguồn Frontend (React/Vue/Angular) lên môi trường Production.

Chúng ta sẽ sử dụng các dịch vụ:
1. **AWS Amplify Hosting (hoặc S3 + CloudFront):** Triển khai tĩnh (Static Hosting) các file giao diện với hiệu năng cực cao, tự động phân phối nội dung trên toàn cầu (CDN).
2. **Amazon Route 53:** Quản lý tên miền tùy chỉnh (Custom Domain) và thiết lập chứng chỉ SSL/TLS miễn phí qua AWS Certificate Manager (ACM) để đảm bảo kết nối HTTPS.

*(Nội dung chi tiết sẽ được cập nhật trong các bài viết con).*
