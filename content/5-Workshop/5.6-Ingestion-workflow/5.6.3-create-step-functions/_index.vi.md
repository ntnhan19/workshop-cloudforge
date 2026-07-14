---
title : "Điều phối Workflow với Step Functions"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

Sau khi thiết lập hệ thống định tuyến sự kiện (EventBridge) và hàng đợi thông điệp (SQS), bước tiếp theo là xây dựng tầng điều phối quy trình xử lý phức tạp. **AWS Step Functions** đóng vai trò như một bộ điều phối trung tâm (Orchestrator), quản lý các máy trạng thái (State Machines) để điều khiển luồng công việc xử lý Media một cách trực quan, bền bỉ và có khả năng kiểm soát lỗi cao.

#### 1. Thiết lập EventBridge Connection cho Webhook
Để Step Functions có thể gọi một API ngoại vi, AWS yêu cầu cấu hình một Connection để lưu trữ phương thức xác thực. Do API Backend hiện tại không yêu cầu xác thực bằng token, nhóm đã chọn phương thức **API Key** với một giá trị giả định (dummy) để vượt qua ràng buộc bắt buộc của hệ thống:
- Truy cập **Amazon EventBridge** → **API destinations** → tab **Connections** → **Create connection**.
- **Connection name:** `cloudforge-webhook-conn`.
- **API type:** Chọn `Public` (do ALB của hệ thống có thể gọi qua Public DNS hoặc có thể chọn `Private` nếu VPC nội bộ).
- **Configure authorization:** Chọn `Custom configuration`.
- **Authorization type:** Chọn `API Key` (với `API Key Name` nhập `X-Api-Key` và `Value` nhập `dummy`).

![Create Connection](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/eventbridge_connection_1.png)
![Configure Authorization](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/eventbridge_connection_2.png)

#### 2. Khởi tạo và Thiết lập thuộc tính State Machine
Truy cập dịch vụ **AWS Step Functions** → **State machines** → chọn **Create state machine**. Tiến hành cấu hình các thông số nền tảng tại giao diện khởi tạo mới:

- **Template:** Lựa chọn `Create from blank` để tự định nghĩa luồng xử lý từ đầu.
- **State machine name:** Định danh quy tắc là `cloudforge-media-workflow`.
- **State machine type:** Chọn `Standard` (Đảm bảo thời gian thực thi tối đa lên đến 1 năm và hỗ trợ lưu trữ lịch sử thực thi chi tiết, phù hợp với các tác vụ phân tích Media dài hạn).
- **State machine query language:** Hệ thống mặc định sử dụng **JSONata** (Tiêu chuẩn truy vấn mới giúp tối ưu hóa việc biến đổi dữ liệu trực tiếp trong các bước chuyển trạng thái mà không cần phụ thuộc vào mã nguồn trung gian).

![Step Functions Config](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_config.png)

Bấm **Continue** để chuyển sang giao diện thiết kế trực quan.

#### 3. Xây dựng Sơ đồ Điều phối
Tại không gian Canvas của Workflow Studio, tiến hành kéo thả các khối trạng thái logic để thiết lập tích hợp với ECS AI Worker:

1. **Khối Kích hoạt Worker (Amazon ECS - RunTask):** Từ thanh công cụ bên trái (tab Actions), tìm kiếm **ECS** và kéo khối **RunTask** thả ngay dưới điểm `Start`. Đổi tên thành `Launch AI Worker`.
2. **Cấu hình tham số Arguments cho RunTask (JSONata):**
   - Click vào khối `Launch AI Worker`. Ở tab **Configuration** bên phải, hãy tích vào ô **Wait for task to complete** (Cực kỳ quan trọng: Điều này buộc Step Functions phải chờ AI Worker chạy xong 100% rồi mới đi tiếp xuống khối gọi Webhook).
   - Chuyển sang tab **Arguments & Output**, sẽ thấy một khung soạn thảo JSON.
   - Vì ở bước trước chúng ta đã chọn ngôn ngữ truy vấn là **JSONata**, giao diện mới của AWS Step Functions sẽ gộp chung tất cả cấu hình (Cluster, Network, Overrides...) vào một file JSON duy nhất thay vì các ô nhập liệu rời rạc.
   - Xóa đoạn JSON mặc định và dán đoạn mã sau vào ô **Arguments**:

