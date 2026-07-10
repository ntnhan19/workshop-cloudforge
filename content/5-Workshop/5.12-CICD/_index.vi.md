---
title : "CI/CD Pipeline"
date : 2026-07-10
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

### Tổng quan Kiến trúc CI/CD Pipeline

Sau khi toàn bộ hệ thống Smart Media Analytics từ Database, Backend, AI/ML đến Frontend đã đi vào hoạt động ổn định, nhóm dự án tiến hành thiết lập quy trình Tích hợp liên tục và Triển khai liên tục (CI/CD) nhằm tối ưu hóa vòng đời phát triển phần mềm (SDLC) theo tiêu chuẩn DevOps hiện đại.

Việc triển khai thủ công mã nguồn từ máy trạm (Local) lên hệ thống AWS không chỉ gây ra nút thắt cổ chai về mặt thời gian mà còn tiềm ẩn rủi ro do thao tác lỗi của con người (Human error). Do đó, nhóm dự án quyết định xây dựng một luồng CI/CD hoàn toàn tự động sử dụng **GitHub Actions**.

#### Tại sao lại sử dụng GitHub Actions với OIDC?
Hệ thống CI/CD được thiết kế tuân thủ nghiêm ngặt các tiêu chuẩn bảo mật của AWS:

- **Xác thực phi phím (Keyless Authentication):** Thay vì lưu trữ tĩnh Access Key IAM trên GitHub Secrets vốn mang rủi ro bảo mật lớn, GitHub Actions xác thực danh tính với AWS một cách an toàn thông qua **OpenID Connect (OIDC)**, loại bỏ hoàn toàn việc sử dụng khóa tĩnh.
- **Quyền hạn tối thiểu (Least Privilege):** IAM Role cấp cho GitHub Actions bị giới hạn chặt chẽ chỉ được thực thi các tác vụ Push ECR và cập nhật ECS, đồng thời chỉ được kích hoạt từ đúng nhánh `main` của Repository dự án.

#### Lộ trình triển khai CI/CD:
1. **Thiết lập Xác thực OIDC:** Cấu hình Identity Provider trên AWS IAM và tạo Role chuyên biệt cho GitHub Actions.
2. **Cấu hình Đẩy Docker Image:** Viết quy trình GitHub Actions tự động Build mã nguồn và Push Image lên kho lưu trữ riêng tư Amazon ECR.
3. **Tự động Triển khai lên ECS:** Cập nhật bản ghi cấu hình (Task Definition) trên Amazon ECS và thực hiện Rolling Update không làm gián đoạn dịch vụ (Zero-downtime).

### Nội dung thực hành

- [Thiết lập Xác thực OIDC](5.12.1-github-actions-oidc/)
- [Tự động đẩy Image lên ECR](5.12.2-ecr-push/)
- [Triển khai liên tục lên ECS](5.12.3-deploy-pipeline/)
