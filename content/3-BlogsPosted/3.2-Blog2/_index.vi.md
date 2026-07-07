---
title: "Blog 2"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
# Tự động số hóa hồ sơ bệnh án với Amazon Bedrock Data Automation và AWS HealthLake

Hôm nay mình muốn chia sẻ một kiến trúc khá thú vị từ AWS dành cho lĩnh vực y tế: **tự động số hóa hồ sơ bệnh án bằng AI với Amazon Bedrock Data Automation kết hợp AWS HealthLake**.

Trong quá trình chuyển đổi số, rất nhiều bệnh viện vẫn đang lưu trữ bệnh án dưới dạng giấy hoặc PDF scan. Điều này khiến việc tìm kiếm thông tin, tổng hợp lịch sử điều trị hay chia sẻ dữ liệu giữa các hệ thống mất khá nhiều thời gian. Nếu nhập liệu thủ công thì vừa tốn nhân lực, vừa dễ xảy ra sai sót.

Bài toán đặt ra là: làm thế nào để tự động chuyển các tài liệu y tế không có cấu trúc thành dữ liệu số có thể tìm kiếm, phân tích và tích hợp với các hệ thống quản lý bệnh viện? AWS đưa ra một kiến trúc serverless kết hợp AI để giải quyết bài toán này.

### Kiến trúc hoạt động như thế nào?

![Kiến trúc AWS HealthLake và Bedrock](/images/3-BlogsPosted/3.2-Blog2/blog2.jpg)

Toàn bộ quy trình được kích hoạt tự động ngay khi hồ sơ được tải lên Amazon S3. Luồng xử lý gồm các bước:

1. Người dùng upload hồ sơ bệnh án (PDF hoặc hình ảnh) lên Amazon S3.
2. AWS Lambda nhận sự kiện upload và khởi động pipeline.
3. Amazon Bedrock Data Automation phân tích tài liệu, tự động trích xuất các thông tin quan trọng như:
   - Thông tin bệnh nhân
   - Chẩn đoán
   - Kết quả xét nghiệm
   - Đơn thuốc
   - Dấu hiệu sinh tồn
4. Dữ liệu sau đó được chuyển sang chuẩn **FHIR R4**.
5. **AWS HealthLake** lưu trữ và lập chỉ mục để các hệ thống y tế có thể truy vấn thông qua API tiêu chuẩn.

Điểm mình thấy hay là toàn bộ quy trình hoạt động theo mô hình event-driven, nghĩa là chỉ xử lý khi có tài liệu mới được tải lên, không cần duy trì máy chủ chạy liên tục.

### Amazon Bedrock Data Automation có gì đặc biệt?

Điểm nổi bật nhất là không cần tự xây dựng pipeline OCR và NLP riêng. Thông thường để xử lý hồ sơ bệnh án, doanh nghiệp sẽ phải kết hợp nhiều bước:

- OCR
- Layout Analysis
- Entity Extraction
- Chuẩn hóa dữ liệu
- Mapping sang FHIR

Giờ đây, Amazon Bedrock Data Automation giúp tự động hóa phần lớn quy trình này, giảm đáng kể công sức phát triển và bảo trì. Đây là hướng tiếp cận rất phù hợp với các tổ chức muốn nhanh chóng triển khai AI mà không cần xây dựng mô hình từ đầu.

### Use Case 1: Số hóa bệnh án cũ

Một bệnh viện có thể có hàng triệu hồ sơ bệnh án giấy được lưu trữ nhiều năm. Thay vì phải nhập liệu thủ công, toàn bộ tài liệu có thể được upload lên Amazon S3. Pipeline sẽ tự động:

- Trích xuất dữ liệu
- Chuẩn hóa
- Lưu vào AWS HealthLake

Sau đó bác sĩ chỉ cần tìm kiếm theo tên bệnh nhân, mã bệnh hoặc lịch sử điều trị thay vì mở từng tập hồ sơ giấy.

### Use Case 2: Hỗ trợ AI phân tích hồ sơ bệnh nhân

Khi dữ liệu đã được chuẩn hóa theo FHIR, việc xây dựng các ứng dụng AI sẽ trở nên dễ dàng hơn rất nhiều. Ví dụ:

- AI tóm tắt lịch sử điều trị
- Chatbot hỗ trợ bác sĩ tra cứu bệnh án
- Phân tích xu hướng điều trị
- Hỗ trợ nghiên cứu lâm sàng
- Kết nối dữ liệu giữa nhiều bệnh viện

Đây cũng là nền tảng quan trọng để phát triển các hệ thống Generative AI trong lĩnh vực y tế.

### Vì sao AWS sử dụng HealthLake?

Điểm mình đánh giá cao là AWS không chỉ tập trung vào việc "đọc được tài liệu", mà còn chuẩn hóa dữ liệu theo **FHIR (Fast Healthcare Interoperability Resources)** — tiêu chuẩn trao đổi dữ liệu y tế được sử dụng rộng rãi trên thế giới.

Điều này giúp dữ liệu sau khi xử lý không bị "khóa" trong một ứng dụng riêng, mà có thể tích hợp với nhiều hệ thống HIS, EMR hoặc các ứng dụng AI khác.

### Một vài lưu ý khi triển khai

AWS cũng lưu ý rằng kiến trúc trong bài viết được xây dựng để minh họa. Nếu triển khai trên dữ liệu bệnh nhân thật, cần bổ sung thêm các yêu cầu về bảo mật như:

- Mã hóa dữ liệu khi lưu trữ và truyền tải.
- Quản lý quyền truy cập bằng IAM theo nguyên tắc quyền tối thiểu.
- Theo dõi hoạt động bằng CloudTrail.
- Triển khai trong môi trường đáp ứng các yêu cầu tuân thủ như HIPAA (nếu áp dụng).

### Tổng kết

Theo mình, điều đáng giá nhất của kiến trúc này không nằm ở AI hay OCR riêng lẻ, mà là khả năng tự động chuyển đổi tài liệu y tế không có cấu trúc thành dữ liệu chuẩn hóa, sẵn sàng cho các hệ thống và ứng dụng AI sử dụng. 

Đây là một ví dụ điển hình cho cách AWS kết hợp **Serverless + AI + Chuẩn dữ liệu ngành** để giải quyết một bài toán rất thực tế trong lĩnh vực chăm sóc sức khỏe.

---

### Nguồn tham khảo & Hướng dẫn triển khai

- **Tham khảo từ AWS Architecture Blog:** [Automate medical record digitization with Amazon Bedrock Data Automation and AWS HealthLake](https://aws.amazon.com/blogs/architecture/automate-medical-record-digitization-with-amazon-bedrock-data-automation-and-aws-healthlake/)