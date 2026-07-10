---
title : "CI/CD & Tự động hóa"
date : 2026-07-10
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

### Tổng quan CI/CD

Việc triển khai thủ công không chỉ tốn thời gian mà còn dễ gây ra lỗi con người. Để đạt chuẩn DevOps, chúng ta cần thiết lập một luồng Tích hợp liên tục và Triển khai liên tục (CI/CD).

Trong chương này, chúng ta sẽ sử dụng **GitHub Actions** (hoặc AWS CodePipeline) để:
1. Tự động chạy Unit Test mỗi khi có mã nguồn mới được push lên.
2. Tự động Build Docker Image và đẩy lên Amazon ECR.
3. Tự động cập nhật Task Definition trên ECS để tiến hành Rolling Update không gián đoạn dịch vụ.

*(Nội dung chi tiết sẽ được cập nhật trong các bài viết con).*