```json
{
  "LaunchType": "FARGATE",
  "Cluster": "cloudforge-compute-cluster",
  "TaskDefinition": "cloudforge-ai-worker-task",
  "NetworkConfiguration": {
    "AwsvpcConfiguration": {
      "Subnets": [
        "<YOUR_SUBNET_ID_1>",
        "<YOUR_SUBNET_ID_2>"
      ],
      "SecurityGroups": [
        "<YOUR_SECURITY_GROUP_ID>"
      ],
      "AssignPublicIp": "ENABLED"
    }
  },
  "Overrides": {
    "ContainerOverrides": [
      {
        "Name": "worker-container",
        "Command": [
          "python",
          "entrypoint.py",
          "--config-json",
          "{% $string({'bucket': $states.input.detail.bucket.name, 'video_s3_key': $states.input.detail.object.key}) %}"
        ]
      }
    ]
  }
}
```
   - *(Lưu ý: Các giá trị `<YOUR_SUBNET_ID>` và `<YOUR_SECURITY_GROUP_ID>` được thiết lập tương ứng với VPC Subnet và Security Group thực tế của hệ thống. Cú pháp `{% $string(...) %}` là sức mạnh của JSONata, giúp tự động trích xuất metadata từ EventBridge S3 event và biến nó thành chuỗi JSON truyền vào cho Container).*

3. **Khối Gọi Webhook Backend (HTTP - Invoke):** Để hoàn thiện kiến trúc Event-Driven, sau khi AI Worker hoàn thành trích xuất cảnh, hệ thống cần thông báo cho Backend để tiếp tục chạy các Model AI nặng (Whisper, Bedrock) trên cùng một `job_id`. 
   - Từ thanh tìm kiếm bên trái, gõ chữ `invoke` hoặc `HTTP`. Kéo khối **HTTP Endpoint (Call HTTPS APIs)** thả ngay dưới khối `Launch AI Worker`. Đổi tên thành `Call Backend Webhook` (Lưu ý: TUYỆT ĐỐI KHÔNG chọn khối Lambda Invoke).
   - Trong phần cấu hình **API Endpoint**, do Step Functions yêu cầu bắt buộc giao thức bảo mật `https://` và ALB Backend nằm trong mạng Private, nhóm đã điền URL của cổng **API Gateway** (nó sẽ đóng vai trò trung gian nhận HTTPS và chuyển tiếp qua VPC Link vào ALB). Cú pháp: `https://<API-GATEWAY-ID>.execute-api.<REGION>.amazonaws.com/api/v1/ingest/webhook` với Method `POST`.
   - **Authentication:** Để tích hợp HTTP Invoke trên Step Functions, nhóm đã sử dụng **EventBridge Connection** (`cloudforge-webhook-conn` vừa tạo ở bước trên) nhằm quản lý truy cập giữa Step Functions và ALB.
   - Ngay tại tab **Configuration**, cuộn xuống mục **Request body**, định nghĩa payload (sử dụng JSONata) để báo cáo trạng thái thành công kèm theo `job_id`. Vì Step Functions đã chạy qua bước ECS nên đầu vào gốc đã bị thay đổi, chúng ta phải dùng biến Context `$states.context.Execution.Input` để lấy lại sự kiện gốc từ EventBridge:
     ```json
     {
       "job_id": "{% $split($states.context.Execution.Input.detail.object.key, '/')[1] %}",
       "status": "success"
     }
     ```

![Workflow Studio Setup](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/workflow_studio_setup.png)

