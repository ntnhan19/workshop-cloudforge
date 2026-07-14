---
title : "Tạo Vector Truy vấn"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.10.1. </b> "
---

Để cơ sở dữ liệu Vector có thể so sánh và tìm kiếm, câu truy vấn (Query) bằng ngôn ngữ tự nhiên của người dùng bắt buộc phải được chuyển đổi sang cùng một không gian toán học với dữ liệu lưu trữ. Bước đầu tiên trong chu trình Semantic Search là quá trình tạo Vector Truy vấn (Query Embeddings).

#### Kiến trúc luồng xử lý truy vấn
Khi người dùng nhập một câu lệnh tìm kiếm trên Frontend, luồng xử lý dữ liệu được thiết kế như sau:

1. Frontend gửi một HTTP POST request chứa chuỗi tìm kiếm (Ví dụ: `{"query": "cách bảo mật dữ liệu đám mây"}`) tới API Gateway.
2. API Gateway định tuyến yêu cầu qua VPC Link tới Application Load Balancer (ALB).
3. ALB phân phối yêu cầu tới một API Pod (chạy trên ECS Fargate) phụ trách xử lý tác vụ tìm kiếm (Search Service).
4. API Pod tiếp nhận chuỗi tìm kiếm và khởi tạo lệnh gọi tới dịch vụ Amazon Bedrock.

![Query Flow](/images/5-Workshop/5.10-Semantic-search/5.10.1-generate-query-vector/5.10.1-query-flow.png)

#### Thiết lập tương tác Amazon Bedrock
Nhóm dự án cấu hình API Pod sử dụng AWS SDK (`boto3`) để tương tác với Amazon Bedrock. Tương tự như quá trình Ingestion, mô hình được cấu hình sử dụng là **Amazon Titan Text Embeddings v2** (`amazon.titan-embed-text-v2:0`).

Việc đảm bảo tính nhất quán của mô hình là yếu tố kỹ thuật then chốt: **Vector của câu truy vấn phải được sinh ra từ cùng một mô hình (Model) và cùng số lượng chiều (Dimensions) với Vector của dữ liệu gốc.** Sự sai lệch về mô hình sẽ dẫn đến việc hai Vector nằm ở hai không gian toán học hoàn toàn khác nhau, khiến thuật toán so sánh trở nên vô nghĩa.

{{% notice tip %}}
**Cấu hình kỹ thuật:** Trong Workshop này, mô hình Titan v2 được thiết lập phân tách chuỗi văn bản đầu vào với số chiều đầu ra cố định là **1536 dimensions** (đảm bảo khớp 100% với kiểu dữ liệu `vector(1536)` trong cấu trúc bảng PostgreSQL).
{{% /notice %}}

Dưới đây là đoạn mã xử lý Core Logic được triển khai trên Search Service để tương tác với Bedrock API:

```python
import json
import boto3

def generate_query_vector(query_text: str):
    # Khởi tạo Bedrock Runtime Client trong mạng nội bộ VPC
    bedrock_client = boto3.client(service_name="bedrock-runtime", region_name="ap-southeast-1")
    
    # Cấu hình Body theo đúng đặc tả của Amazon Titan Text Embeddings v2
    body = json.dumps({
        "inputText": query_text,
        "dimensions": 1536,
        "normalize": True
    })
    
    # Khởi chạy ra lệnh cho mô hình ML
    response = bedrock_client.invoke_model(
        body=body,
        modelId="amazon.titan-embed-text-v2:0",
        accept="application/json",
        contentType="application/json"
    )
    
    response_body = json.loads(response.get("body").read())
    
    # Trả về mảng số thực float đại diện cho Vector truy vấn
    return response_body.get("embedding")
```

#### Phân quyền IAM (IAM Roles)
Trong giai đoạn này, ECS Task Role của dịch vụ Search Service (Backend API) được bổ sung chính sách (Policy) cấp quyền:
- `bedrock:InvokeModel`: Cho phép API Pod gọi hàm khởi tạo Embeddings của mô hình Titan.

*Ghi chú kỹ thuật: Nhờ kiến trúc mạng nội bộ VPC, dữ liệu truy vấn của người dùng được gửi thẳng từ Backend API tới Amazon Bedrock Endpoint thông qua VPC Endpoint mà không truyền tải qua Internet công cộng, đáp ứng tiêu chuẩn an toàn thông tin.*

***

**Bước tiếp theo:** Sau khi API Pod nhận được mảng Vector trả về từ Amazon Bedrock, chuỗi số liệu toán học này sẽ được sử dụng làm tham số đầu vào (Input Parameter) cho lệnh truy vấn cơ sở dữ liệu PostgreSQL ở bài 5.10.2.
