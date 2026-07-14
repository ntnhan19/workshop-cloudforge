---
title : "Khởi tạo ECS Cluster"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.7.1. </b> "
---

Trước khi tiến hành cấu hình chi tiết cho các tác vụ xử lý (Task Definitions), cần chuẩn bị không gian lưu trữ mã nguồn đóng gói và thiết lập nhóm quản lý logic tài nguyên. Phân đoạn này sẽ tập trung vào việc cấu hình kho lưu trữ **Amazon ECR** và khởi tạo cụm **Amazon ECS Cluster**.

Do dự án áp dụng công nghệ **AWS Fargate**, cụm Cluster này sẽ hoạt động như một thực thể logic trống lúc ban đầu. Tài nguyên Compute thực tế sẽ chỉ được AWS cấp phát động ngay tại thời điểm ứng dụng được kích hoạt.

#### 1. Khởi tạo kho lưu trữ hình ảnh Docker (Amazon ECR)
Chúng ta tiến hành khởi tạo hai kho lưu trữ riêng biệt nhằm phân tách hoàn toàn mã nguồn của tầng giao tiếp Backend API và tầng xử lý tác vụ nặng AI Worker.

1. Truy cập dịch vụ **Amazon ECR** trên AWS Console → Chọn **Private registry** → **Repositories** → Bấm **Create repository**.
2. Tiến hành thiết lập các thông số tại màn hình khởi tạo:
   - **Repository name:** Nhập tên `cloudforge-backend`.
   - **Image tag mutability:** Chọn **Immutable** *(Best Practice nhằm ngăn chặn hành vi ghi đè lên các bản build cũ có cùng tag, đảm bảo tính nhất quán của lịch sử triển khai).*
   - **Encryption settings:** Giữ nguyên mặc định **AES-256**.
   - **Image scanning settings:** Tùy chọn này hiện đã được AWS đánh dấu là *deprecated* (lỗi thời) nhằm chuyển hướng sang kiến trúc quét lỗ hổng bảo mật tập trung bằng Amazon Inspector. Bỏ qua cấu hình này.

![ECR Config](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecr_config.png)
3. Cuộn xuống cuối trang và bấm **Create repository**.
4. Lặp lại toàn bộ chu trình trên để tạo thêm kho lưu trữ thứ hai với tên định danh `cloudforge-ai-worker`.

Sau khi hoàn tất, ghi nhận lại chuỗi đường dẫn **URI** của cả hai kho lưu trữ (ví dụ: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend`) để phục vụ cho công đoạn đóng gói và đẩy mã nguồn lên Cloud ở các bài học tiếp theo.

![ECR Repositories Created](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecr_repositories_created.png)

#### 2. Khởi tạo Amazon ECS Cluster
1. Truy cập dịch vụ **Amazon ECS** trên AWS Console → Chọn **Clusters** tại thanh điều hướng bên trái → Bấm **Create cluster**.
2. **Cluster configuration:** Nhập tên định danh tập trung `cloudforge-compute-cluster` vào trường Cluster name.
3. **Infrastructure:** Đảm bảo tùy chọn **AWS Fargate (serverless)** đang được chọn mặc định. Hệ thống hoàn toàn không sử dụng các thực thể EC2 instances nhằm loại bỏ công tác vận hành hệ điều hành.
4. **Monitoring (Tùy chọn nâng cao):** Chọn **Use Container Insights** để kích hoạt tính năng tự động đẩy các chỉ số chuyên sâu về hiệu năng phần cứng (vCPU, Memory Utilization) lên hệ thống giám sát Amazon CloudWatch.

![ECS Cluster Config](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecs_cluster_config.png)
5. Bấm **Create** và đợi hệ thống phê duyệt khởi tạo chu trình trong vài giây.

![ECS Cluster Created](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecs_cluster_created.png)

{{% notice tip %}}
**Fargate Cost Optimization:** Vì Amazon ECS Cluster khi chạy ở chế độ Fargate chỉ đóng vai trò là một nhóm quản lý logic, bạn **hoàn toàn không tốn bất kỳ chi phí nào** khi duy trì một Cluster trống. Chi phí hạ tầng Compute sẽ chỉ bắt đầu được tính toán dựa trên lượng vCPU và dung lượng RAM thực tế tiêu thụ theo giây khi các tác vụ (Tasks) chính thức chuyển sang trạng thái hoạt động (`RUNNING`).
{{% /notice %}}

***

**Bước tiếp theo:** Hạ tầng quản lý và kho lưu trữ hình ảnh Docker đã sẵn sàng. Hãy chuyển sang bài học [**5.7.2: Triển khai ECS Backend**](../5.7.2-deploy-ecs-backend/) để tiến hành tìm hiểu phương thức thiết lập bộ cân bằng tải Application Load Balancer (ALB) kết nối với mã nguồn API ứng dụng.
