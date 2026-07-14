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

![IAM Task Role](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/iam_task_role.png)

#### Bước 2: Thêm X-Ray Daemon Sidecar vào Task Definition
Trên môi trường ECS Fargate, phương pháp chuẩn nhất để chạy X-Ray là mô hình **Sidecar**: Chạy một container phụ (Daemon) song song với container chính (Backend) bên trong cùng một Task. Daemon này sẽ lắng nghe, thu thập các gói dữ liệu Trace từ ứng dụng và đẩy lên AWS thông qua giao thức UDP.

Truy cập **Amazon ECS Console** -> **Task Definitions**, chọn Task Definition của Backend API và bấm **Create new revision** -> **Create new revision**.

Tại giao diện tạo Revision mới, giữ nguyên các cấu hình của Container 1 (Backend API) và Task size. Kéo xuống dưới, bấm **Add more containers** để thêm container thứ hai với các thông số sau:

*   **Name**: `xray-daemon`
*   **Image URI**: `public.ecr.aws/xray/aws-xray-daemon:latest`
*   **Essential container**: Yes

![X-Ray Container Setup](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/xray_container_setup.png)

**Port mappings:**
*   **Container port**: `2000`
*   **Protocol**: `UDP` *(Bắt buộc chọn UDP để container Backend có thể giao tiếp nội bộ với X-Ray Daemon)*.

**Resource allocation limits:**
*   **Memory hard limit**: `256` *(Cấp 256 MB RAM cho Daemon hoạt động)*.

![X-Ray Port & Resource Setup](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/xray_port_resource.png)

**Log collection:**
*   Đánh dấu chọn **Use log collection** để đẩy log của X-Ray Daemon lên CloudWatch, giúp dễ dàng debug nếu Daemon gặp lỗi khởi động.

![X-Ray Log Collection](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/xray_log_collection.png)

Sau khi điền đủ thông tin, cuộn xuống cuối trang và bấm **Create**. ECS sẽ tạo ra một phiên bản Task Definition mới (chứa 2 containers). Tiếp theo, bạn chỉ việc cập nhật ECS Service để sử dụng Revision mới này.

#### Bước 3: Tích hợp X-Ray SDK vào Mã nguồn ứng dụng (FastAPI)
Sau khi hạ tầng đã sẵn sàng đón nhận dữ liệu, bước cuối cùng là khai báo thư viện trong mã nguồn Backend (Python/FastAPI) để hệ thống tự động sinh ra các gói Trace cho mọi Request.

1. Cài đặt thư viện AWS X-Ray SDK vào tệp `requirements.txt` của dự án:

```text
aws-xray-sdk==2.12.0
```

2. Bổ sung Middleware tùy chỉnh của X-Ray vào tệp tin khởi chạy máy chủ (ví dụ `main.py`). Middleware này sẽ tự động đo lường thời gian xử lý của mỗi API Endpoints:

```python
from fastapi import FastAPI
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core.async_context import AsyncContext
from starlette.types import ASGIApp, Receive, Scope, Send

app = FastAPI()

# Cấu hình X-Ray Recorder và định danh tên dịch vụ hiển thị trên Service Map
xray_recorder.configure(service='SmartMedia-BackendAPI', context=AsyncContext())

class CustomXRayMiddleware:
    def __init__(self, app: ASGIApp):
        self.app = app

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        if scope["type"] != "http":
            return await self.app(scope, receive, send)
            
        segment = xray_recorder.begin_segment('SmartMedia-BackendAPI')
        try:
            segment.put_http_meta('url', scope.get('path'))
            segment.put_http_meta('method', scope.get('method'))
            await self.app(scope, receive, send)
        except Exception as e:
            segment.add_exception(e)
            raise
        finally:
            xray_recorder.end_segment()

# Tích hợp Middleware đo lường vào FastAPI
app.add_middleware(CustomXRayMiddleware)
```

#### Bước 4: Quan sát Trace Map và Trace details
Sau khi cập nhật ECS Service thành công, hãy thử gửi vài Request (ví dụ gọi API `/health` hoặc API `/search`) thông qua Postman hoặc trình duyệt.

Truy cập **AWS CloudWatch Console**, nhìn sang menu bên trái, dưới mục **Application Signals (APM)**, chọn **Trace Map** (hoặc Application Map). Sẽ nhìn thấy một biểu đồ mạng lưới trực quan mô phỏng đường đi của dữ liệu (các luồng Client gọi vào hệ thống Backend).

Tiếp tục chọn mục **Traces** ở menu trái, quản trị viên có thể click vào từng Request cụ thể để xem biểu đồ phân tích chính xác thời gian thi hành của mỗi tác vụ con tính bằng mili-giây, giúp dễ dàng khoanh vùng các truy vấn SQL chậm hoặc các logic xử lý đang ngốn nhiều tài nguyên.

![AWS X-Ray Service Map](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/5.13.2-xray-service-map.png)

***

**Bước tiếp theo:** Toàn bộ kiến trúc lõi của dự án Smart Media Analytics đã được xây dựng và giám sát thành công. [**Chương 5.14: Trải nghiệm Hệ thống (System Walkthrough)**](../../5.14-User-guide/) sẽ trình diễn cách người dùng cuối tương tác với giao diện, tải lên video và trải nghiệm tính năng tìm kiếm theo thời gian thực.
