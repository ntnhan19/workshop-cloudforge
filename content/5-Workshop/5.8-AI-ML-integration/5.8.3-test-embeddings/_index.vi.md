---
title : "Kiểm thử Embeddings"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.8.3. </b> "
---

Sau khi đã bóc tách thành công văn bản thô từ dịch vụ Amazon Transcribe, hệ thống của chúng ta tiếp tục đối mặt với một bài toán dữ liệu mới: Làm thế nào để máy tính có thể thực sự hiểu được "ngữ nghĩa" sâu xa của đoạn hội thoại, thay vì chỉ tìm kiếm khớp từ khóa (Keyword matching) một cách máy móc?

Giải pháp tối ưu nhất cho kiến trúc của Smart Media Analytics chính là sử dụng các **Mô hình nhúng (Embeddings Models)**. Trong phân đoạn này, chúng ta sẽ ghi nhận lại cách AI Worker giao tiếp với Amazon Bedrock để chuyển đổi ngôn ngữ tự nhiên thành các Vector toán học.

#### Cơ chế hoạt động của Text Embeddings

Trong thế giới của Trí tuệ Nhân tạo, "Embedding" là quá trình mã hóa một đoạn văn bản thành một chuỗi các số thực (Vector đa chiều). Các đoạn văn bản có ý nghĩa tương đồng sẽ tạo ra các Vector nằm gần nhau trong không gian toán học.

Ứng dụng AI Worker sử dụng mô hình **Amazon Titan Text Embeddings v2** (mô hình Serverless đã được tự động kích hoạt ở Bài 5.8.1) để thực hiện quy trình sau:

1. **Phân mảnh dữ liệu (Chunking):** Tệp văn bản JSON chứa toàn bộ lời thoại trả về từ Amazon Transcribe sẽ được chia nhỏ thành các đoạn ngắn (Chunks) hợp lý. Điều này giúp tối ưu hóa độ chính xác và đảm bảo không vượt quá giới hạn dung lượng đầu vào (Context window) của Bedrock.
2. **Gọi API tạo Vector:** AI Worker đóng gói các đoạn văn bản này và gửi tới Amazon Bedrock thông qua hàm API `InvokeModel`.
3. **Tiếp nhận mảng Vector:** Bedrock tính toán và phản hồi lại bằng một mảng dữ liệu (Array) chứa hàng ngàn giá trị số thực (ví dụ: `[0.012, -0.453, 0.887, ...]`). Đây chính là "chữ ký ngữ nghĩa" đặc trưng của đoạn văn bản đó.
4. **Lưu trữ tri thức:** Mảng Vector này sau đó sẽ được đưa vào Cơ sở dữ liệu Vector (PostgreSQL kết hợp `pgvector`) cùng với các siêu dữ liệu (Metadata) như ID của video và mốc thời gian.

![Embeddings Flow](/images/5-Workshop/5.8-AI-ML-integration/5.8.3-test-embeddings/embeddings_flow.png)

#### Yêu cầu quyền hạn hạ tầng (IAM Role Permissions)
Để tiến trình giao tiếp với Amazon Bedrock diễn ra trơn tru mà không bị chặn, **ECS Task Role** của AI Worker đã được cấp sẵn chính sách quyền hạn:
- `bedrock:InvokeModel` (Cho phép gọi và thực thi mô hình AI).

{{% notice tip %}}
**Lợi thế Kiến trúc:** Việc sử dụng API Embeddings của Bedrock trực tiếp từ môi trường Amazon ECS trong cùng một VPC cho phép dữ liệu nội bộ của hệ thống hoàn toàn không phải đi qua mạng Internet công cộng. Điều này đáp ứng các tiêu chuẩn khắt khe nhất về bảo mật và tuân thủ (Compliance) của doanh nghiệp.
{{% /notice %}}

***

**Bước tiếp theo:** Với toàn bộ dữ liệu Vector đã được định hình cấu trúc, luồng xử lý nền (Background Processing) của chúng ta coi như đã hoàn thiện. Ở chương tiếp theo (**Chương 5.9: API & Real-time**), chúng ta sẽ quay trở lại với tầng Backend để cấu hình các luồng giao tiếp thời gian thực, giúp thông báo cho người dùng ngay khi AI Worker phân tích xong!
