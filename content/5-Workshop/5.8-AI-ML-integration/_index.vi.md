---
title : "Tích hợp AI/ML"
date : 2026-07-10
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

### Tổng quan Tích hợp Trí tuệ Nhân tạo (AI/ML Integration)

Sức mạnh cốt lõi và điểm nhấn kiến trúc của hệ thống Smart Media Analytics nằm ở khả năng phân tích nội dung đa phương tiện tự động bằng Trí tuệ Nhân tạo. Trong chương này, chúng sẽ tiến hành kết nối ứng dụng AI Worker (đã triển khai trên hạ tầng ECS Fargate) với hệ sinh thái Generative AI và Machine Learning mạnh mẽ của AWS.

Thay vì tự xây dựng, tối ưu và huấn luyện các mô hình học máy phức tạp vốn đòi hỏi lượng dữ liệu lớn cùng chi phí tính toán khổng lồ, dự án lựa chọn giải pháp tích hợp các dịch vụ AI được quản lý toàn diện (Managed Services) thông qua các lời gọi API. Hướng đi này giúp tối ưu hóa tối đa thời gian triển khai hạ tầng và chi phí vận hành thực tế.

#### Lợi ích cốt lõi của việc sử dụng Managed AI Services:
- **Tích hợp liền mạch:** Các dịch vụ AI trong hệ sinh thái AWS được thiết kế để giao tiếp tự nhiên với nhau và với các thành phần hạ tầng cốt lõi (Amazon S3, Amazon ECS) trong cùng một mạng lưới bảo mật nội bộ.
- **Tiếp cận mô hình ngôn ngữ hàng đầu:** Dịch vụ Amazon Bedrock cung cấp quyền truy cập tập trung vào các Mô hình Nền tảng (Foundation Models - FMs) tiên tiến nhất thế giới hiện nay như Anthropic Claude và Amazon Titan.
- **Bảo mật dữ liệu tuyệt đối:** Khác với việc gọi API ra các dịch vụ công cộng bên ngoài (như ChatGPT), toàn bộ dữ liệu âm thanh, hình ảnh và văn bản của hệ thống khi gửi lên Amazon Bedrock hay Amazon Transcribe đều được cam kết mã hóa toàn vẹn và tuyệt đối không bị sử dụng để huấn luyện lại các mô hình gốc.

#### Lộ trình tích hợp phân lớp AI/ML:
1. **Amazon Bedrock:** Cấu hình kích hoạt và cấp quyền truy cập các mô hình Large Language Models (LLMs) đại diện là Anthropic Claude 3 nhằm phục vụ tác vụ phân tích cảm xúc và tóm tắt nội dung Media.
2. **Amazon Transcribe:** Tích hợp dịch vụ chuyển đổi giọng nói thành văn bản (Speech-to-Text) bất đồng bộ với độ chính xác cao, đóng vai trò trích xuất dữ liệu hội thoại thô từ các tệp tin âm thanh và video.
3. **Mô hình nhúng (Embeddings):** Sử dụng dòng mô hình Amazon Titan Text Embeddings để chuyển đổi văn bản thành các Vector toán học, phục vụ trực tiếp cho tính năng tìm kiếm ngữ nghĩa (Semantic Search) chuyên sâu.

Hoàn thành chương này, AI Worker của bạn sẽ sở hữu "trí thông minh" thực sự, có khả năng bóc tách, hiểu sâu và chuyển đổi các tệp đa phương tiện phi cấu trúc thành các khối dữ liệu tri thức có cấu trúc rõ ràng.

### Nội dung thực hành

- [Cấp quyền Amazon Bedrock](5.8.1-enable-bedrock-access/)
- [Tích hợp Amazon Transcribe](5.8.2-setup-transcribe/)
- [Kiểm thử Embeddings](5.8.3-test-embeddings/)
