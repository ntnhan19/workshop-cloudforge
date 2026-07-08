---
title : "Tổng quan Kiến trúc"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Kiến trúc hệ thống Smart Media Analytics (Team CloudForge)

Dự án **Smart Media Analytics** là một giải pháp phân tích video toàn diện, kết hợp sức mạnh của AI và kiến trúc Serverless/Microservices trên AWS. Hệ thống được thiết kế để xử lý video dung lượng lớn, trích xuất dữ liệu thông minh (nhận diện khung cảnh, giọng nói, vật thể) và cho phép người dùng tìm kiếm theo ngữ nghĩa (Semantic Search).

Dưới đây là sơ đồ kiến trúc tổng thể của toàn bộ hệ thống khi được triển khai trên Cloud:

![Tổng quan kiến trúc nền tảng](/images/2-Proposal/platform_architecture.png)

#### Các luồng xử lý chính (Key Workflows)

1. **Luồng Ingestion (Tải lên & Tiền xử lý):**
   - Người dùng tải video lên thông qua giao diện Web (React).
   - Video gốc được lưu trữ an toàn tại **Amazon S3**.
   - Một bản ghi Job được tạo trong **RDS PostgreSQL**, và một thông báo được đẩy qua **Redis Pub/Sub** để Frontend cập nhật trạng thái theo thời gian thực (Real-time).

2. **Luồng AI Pipeline (Phân tích & Trích xuất):**
   - Nền tảng tính toán chính là **Amazon ECS (AWS Fargate)** hoạt động trong vùng an toàn (Private Subnet).
   - ECS sẽ kéo video từ S3 (thông qua S3 Gateway Endpoint để tối ưu băng thông mạng nội bộ).
   - **FFmpeg & PySceneDetect** sẽ cắt video thành các phân cảnh (scenes) và tách ảnh đại diện (keyframes).
   - **Whisper AI** xử lý âm thanh để tạo phụ đề (Transcript/Captions).
   - Các keyframes được gửi tới mô hình **Vision AI (Qwen-VL)** để nhận diện và mô tả cảnh vật, sau đó được chuyển thành vector (Embeddings) để phục vụ cho tính năng tìm kiếm.

3. **Luồng Serving & Semantic Search (Truy vấn & Phân phối):**
   - API Backend được xây dựng bằng **FastAPI** và bảo vệ bởi Rate Limiter.
   - Toàn bộ vector nhúng và metadata được lưu tại **PostgreSQL (với extension pgvector)**. Khi người dùng nhập từ khóa tìm kiếm (ví dụ: "Người đàn ông đạp xe"), hệ thống sẽ thực hiện tìm kiếm vector độ tương đồng cosine (Cosine Similarity) ngay trong Database.
   - Ứng dụng Backend được host trực tiếp trên **AWS App Runner**, sử dụng VPC Connector để thọc sâu vào truy vấn Database nằm trong vùng an toàn (Private Subnet).