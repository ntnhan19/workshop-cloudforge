---
title : "Điều phối Workflow"
date : 2026-07-10
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

### Tổng quan Điều phối (Ingestion Workflow)

Trong các chương trước, chúng ta đã hoàn thiện phân lớp lưu trữ, cơ sở dữ liệu và bảo mật nền tảng. Tại chương 5.6 này, hệ thống sẽ được thiết lập **kiến trúc hướng sự kiện (Event-driven Architecture)** để liên kết các thành phần dịch vụ một cách lỏng lẻo (decoupled) nhưng vẫn đảm bảo tính bền bỉ và khả năng mở rộng cao.

Với đặc thù của hệ thống Smart Media Analytics là phải xử lý khối lượng lớn tệp đa phương tiện, các tác vụ phân tích AI (nhận diện hình ảnh, bóc băng âm thanh, trích xuất metadata) đòi hỏi rất nhiều thời gian tính toán. Nếu sử dụng kiến trúc gọi API đồng bộ (Synchronous) truyền thống, hệ thống sẽ dễ dàng đối mặt với tình trạng thắt cổ chai (bottleneck) hoặc ngắt kết nối (timeout). Do đó, việc chuyển đổi sang mô hình xử lý bất đồng bộ (Asynchronous) dựa trên thông điệp là bắt buộc.

Trong chương này, chúng ta sẽ tập trung triển khai các thành phần điều phối lõi:
1. **Amazon SQS (Simple Queue Service):** Đóng vai trò là bộ đệm (buffer) và hàng đợi chứa các tác vụ xử lý video/âm thanh, phân phối công việc đến các AI Worker một cách an toàn mà không rủi ro mất mát dữ liệu khi tải tăng đột biến.
2. **Amazon EventBridge:** Trung tâm định tuyến sự kiện (Event Bus), có nhiệm vụ lắng nghe các thay đổi trạng thái (ví dụ: tệp tin được tải lên S3 thành công, hoặc AI xử lý hoàn tất/thất bại) để kích hoạt tự động luồng xử lý tiếp nối.
3. **AWS Step Functions (Tùy chọn):** Máy trạng thái (State Machine) điều phối các quy trình xử lý luồng phức tạp, rẽ nhánh đa điều kiện và cung cấp khả năng giám sát trực quan toàn bộ vòng đời của một tác vụ Media.

Hoàn thành chương này sẽ giúp dự án của chúng ta chuyển mình từ một hệ thống gọi API đồng bộ dễ bị nghẽn sang một kiến trúc xử lý tin nhắn bền bỉ, sẵn sàng tiếp nhận hàng ngàn video tải lên cùng lúc.

### Nội dung thực hành

- [Khởi tạo Hàng đợi SQS](5.6.1-create-sqs-queue/)
- [Cấu hình EventBridge](5.6.2-create-eventbridge-rule/)
- [Điều phối Workflow với Step Functions](5.6.3-create-step-functions/)
- [Kiểm tra luồng Ingestion](5.6.4-test-ingestion-flow/)
