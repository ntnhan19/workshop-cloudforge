---
title : "Tự động đẩy Image lên ECR"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.12.2. </b> "
---

Sau khi đã thiết lập thành công luồng xác thực bảo mật OIDC ở bài 5.12.1, bước tiếp theo trong quy trình CI/CD là tự động hóa khâu Build (Đóng gói mã nguồn) và Push (Đẩy Image) lên Amazon Elastic Container Registry (ECR) thông qua GitHub Actions.

Amazon ECR đóng vai trò là một kho lưu trữ an toàn (Private Registry) chứa các phiên bản Docker Image của ứng dụng Backend, sẵn sàng để triển khai lên hạ tầng máy chủ.

#### Bước 1: Khởi tạo Repository trên Amazon ECR
Trước khi GitHub Actions có thể đẩy Image lên, cần tạo một kho lưu trữ trống trên AWS.

1. Truy cập **Amazon ECR** Console.
2. Tại menu bên trái, chọn **Repositories** và nhấn **Create repository**.
3. Cấu hình các thông số cơ bản:
   - **Visibility settings:** Chọn `Private`.
   - **Repository name:** Đặt tên kho lưu trữ là `cloudforge-backend`.
4. Nhấn **Create repository** ở cuối trang để hoàn tất.

Lưu ý và sao chép **URI** của Repository vừa tạo (Ví dụ: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend`).

![Create ECR Repository](/images/5-Workshop/5.12-CICD/5.12.2-ecr-push/5.12.2-create-ecr.png)

#### Bước 2: Thiết lập Workflow GitHub Actions
Nhóm dự án tạo một tệp định nghĩa Workflow tại thư mục gốc của dự án trên VS Code: `.github/workflows/deploy.yml`.

Tệp cấu hình này định nghĩa các bước (Steps) sẽ được thực thi mỗi khi có mã nguồn mới được đẩy lên nhánh `main`.

```yaml
name: Deploy Backend to ECR

on:
  push:
    branches:
      - main

# Cấp quyền bắt buộc để GitHub Actions có thể nhận OIDC Token
permissions:
  id-token: write 
  contents: read 

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::236320489525:role/GitHubActionsDeployRole
          aws-region: ap-southeast-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push Docker image to ECR
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: cloudforge-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Di chuyển vào thư mục backend vì dự án là monorepo
          cd backend
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
```

**Giải thích luồng thực thi:**
- **permissions (`id-token: write`):** Đây là thiết lập quan trọng nhất. Thiếu dòng này, GitHub sẽ không thể yêu cầu JWT token từ STS, dẫn đến lỗi xác thực OIDC.
- **configure-aws-credentials:** Step mượn quyền `GitHubActionsDeployRole` (đã tạo ở 5.12.1) để lấy Temporary Credentials.
- **amazon-ecr-login:** Step tự động đăng nhập vào kho ECR nội bộ.
- **Build & Push:** Đóng gói Docker Image dựa trên `Dockerfile` trong thư mục `backend`, sau đó dán nhãn (tag) bằng mã SHA của commit (`github.sha`) để dễ dàng quản lý phiên bản (Versioning) và đẩy lên ECR.

#### Bước 3: Kiểm tra quá trình tự động Build & Push
Sau khi tạo tệp `.github/workflows/deploy.yml` trên VS Code, lưu lại và thực hiện lệnh `git push` lên nhánh `main`. Tiếp theo, điều hướng tới thẻ **Actions** trên giao diện kho lưu trữ GitHub của hệ thống.

Sẽ thấy một tiến trình mang tên `Deploy Backend to ECR` đang được chạy tự động. Nếu toàn bộ cấu hình OIDC và mã nguồn hợp lệ, quy trình sẽ kết thúc với một dấu tích xanh (Success).

Truy cập lại vào kho lưu trữ `cloudforge-backend` trên Amazon ECR Console, sẽ thấy một Docker Image mới tinh vừa được đẩy lên, gắn nhãn (tag) bằng chuỗi mã SHA của commit.

![GitHub Actions Success](/images/5-Workshop/5.12-CICD/5.12.2-ecr-push/5.12.2-actions-success.png)

***

**Bước tiếp theo:** Docker Image đã được chuẩn bị sẵn sàng trên lưu trữ riêng tư. Trong [bài 5.12.3](../5.12.3-deploy-pipeline/) cuối cùng, nhóm dự án sẽ nối thêm một Step vào Workflow hiện tại để kích hoạt lệnh triển khai Image mới lên Amazon ECS theo chiến lược Rolling Update (Không gián đoạn).
