---
title : "Tìm kiếm Ngữ nghĩa"
date : 2026-07-10
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Tổng quan Tìm kiếm Ngữ nghĩa (Semantic Search)

Thay vì tìm kiếm theo từ khóa truyền thống (Keyword-based search), Smart Media Analytics cho phép người dùng tìm kiếm video bằng ngôn ngữ tự nhiên. Ví dụ: *"Tìm các đoạn video có không khí vui vẻ nhắc đến lễ hội"*.

Để hiện thực hóa điều này, chúng ta sẽ tận dụng **RDS PostgreSQL** đã được cài đặt extension **pgvector** ở Chương 5.4.

Trọng tâm của chương:
1. **Vector Indexing:** Cách lưu trữ và lập chỉ mục (Index) các vector đặc trưng sinh ra từ Amazon Bedrock.
2. **Cosine Similarity:** Thuật toán tính toán khoảng cách vector để tìm ra các kết quả có độ tương đồng ngữ nghĩa cao nhất với câu truy vấn của người dùng.

*(Nội dung chi tiết sẽ được cập nhật trong các bài viết con).*
