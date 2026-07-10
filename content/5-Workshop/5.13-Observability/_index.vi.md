---
title : "Giám sát Hệ thống"
date : 2026-07-10
weight : 13
chapter : false
pre : " <b> 5.13. </b> "
---

### Tổng quan Giám sát (Observability)

Một hệ thống Production không thể thiếu "đôi mắt" để theo dõi "sức khỏe" và gỡ lỗi khi có sự cố. AWS cung cấp bộ công cụ mạnh mẽ để đảm bảo khả năng quan sát (Observability) toàn diện.

Trọng tâm của chương bao gồm:
1. **Amazon CloudWatch:** Quản lý Log tập trung từ ECS Container, Lambda, và RDS. Đặt cảnh báo (Alarms) khi CPU tăng cao hoặc có lỗi 500.
2. **AWS X-Ray:** Theo dõi dấu vết phân tương (Distributed Tracing) để biết chính xác một Request từ lúc vào API Gateway đến lúc truy vấn Database mất bao nhiêu mili-giây ở từng giai đoạn.

*(Nội dung chi tiết sẽ được cập nhật trong các bài viết con).*
