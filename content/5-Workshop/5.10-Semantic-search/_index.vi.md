---
title : "Tìm kiếm Ngữ nghĩa"
date : 2026-07-10
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Tổng quan Kiến trúc Tìm kiếm Ngữ nghĩa (Semantic Search)

Chức năng cốt lõi tạo nên giá trị của hệ thống Smart Media Analytics là khả năng truy xuất dữ liệu đa phương tiện dựa trên ý nghĩa ngữ cảnh (Contextual meaning), thay vì phụ thuộc vào phương pháp khớp từ khóa (Keyword matching) truyền thống. Khách hàng có thể truy vấn bằng ngôn ngữ tự nhiên, ví dụ: *"Tìm các đoạn hội thoại thể hiện sự lo lắng về an ninh mạng"*.

Để hiện thực hóa năng lực này, nhóm dự án thiết lập kiến trúc tích hợp giữa Mô hình học máy (Amazon Bedrock) và Cơ sở dữ liệu Vector (Amazon RDS PostgreSQL kết hợp extension `pgvector`).

#### Nguyên lý vận hành của Vector Search
Kiến trúc Semantic Search của dự án hoạt động dựa trên cơ chế đối chiếu không gian toán học:
- **Biểu diễn Dữ liệu:** Mọi đoạn văn bản (từ Video Transcript) đều đã được mã hóa thành các Vector đa chiều ở giai đoạn Ingestion (Chương 5.8.3) và lưu trữ trong PostgreSQL.
- **Biểu diễn Truy vấn:** Câu lệnh tìm kiếm của người dùng cũng được mã hóa thành một Vector đa chiều bằng chính mô hình nhúng (Embeddings Model) đã sử dụng ở khâu Ingestion.
- **Đo lường Khoảng cách:** Hệ thống sử dụng thuật toán **Cosine Similarity** (Độ tương đồng Cosin) được hỗ trợ bởi `pgvector` để tính toán khoảng cách giữa Vector truy vấn và toàn bộ các Vector trong cơ sở dữ liệu. Các Vector có khoảng cách hình học gần nhất sẽ được trả về dưới dạng kết quả tương đồng ngữ nghĩa cao nhất.

#### Lộ trình triển khai Semantic Search:
1. **Tạo Vector Truy vấn (Query Embeddings):** Xây dựng luồng API xử lý yêu cầu tìm kiếm, gọi đến Amazon Bedrock để chuyển đổi câu truy vấn (Query) của người dùng thành cấu trúc Vector.
2. **Tìm kiếm Vector với pgvector:** Thiết lập các lệnh truy vấn cơ sở dữ liệu nâng cao (Advanced SQL Queries) để thực thi tính toán khoảng cách toán học và trả về dữ liệu kết quả theo thời gian thực.

Với kiến trúc này, dự án giải quyết triệt để bài toán thấu hiểu ý định tìm kiếm (Search Intent) của người dùng cuối.

### Nội dung thực hành

- [Tạo Vector Truy vấn](5.10.1-generate-query-vector/)
- [Thực thi Vector Search với pgvector](5.10.2-vector-search-pgvector/)
