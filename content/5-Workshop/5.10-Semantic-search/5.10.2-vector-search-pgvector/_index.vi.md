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

Dưới đây là cấu trúc lệnh SQL tương đương đằng sau ORM:
```sql
SELECT 
    asset_id, 
    transcript_snippet, 
    timestamp_start_sec, 
    timestamp_end_sec
FROM 
    scenes
ORDER BY 
    embedding <=> '[0.013, -0.024, ...]' ASC
LIMIT 5;
```
*Ghi chú kỹ thuật: Mảng `[0.013, -0.024, ...]` là giá trị Query Embeddings được trả về từ Amazon Bedrock. Toán tử `<=>` và lệnh `ORDER BY` đảm bảo các kết quả gần gũi nhất về mặt ngữ nghĩa (khoảng cách nhỏ nhất) sẽ được xếp lên đầu.*

#### Triển khai Logic Truy vấn trên Backend
Tại Search Service (chạy trên ECS Fargate), nhóm dự án sử dụng **SQLAlchemy** và thư viện **pgvector-python** (`pgvector.sqlalchemy`) để thực thi câu truy vấn một cách bất đồng bộ và an toàn.

Mã nguồn thực thi từ file `backend/core/embeddings/pgvector_store.py`:

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from pgvector.sqlalchemy import Vector
from models.scene import Scene
from database import SessionLocal

class PGVectorStore:
    # ...
    async def search(self, query_embedding: list[float], n_results: int = 10):
        """
        Thực thi Vector Similarity Search sử dụng Cosine Distance (<=>).
        """
        async with SessionLocal() as db:
            # Query the scenes table, order by cosine distance
            stmt = select(Scene).order_by(
                Scene.embedding.cosine_distance(query_embedding)
            ).limit(n_results)
            
            result = await db.execute(stmt)
            scenes = result.scalars().all()
            
            # Map ORM objects to response dictionary...
            return scenes
```

#### Kịch bản Kiểm thử Vector Search (E2E)
Để chứng minh Backend API đã kết nối thành công với pgvector và trả về kết quả tìm kiếm ngữ nghĩa, tiến hành thực thi một truy vấn mẫu thông qua cURL (hoặc PowerShell).

Tại màn hình Terminal, chạy lệnh sau (thay thế `[ALB-DNS]` bằng DNS thật của hệ thống):
```bash
curl -X POST http://[ALB-DNS]/api/v1/search \
  -H "Content-Type: application/json" \
  -d '{"query": "cloud native", "top_k": 3}'
```

Kết quả trả về ở thời điểm hiện tại (khi chưa có video nào được xử lý xong) sẽ là một mảng rỗng. Điều này hoàn toàn bình thường, và nó chứng minh rằng API Search đã kết nối thành công tới Database để thực hiện truy vấn Vector mà không gặp lỗi:
```json
{
    "query":  "cloud native",
    "total_results":  0,
    "results":  [

                ],
    "processing_time_ms":  12.32
}
```

![Search API Test](/images/5-Workshop/5.10-Semantic-search/5.10.2-vector-search-pgvector/search_api_test.png)

{{% notice tip %}}
**Tối ưu hóa Hiệu suất (Performance Optimization):** Đối với các tập dữ liệu có quy mô lớn, việc quét toàn bộ bảng (Exact Nearest Neighbor Search) sẽ làm suy giảm hiệu suất nghiêm trọng. Nhóm dự án đã thiết lập chỉ mục **HNSW** (Hierarchical Navigable Small World) trên cột `embedding` ở Chương 5.4, giúp tăng tốc độ tìm kiếm (Approximate Nearest Neighbor) lên gấp nhiều lần với độ chính xác xấp xỉ hoàn hảo, đảm bảo độ trễ truy vấn (Latency) luôn ở mức thấp.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống Backend và các luồng xử lý dữ liệu AI cốt lõi đã hoàn thiện toàn vẹn. Ở chương tiếp theo ([**Chương 5.11: Frontend Deployment**](../../5.11-Frontend-deployment/)), nhóm dự án sẽ triển khai giao diện người dùng tĩnh lên Amazon S3 và CloudFront để tương tác trực tiếp với kiến trúc Cloud mà chúng ta vừa xây dựng.
