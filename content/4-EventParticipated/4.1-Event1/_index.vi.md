---
title: "Event 1"
date: 2026-05-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Bài thu hoạch “GenAI-powered App-DB Modernization workshop”

### Mục Đích Của Sự Kiện

- Chia sẻ các phương pháp tiếp cận học tập (study approaches), khung tư duy (learning frameworks), và sự thay đổi tư duy (mindset shifts) trong quá trình hấp thụ và ứng dụng kiến thức công nghệ.
- Đi sâu vào Nghệ thuật giao tiếp với AI (Prompt Engineering) và cách tối ưu hóa chất lượng đầu ra của các mô hình LLM.
- Chia sẻ best practices trong thiết kế ứng dụng hiện đại, chuyển đổi kiến trúc và sử dụng các dịch vụ Cloud/AI.
- Xây dựng văn hóa học tập liên tục, nơi mọi người không ngại thể hiện điểm yếu (vulnerable) để cùng nhau phát triển bền vững.

### Danh Sách Diễn Giả

- **Huỳnh Hoàng Long** - Cloud Engineer, eCloudvalley Vietnam
- **Nguyễn Thịnh** - DevOps Engineer Intern, STYL Solution Pte.Ltd
- **Nguyen Khang** - Cloud Solution Architect, Cloud Kinetics Vietnam
- **Nguyen Phuong Thao** - Cloud Engineer Intern, Vietnam International Bank (VIB)

### Nội Dung Nổi Bật

