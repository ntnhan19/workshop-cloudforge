---
title : "Trải nghiệm Hệ thống (System Walkthrough)"
date : 2026-07-14
weight : 14
chapter : false
pre : " <b> 5.14. </b> "
---

Sau khi triển khai thành công toàn bộ các thành phần Frontend và Backend, quá trình kiểm thử (end-to-end testing) toàn diện được tiến hành để xác thực hệ thống dưới góc độ của người dùng cuối.

Chương này trình diễn cách nền tảng Smart Media Analytics vận hành trong kịch bản thực tế.

### 1. Đăng ký & Đăng nhập
1. [Đường dẫn giao diện web Amplify](https://main.d10clqxr7o0tzf.amplifyapp.com/) được triển khai ở Chương 5.11 được truy cập.
2. Tại tab **Sign Up**, thông tin bao gồm email, mật khẩu và tên được điền để tạo tài khoản mới.
3. Một **Mã xác nhận (Confirmation Code)** được Amazon Cognito gửi tự động và bảo mật đến email để xác thực.
4. Sau khi xác thực thành công, thao tác đăng nhập được thực hiện để truy cập vào bảng điều khiển (Dashboard) chính.

![Giao diện Đăng nhập Cognito](/images/5-Workshop/5.14-User-guide/5.14-login-ui.png)

### 2. Tải lên Video (Ingestion)
1. Phần **Upload** được truy cập để tiến hành chọn một tệp video mẫu.
2. Hệ thống Frontend tiến hành upload trực tiếp file lên bucket S3 thông qua cơ chế Pre-signed URL bảo mật, kèm theo thanh tiến trình trực quan.

![Tiến trình Upload Video](/images/5-Workshop/5.14-User-guide/5.14-upload-progress.png)

3. Khi quá trình upload kết thúc, video xuất hiện trong thư viện với trạng thái **Processing (Đang xử lý)**. Ở phía sau, EventBridge kích hoạt Step Functions, và cụm ECS AI Worker tiến hành trích xuất và phân tích khung hình video.

![Trạng thái Đang Xử lý](/images/5-Workshop/5.14-User-guide/5.14-processing-status.png)

4. Sau khi AI pipeline hoàn thành việc phân tích và đánh chỉ mục (indexing), trạng thái tự động chuyển sang **Completed** và hiển thị đầy đủ thông tin media.

![Trạng thái Đã xử lý](/images/5-Workshop/5.14-User-guide/5.14-completed-status.png)

### 3. Chi tiết Media & Tính năng cho Editor
1. Khi bấm chọn một video đã được xử lý hoàn tất, giao diện Chi tiết Asset (Asset Detail) được mở ra.
2. Một giao diện toàn diện dành cho Editor được cung cấp, nổi bật với hệ thống **Phân loại Tag (Tag Classification)** tự động dựa trên kết quả phân tích AI.
3. Toàn bộ **Bản dịch thoại (Transcript)** của video cũng được hiển thị chi tiết, giúp quá trình theo dõi và kiểm duyệt nội dung diễn ra nhanh chóng.

![Chi tiết Asset, Tag và Transcript](/images/5-Workshop/5.14-User-guide/5.14-asset-detail.png)

### 4. Tìm kiếm Ngữ nghĩa (Search in Video)
1. Bên trong giao diện Asset Detail, tính năng **Search in Video** cho phép tìm kiếm ngữ nghĩa chuyên sâu ngay bên trong nội dung của video đó.
2. Khi một câu mô tả tự nhiên được nhập vào (ví dụ: *"cảnh con báo đang chạy trên tuyết"* hoặc *"mặt trời lặn trên biển"*), hệ thống Backend mã hóa câu văn thành vector (thông qua Amazon Titan) và đối chiếu trong CSDL RDS PostgreSQL `pgvector`.
3. Hệ thống trả về chính xác các phân cảnh (scene) kèm theo **mốc thời gian (timestamp)** khớp với miêu tả.
4. Khi một phân cảnh được click chọn, trình phát (player) sẽ tự động bật video ngay tại đúng thời điểm đó.

![Kết quả Search in Video](/images/5-Workshop/5.14-User-guide/5.14-search-in-video.png)

---

**Bước tiếp theo:** Với hệ thống kiến trúc hướng sự kiện tích hợp AI đã được xây dựng và kiểm chứng thành công, giai đoạn cuối cùng là dọn dẹp môi trường. [**Chương 5.15: Dọn dẹp Tài nguyên (Cleanup)**](../5.15-Cleanup/) trình bày các thao tác gỡ bỏ hạ tầng nhằm đảm bảo không phát sinh chi phí AWS ngoài ý muốn.
