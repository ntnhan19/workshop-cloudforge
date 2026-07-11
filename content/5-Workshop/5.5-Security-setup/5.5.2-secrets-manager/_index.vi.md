---
title : "Quản lý cấu hình với Secrets Manager"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

Trong các chương trước, chúng ta đã thu thập được rất nhiều thông số cấu hình nhạy cảm bao gồm mật khẩu cơ sở dữ liệu (`DB_PASSWORD`), mã định danh người dùng (`User Pool ID`), và mã ứng dụng (`App Client ID`). Việc lưu trữ trực tiếp các chuỗi ký tự này dưới dạng văn bản thuần (Plaintext) trong file `.env` hoặc mã nguồn là một lỗ hổng bảo mật nghiêm trọng.

Để giải quyết bài toán này theo tiêu chuẩn doanh nghiệp, chúng ta sẽ sử dụng **AWS Secrets Manager** để mã hóa, lưu trữ tập trung và quản lý vòng đời của các cấu hình nhạy cảm này.

#### 1. Khởi tạo Secret lưu trữ cấu hình ứng dụng
1. Trên thanh tìm kiếm của AWS Console, gõ **`Secrets Manager`** và chọn dịch vụ tương ứng. Đảm bảo bạn vẫn đang ở Region `ap-southeast-1`.
2. Tại giao diện chính, bấm nút **Store a new secret** (Lưu trữ một bí mật mới).
3. Tiến hành cấu hình qua các bước sau:

**Step 1: Choose secret type**
- **Secret type:** Tích chọn **Other type of secret** (Loại bí mật khác - phù hợp để gom nhiều cặp Key/Value cấu hình ứng dụng).
- **Key/value pairs:** Chuyển sang tab **Key/value** và tiến hành thêm các tham số cốt lõi sau bằng cách bấm *Add row*:
  
  | Key | Value |
  | :--- | :--- |
  | `DB_PASSWORD` | *<Nhập mật khẩu RDS PostgreSQL thật của bạn>* |
  | `COGNITO_USER_POOL_ID` | *<Nhập User Pool ID vừa lấy ở bài 5.5.1>* |
  | `COGNITO_APP_CLIENT_ID` | *<Nhập App Client ID vừa lấy ở bài 5.5.1>* |

- **Encryption key:** Giữ nguyên mặc định là `aws/secretsmanager` (AWS sẽ tự động dùng dịch vụ KMS để mã hóa két sắt này).
- Bấm **Next**.

**Step 2: Configure secret**
- **Secret name:** Nhập tên theo cấu trúc phân cấp thư mục để dễ quản lý: `cloudforge/backend/config`
- **Description:** Nhập mô tả ngắn gọn (ví dụ: `Chứa cấu hình nhạy cảm cho ứng dụng Backend Smart Media Analytics`).
- Các mục khác giữ nguyên mặc định và bấm **Next**.

**Step 3: Configure rotation (Tùy chọn xoay vòng mật khẩu)**
- **Automatic rotation:** Chọn **Disable automatic rotation** (Tắt tính năng tự động đổi mật khẩu định kỳ để phục vụ môi trường Lab đơn giản).
- Bấm **Next**.

**Step 4: Review**
- Cuộn xuống dưới cùng để kiểm tra lại cấu trúc cấu hình và bấm nút **Store** để hoàn tất khởi tạo.

#### 2. Kiểm tra thông tin két sắt bảo mật
Sau khi bấm Store thành công, hệ thống sẽ đưa bạn quay lại danh sách. Hãy click vào Secret mang tên `cloudforge/backend/config` bạn vừa tạo.

Tại đây, bạn cần lưu ý thông số cực kỳ quan trọng sau:
* **Secret ARN:** Chuỗi định danh duy nhất trên toàn cầu của AWS (Có dạng `arn:aws:secretsmanager:ap-southeast-1:xxxxxxxxxxxx:secret:cloudforge/backend/config-xxxxxx`). 

{{% notice info %}}
**Cơ chế vận hành trong tương lai:** Sau này khi cấu hình dịch vụ AWS ECS (Elastic Container Service) để chạy ứng dụng Backend, chúng ta sẽ không cần truyền file `.env` vào Container nữa. Thay vào đó, chúng ta chỉ cần gắn chuỗi **Secret ARN** này vào phần cấu hình Task Definition. Khi Container khởi động, AWS ECS sẽ tự động dùng quyền IAM để đọc két sắt này và nạp thẳng các biến môi trường vào bộ nhớ RAM của Container một cách tuyệt đối an toàn.
{{% /notice %}}

![Secrets Manager Created](/images/5-Workshop/5.5-Security-setup/5.5.2-secrets-manager/secrets_manager_created.png)

***

**Bước tiếp theo:** Kho lưu trữ cấu hình bảo mật đã thiết lập xong. Chúng ta sẽ tiến tới phân lớp phòng thủ ngoài cùng tại **5.5.3: Cấu hình WAF Rules bảo vệ API** để thiết lập tường lửa ngăn chặn các cuộc tấn công mạng phá hoại từ Internet!