#### 4. Thực thi Triển khai Hạ tầng
Sau khi sơ đồ hình chữ Y ngược hoàn thành đúng logic điều phối, thực hiện các bước lưu trữ:
- Di chuyển lên góc trên cùng bên phải và bấm nút **Create**.
- **Cấu hình quyền IAM Role:** Do chúng ta sử dụng JSONata với payload tự viết, AWS Console sẽ KHÔNG THỂ tự động nhận diện được quyền `ecs:RunTask` cần thiết. Bạn BẮT BUỘC phải cấp quyền thủ công cho Role này:
  1. Sau khi lưu thành công, tại trang chi tiết của State Machine, hãy bấm vào đường link ở mục **IAM role ARN** để mở giao diện quản lý IAM.
  2. Tại trang IAM Role, bấm nút **Add permissions** -> chọn **Create inline policy**.
  3. Chuyển sang tab **JSON**, xóa mã cũ và dán đoạn mã sau vào (bao gồm quyền RunTask, PassRole và các quyền EventBridge để Step Functions có thể "chờ" ECS Task chạy xong):
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": [
                     "ecs:RunTask",
                     "ecs:StopTask",
                     "ecs:DescribeTasks"
                 ],
                 "Resource": "*"
             },
             {
                 "Effect": "Allow",
                 "Action": [
                     "events:PutTargets",
                     "events:PutRule",
                     "events:DescribeRule"
                 ],
                 "Resource": [
                     "arn:aws:events:*:*:rule/StepFunctionsGetEventsForECSTaskRule"
                 ]
             },
             {
                 "Effect": "Allow",
                 "Action": "iam:PassRole",
                 "Resource": "*"
             }
         ]
     }
     ```
  4. Bấm **Next**, đặt tên policy là `AllowECSRunTask`, rồi bấm **Create policy** để hoàn tất.
  
![IAM Inline Policy](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/iam_inline_policy.png)

#### 5. Cập nhật EventBridge Rule (Định tuyến Fan-out)
Sau khi đã có Step Functions, cần quay lại EventBridge để gắn nó thành đích đến thứ 2:
1. Quay lại dịch vụ **Amazon EventBridge** -> **Rules**, chọn Rule `cloudforge-s3-to-sqs-rule` và bấm **Edit**.
2. Tại màn hình **Enhanced builder**, bấm vào mũi tên `>` ở viền trái màn hình để mở thanh công cụ **Events and Targets**.
3. Tìm kiếm dịch vụ **AWS Step Functions state machine**, sau đó kéo thả khối này vào khu vực **Targets** (nằm cùng chỗ với SQS queue).
4. Tại bảng cấu hình bên phải, chọn State machine là `cloudforge-media-workflow`.
5. Bấm **Update** (nút màu cam ở góc phải) để lưu lại. Kể từ lúc này, 1 sự kiện từ S3 sẽ được đẩy song song ra cả 2 nơi.

![EventBridge Target Update](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/eventbridge_target_update.png)

#### 6. Kết quả Khởi tạo State Machine
Quy trình điều phối bất đồng bộ đã được cấu hình thành công, tạo ra một kiến trúc phân tách độc lập (Decoupled Architecture) bền bỉ.

![Step Functions Workflow](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_diagram.png)

{{% notice tip %}}
**System Design Note (Tách biệt Logic Nghiệp vụ):** Việc đưa toàn bộ luồng rẽ nhánh và điều phối lên tầng Step Functions giúp cô lập hoàn toàn Logic nghiệp vụ (Business Logic) ra khỏi mã nguồn của máy chủ ứng dụng (Workers). Bằng cách thiết lập Step Functions gọi Webhook, hệ thống đảm bảo AI Worker chỉ thực hiện duy nhất nhiệm vụ xử lý Video, trong khi Backend chịu trách nhiệm tổng hợp AI và cập nhật cơ sở dữ liệu.
{{% /notice %}}

***

**Bước tiếp theo:** Ba thành phần cốt lõi của tầng Ingestion Workflow (SQS, EventBridge, Step Functions) đã được khởi tạo thành công. Tiến hành chuyển sang bài [**5.6.4: Test Ingestion Flow**](../5.6.4-test-ingestion-flow/) để tiến hành kiểm thử tính thông suốt của toàn bộ hệ thống định tuyến tin nhắn trước khi bước sang phân lớp Compute.
