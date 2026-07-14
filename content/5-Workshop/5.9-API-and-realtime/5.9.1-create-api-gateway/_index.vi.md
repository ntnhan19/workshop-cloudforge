---
title : "Tạo API Gateway"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.9.1. </b> "
---

Để ứng dụng Frontend (dạng SPA tĩnh lưu trữ trên S3) có thể tương tác an toàn với hệ thống Backend API đang ẩn mình trong mạng lưới Private Subnets, cần một điểm trung chuyển lưu lượng (Entry point) đủ mạnh mẽ và thông minh ở vùng ranh giới công cộng. 

Đó chính là vai trò của **Amazon API Gateway**. Trong phần này, tiến hành xây dựng một HTTP API hiệu suất cao nhằm hứng toàn bộ yêu cầu (Requests) từ Internet.

#### Tại sao lại là HTTP API?
Amazon API Gateway hỗ trợ nhiều giao thức khác nhau (REST, HTTP, WebSocket). Đối với luồng giao tiếp tiêu chuẩn của dự án, **HTTP API** được ưu tiên lựa chọn nhờ vào các đặc tính vượt trội:
- **Tốc độ phản hồi:** Độ trễ nội tại (Latency) thấp hơn đáng kể so với REST API truyền thống.
- **Tối ưu chi phí:** Chi phí vận hành rẻ hơn tới 71% so với REST API, cực kỳ lý tưởng cho các kiến trúc Microservices chịu tải cao.
- **Tích hợp tự nhiên:** Hỗ trợ tích hợp hoàn hảo với tính năng VPC Link để đẩy lưu lượng trực tiếp vào Application Load Balancer (ALB) bên trong mạng nội bộ.

#### Khởi tạo HTTP API
1. Truy cập dịch vụ **API Gateway** thông qua thanh tìm kiếm trên AWS Management Console.
2. Tại bảng điều khiển chính, tìm khối có tên **HTTP API** và nhấp chọn nút **Build**.
3. Ở bước cấu hình đầu tiên (Step 1), bạn không cần thiết lập tích hợp (Integration) vội. Hãy nhập tên API là `CloudForge-Media-API` để dễ dàng nhận diện.
4. Bấm **Next** lướt qua các bước cấu hình Route mặc định.
5. Ở bước cuối cùng, kiểm tra lại thông tin và nhấn **Create** để hệ thống khởi tạo cổng API.

![Create HTTP API](/images/5-Workshop/5.9-API-and-realtime/5.9.1-create-api-gateway/create_http_api.png)

#### Cấu hình chia sẻ tài nguyên chéo (CORS)
Do API của tiến hành được gọi từ một trình duyệt Frontend nằm ở tên miền (Domain) khác, trình duyệt sẽ tự động chặn các yêu cầu này vì lý do an ninh (lỗi CORS) nếu chúng ta không thiết lập rõ ràng.

1. Tại thanh Menu điều hướng bên trái của `CloudForge-Media-API`, chọn mục **CORS**.
2. Nhấn nút **Configure** và thiết lập các thông số bảo mật như sau:
   - **Access-Control-Allow-Origin:** Thêm `*` (Cho phép mọi nguồn gốc. *Lưu ý: Trong thực tế Production, cần chỉ định đích danh tên miền của ứng dụng để đảm bảo bảo mật*).
   - **Access-Control-Allow-Methods:** Thêm `*` (Cho phép mọi phương thức HTTP như GET, POST, PUT, DELETE).
   - **Access-Control-Allow-Headers:** Thêm `*` (Cho phép truyền mọi Headers tùy chỉnh).
3. Nhấn **Save** để áp dụng bộ quy tắc cấu hình phân giải chéo.

{{% notice tip %}}
**Cốt lõi Vận hành:** Ở thời điểm hiện tại, HTTP API này mới chỉ là "một cánh cửa trống" dẫn ra Internet. Nó đã được cấp phát một địa chỉ URL (Invoke URL) hợp lệ nhưng hoàn toàn chưa biết cách điều hướng luồng dữ liệu đi đâu vào bên trong hệ thống.
{{% /notice %}}

***

**Bước tiếp theo:** Để cánh cửa API này thực sự phát huy tác dụng, tiến hành thiết lập "Cây cầu kết nối" (VPC Link) để dẫn luồng dữ liệu đâm xuyên qua mạng riêng ảo và tới trúng đích hạ tầng Backend trong bài học tiếp theo.
