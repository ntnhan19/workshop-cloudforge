---
title : "Kiểm tra luồng Ingestion"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---

Ba thành phần cốt lõi của kiến trúc điều phối tin nhắn (SQS, EventBridge, Step Functions) đã được thiết lập. Bây giờ là lúc chúng ta kiểm chứng (End-to-End Test) xem đường ống dữ liệu (Data Pipeline) có hoạt động thông suốt từ đầu đến cuối hay không.

Mục tiêu của bài thực hành này là: Giả lập một người dùng tải video lên S3, và kiểm tra xem SQS có nhận được tin nhắn báo hiệu có video mới hay không mà không cần bất kỳ sự can thiệp thủ công nào.

#### 1. Tải tệp tin mẫu lên Amazon S3
Đầu tiên, chúng ta sẽ đóng vai người dùng cuối thực hiện thao tác tải tệp tin lên hệ thống lưu trữ.

1. Truy cập dịch vụ **Amazon S3** trên AWS Console.
2. Tìm và bấm vào Bucket dùng để lưu trữ video upload của dự án (ví dụ: `cloudforge-media-upload-xxxxxx`).
3. Bấm nút **Upload** màu cam.
4. Bấm **Add files**, chọn một tệp tin bất kỳ từ máy tính của bạn (có thể là một đoạn video `.mp4` ngắn, hoặc đơn giản là một file `.txt` mẫu).
5. Cuộn xuống dưới cùng và bấm **Upload**. Đợi vài giây cho đến khi thanh tiến trình báo màu xanh (Succeeded).

Ngay tại khoảnh khắc tệp tin này được đưa lên S3 thành công, S3 đã phát ra một sự kiện (Event), EventBridge đã lập tức bắt lấy nó và định tuyến thẳng vào hàng đợi SQS. Hãy cùng kiểm chứng điều này!

#### 2. Kiểm tra tin nhắn tại SQS Queue
1. Mở một tab trình duyệt mới và truy cập dịch vụ **Amazon SQS**.
2. Bấm vào hàng đợi tác vụ chính **`cloudforge-media-task-queue`**.
3. Ở góc trên cùng bên phải, bấm nút **Send and receive messages**.
4. Cuộn xuống phần **Receive messages**, bạn sẽ thấy thông số *Messages available* đang có giá trị lớn hơn 0.
5. Bấm nút **Poll for messages** để hệ thống lấy tin nhắn ra khỏi hàng đợi.
6. Một tin nhắn mới sẽ xuất hiện trong danh sách bên dưới. Hãy bấm vào **ID** của tin nhắn đó.

Tại tab **Body** của tin nhắn, bạn sẽ thấy một cấu trúc JSON chi tiết được gửi từ EventBridge. Nếu nhìn kỹ, bạn sẽ tìm thấy tên Bucket S3 và tên File (Object Key) mà bạn vừa tải lên ở Bước 1. 

{{% notice tip %}}
**Sức mạnh của kiến trúc lỏng lẻo (Decoupled):** Tin nhắn này sẽ nằm chờ an toàn trong hàng đợi SQS (tối đa 4 ngày theo cấu hình mặc định) cho đến khi có một máy chủ AI Worker rảnh rỗi đến lấy (Receive) và xóa (Delete) nó đi. Dù hệ thống Backend của bạn hiện tại chưa có, hoặc có bị sập nguồn đi chăng nữa, bạn cũng không bao giờ lo mất dữ liệu khách hàng!
{{% /notice %}}

***

**Bước tiếp theo:** Chúc mừng bạn! Luồng Ingestion phi kết nối (Serverless Ingestion Pipeline) đã hoạt động hoàn hảo. Nhưng ai sẽ là người đọc những tin nhắn đang nằm chờ trong SQS và thực thi các khối xử lý AI? 

Hãy bước sang chương quan trọng nhất của hệ thống: **Chương 5.7: Compute Setup** để triển khai cụm máy chủ ứng dụng Amazon ECS (Elastic Container Service).
