---
title : "Điều phối Workflow với Step Functions"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

Sau khi thiết lập hệ thống định tuyến sự kiện (EventBridge) và hàng đợi thông điệp (SQS), bước tiếp theo là xây dựng tầng điều phối quy trình xử lý phức tạp. **AWS Step Functions** đóng vai trò như một bộ điều phối trung tâm (Orchestrator), quản lý các máy trạng thái (State Machines) để điều khiển luồng công việc xử lý Media một cách trực quan, bền bỉ và có khả năng kiểm soát lỗi cao.

#### 1. Khởi tạo và Thiết lập thuộc tính State Machine
Truy cập dịch vụ **AWS Step Functions** → **State machines** → chọn **Create state machine**. Tiến hành cấu hình các thông số nền tảng tại giao diện khởi tạo mới:

- **Template:** Lựa chọn `Create from blank` để tự định nghĩa luồng xử lý từ đầu.
- **State machine name:** Định danh quy tắc là `cloudforge-media-workflow`.
- **State machine type:** Chọn `Standard` (Đảm bảo thời gian thực thi tối đa lên đến 1 năm và hỗ trợ lưu trữ lịch sử thực thi chi tiết, phù hợp với các tác vụ phân tích Media dài hạn).
- **State machine query language:** Hệ thống mặc định sử dụng **JSONata** (Tiêu chuẩn truy vấn mới giúp tối ưu hóa việc biến đổi dữ liệu trực tiếp trong các bước chuyển trạng thái mà không cần phụ thuộc vào mã nguồn trung gian).

![Step Functions Config](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_config.png)

Bấm **Continue** để chuyển sang giao diện thiết kế trực quan.

#### 2. Xây dựng Sơ đồ Điều phối (Workflow Studio v2)
Tại không gian Canvas của Workflow Studio, tiến hành kéo thả các khối trạng thái logic để thiết lập bộ khung kiến trúc (Skeleton Architecture) cho luồng xử lý:

1. **Trạng thái Khởi đầu (Pass State - Start Processing):** Kéo khối `Pass` và thả ngay dưới điểm `Start`, đổi tên thành `Start Processing`. 
2. **Trạng thái Rẽ nhánh (Choice State - Check Status):** Kéo khối `Choice` thả ngay dưới khối xử lý ban đầu, đổi tên thành `Check Status`.
   - *Cấu hình điều kiện rẽ nhánh (JSONata):* Tại mục Rule #1, thiết lập Condition là `true` (hệ thống sẽ hiển thị dưới dạng `{% true %}`). Đây là giá trị giữ chỗ (placeholder) bắt buộc để hoàn thiện khung định tuyến rẽ nhánh trước khi tích hợp dữ liệu thực tế từ máy chủ Worker.
3. **Nhánh Xử lý Thành công (Pass State - Update Metadata):** Tại nhánh `Rule #1` của khối Choice, kéo một khối `Pass` vào và cấu hình tên là `Update Metadata` (Định hình tác vụ ghi nhận dữ liệu kết quả vào Cơ sở dữ liệu).
4. **Nhánh Xử lý Thất bại (Pass State - Notify Error):** Tại nhánh `Default` còn lại, kéo một khối `Pass` vào và cấu hình tên là `Notify Error` (Định hình tác vụ kích hoạt hệ thống cảnh báo lỗi).

![Workflow Studio Setup](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/workflow_studio_setup.png)

#### 3. Thực thi Triển khai Hạ tầng
Sau khi sơ đồ hình chữ Y ngược hoàn thành đúng logic điều phối, thực hiện các bước lưu trữ:
- Di chuyển lên góc trên cùng bên phải và bấm nút **Create**.
- **Xác nhận IAM Role:** Hệ thống sẽ tự động tạo một IAM Role chuyên dụng để cấp quyền cho phép Step Functions tương tác an toàn với các tài nguyên AWS liên quan. Bấm **Confirm** để hoàn tất.

#### 4. Kết quả Khởi tạo State Machine
Quy trình điều phối bất đồng bộ đã được cấu hình thành công, tạo ra một kiến trúc phân tách độc lập (Decoupled Architecture) bền bỉ.

![Step Functions Workflow](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_diagram.png)

{{% notice tip %}}
**System Design Note (Tách biệt Logic Nghiệp vụ):** Việc đưa toàn bộ luồng rẽ nhánh và điều phối lên tầng Step Functions giúp cô lập hoàn toàn Logic nghiệp vụ (Business Logic) ra khỏi mã nguồn của máy chủ ứng dụng (Workers). Khi hệ thống cần bổ sung tính năng (ví dụ: thêm bước nén video hoặc gửi SMS), kỹ sư chỉ cần tái cấu hình sơ đồ trên Step Functions mà không cần biên dịch hay triển khai lại mã nguồn của toàn bộ cụm máy chủ xử lý.
{{% /notice %}}

***

**Bước tiếp theo:** Ba thành phần cốt lõi của tầng Ingestion Workflow (SQS, EventBridge, Step Functions) đã được khởi tạo thành công. Chúng ta sẽ chuyển sang bài **5.6.4: Test Ingestion Flow** để tiến hành kiểm thử tính thông suốt của toàn bộ hệ thống định tuyến tin nhắn trước khi bước sang phân lớp Compute.
