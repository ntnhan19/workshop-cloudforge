---
title : "Tích hợp Truy vết X-Ray"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.13.2. </b> "
---

Trong một kiến trúc phân tán (Microservices/Serverless), khi người dùng gọi một API, luồng dữ liệu có thể đi xuyên qua hàng loạt dịch vụ: API Gateway -> Load Balancer -> ECS Container -> Cơ sở dữ liệu (Aurora/DynamoDB). Nếu API phản hồi chậm hoặc báo lỗi 500, việc chỉ đọc text log tại CloudWatch (Bài 5.13.1) là chưa đủ để tìm ra chính xác nút thắt (Bottleneck) nằm ở dịch vụ nào.

Giải pháp tối ưu của AWS cho bài toán này là sử dụng **AWS X-Ray** để truy vết phân tán (Distributed Tracing).

Bài viết này hướng dẫn cách cấu hình chạy X-Ray Daemon theo mô hình Sidecar trên Amazon ECS và tích hợp bộ thư viện X-Ray SDK vào mã nguồn ứng dụng Backend.

#### Bước 1: Bổ sung quyền cho ECS Task Role
Để các container bên trong ECS Task có thể gửi dữ liệu truy vết về dịch vụ AWS X-Ray một cách hợp lệ, chúng cần được cấp quyền IAM tương ứng.

1. Truy cập **AWS IAM Console** -> **Roles**.
2. Tìm và mở Role đang được gán làm **Task Role** cho dịch vụ Backend (Lưu ý: Là Task Role, không phải Task Execution Role).
3. Chọn **Add permissions** -> **Attach policies**.
4. Tìm kiếm và tick chọn chính sách **`AWSXRayDaemonWriteAccess`**.
5. Bấm **Add permissions**.

#### Bước 2: Thêm X-Ray Daemon Sidecar vào Task Definition
Trên môi trường ECS Fargate, phương pháp chuẩn nhất để chạy X-Ray là mô hình **Sidecar**: Chạy một container phụ (Daemon) song song với container chính (Backend) bên trong cùng một Task. Daemon này sẽ lắng nghe, thu thập các gói dữ liệu Trace từ ứng dụng và đẩy lên AWS thông qua giao thức UDP.

Truy cập **Amazon ECS Console** -> **Task Definitions**, chọn tạo một bản cập nhật mới (Create new revision) và thêm khối định nghĩa container thứ hai:

```json
{
    "name": "xray-daemon",
    "image": "public.ecr.aws/xray/aws-xray-daemon:latest",
    "cpu": 32,
    "memoryReservation": 256,
    "portMappings": [
        {
            "containerPort": 2000,
            "protocol": "udp"
        }
    ]
}
```

*Lưu ý: Bắt buộc mở cổng UDP 2000 để container Backend có thể giao tiếp nội bộ với X-Ray Daemon Container.*

#### Bước 3: Tích hợp X-Ray SDK vào Mã nguồn ứng dụng (FastAPI)
Sau khi hạ tầng đã sẵn sàng đón nhận dữ liệu, bước cuối cùng là khai báo thư viện trong mã nguồn Backend (Python/FastAPI) để hệ thống tự động sinh ra các gói Trace cho mọi Request.

1. Cài đặt thư viện AWS X-Ray SDK vào tệp `requirements.txt` của dự án:

```text
aws-xray-sdk==2.12.0
```

2. Bổ sung Middleware của X-Ray vào tệp tin khởi chạy máy chủ (ví dụ `main.py`). Middleware này sẽ tự động đo lường thời gian xử lý của mọi API Endpoints:

```python
from fastapi import FastAPI
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.starlette.middleware import XRayMiddleware

app = FastAPI()

# Cấu hình X-Ray Recorder và định danh tên dịch vụ hiển thị trên Service Map
xray_recorder.configure(service='SmartMedia-BackendAPI')

# Tích hợp Middleware đo lường vào FastAPI
app.add_middleware(XRayMiddleware, app=app, recorder=xray_recorder)

# --- Khai báo các Routes xử lý nghiệp vụ tại đây ---
@app.get("/api/health")
def health_check():
    return {"status": "Healthy"}
```

#### Bước 4: Quan sát Service Map và Trace
Sau khi commit mã nguồn và để luồng CI/CD (GitHub Actions) tự động triển khai phiên bản mới lên ECS, hãy thử gửi vài Request từ trình duyệt/Postman vào API của hệ thống.

Truy cập **AWS X-Ray Console** -> **Service map**. Bạn sẽ nhìn thấy một biểu đồ mạng lưới trực quan mô phỏng đường đi của dữ liệu (Client gọi vào Backend, Backend truy vấn vào PostgreSQL).

Chuyển sang tab **Traces**, quản trị viên có thể click vào từng Request cụ thể để xem biểu đồ Gantt phân tích chính xác thời gian thi hành của mỗi tác vụ con tính bằng mili-giây, giúp dễ dàng khoanh vùng các truy vấn SQL chậm hoặc các logic xử lý đang ngốn nhiều CPU.

![AWS X-Ray Service Map](/images/5-Workshop/5.13-observability/5.13.2-xray-service-map.png)
*(Hướng dẫn chụp: Chụp lại màn hình Service Map trên AWS Console hiển thị các node giao tiếp trực quan từ Client -> Backend -> Database).*

***

**Tổng kết Chương 5.13:** Với bộ đôi Amazon CloudWatch và AWS X-Ray, hệ thống Smart Media Analytics đã đạt mức độ trưởng thành cao nhất về năng lực vận hành (Operational Excellence). Quản trị viên giờ đây có toàn quyền kiểm soát log tập trung, nhận cảnh báo tự động khi quá tải, và truy vết trực quan từng luồng dữ liệu phân tán.

**Bước tiếp theo:** Xin chúc mừng! Bạn đã hoàn thành xuất sắc việc xây dựng toàn bộ kiến trúc lõi của dự án CloudForge. Tại **Chương 5.14: Dọn dẹp tài nguyên (Cleanup)**, chúng ta sẽ thực hiện các bước thu hồi hạ tầng để tránh phát sinh chi phí AWS ngoài ý muốn sau khi kết thúc Workshop.
