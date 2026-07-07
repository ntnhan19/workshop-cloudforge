---
title: "Event 2"
date: 2026-05-23
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch: AWS Vietnam Community Day & First Cloud Journey 2026

### Mục tiêu sự kiện
- Chia sẻ các kiến thức chuyên sâu về ứng dụng AI tạo sinh (GenAI), cách tinh chỉnh và xây dựng hệ thống tác tử (Multi-Agent).
- Cung cấp các phương pháp tối ưu hóa trải nghiệm sử dụng AI thông qua việc xây dựng ngữ cảnh (Context) và bộ nhớ (Memory).
- Hướng dẫn xây dựng kiến trúc nền tảng đám mây an toàn, tối ưu chi phí và hiệu suất với Amazon CloudFront.
- Truyền cảm hứng từ hành trình phát triển sản phẩm thực tế trong các cuộc thi Hackathon.

### Danh sách Diễn giả (Speakers)
- **Tinh Truong** – Platform Engineer, GoTymeX
- **Anh Pham** – Cloud Consultant (G-AsiaPacific Vietnam) & AWS Community Builder
- **Thuan Nguyen** – DevOps Engineer, First Cloud AI Journey
- **Thao Nguyen & Mai Nguyen & Uyen Le** – GenAI Engineer, VIB
- **Duc Dao** – Solution Architect, Cloud Kinetics
- **Vy Lam** – Senior Business Systems Analyst, VPBank

### Những Điểm Nhấn Quan Trọng (Key Highlights)

#### 1. Tầm quan trọng của Ngữ cảnh trong AI (Tinh Truong)
- **Lỗi sai phổ biến:** AI đưa ra câu trả lời kém thường do đầu vào (ngữ cảnh) yếu chứ không phải do mô hình. Việc nhồi nhét quá nhiều thông tin rác (Internet Puller) sẽ làm giảm độ chính xác và tốn kém token.
- **Công thức xây dựng Context:** Một ngữ cảnh tốt cần có đủ 4 yếu tố: Mục tiêu (Goal), Thông tin liên quan (Relevant info), Các ràng buộc (Constraints), và Tiêu chí thành công (Success criteria).
- **Sự tiến hóa của AI:** Cách chúng ta tương tác với AI đang dịch chuyển từ việc chỉ dùng Prompt đơn thuần -> Xây dựng Context -> Hình thành Memory (Trí não AI thứ hai) để hỗ trợ cá nhân hóa dài hạn.

#### 2. Trợ lý AI thân thiện với Amazon Q (Anh Pham)
- **Q Chat Agent:** Trợ lý AI giúp khám phá dữ liệu, phân tích insight, tự động hóa công việc cho PM như tóm tắt biên bản họp (MoM), gửi email và lên lịch.
- **Q Flows:** Tạo các luồng làm việc thông minh bằng ngôn ngữ tự nhiên — không cần viết code.
- **Q Spaces & Sight:** Không gian làm việc chung biến insight cá nhân thành kiến thức nhóm, đồng thời xây dựng dashboard và báo cáo từ dữ liệu thô bằng ngôn ngữ tự nhiên.

#### 3. Tối ưu kiến trúc nền tảng với Amazon CloudFront (Thuan Nguyen)
- **Vấn đề chi phí và bảo mật:** Mô hình pay-as-you-go đôi khi mang lại hóa đơn CDN ngoài dự kiến. Cần các gói định tuyến và bảo vệ để giảm rủi ro tài chính.
- **Origin Cloaking:** Sử dụng Origin Access Control (OAC) để khóa quyền truy cập trực tiếp vào S3, Lambda, đảm bảo luồng traffic bắt buộc phải đi qua CloudFront.
- **Tính khả dụng cao:** CloudFront hỗ trợ cấu hình Failover (chuyển đổi dự phòng) khi origin chính gặp lỗi, duy trì trải nghiệm người dùng không bị gián đoạn và tăng tốc độ bằng tính năng HTTP Compression (có thể giảm tới 81% thời gian tải).

#### 4. 36 giờ với LotusHacks – Xây dựng UTMorpho từ Ý tưởng đến Thực tế (Thao Nguyen & Mai Nguyen & Uyen Le)
- **Từ số 0 đến Ý tưởng:** Hành trình brainstorm, xác định vấn đề và định hình sản phẩm UTMorpho.
- **Phát triển dưới áp lực:** Quá trình chạy nước rút 36 giờ đối mặt với nhiều thách thức, thất bại và những bước ngoặt.
- **Bài học đắt giá:** Những ý tưởng tốt nhất xuất phát từ sự bực bội trong thực tế. Trong làm việc nhóm, sự đồng bộ quan trọng hơn tất cả và cần tránh "scope creep" (ôm đồm quá nhiều tính năng).

