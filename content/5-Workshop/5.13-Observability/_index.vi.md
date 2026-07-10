---
title : "Giám sát Hệ thống"
date : 2026-07-10
weight : 13
chapter : false
pre : " <b> 5.13. </b> "
---

### Tổng quan Kiến trúc Giám sát (Observability)

Sau khi hệ thống Smart Media Analytics đã được triển khai hoàn tất và có khả năng tự động hóa kiểm thử, đóng gói, phân phối thông qua luồng CI/CD Pipeline, vấn đề quan trọng cuối cùng là đảm bảo khả năng quan sát (**Observability**) và vận hành hệ thống ổn định trong môi trường Production.

Một kiến trúc phân tán (Microservices/Serverless) bao gồm nhiều thành phần giao tiếp bất đồng bộ với nhau (API Gateway, Lambda, ECS Fargate, Aurora PostgreSQL, S3) đòi hỏi một cơ chế giám sát tập trung. Điều này giúp quản trị viên có thể theo dõi sức khỏe hệ thống, thu thập nhật ký (Logs), đo lường hiệu suất (Metrics) và truy vết các luồng giao tiếp dữ liệu (Distributed Tracing).

Nhóm dự án quyết định triển khai bộ công cụ giám sát chính chủ của AWS bao gồm **Amazon CloudWatch** và **AWS X-Ray** để giải quyết bài toán này.

#### Tại sao lại sử dụng CloudWatch và X-Ray?
Kiến trúc Observability được xây dựng tuân thủ nghiêm ngặt các tiêu chí vận hành lõi:

- **Tập trung hóa Nhật ký và Chỉ số (Centralized Logging & Metrics):** Amazon CloudWatch đóng vai trò là điểm hội tụ duy nhất. Mọi dịch vụ từ tầng Compute (ECS, Lambda) đến tầng Database (Aurora) đều tự động đẩy log (`stdout`/`stderr`) và chỉ số hiệu năng về CloudWatch, giúp loại bỏ hoàn toàn việc phải cấu hình kết nối từ xa vào từng container để đọc nhật ký thủ công.
- **Truy vết phân tán (Distributed Tracing):** AWS X-Ray hỗ trợ vẽ lại toàn bộ bản đồ dịch vụ (**Service Map**) và theo dõi chính xác từng mili-giây thời gian xử lý của một API Request khi nó đi qua API Gateway, gọi vào xử lý tại Lambda, và truy xuất cơ sở dữ liệu. Tính năng này đóng vai trò quyết định trong việc phát hiện và cô lập các điểm nghẽn hiệu năng (Bottlenecks).

#### Lộ trình triển khai Giám sát:
1. **Thiết lập Amazon CloudWatch:** Cấu hình Log Groups cho cụm dịch vụ ECS Fargate, tạo Dashboards trực quan hóa Metrics (CPU, Memory, API 5XX Errors) và thiết lập Cảnh báo tự động (Alarms).
2. **Tích hợp AWS X-Ray:** Bổ sung thư viện X-Ray SDK vào mã nguồn backend ứng dụng và cấu hình ECS Task Role để cấp quyền đẩy dữ liệu Trace về hệ thống AWS X-Ray Daemon.

### Nội dung thực hành

- [Thiết lập Amazon CloudWatch](5.13.1-cloudwatch-setup/)
- [Tích hợp Truy vết X-Ray](5.13.2-xray-tracing/)
