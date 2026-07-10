---
title : "Thực thi Vector Search với pgvector"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.10.2. </b> "
---

Sau khi đã chuyển đổi thành công câu truy vấn của người dùng thành một mảng Vector (Query Embeddings) thông qua Amazon Bedrock ở bài 5.10.1, bước tiếp theo trong kiến trúc là thực thi truy vấn tìm kiếm ngữ nghĩa trực tiếp trên Cơ sở dữ liệu Amazon RDS PostgreSQL.

#### Cơ chế đối chiếu Vector (Vector Similarity)
Dự án sử dụng extension `pgvector` đã được kích hoạt trên PostgreSQL để hỗ trợ lưu trữ và truy vấn kiểu dữ liệu `vector(1536)`.
Để đo lường mức độ tương đồng giữa câu truy vấn và các đoạn Transcript, kiến trúc sử dụng toán tử **Cosine Distance** (Ký hiệu trong pgvector là `<=>`). Khoảng cách Cosine càng nhỏ, hai đoạn văn bản càng có ý nghĩa ngữ cảnh giống nhau.

#### Cấu trúc truy vấn SQL (SQL Query Structure)
Nhóm dự án thiết lập truy vấn SQL nâng cao trên Backend API để tính toán và trích xuất các kết quả (Top-K) có độ tương đồng cao nhất.

Dưới đây là cấu trúc lệnh SQL thô (Raw SQL) cốt lõi:
```sql
SELECT 
    video_id, 
    transcript_text, 
    start_time, 
    end_time,
    (embedding <=> '[0.013, -0.024, ...]') AS cosine_distance
FROM 
    video_transcripts
ORDER BY 
    cosine_distance ASC
LIMIT 5;
```
*Ghi chú kỹ thuật: Mảng `[0.013, -0.024, ...]` là giá trị Query Embeddings được trả về từ Amazon Bedrock. Lệnh `ORDER BY cosine_distance ASC` đảm bảo các kết quả gần gũi nhất về mặt ngữ nghĩa (khoảng cách nhỏ nhất) sẽ được xếp lên đầu.*

#### Triển khai Logic Truy vấn trên Backend
Tại Search Service (chạy trên ECS Fargate), đoạn lệnh SQL trên được đóng gói và thực thi thông qua thư viện kết nối cơ sở dữ liệu. Để đảm bảo chuẩn bảo mật, thông tin xác thực Database được truy xuất động từ AWS Secrets Manager (đã thiết lập ở Chương 5.5.2).

Mã nguồn mô phỏng luồng thực thi (`psycopg2` & `boto3`):

```python
import psycopg2
import boto3
import json
import os

def get_db_credentials():
    # Lấy thông tin đăng nhập DB an toàn từ Secrets Manager
    client = boto3.client('secretsmanager', region_name='ap-southeast-1')
    secret_name = os.environ.get('DB_SECRET_NAME', 'cloudforge-db-credentials')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

def search_similar_transcripts(query_vector: list, limit: int = 5):
    creds = get_db_credentials()
    
    # Khởi tạo kết nối tới RDS PostgreSQL trong Private Subnet
    conn = psycopg2.connect(
        host=creds['host'],
        database=creds['dbname'],
        user=creds['username'],
        password=creds['password']
    )
    cursor = conn.cursor()
    
    # Chuyển đổi list vector thành định dạng string chuẩn của pgvector
    vector_str = '[' + ','.join(map(str, query_vector)) + ']'
    
    # Thực thi truy vấn tính toán Cosine Distance
    query = """
        SELECT video_id, transcript_text, start_time 
        FROM video_transcripts 
        ORDER BY embedding <=> %s ASC 
        LIMIT %s;
    """
    cursor.execute(query, (vector_str, limit))
    results = cursor.fetchall()
    
    cursor.close()
    conn.close()
    
    return results
```

{{% notice tip %}}
**Tối ưu hóa hiệu suất (Performance Optimization):** Đối với các tập dữ liệu có quy mô lớn, việc quét toàn bộ bảng (Exact Nearest Neighbor Search) sẽ làm suy giảm hiệu suất nghiêm trọng. Nhóm dự án đã thiết lập chỉ mục **HNSW** (Hierarchical Navigable Small World) trên cột `embedding` ở Chương 5.4, giúp tăng tốc độ tìm kiếm (Approximate Nearest Neighbor) lên gấp nhiều lần với độ chính xác xấp xỉ hoàn hảo, đảm bảo độ trễ truy vấn (Latency) luôn ở mức thấp.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống Backend và các luồng xử lý dữ liệu AI cốt lõi đã hoàn thiện toàn vẹn. Ở chương tiếp theo (**Chương 5.11: Frontend Deployment**), nhóm dự án sẽ triển khai giao diện người dùng tĩnh lên Amazon S3 và CloudFront để tương tác trực tiếp với kiến trúc Cloud mà chúng ta vừa xây dựng.
