---
title : "Triển khai liên tục lên ECS"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.12.3. </b> "
---

Sau khi Docker Image mới được đóng gói và đẩy thành công lên Amazon ECR ở bài 5.12.2, bước cuối cùng của quy trình CI/CD là tự động cập nhật hệ thống máy chủ Amazon ECS để chạy phiên bản mã nguồn mới nhất.

Để đảm bảo hệ thống không bị gián đoạn (Zero-downtime) trong quá trình cập nhật, nhóm dự án thiết kế luồng triển khai dựa trên cơ chế **Rolling Update** mặc định của Amazon ECS Fargate.

#### Bước 1: Cập nhật luồng Workflow trên GitHub Actions
Mở lại tệp `.github/workflows/deploy.yml` đã tạo ở bài trước trên VS Code và bổ sung thêm các bước (Steps) tải cấu hình Task Definition hiện tại, cập nhật Image ID mới, và ra lệnh triển khai trực tiếp lên ECS Cluster.

Nội dung tệp `deploy.yml` hoàn chỉnh sau khi tích hợp luồng CD:

```yaml
name: Deploy Backend to ECR and ECS

on:
  push:
    branches:
      - main

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
        id: build-image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: cloudforge-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          cd backend
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
          echo "image=$REGISTRY/$REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Download current ECS task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition cloudforge-backend-task \
            --query taskDefinition > task-definition.json

      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: backend-container
          image: ${{ steps.build-image.outputs.image }}

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: cloudforge-backend-service
          cluster: cloudforge-compute-cluster
          wait-for-service-stability: true
```

**Giải thích luồng triển khai bổ sung:**
- **Download task definition:** Sử dụng AWS CLI (đã được xác thực qua OIDC ngắn hạn) để tải xuống cấu hình Task Definition đang hoạt động (Active) dưới dạng tệp `task-definition.json`, giúp Pipeline luôn đồng bộ với cấu hình thực tế trên hạ tầng.
- **render-task-definition:** Tìm kiếm container có tên `cloudforge-backend-container` bên trong tệp định nghĩa JSON vừa tải và thay thế trường `image` bằng đường dẫn URI của Docker Image chứa mã SHA commit vừa được đẩy lên ECR.
- **deploy-task-definition:** Đăng ký một phiên bản mới (New Revision) của Task Definition lên AWS và cập nhật ECS Service để kích hoạt tiến trình Rolling Update. Tham số `wait-for-service-stability: true` buộc GitHub Actions giữ kết nối chờ cho đến khi các container mới vượt qua Health Check ổn định thì mới đóng phiên chạy thành công.

#### Bước 2: Quan sát tiến trình Rolling Update
Lưu lại tệp cấu hình trên máy cục bộ, thực hiện commit và `git push` lên nhánh `main`.

1. **Trên GitHub Actions:** Theo dõi tiến trình CI/CD. Giai đoạn *Deploy Amazon ECS task definition* sẽ duy trì trạng thái chạy khoảng 3 đến 5 phút để đợi ECS container khởi tạo xong.
2. **Trên AWS ECS Console:** Chuyển sang giao diện quản lý cụm máy chủ, truy cập vào dịch vụ `cloudforge-backend-service` và chọn thẻ **Deployments**.

Tại đây, cơ chế Rolling Update sẽ diễn ra một cách tự động và trực quan:
- ECS tự động tính toán và khởi tạo (Provision) các tác vụ Task mới song song với các Task cũ dựa trên Image mới được chỉ định.
- Các Task mới tự động đăng ký vào Target Group của Application Load Balancer (ALB) và thực hiện quá trình kiểm tra sức khỏe đầu vào.
- Khi các Task mới đạt trạng thái RUNNING và HEALTHY, ECS sẽ từ từ điều hướng toàn bộ lưu lượng người dùng sang phiên bản mới, đồng thời rút và hủy (Deregister) các Task cũ, đảm bảo hệ thống duy trì hoạt động liên tục 100% không xuất hiện lỗi gián đoạn.

![ECS Rolling Update](/images/5-Workshop/5.12-CICD/5.12.3-deploy-pipeline/5.12.3-ecs-deploy.png)

***

**Bước tiếp theo:** Hệ thống đã tự vận hành hoàn hảo, nhưng làm sao để quản trị viên theo dõi được sức khỏe hệ thống và truy vết lỗi nếu có sự cố xảy ra? Ở chương cuối cùng ([**Chương 5.13: Observability**](../../5.13-Observability/)), tiến hành khám phá cách giám sát toàn diện ứng dụng với AWS CloudWatch và AWS X-Ray.
