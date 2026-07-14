---
title : "Thiết lập Xác thực OIDC"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.12.1. </b> "
---

Vấn đề bảo mật lớn nhất khi thiết lập CI/CD thông qua nền tảng bên thứ ba (như GitHub Actions) là việc cấp quyền truy cập vào môi trường AWS. Thay vì tạo một IAM User và lưu trữ vĩnh viễn khóa tĩnh `AWS_ACCESS_KEY_ID` cùng `AWS_SECRET_ACCESS_KEY` trên GitHub Secrets – một phương pháp tiềm ẩn rủi ro rò rỉ dữ liệu cực cao, nhóm dự án áp dụng chuẩn bảo mật **OpenID Connect (OIDC)**.

OIDC cho phép GitHub Actions trao đổi một JWT token ngắn hạn với AWS Security Token Service (STS) để nhận về thông tin đăng nhập tạm thời (Temporary Credentials), sau đó tự động hủy khi phiên chạy kết thúc.

#### Bước 1: Tạo OIDC Identity Provider trên AWS
Nhóm dự án thiết lập AWS IAM nhận diện GitHub là một nhà cung cấp định danh hợp lệ.

1. Truy cập **AWS IAM** Console, chọn **Identity providers** ở menu bên trái.
2. Nhấn **Add provider** và chọn loại **OpenID Connect**.
3. Điền thông tin cấu hình:
   - **Provider URL:** `https://token.actions.githubusercontent.com`
   - **Audience:** `sts.amazonaws.com`
4. Bấm **Get thumbprint** để hệ thống tự động xác thực chứng chỉ từ GitHub.
5. Bấm **Add provider** để hoàn tất.

![Thiết lập OIDC Provider](/images/5-Workshop/5.12-CICD/5.12.1-github-actions-oidc/5.12.1-oidc-provider.png)

#### Bước 2: Khởi tạo IAM Role cho GitHub Actions
Sau khi có Provider, hệ thống cần một IAM Role cụ thể để GitHub có thể mượn quyền (Assume Role).

1. Tại IAM Console, chọn **Roles** -> **Create role**.
2. Loại Trusted entity: Chọn **Web identity**.
3. Chọn Identity provider là `token.actions.githubusercontent.com` và Audience là `sts.amazonaws.com`. Nhấn **Next**.
4. Chọn các chính sách ủy quyền (Permission policies) cần thiết cho luồng CI/CD:
   - `AmazonEC2ContainerRegistryPowerUser` (Quyền đẩy Image lên ECR).
   - `AmazonECS_FullAccess` (Quyền cập nhật cấu hình dịch vụ trên ECS).
5. Đặt tên cho Role, ví dụ: `GitHubActionsDeployRole`.

#### Bước 3: Cấu hình Trust Policy giới hạn truy cập (Least Privilege)
Để ngăn chặn bất kỳ kho lưu trữ (Repository) nào khác trên GitHub mượn quyền của Role này, nhóm dự án thiết lập **Condition** cực kỳ nghiêm ngặt trong Trust Policy.

1. Mở Role vừa tạo, chuyển sang tab **Trust relationships**.
2. Chọn **Edit trust policy** và bổ sung điều kiện `StringLike` cho trường `sub` (Subject). Đảm bảo chỉ định đúng tên repository `smart_media_analytics_cloudforge`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::[ACCOUNT_ID]:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:ntnhan19/smart_media_analytics_cloudforge:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Với cấu hình này, IAM Role sẽ từ chối mọi yêu cầu cấp quyền trừ khi luồng CI/CD được thực thi chính xác từ nhánh `main` của repository dự án.

![IAM Trust Relationship](/images/5-Workshop/5.12-CICD/5.12.1-github-actions-oidc/5.12.1-trust-policy.png)

***

**Bước tiếp theo:** Nền tảng xác thực bảo mật OIDC đã được thiết lập vững chắc. Ở [bài 5.12.2](../5.12.2-ecr-push/), nhóm dự án sẽ tiến hành viết tệp workflow cho GitHub Actions để biên dịch mã nguồn và đẩy Docker Image lên Amazon ECR.
