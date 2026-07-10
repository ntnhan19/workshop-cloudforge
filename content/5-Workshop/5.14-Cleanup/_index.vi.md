---
title : "Dọn dẹp Tài nguyên"
date : 2026-07-10
weight : 14
chapter : false
pre : " <b> 5.14. </b> "
---

### Tổng quan Dọn dẹp (Clean Up)

**CẢNH BÁO CHI PHÍ QUAN TRỌNG!**

Hệ thống Smart Media Analytics sử dụng nhiều dịch vụ mạnh mẽ của AWS như RDS, NAT Gateway, WAF và ECS Fargate. Mặc dù một số tài nguyên nằm trong Free-tier, việc duy trì chúng 24/7 sẽ tạo ra các hóa đơn đáng kể nếu bạn quên xóa sau khi thực hành xong.

Trong chương cuối cùng này, chúng ta sẽ thực hiện các bước phá hủy tài nguyên theo thứ tự an toàn (đảo ngược lại với lúc tạo) để đảm bảo không có bất kỳ "tài nguyên mồ côi" (orphan resources) nào bị bỏ sót, giúp bảo vệ túi tiền của bạn.

*(Hướng dẫn dọn dẹp chi tiết sẽ được cập nhật).*
