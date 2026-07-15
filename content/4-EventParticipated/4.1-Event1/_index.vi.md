---
title: "Event 1"
date: 2026-05-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Bài thu hoạch Sự kiện First Cloud Journey

### Mục Đích Của Sự Kiện

- Chia sẻ các phương pháp tiếp cận học tập (study approaches), khung tư duy (learning frameworks), và sự thay đổi tư duy (mindset shifts) trong quá trình hấp thụ và ứng dụng kiến thức công nghệ.
- Đi sâu vào Nghệ thuật giao tiếp với AI (Prompt Engineering) và cách tối ưu hóa chất lượng đầu ra của các mô hình LLM.
- Khám phá sức mạnh của các AI Agent (như Developer Agent) trong việc tự động hóa và duy trì kỷ luật phát triển phần mềm.
- Xây dựng văn hóa học tập liên tục, nơi mọi người không ngại thể hiện điểm yếu (vulnerable) để cùng nhau phát triển bền vững.

### Danh Sách Diễn Giả

- **Huỳnh Hoàng Long** - Cloud Engineer, eCloudvalley Vietnam
- **Nguyễn Thịnh** - DevOps Engineer Intern, STYL Solution Pte.Ltd
- **Nguyen Khang** - Cloud Solution Architect, Cloud Kinetics Vietnam
- **Nguyen Phuong Thao** - Cloud Engineer Intern, Vietnam International Bank (VIB)

### Nội Dung Nổi Bật

#### Session 1: Huỳnh Hoàng Long - Steal the Formula Used by Social Media
![Session 1](/images/4-EventParticipated/4.1-Event1/session1.jpg)
- **Use the fear of loss**: "Humans fear losing more than they enjoy gaining" (Con người sợ mất mát hơn là thích được nhận thêm).
- **Học hỏi từ mạng xã hội**: Khám phá bí quyết và công thức thành công (Formula) được các nền tảng mạng xã hội sử dụng để thu hút và giữ chân người dùng.
- **Áp dụng vào việc học**: Sử dụng các cơ chế tâm lý của mạng xã hội để áp dụng vào việc học, cụ thể là lập chuỗi (streak) và tự thưởng cho bản thân nhằm duy trì động lực và kỷ luật liên tục.

