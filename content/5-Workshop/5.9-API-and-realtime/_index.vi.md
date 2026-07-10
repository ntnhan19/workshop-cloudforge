---
title : "API & Real-time"
date : 2026-07-10
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### Tổng quan API & Real-time

Với kiến trúc xử lý dữ liệu bất đồng bộ (Asynchronous), làm thế nào để Frontend biết được khi nào một video đã được AI xử lý xong? Việc yêu cầu Frontend liên tục gọi API (Polling) sẽ làm quá tải hệ thống.

Chương này sẽ giải quyết bài toán giao tiếp thời gian thực (Real-time) và quản lý API:
1. **Amazon API Gateway (WebSocket):** Thiết lập kết nối hai chiều dai dẳng. Khi AI Worker xử lý xong, nó có thể chủ động đẩy (Push) thông báo trực tiếp về trình duyệt của người dùng.
2. **API Gateway (REST API):** Bọc các endpoint của Application Load Balancer để cung cấp khả năng điều tiết lưu lượng (Throttling) và quản lý phiên bản API.

*(Nội dung chi tiết sẽ được cập nhật trong các bài viết con).*