#### Session 1: Huỳnh Hoàng Long - Chuyển Đổi Sang Kiến Trúc Ứng Dụng Mới
![Session 1 - Long](https://placeholder-image-url.com/session1-long.jpg)
- **Đưa ra các ảnh hưởng tiêu cực của kiến trúc ứng dụng cũ**:
  - Thời gian release sản phẩm lâu → Mất doanh thu/bỏ lỡ cơ hội
  - Hoạt động kém hiệu quả → Mất năng suất, tốn kém chi phí
  - Không tuân thủ các quy định về bảo mật → Mất an ninh, uy tín
- **Chuyển đổi sang kiến trúc ứng dụng mới - Microservice Architecture**:
  - Chuyển đổi thành hệ thống modular – từng chức năng là một **dịch vụ độc lập** giao tiếp với nhau qua **sự kiện** với 3 trụ cột cốt lõi:
    - **Queue Management**: Xử lý tác vụ bất đồng bộ
    - **Caching Strategy:** Tối ưu performance
    - **Message Handling:** Giao tiếp linh hoạt giữa services

#### Session 2: Nguyễn Thịnh - Domain-Driven Design & Event-Driven Architecture
![Session 2 - Thịnh](https://placeholder-image-url.com/session2-thinh.jpg)
- **Domain-Driven Design (DDD)**:
  - **Phương pháp 4 bước**: Xác định domain events → sắp xếp timeline → identify actors → xác định bounded contexts
  - **Case study bookstore**: Minh họa cách áp dụng DDD thực tế
  - **Context mapping**: 7 patterns tích hợp bounded contexts
- **Event-Driven Architecture**:
  - **3 patterns tích hợp**: Publish/Subscribe, Point-to-point, Streaming
  - **Lợi ích**: Loose coupling, scalability, resilience
  - **So sánh sync vs async**: Hiểu rõ trade-offs (sự đánh đổi)

#### Session 3: Nguyen Khang - Compute Evolution
![Session 3 - Khang](https://placeholder-image-url.com/session3-khang.jpg)
- **Shared Responsibility Model**: Từ EC2 → ECS → Fargate → Lambda
- **Serverless benefits**: No server management, auto-scaling, pay-for-value
- **Functions vs Containers**: Criteria lựa chọn phù hợp

#### Session 4: Nguyen Phuong Thao - Amazon Q Developer
![Session 4 - Thảo](https://placeholder-image-url.com/session4-thao.jpg)
- **SDLC automation**: Từ planning đến maintenance
- **Code transformation**: Java upgrade, .NET modernization
- **AWS Transform agents**: VMware, Mainframe, .NET migration

### Những Gì Học Được

#### Tư Duy Thiết Kế

- **Business-first approach**: Luôn bắt đầu từ business domain, không phải technology
- **Ubiquitous language**: Importance của common vocabulary giữa business và tech teams
- **Bounded contexts**: Cách identify và manage complexity trong large systems

#### Kiến Trúc Kỹ Thuật

- **Event storming technique**: Phương pháp thực tế để mô hình hóa quy trình kinh doanh
- Sử dụng **Event-driven communication** thay vì synchronous calls
- **Integration patterns**: Hiểu khi nào dùng sync, async, pub/sub, streaming
- **Compute spectrum**: Criteria chọn từ VM → containers → serverless

#### Chiến Lược Hiện Đại Hóa

- **Phased approach**: Không rush, phải có roadmap rõ ràng
- **7Rs framework**: Nhiều con đường khác nhau tùy thuộc vào đặc điểm của mỗi ứng dụng
- **ROI measurement**: Cost reduction + business agility

### Ứng Dụng Vào Công Việc

- **Áp dụng DDD** cho project hiện tại: Event storming sessions với business team
- **Refactor microservices**: Sử dụng bounded contexts để identify service boundaries
- **Implement event-driven patterns**: Thay thế một số sync calls bằng async messaging
- **Serverless adoption**: Pilot AWS Lambda cho một số use cases phù hợp
- **Try Amazon Q Developer**: Integrate vào development workflow để boost productivity

### Trải nghiệm trong event

Tham gia workshop **“GenAI-powered App-DB Modernization”** là một trải nghiệm rất bổ ích, giúp tôi có cái nhìn toàn diện về cách hiện đại hóa ứng dụng và cơ sở dữ liệu bằng các phương pháp và công cụ hiện đại. Một số trải nghiệm nổi bật:

#### Học hỏi từ các diễn giả có chuyên môn cao
- Các diễn giả đến từ AWS và các tổ chức công nghệ lớn đã chia sẻ **best practices** trong thiết kế ứng dụng hiện đại.
- Qua các case study thực tế, tôi hiểu rõ hơn cách áp dụng **Domain-Driven Design (DDD)** và **Event-Driven Architecture** vào các project lớn.

#### Trải nghiệm kỹ thuật thực tế
- Tham gia các phiên trình bày về **event storming** giúp tôi hình dung cách **mô hình hóa quy trình kinh doanh** thành các domain events.
- Học cách **phân tách microservices** và xác định **bounded contexts** để quản lý sự phức tạp của hệ thống lớn.
- Hiểu rõ trade-offs giữa **synchronous và asynchronous communication** cũng như các pattern tích hợp như **pub/sub, point-to-point, streaming**.

#### Ứng dụng công cụ hiện đại
- Trực tiếp tìm hiểu về **Amazon Q Developer**, công cụ AI hỗ trợ SDLC từ lập kế hoạch đến maintenance.
- Học cách **tự động hóa code transformation** và pilot serverless với **AWS Lambda**, từ đó nâng cao năng suất phát triển.

#### Kết nối và trao đổi
- Workshop tạo cơ hội trao đổi trực tiếp với các chuyên gia, đồng nghiệp và team business, giúp **nâng cao ngôn ngữ chung (ubiquitous language)** giữa business và tech.
- Qua các ví dụ thực tế, tôi nhận ra tầm quan trọng của **business-first approach**, luôn bắt đầu từ nhu cầu kinh doanh thay vì chỉ tập trung vào công nghệ.

#### Bài học rút ra
- Việc áp dụng DDD và event-driven patterns giúp giảm **coupling**, tăng **scalability** và **resilience** cho hệ thống.
- Chiến lược hiện đại hóa cần **phased approach** và đo lường **ROI**, không nên vội vàng chuyển đổi toàn bộ hệ thống.
- Các công cụ AI như Amazon Q Developer có thể **boost productivity** nếu được tích hợp vào workflow phát triển hiện tại.

#### Một số hình ảnh khi tham gia sự kiện
* Thêm các hình ảnh của các bạn tại đây
> Tổng thể, sự kiện không chỉ cung cấp kiến thức kỹ thuật mà còn giúp tôi thay đổi cách tư duy về thiết kế ứng dụng, hiện đại hóa hệ thống và phối hợp hiệu quả hơn giữa các team.
