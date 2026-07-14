---
title : "Thiết lập Amazon CloudWatch"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.13.1. </b> "
---

Bài viết này hướng dẫn chi tiết cách cấu hình Amazon CloudWatch để thu thập log tập trung (Centralized Logging) từ hệ thống máy chủ container Amazon ECS Fargate và thiết lập các cảnh báo (Alarms) tự động khi tài nguyên có dấu hiệu quá tải.

Trong môi trường container hóa vô định hình như Amazon ECS Fargate, các container có thể được khởi tạo, scale-up hoặc hủy bỏ liên tục theo nhu cầu tải. Do đó, việc lưu trữ log cục bộ trực tiếp trên container là bất khả thi (dữ liệu log sẽ bị xóa vĩnh viễn khi container bị hủy). Giải pháp tiêu chuẩn là cấu hình **awslogs** log driver để bắt trọn luồng dữ liệu log hệ thống và đẩy thẳng về dịch vụ CloudWatch Logs.

#### Bước 1: Cấu hình thời gian lưu trữ CloudWatch Log Group
Nhóm dự án cần thiết lập thời gian lưu trữ (Retention) cho nhóm log mặc định của ứng dụng Backend, nhằm tối ưu hóa chi phí lưu trữ trên AWS, tránh để mặc định `Never expire` gây lãng phí ngân sách dự án.

1. Truy cập bảng điều khiển **Amazon CloudWatch**.
2. Tại menu bên trái, dưới mục **Logs**, chọn **Log groups**.
3. Hệ thống ECS đã tự động tạo sẵn một Log Group tên là `/ecs/cloudforge-backend-task`. Hãy bấm vào tên Log Group này.
4. Chọn tab **Actions** (hoặc nhấn trực tiếp vào mục Retention đang hiển thị `Never expire`), chọn **Edit retention setting**.
5. Chọn **14 days** (2 tuần) và bấm **Save changes**.

#### Bước 2: Cập nhật ECS Task Definition sử dụng awslogs driver
Để hệ thống máy chủ ECS biết được cần đẩy log về đâu, cấu hình log driver phải được tích hợp chặt chẽ bên trong bản thiết kế tác vụ (Task Definition).

Khối cấu hình `logConfiguration` được khai báo cụ thể trong định dạng JSON của Task Definition như sau:

```json
"logConfiguration": {
    "logDriver": "awslogs",
    "options": {
        "awslogs-group": "/ecs/cloudforge-backend",
        "awslogs-region": "ap-southeast-1",
        "awslogs-stream-prefix": "ecs"
    }
}
```

Sau khi ứng dụng được cập nhật thông qua luồng CI/CD, mã nguồn Backend chạy trong container sẽ tự động chuyển hướng toàn bộ dữ liệu log ra ngoài. Khi truy cập vào Log Group `/ecs/cloudforge-backend-task`, quản trị viên có thể mở các **Log streams** để theo dõi toàn bộ tiến trình khởi chạy ứng dụng theo thời gian thực (Real-time).

![CloudWatch Log Stream](/images/5-Workshop/5.13-Observability/5.13.1-cloudwatch-setup/5.13.1-log-stream.png)

#### Bước 3: Thiết lập CloudWatch Alarm cảnh báo CPU
Để chủ động xử lý các tình huống nghẽn mạng hoặc quá tải hệ thống trước khi dịch vụ rơi vào trạng thái sập (Crash), nhóm dự án thiết lập một hệ thống cảnh báo (Alarm) đo lường mức độ tiêu thụ CPU của ECS Service.

1. Tại menu bên trái của CloudWatch Console, dưới mục **Alarms**, chọn **All alarms**.
2. Bấm **Create alarm** -> Chọn **Select metric**.
3. Chọn danh mục **ECS** -> **ClusterName, ServiceName**.
4. Tìm kiếm và chọn đúng metric **CPUUtilization** ứng với service `cloudforge-backend-service` thuộc cụm cluster `cloudforge-compute-cluster`. Bấm **Select metric**.
5. Cấu hình điều kiện kích hoạt cảnh báo (Conditions):
   - **Statistic:** `Average`
   - **Period:** `1 minute` (Đo lường chi tiết theo từng phút).
   - **Threshold type:** `Static`
   - **Whenever CPUUtilization is:** Chọn `Greater/Equal` và điền giá trị `80` (Kích hoạt trạng thái cảnh báo khi CPU chạm ngưỡng hoặc vượt quá 80%).
6. Bấm **Next**. Tại bước Configure actions, chọn gửi thông báo (Send notification) đến một Amazon SNS Topic chuyên trách (Ví dụ: `DevOps-Alerts`) để tự động chuyển tiếp email cảnh báo về hộp thư của đội ngũ vận hành.
7. Đặt tên cho Alarm là `ECS-High-CPU-Alert` và nhấn **Create alarm**.

![CloudWatch CPU Alarm](/images/5-Workshop/5.13-Observability/5.13.1-cloudwatch-setup/5.13.1-cpu-alarm.png)

***

**Bước tiếp theo:** Hệ thống đã được trang bị năng lực giám sát nhật ký tập trung và chủ động phát hiện sự cố tài nguyên phần cứng. Ở bài viết tiếp theo ([**Bài 5.13.2: Tích hợp Truy vết X-Ray**](../5.13.2-xray-tracing/)), nhóm dự án sẽ tiếp tục giải quyết bài toán phức tạp hơn: cấu hình công cụ AWS X-Ray để theo dõi chuyên sâu đường đi của dữ liệu xuyên qua các lớp dịch vụ Serverless.
