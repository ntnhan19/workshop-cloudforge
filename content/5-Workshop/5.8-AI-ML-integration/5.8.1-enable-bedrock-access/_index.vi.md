---
title : "Cấu hình Amazon Bedrock"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.8.1. </b> "
---

Một trong những khối kiến trúc quan trọng nhất cấu thành nên năng lực phân tích của Smart Media Analytics là **Amazon Bedrock** - dịch vụ đám mây được quản lý toàn diện, cung cấp quyền truy cập vào các Mô hình Nền tảng (Foundation Models - FMs) hàng đầu thị trường thông qua một giao diện API thống nhất. 

#### Cập nhật chính sách truy cập Mô hình mới nhất từ AWS
Trước đây, AWS yêu cầu kiến trúc sư hệ thống phải thao tác yêu cầu kích hoạt thủ công từng mô hình để kiểm soát rủi ro. Tuy nhiên, theo bản cập nhật nền tảng mới nhất của AWS, **trang quản lý Model Access truyền thống đã được gỡ bỏ**.

Hiện tại, các mô hình nền tảng Serverless (như Amazon Titan Text Embeddings v2 mà chúng ta sử dụng để tạo Vector) sẽ được **tự động kích hoạt (Automatically enabled)** trên toàn bộ các Region thương mại ngay trong lần gọi API đầu tiên. Điều này giúp loại bỏ hoàn toàn nút thắt cổ chai về mặt vận hành, cho phép hệ thống AI Worker kết nối và sử dụng ngay lập tức.

#### Lưu ý đối với các mô hình của Anthropic (Claude 3)
Trong dự án này, chúng ta sử dụng dòng mô hình **Anthropic Claude 3** để xử lý logic lập luận sâu và tóm tắt nội dung Media. 

Mặc dù cơ chế kích hoạt đã được tự động hóa, quy định của AWS đối với các mô hình của hãng Anthropic yêu cầu người dùng lần đầu tiên (First-time users) vẫn phải cung cấp thông tin Use Case (Mục đích sử dụng). 

1. Tìm kiếm và truy cập dịch vụ **Amazon Bedrock** (Đảm bảo đang ở Region `ap-southeast-1`).
2. Truy cập mục **Model access** ở thanh điều hướng bên trái.
3. Hệ thống sẽ hiển thị thông báo xác nhận chính sách tự động cấp quyền: *"Model access page has been retired"*.
4. Để sử dụng mô hình Claude 3, trong lần đầu tiên AI Worker gọi API (hoặc khi bạn test trên giao diện Playground của Bedrock), nếu hệ thống yêu cầu khai báo *Use case details*, bạn có thể sử dụng nội dung: *"Testing Generative AI capabilities for an educational workshop and cloud deployment pipeline simulation"*.

![Bedrock Auto Access](/images/5-Workshop/5.8-AI-ML-integration/5.8.1-enable-bedrock-access/bedrock_model_access.png)

{{% notice tip %}}
**Best Practice:** Việc AWS chuyển sang mô hình "Tự động cấp quyền" (Auto-enable upon first invocation) giúp kiến trúc sư hệ thống không cần bận tâm đến quá trình Provisioning thủ công, cực kỳ phù hợp với triết lý tự động hóa và hạ tầng linh hoạt của Cloud-native.
{{% /notice %}}

***

**Bước tiếp theo:** Với việc Amazon Bedrock đã sẵn sàng ở trạng thái tự động kích hoạt, chúng ta sẽ tiếp tục thiết lập dịch vụ **Amazon Transcribe** để xử lý luồng dữ liệu âm thanh ở phần tiếp theo.
