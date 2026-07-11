---
title : "Cấu hình WAF Rules bảo vệ API"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

Sau khi đã thiết lập định danh (Cognito) và bảo vệ dữ liệu nhạy cảm (Secrets Manager), bước cuối cùng trong chiến lược bảo mật nhiều lớp là xây dựng một "tấm khiên" ở vòng ngoài cùng: **AWS WAF (Web Application Firewall)**.

AWS WAF sẽ đóng vai trò như một lực lượng gác cổng, phân tích mọi luồng truy cập (HTTP/HTTPS) đi vào hệ thống và tự động đánh chặn các cuộc tấn công phá hoại phổ biến trước khi chúng có thể chạm tới mã nguồn ứng dụng hoặc Database.

#### 1. Khởi tạo Protection Pack (Web ACL)
Từ AWS Console, truy cập dịch vụ **WAF & Shield** → **Protection packs (web ACLs)** và chọn **Create protection pack (web ACL)**. Do AWS liên tục cập nhật giao diện, hãy thiết lập cẩn thận theo các bước sau để tối ưu hóa chi phí:

**Bước 1: Tell us about your app**
- **App category:** Chọn **API & integration services** (Hoặc **Other**).
- **App focus:** Hệ thống sẽ tự động chọn **API** (Phù hợp với kiến trúc Backend API của chúng ta).

**Bước 2: Select resources to protect**
- Chọn **Skip for now** (Tạm thời bỏ qua). Chúng ta sẽ đính kèm Web ACL này vào Application Load Balancer ở chương sau.

**Bước 3: Choose protection pack (web ACL) resource types**
- Tích chọn loại tài nguyên khu vực: **Amazon API Gateway REST API, Application Load Balancer...**

**Bước 4: Choose initial protections (Lưu ý né bẫy chi phí)**
Theo mặc định, AWS sẽ đề xuất gói *Recommended* với chi phí khá cao. Để tối ưu chi phí cho môi trường Lab, hãy thao tác như sau:
- Bắt buộc chọn ô thứ 3: **Build your own pack from all of the protections AWS WAF offers** (Gói Tự xây dựng).

**Bước 5: Name and describe**
- **Name:** Nhập `cloudforge-api-waf`.

**Bước 6: Add rules (Thêm luật bảo vệ)**
Tại bảng **Add rules** ở bên phải màn hình, tiến hành thêm các bộ luật của AWS (AWS-managed rule group) để bảo vệ ứng dụng:
1. Thêm bộ luật **Core rule set** (AWSManagedRulesCommonRuleSet): Đánh chặn các loại tấn công web cơ bản theo OWASP Top 10.
2. Thêm bộ luật **SQL database** (AWSManagedRulesSQLiRuleSet): Ngăn chặn tuyệt đối kỹ thuật chèn mã độc cơ sở dữ liệu (SQL Injection).
*(Đảm bảo cả 2 bộ luật đều hiển thị trạng thái **Saved** màu xanh lá).*

Cuối cùng, cuộn xuống dưới cùng của màn hình chính và bấm nút **Create protection pack (web ACL)** để hoàn tất.

#### 2. Kết quả cấu hình tường lửa ứng dụng
Sau khi hệ thống khởi tạo thành công, Web ACL đã sẵn sàng ở trạng thái kích hoạt với các bộ luật tối ưu.

![WAF Rules Created](/images/5-Workshop/5.5-Security-setup/5.5.3-waf-rules/waf_rules_created.png)

{{% notice tip %}}
**Cost Optimization (Tối ưu chi phí):** Việc chủ động lựa chọn tự xây dựng bộ quy tắc (Build your own pack) thay vì sử dụng gói Recommended giúp giảm thiểu chi phí duy trì WAF từ mức ~$55/tháng xuống chỉ còn ~$5/tháng (phí duy trì Web ACL cơ bản). Thiết lập này là phù hợp và tối ưu nhất cho môi trường thử nghiệm (PoC) trong khi vẫn đảm bảo chặn đứng các rủi ro chí mạng.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống Smart Media Analytics hiện đã được triển khai các lớp bảo mật nền tảng (Cognito, Secrets Manager, AWS WAF), đáp ứng tiêu chuẩn an toàn cốt lõi. Hãy sẵn sàng bước sang **Chương 5.6: Ingestion Workflow** để bắt đầu thiết lập kiến trúc điều phối sự kiện bất đồng bộ!