#### 5. Tính không tất định của các thiết lập LLM (Duc Dao)
- **Thực tế về Temperature = 0:** Về mặt lý thuyết, Temp=0 sẽ cho ra kết quả mang tính xác định (đầu ra giống nhau 100%). Tuy nhiên thực tế, không có mô hình LLM nào đảm bảo tính nhất quán trên mọi tác vụ.
- **Nguyên nhân kỹ thuật:** Kiến trúc GPU thực hiện các phép toán dấu phẩy động (floating-point) không có tính kết hợp, và các nhà cung cấp API thường gộp chung (batching) yêu cầu của nhiều người dùng để tối ưu chi phí, dẫn đến những sai số nhỏ làm thay đổi kết quả đầu ra.
- **Chiến lược giảm thiểu:** Xây dựng hệ thống với tư duy "chấp nhận sự biến thiên" bằng cách dùng majority voting (chạy nhiều lần và lấy kết quả phổ biến nhất), ép kiểu đầu ra (JSON, YAML) và sử dụng temp=0.1 để tránh việc mô hình bị kẹt trong vòng lặp lặp đi lặp lại.

#### 6. Hệ thống Multi-Agent cấp Doanh nghiệp - Bài toán chấm điểm Startup (Vy Lam)
- **Hạn chế của Single Agent:** Khi dùng một AI Agent duy nhất để duyệt tín dụng, hệ thống dễ gặp vấn đề về giới hạn ngữ cảnh, pha loãng chuyên môn, thiếu cơ chế kiểm tra chéo và trở thành điểm nghẽn duy nhất.
- **Sức mạnh của Multi-Agent:** Kiến trúc mô phỏng một hội đồng tín dụng ảo gồm nhiều Agent chuyên biệt (Phân tích tài chính, Thị trường, Đội ngũ, Rủi ro, và Tuân thủ) giúp đưa ra quyết định chính xác hơn.
- **Tư duy Enterprise-Grade:** Để đưa từ POC lên Production, hệ thống cần được trang bị Guardrails (bảo vệ đầu vào/đầu ra), bảo mật (MFA, mã hóa), Data Governance (xử lý PII) và tuân thủ các khung quy định như SOC 2, GDPR.

### Tổng kết (Key Takeaways)

#### Về Tư duy AI & Phát triển ứng dụng
- **Chất lượng hơn Số lượng:** Cung cấp Context chất lượng cho AI thay vì một đống dữ liệu rác.
- **Cơ chế Multi-Agent:** Các bài toán doanh nghiệp phức tạp nên được chia nhỏ và xử lý chéo bằng nhiều AI Agents thay vì phụ thuộc vào một tác tử duy nhất.
- **Hiểu bản chất công nghệ:** Đừng tin tưởng mù quyáng vào cài đặt tính tất định (temperature=0) của LLM cho các ứng dụng có mức rủi ro cao.

#### Kiến trúc Kỹ thuật
- Bảo mật hệ thống từ cấp độ mạng (Edge) đến máy chủ gốc (Origin) bằng CloudFront và Guardrails.
- Chuyển đổi từ các quy trình thủ công sang việc tích hợp AI Agents và các hệ thống "Second Brain" với khả năng truy xuất bộ nhớ.

### Kế hoạch Ứng dụng vào Công việc (Applying to Work)
- **Tối ưu hóa các hệ thống LLM nội bộ:** Thay đổi tư duy khi viết Prompt, sử dụng framework 4 bước (Goal, Info, Constraints, Criteria) và thử nghiệm temp=0.1 nếu gặp lỗi lặp phản hồi.
- **Thử nghiệm kiến trúc Multi-Agent:** Xây dựng các luồng đánh giá tự động thay vì dùng 1 single prompt dài dòng.
- **Bảo mật CDN:** Đánh giá lại quyền truy cập (bucket policies) trên AWS S3, triển khai OAC thay cho OAI cũ để tăng cường bảo mật cho nội dung tĩnh.

### Trải nghiệm sự kiện (Event Experience)
Tham dự AWS Vietnam Community Day 2026 mang lại cho tôi góc nhìn vô cùng thực tế về những khó khăn khi triển khai AI vào môi trường Doanh nghiệp. Từ những vấn đề kỹ thuật sâu sắc như sai số dấu phẩy động của GPU làm sai lệch LLM, cho đến cách xây dựng một kiến trúc nhiều lớp bảo mật với Multi-Agent và CloudFront.

Sự kiện không chỉ truyền tải kiến thức công nghệ mà còn nhắc nhở tôi về tư duy làm sản phẩm: Luôn bắt đầu từ "nỗi đau" thực tế, giữ vững sự tập trung, và thiết lập những lớp bảo vệ (guardrails) trước khi mở rộng quy mô. Điểm nhấn lớn nhất là nhận ra ranh giới giữa việc "biết dùng AI" và "làm cho AI thực sự hoạt động hiệu quả" nằm ở chính khả năng cung cấp và quản lý ngữ cảnh (Context) của chúng ta.

### Một số hình ảnh tại sự kiện

![Event Photo 1](../../images/4-EventParticipated/4.2-Event2/event_23_05%281%29.jpg)

![Event Photo 2](../../images/4-EventParticipated/4.2-Event2/event_23_05%282%29.jpg)