#### Session 2: Nguyễn Tuấn Thịnh - Nghệ thuật giao tiếp với AI & Automated Prompt Engineering
![Session 2](/images/4-EventParticipated/4.1-Event1/session2.jpg)
- **Tầm quan trọng của Prompt**: Lệnh chung chung sẽ sinh ra kết quả chung chung, gây lãng phí token và kết quả không nhất quán. Một câu lệnh chuẩn cần có: Vai trò (Role), Hướng dẫn (Instruction), Ngữ cảnh (Context), Dữ liệu đầu vào (Input), Định dạng đầu ra (Output), Ví dụ (Examples) và Các ràng buộc (Constraints).
- **Nguyên tắc "Mastering LLM"**: Cần rõ ràng, dùng ngôn ngữ chỉ thị và dấu phân cách. Đặc biệt, nên yêu cầu AI NÊN LÀM GÌ (DOs) thay vì KHÔNG NÊN LÀM GÌ (DON'Ts), không bắt AI tự làm toán, và cho phép mô hình trả lời "Tôi không biết".
- **Kỹ thuật tối ưu Nâng cao**: Áp dụng Chain-of-Thought (CoT), Self-Consistency, Tree-of-Thoughts (ToT) và RAG để AI xử lý logic tốt hơn.
- **Kiến trúc Proptimizer**: Tiện ích mở rộng trình duyệt giúp tự động hóa Prompt với chi phí nền tảng $0 (không tính phí API của AI). Hệ thống chạy hoàn toàn trên Serverless với S3/CloudFront (Frontend), Cognito (Xác thực), API Gateway/Lambda (Backend), DynamoDB (Lưu trữ tốc độ cao) và Amazon Bedrock để gọi các mô hình AI (Claude, GPT).

#### Session 3: Nguyen Khang - Growth Mindset
![Session 3](/images/4-EventParticipated/4.1-Event1/session3.jpg)
- **Growth Mindset – Ask "Why?"**: Luôn đặt câu hỏi "Tại sao?" – sự tò mò chính là tài sản lớn nhất của bạn.
- **Question Everything**: Không chấp nhận mọi thứ như một lẽ dĩ nhiên, luôn tò mò và tìm hiểu sâu bản chất vấn đề.
- **Embrace Mistakes**: Ở giai đoạn đầu của việc học, sai lầm là những bài học giá trị, không phải là sự thất bại.
- **Stay Hungry**: Khoảnh khắc bạn ngừng học hỏi là lúc bạn bị tụt lại phía sau. Luôn sẵn sàng đối mặt với những thách thức mới.

#### Session 4: Nguyen Phuong Thao - Developer Agent
![Session 4](/images/4-EventParticipated/4.1-Event1/session4.jpg)
- **Developer Agent: Kỷ luật Code từng Story riêng biệt**: Ứng dụng AI Agent vào quy trình phát triển phần mềm một cách có kỷ luật.
- **Quy trình 4 bước của Dev Agent**:
  1. **Khởi động Dev Agent**: Tự động tìm các Story có trạng thái Approved.
  2. **Phân tách Subtasks**: Bẻ gãy Story thành các tác vụ (task) cực kỳ chi tiết.
  3. **Thực thi Code**: Code chính xác, không để lại khoảng trống mơ hồ, không đoán mò.
  4. **Hoàn tất Code**: Đổi trạng thái Story thành "Ready for Review" sau khi hoàn thiện.

### Những Gì Học Được

#### Tư Duy & Phương Pháp Học Tập
- **Growth Mindset**: Luôn giữ sự tò mò, đặt câu hỏi "Tại sao" và xem sai lầm là cơ hội để học hỏi thay vì chán nản.
- **Động lực học tập**: Tận dụng tâm lý "sợ mất mát" (fear of loss) và cơ chế phần thưởng (reward) để tạo ra các chuỗi học tập (streak) liên tục, tương tự cách mạng xã hội giữ chân người dùng.

#### Tối Ưu Hóa Kỹ Năng Kỹ Thuật & AI
- **Kỹ năng Prompting**: Việc viết câu lệnh rõ ràng, cung cấp đầy đủ Role, Instruction, Context sẽ quyết định chất lượng phản hồi từ AI. Áp dụng tư duy chỉ định rõ AI cần làm gì (DOs) thay vì cấm đoán.
- **Automation với AI Agent**: Tận dụng AI không chỉ để chat mà còn để tạo ra các Agent (như Developer Agent) giúp xử lý các quy trình phát triển phần mềm kỷ luật (từ phân tách task đến khi hoàn thành code).

### Ứng Dụng Vào Công Việc & Học Tập

- **Thiết lập chuỗi học tập (Streak)**: Áp dụng cơ chế fear of loss để duy trì việc học công nghệ mới (Cloud, AI) mỗi ngày mà không bị ngắt quãng.
- **Tối ưu hóa Prompts**: Cấu trúc lại cách đặt câu hỏi cho AI (ChatGPT, Claude, Amazon Q) trong công việc hàng ngày theo chuẩn framework đã học để có câu trả lời chính xác nhất.
- **Khám phá Developer Agent**: Tìm hiểu và nghiên cứu cách tích hợp các Agent vào quy trình xử lý code để tăng năng suất và giảm thiểu sai sót.

### Trải nghiệm trong event

Tham gia sự kiện First Cloud Journey lần này là một trải nghiệm mở mang tầm mắt, không chỉ về mặt công nghệ mà còn về tư duy phát triển bản thân. Một số trải nghiệm nổi bật:

#### Học hỏi từ các diễn giả nhiệt huyết
- Các diễn giả mang đến năng lượng tươi mới, chia sẻ những góc nhìn cực kỳ thực tế từ kinh nghiệm cá nhân, từ cách vượt qua sự trì hoãn đến cách làm chủ công nghệ AI.

#### Thay đổi tư duy (Mindset Shift)
- Tôi nhận ra rằng việc học không cần phải gò ép mà có thể "hack" bằng các công cụ tâm lý học (như streak, phần thưởng) mượn từ các nền tảng mạng xã hội.
- Hiểu được giá trị to lớn của "Growth Mindset" – không ngại sai sót, bởi mỗi lỗi sai đều là một bài học đắt giá ở giai đoạn đầu.

#### Một số hình ảnh khi tham gia sự kiện
![Event Photo 1](/images/4-EventParticipated/4.1-Event1/event_09_05(1).jpg)
![Event Photo 2](/images/4-EventParticipated/4.1-Event1/event_09_05(2).jpg)
> Tổng thể, sự kiện đã giúp tôi thay đổi sâu sắc cách tiếp cận việc học, cũng như cung cấp các phương pháp tối ưu hóa năng suất bằng trí tuệ nhân tạo, tạo động lực to lớn cho hành trình phát triển sắp tới.
