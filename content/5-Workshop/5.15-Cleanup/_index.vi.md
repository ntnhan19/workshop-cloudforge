---
title : "Dọn dẹp Tài nguyên"
date : 2026-07-10
weight: 15
chapter: false
pre: " <b> 5.15. </b> "
---

### Tổng quan Dọn dẹp (Clean Up)

**CẢNH BÁO CHI PHÍ QUAN TRỌNG!**

Hệ thống Smart Media Analytics sử dụng nhiều dịch vụ mạnh mẽ của AWS như RDS, NAT Gateway, WAF và ECS Fargate. Mặc dù một số tài nguyên nằm trong Free-tier, việc duy trì chúng 24/7 sẽ tạo ra các hóa đơn đáng kể nếu bạn quên xóa sau khi thực hành xong.

Trong chương cuối cùng này, tiến hành thực hiện các bước phá hủy tài nguyên theo thứ tự an toàn (đảo ngược lại với lúc tạo) để đảm bảo không có bất kỳ "tài nguyên mồ côi" (orphan resources) nào bị bỏ sót, giúp bảo vệ túi tiền của hệ thống.

#### Bước 1: Xóa trắng và Xóa S3 Buckets
AWS yêu cầu S3 bucket phải trống trước khi có thể xóa.
1. Truy cập **S3 Console**.
2. Tìm bucket lưu trữ media của hệ thống (ví dụ: `cloudforge-media-storage-...`).
3. Chọn bucket và nhấn **Empty**. Nhập `permanently delete` để xác nhận.
4. Sau khi đã làm trống, chọn lại bucket và nhấn **Delete**.

#### Bước 2: Xóa Frontend và CDN
1. Truy cập **AWS Amplify Console**.
2. Chọn ứng dụng frontend của hệ thống và nhấn **Actions > Delete app**.
3. Truy cập **CloudFront Console**.
4. Chọn distribution của hệ thống, nhấn **Disable**, và chờ trạng thái chuyển sang *Disabled* (có thể mất vài phút).
5. Khi đã disable xong, chọn distribution đó và nhấn **Delete**.

#### Bước 3: Xóa Tài nguyên Compute (ECS & ECR)
1. Truy cập **Amazon ECS Console**.
2. Vào phần **Clusters** và chọn cluster của hệ thống (ví dụ: `cloudforge-compute-cluster`).
3. Tại tab **Services**, chọn service backend của hệ thống, nhấn **Delete**, và xác nhận bằng cách gõ `delete`.
4. Khi service đã bị xóa, nhấn **Delete cluster** ở góc trên cùng bên phải.
5. Truy cập **Amazon ECR Console**.
6. Chọn repository của hệ thống (ví dụ: `cloudforge-backend`) và nhấn **Delete**.

#### Bước 4: Xóa Database (RDS & ElastiCache)
1. Truy cập **Amazon RDS Console**.
2. Chọn database instance của hệ thống (ví dụ: `cloudforge-db`).
3. Vào **Modify**, cuộn xuống và BỎ CHỌN **Enable deletion protection**, sau đó apply thay đổi ngay lập tức.
4. Chọn lại database và nhấn **Delete**. **Bỏ chọn** tùy chọn tạo final snapshot. Nhập `delete me` để xác nhận.
5. Truy cập **Amazon ElastiCache Console**.
6. Vào phần **Redis clusters**, chọn cluster của hệ thống và nhấn **Delete**.

#### Bước 5: Xóa Bảo mật và Mạng (WAF & VPC)
1. Truy cập **AWS WAF Console**.
2. Vào **Web ACLs** (đảm bảo bạn đang ở region *Global (CloudFront)*).
3. Chọn Web ACL của hệ thống, gỡ liên kết (disassociate) khỏi bất kỳ tài nguyên nào còn sót lại và nhấn **Delete**.
4. Truy cập **VPC Console**.
5. Vào phần **NAT Gateways**. Chọn NAT Gateway của hệ thống và nhấn **Delete**. *Bạn BẮT BUỘC phải đợi trạng thái chuyển thành 'Deleted' (khoảng 3-5 phút) trước khi làm bước tiếp theo.*
6. Vào phần **Elastic IPs**, chọn IP đã cấp phát, nhấn **Actions > Release Elastic IP address**.
7. Vào phần **Your VPCs**, chọn VPC của hệ thống, nhấn **Actions > Delete VPC**. Hành động này sẽ tự động dọn dẹp các subnet, route table và internet gateway liên quan.

