---
title : "Cấu hình EventBridge"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---

Sau khi hàng đợi SQS đã sẵn sàng tiếp nhận công việc, bước tiếp theo trong kiến trúc hướng sự kiện (Event-driven) là thiết lập bộ định tuyến trung tâm: **Amazon EventBridge**.

Trong hệ thống Smart Media Analytics, khi một tệp đa phương tiện (Video/Audio) được tải lên Amazon S3, S3 sẽ phát ra một sự kiện. EventBridge đóng vai trò "lắng nghe" và ngay lập tức định tuyến thông điệp chứa siêu dữ liệu của tệp tin đó vào hàng đợi SQS `cloudforge-media-task-queue` để các AI Worker lấy ra xử lý.

#### 1. Khởi tạo EventBridge Rule (Enhanced Builder)
Truy cập dịch vụ **Amazon EventBridge** → **Rules** → **Create rule**. Sử dụng giao diện thiết lập trực quan (Enhanced builder) để cấu hình luồng định tuyến:

**Bước 1: Thiết lập Trigger (Nguồn phát sự kiện)**
- Tại thẻ **Build**, khu vực *Events* bên trái, tìm và kéo khối **S3 (Simple Storage Service) Object Created** thả vào khu vực **Triggering Events**.
- *Tùy chọn nâng cao (Lọc sự kiện):* Thông qua tính năng **Event pattern (Filter)**, kiến trúc có thể được cấu hình để chỉ định bắt sự kiện từ một S3 Bucket cụ thể, giúp tối ưu hóa lưu lượng và ngăn chặn các vòng lặp xử lý không mong muốn.

![EventBridge Trigger Setup](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_trigger.png)

**Bước 2: Thiết lập Target (Đích đến)**
- Tại thanh công cụ bên trái, tìm kiếm dịch vụ **SQS** (hoặc mở danh mục AWS Services), kéo khối **Amazon SQS** thả vào khu vực **Targets**.
- Tại bảng cấu hình Target, mục *Queue*, chọn đúng hàng đợi **`cloudforge-media-task-queue`** đã khởi tạo ở phân đoạn trước.

![EventBridge Target Setup](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_target.png)

**Bước 3: Định danh quy tắc**
- Chuyển sang thẻ **Configure** ở thanh điều hướng.
- **Rule name:** Cấp tên quy tắc là `cloudforge-s3-to-sqs-rule`.
- **Activation:** Đảm bảo nút kích hoạt đang ở trạng thái **Active**.

![EventBridge Rule Naming](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_rule_naming.png)

Cuối cùng, kiểm tra lại sơ đồ định tuyến và bấm **Create** ở góc trên cùng bên phải để hoàn tất quá trình triển khai.

#### 2. Kết quả triển khai Event Routing
Khi quy tắc được tạo thành công và kích hoạt, đường ống dữ liệu bất đồng bộ (Asynchronous Data Pipeline) đã chính thức được thiết lập thông suốt từ S3 Storage đi qua EventBridge và tập kết an toàn tại SQS Queue.

![EventBridge Rule Created](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_rule_created.png)

{{% notice tip %}}
**System Design Note (Kiến trúc phi kết nối):** Việc tách rời (Decoupling) các thành phần bằng EventBridge giúp lớp Frontend không cần quan tâm đến trạng thái của hệ thống Backend. Người dùng chỉ việc tải file lên S3 và nhận phản hồi thành công ngay lập tức. Mọi tác vụ nặng phía sau đã có EventBridge và SQS điều phối, loại bỏ hoàn toàn rủi ro tắc nghẽn (bottleneck) hoặc quá tải (timeout) tại cổng giao tiếp API.
{{% /notice %}}

***

**Bước tiếp theo:** Dòng chảy dữ liệu từ khi người dùng Upload (S3) → Nhận diện sự kiện (EventBridge) → Chờ xử lý (SQS) đã được khơi thông. Tiếp theo, chúng ta sẽ làm quen với một khái niệm nâng cao trong kiến trúc điều phối tại bài **5.6.3: Khởi tạo Step Functions (Tùy chọn)**.
