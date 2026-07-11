---
title : "Tích hợp Amazon Transcribe"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.8.2. </b> "
---

Để hiểu và bóc tách được nội dung của một tệp Video hay Audio, bước đầu tiên và quan trọng nhất trong đường ống xử lý (Pipeline) là phải trích xuất được luồng hội thoại thành văn bản thô. Trong kiến trúc tổng thể của Smart Media Analytics, chúng ta sử dụng dịch vụ **Amazon Transcribe** - giải pháp nhận dạng tiếng nói tự động (Automatic Speech Recognition - ASR) dựa trên công nghệ học máy chuyên sâu của AWS.

Không giống như Amazon Bedrock yêu cầu cơ chế tự động kích hoạt hoặc khai báo Use Case từ phía người dùng, Amazon Transcribe được AWS mở khóa sẵn mặc định trên mọi tài khoản. Trong phân đoạn này, chúng ta sẽ tìm hiểu sâu về cơ chế tương tác tự động giữa AI Worker và dịch vụ xử lý âm thanh này.

#### Luồng xử lý âm thanh tự động (Audio Processing Flow)

Ứng dụng AI Worker được lập trình để thực hiện một chu trình tự động hóa hoàn toàn bằng Python (thông qua thư viện SDK `boto3`) để tương tác với Amazon Transcribe:

1. **Trích xuất dữ liệu âm thanh:** Khi AI Worker tiêu thụ một thông điệp (Metadata của Video vừa tải lên) từ hàng đợi Amazon SQS, nó sẽ gọi công cụ `FFmpeg` tích hợp sẵn trong Docker Container để bóc tách riêng luồng âm thanh ra khỏi tệp Video gốc.
2. **Lưu trữ trung gian:** Tệp âm thanh sau khi trích xuất sẽ được tải ngược lên một thư mục tạm thời chuyên biệt trên **Amazon S3** theo cấu trúc đường dẫn `s3://[Bucket-Name]/audio-temp/`.
3. **Kích hoạt Transcribe Job:** AI Worker gửi một lệnh gọi API `StartTranscriptionJob` tới Amazon Transcribe, truyền vào liên kết S3 của tệp âm thanh đồng thời cấu hình tính năng tự động nhận dạng ngôn ngữ (Auto Language Identification).
4. **Kiểm tra trạng thái bất đồng bộ (Polling):** Do tiến trình chuyển đổi giọng nói cần thời gian tính toán và diễn ra bất đồng bộ, AI Worker sẽ liên tục thực hiện vòng lặp thăm dò (Poll) qua API `GetTranscriptionJob` để cập nhật trạng thái xử lý của hệ thống.
5. **Đồng bộ kết quả:** Ngay khi trạng thái chuyển sang nhãn `COMPLETED`, Amazon Transcribe sẽ xuất ra một tệp JSON chứa toàn bộ văn bản (Transcript) đã được bóc tách, đi kèm mốc thời gian chi tiết (Timestamps) của từng từ ngữ.

![Transcribe Flow](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/transcribe_flow.png)

#### Yêu cầu quyền hạn hạ tầng (IAM Role Permissions)
Để quy trình tự động hóa trên chạy trôi chảy không gặp lỗi phân quyền, **ECS Task Role** của dịch vụ AI Worker bắt buộc phải được đính kèm các chính sách quyền hạn sau:
- `transcribe:StartTranscriptionJob` và `transcribe:GetTranscriptionJob` để điều khiển tiến trình chuyển đổi văn bản.
- `s3:GetObject` và `s3:PutObject` trên Bucket chỉ định nhằm mục đích ghi tệp âm thanh đầu vào và đọc kết quả phân tích JSON đầu ra.

{{% notice tip %}}
**Tối ưu hóa chi phí (Cost Optimization):** Việc AI Worker chủ động tách riêng và nén luồng âm thanh đầu vào (như định dạng MP3/OGG) trước khi đẩy lên đám mây không chỉ làm giảm băng thông mạng nội bộ mà còn tiết kiệm đáng kể chi phí lưu trữ. Do Amazon Transcribe tính toán chi phí dựa trên từng giây xử lý của tệp, tệp tin gọn nhẹ sẽ giúp tăng tốc độ xử lý I/O và tối ưu tài nguyên hóa đơn.
{{% /notice %}}

***

**Bước tiếp theo:** Sau khi đã sở hữu toàn bộ dữ liệu văn bản thô được trích xuất từ Video/Audio, chúng ta sẽ chuyển sang phân đoạn cuối cùng của luồng phân tích tri thức: Sử dụng mô hình nhúng toán học nhằm tạo lập các Vector (Embeddings), phục vụ trực tiếp cho tính năng tìm kiếm ngữ nghĩa chuyên sâu (Semantic Search).
