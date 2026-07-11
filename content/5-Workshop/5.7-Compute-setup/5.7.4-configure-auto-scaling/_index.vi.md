---
title : "Thiết lập Auto Scaling"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.7.4. </b> "
---

Một trong những ưu điểm lớn nhất của kiến trúc đám mây và AWS Fargate là khả năng mở rộng tự động (Auto Scaling). Hệ thống có thể tự động tăng số lượng Container khi lưu lượng truy cập hoặc khối lượng công việc tăng cao đột biến, và tự động thu hẹp lại (Scale-in) khi hệ thống nhàn rỗi nhằm tối ưu hóa chi phí.

Trong phân đoạn này, chúng ta sẽ thiết lập chính sách mở rộng tự động (Auto Scaling Policies) cho cả hai tầng dịch vụ: **Backend API** và **AI Worker**.

#### 1. Cấu hình Auto Scaling cho Backend API
Lưu lượng truy cập người dùng thường biến động theo thời gian thực. Cấu hình Auto Scaling dựa trên mức sử dụng CPU (CPU Utilization) là phương pháp phổ biến và hiệu quả nhất cho Backend.

1. Truy cập dịch vụ **Amazon ECS**, chọn Cluster `cloudforge-compute-cluster`.
2. Tại tab **Services**, tích chọn dịch vụ `cloudforge-backend-service` và bấm nút **Update**.
3. Cuộn xuống phần **Service auto scaling** và tích chọn **Use service auto scaling**.
4. Cấu hình **Task capacity** với Minimum number of tasks là `2` và Maximum number of tasks là `10`.
5. Tại mục **Scaling policies**, chọn Policy type là **Target tracking** và đặt Policy name là `backend-cpu-scaling-policy`.
6. Thiết lập **ECS service metric** là **ECSServiceAverageCPUUtilization** với Target value là `70` (Khi CPU trung bình vượt 70%, hệ thống tự động tạo thêm Container).
7. Điền Scale-out cooldown period là `60` và Scale-in cooldown period là `300`.

![Backend Scaling Config](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/backend_scaling_config.png)

8. Cuộn xuống cuối trang và bấm **Update**.

![Backend Auto Scaling](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/backend_auto_scaling.png)

#### 2. Cấu hình Auto Scaling cho AI Worker
Lưu lượng công việc của AI Worker phụ thuộc hoàn toàn vào số lượng thông điệp tồn đọng trong hàng đợi. Trong phạm vi Workshop này, chúng ta sẽ sử dụng CPU Utilization để làm quen với cơ chế Auto Scaling cơ bản.

1. Quay lại danh sách **Services** của Cluster `cloudforge-compute-cluster`.
2. Tích chọn dịch vụ `cloudforge-ai-worker-service` và bấm nút **Update**.
3. Tích chọn **Use service auto scaling** tại mục Service auto scaling.
4. Cấu hình **Task capacity** với Minimum number of tasks là `1` và Maximum number of tasks là `5`.
5. Tại mục **Scaling policies**, chọn Policy type là **Target tracking** và đặt Policy name là `worker-cpu-scaling-policy`.
6. Thiết lập **ECS service metric** là **ECSServiceAverageCPUUtilization** với Target value là `75`.
7. Điền Scale-out cooldown period là `60` và Scale-in cooldown period là `300`.

![Worker Scaling Config](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/worker_scaling_config.png)

8. Cuộn xuống cuối trang và bấm **Update**.

![Worker Auto Scaling](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/worker_auto_scaling.png)

{{% notice tip %}}
**Best Practice cho Background Worker:** Trong các hệ thống Production quy mô lớn, việc mở rộng AI Worker nên dựa vào chỉ số **`ApproximateNumberOfMessagesVisible`** của Amazon SQS thông qua CloudWatch Custom Metric. Điều này đảm bảo hệ thống phản ứng chính xác với khối lượng công việc tồn đọng (Queue Depth) thay vì chỉ số phần cứng đơn thuần.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống hạ tầng Compute đã hoàn thiện với khả năng tự động co giãn thông minh. Bạn có thể tự hào rằng ứng dụng CloudForge giờ đây có khả năng chịu tải cao với kiến trúc Serverless đích thực. Hãy chuyển sang phần tiếp theo để tổng kết kiến trúc và trải nghiệm thực tế!
