---
title: "Blog 1"
date: 2026-06-12
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Xây dựng ứng dụng tìm kiếm đầu tiên với Amazon OpenSearch Service

Trong thời đại dữ liệu bùng nổ, khả năng tìm kiếm và phân tích thông tin theo thời gian thực không còn là lợi thế mà là điều kiện sống còn của doanh nghiệp. Từ các cảm biến IoT công nghiệp truyền hàng triệu chỉ số mỗi giây, đến nền tảng thương mại điện tử cần hiển thị sản phẩm tức thì, hay các đội bảo mật cần phát hiện mối đe dọa trong thời gian thực và tất cả đều đòi hỏi một nền tảng tìm kiếm mạnh mẽ.

Amazon OpenSearch Service ra đời để giải quyết đúng bài toán đó.

### OpenSearch Service là gì?

Đây là dịch vụ tìm kiếm và phân tích được quản lý hoàn toàn bởi AWS. Thay vì lo lắng về hạ tầng, bạn chỉ cần tập trung vào việc xây dựng ứng dụng. Phiên bản Amazon OpenSearch Serverless còn đi xa hơn khi tự động co giãn tài nguyên theo nhu cầu, không cần quản lý server.

### Kiến trúc cốt lõi cần biết

Trước khi bắt tay vào code, bạn cần nắm vài khái niệm nền tảng:

- **Document & Index:** Đơn vị dữ liệu cơ bản là document (định dạng JSON), được tổ chức trong các index tương tự như bảng trong cơ sở dữ liệu.
- **Cluster & Node:** OpenSearch hoạt động theo mô hình phân tán. Các master node quản lý cluster, data node xử lý lưu trữ và truy vấn, còn coordinator node định tuyến yêu cầu để giảm tải cho data node.
- **Shard & Replica:** Index được chia thành các shard (khuyến nghị 10–50 GB mỗi shard) và nhân bản để đảm bảo tính sẵn sàng cao.
- **Inverted Index & BM25:** Công nghệ tìm kiếm full-text được hỗ trợ bởi cấu trúc inverted index và thuật toán xếp hạng BM25.

### Kiến trúc ứng dụng mẫu

Một ứng dụng tìm kiếm hoàn chỉnh trên AWS được xây dựng theo mô hình bảo mật nhiều lớp, với các thành phần phối hợp chặt chẽ từ frontend đến cluster dữ liệu.

![Kiến trúc ứng dụng tìm kiếm OpenSearch](/images/3-BlogsPosted/3.1-Blog1/blog1.jpg)

**Luồng hoạt động chi tiết**

Khi người dùng tương tác với ứng dụng, mọi yêu cầu được xử lý theo một chuỗi bước rõ ràng:

1. **Bước 1 - Truy cập giao diện:** Người dùng mở ứng dụng qua AWS App Runner, nơi frontend được host và phục vụ tự động.
2. **Bước 2 - Xác thực:** Amazon Cognito đảm nhận việc xác minh danh tính và phân quyền, đảm bảo chỉ người dùng hợp lệ mới có thể tiếp tục.
3. **Bước 3 - Định tuyến qua API Gateway:** Mọi yêu cầu từ frontend đều đi qua API Gateway - đây là cổng vào duy nhất cho toàn bộ hệ thống. API Gateway kiểm tra token từ Cognito rồi chuyển tiếp yêu cầu vào các Lambda function bên trong VPC.
4. **Bước 4 - Xử lý logic tại Lambda:** Tùy theo loại yêu cầu, AWS Lambda thực hiện một trong hai nhiệm vụ: đánh index dữ liệu mới vào OpenSearch, hoặc thực thi truy vấn tìm kiếm trên cluster hiện có.
5. **Bước 5 - OpenSearch trong vùng bảo mật:** Toàn bộ OpenSearch cluster nằm trong private subnet của VPC, hoàn toàn không tiếp xúc trực tiếp với internet - đây là lớp bảo mật quan trọng cho dữ liệu nhạy cảm.

### Kết luận

Với kiến trúc trên, bạn có thể xây dựng một ứng dụng tìm kiếm production-ready trên AWS mà không cần quản lý hạ tầng phức tạp. Amazon OpenSearch Service mang lại:

- Tìm kiếm nhanh và có thể mở rộng theo nhu cầu thực tế của doanh nghiệp.
- Bảo mật và tuân thủ tích hợp sẵn không cần cấu hình thủ công.
- Quản lý cluster tự động: AWS lo phần nặng, bạn tập trung vào sản phẩm.
- Mô hình chi phí linh hoạt: chỉ trả tiền cho những gì thực sự dùng.

### Nguồn tham khảo & Hướng dẫn triển khai

- **Source code & Hướng dẫn triển khai chi tiết:** Có sẵn tại [GitHub Repository](https://github.com/aws-samples/sample-for-amazon-opensearch-service-tutorials-101) - bạn có thể clone về và chạy thử ngay hôm nay.
- **Bài viết gốc:** [Amazon OpenSearch Service 101: Create your first search application](https://aws.amazon.com/blogs/big-data/amazon-opensearch-service-101-create-your-first-search-application-with-opensearch/)