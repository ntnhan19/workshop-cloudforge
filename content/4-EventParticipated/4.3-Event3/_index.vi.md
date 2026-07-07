---
title: "Event 3"
date: 2026-06-27
weight: 1
chapter: false
pre: " <b> 4.3. </b> "
---

# Bài thu hoạch: FCAJ Community Day - June 2026

### Mục tiêu sự kiện
- Chia sẻ kinh nghiệm thực chiến từ giai đoạn lên ý tưởng (PoC) đến khi triển khai sản phẩm thực tế (Production) của các startup công nghệ.
- Phân tích kiến trúc tối ưu cho AI Giọng nói (Voice AI) và cách giải quyết các bài toán đặc thù của tiếng Việt.
- Giới thiệu các công cụ AI hiện đại (AWS DevOps Agent, Amazon Q) giúp tự động hóa quy trình vận hành hệ thống và quản trị nguồn nhân lực (HR).
- Cung cấp giải pháp bảo mật nâng cao khi tích hợp AI với dữ liệu nội bộ của doanh nghiệp.

### Danh sách Diễn giả (Speakers)
- **Steve Trần** – Founder & CTO, CloudThinker
- **Trung Vũ** – Founder, Revve AI
- **Kiệt Trần** – Core Team, AWS Student Builder Group HUFLIT
- **Danh Hoàng Hiếu Nghi** – AI Engineer Ambassador, AWS Study Group
- **Bảo Phan Kim & Nguyên Nguyễn** – Cloud Engineers, Cloud Kinetics
- **Trường Trần (Wayne)** – Solution Sales, Noventiq Vietnam
- **Đặng Cao Minh Anh** – Solution Sales Internship, Noventiq Vietnam
- **Nguyễn Đức Toàn** – AWS Security Builder

### Những Điểm Nhấn Quan Trọng (Key Highlights)

#### 1. Deep Response Engine: From Detection to Autonomous Resolution (Steve Trần)
![Session 1](../../images/4-EventParticipated/4.3-Event3/session1.jpg)

- **Từ PoC đến Production:** Đừng dành quá nhiều thời gian suy nghĩ lý thuyết. Hãy bắt tay vào làm (execution) và tìm kiếm "khách hàng champion" (như F88, FPT) để đưa ý tưởng giải quyết đúng bài toán thực tế của doanh nghiệp.

- **Agentic Platform cho Cloud Infrastructure:** Ứng dụng AI Agent để hỗ trợ đội ngũ kỹ sư vận hành hạ tầng Cloud phức tạp, tự động hóa xử lý sự cố (Incident), tối ưu chi phí (FinOps) và kiểm tra bảo mật trước khi lên Production. AI không thay thế con người, nhưng sẽ cần những người dùng AI thật giỏi.

#### 2. Voice Agents: Building Human-Like AI Conversations at Scale (Trung Vũ - Hiếu Nghị - Kiệt Trần)
![Session 2](../../images/4-EventParticipated/4.3-Event3/session2.jpg)

- **Hạn chế của mô hình Speech-to-Speech:** Hiện tại chưa hỗ trợ tốt tiếng Việt (vốn là ngôn ngữ low-resource) và khó kiểm soát được việc AI có "nói bậy" hay không (Guardrails).
- **Kiến trúc 3 bước tối ưu:** Sử dụng *Speech-to-Text -> LLM (Text) -> Text-to-Speech*. Kiến trúc này giúp xử lý tốt tiếng Việt, thực hiện chính xác các thao tác Tool Calling (như khóa thẻ ngân hàng) và dễ dàng chuyển tiếp (pass) cuộc gọi cho nhân viên con người một cách tự nhiên khi AI không thể xử lý.
- **Thách thức ngôn ngữ:** Cần train model để AI hiểu ngữ cảnh ngắt lời (barge-in), phân biệt giới tính (anh/chị) qua giọng nói, và xử lý các accent vùng miền.

#### 3. AWS DevOps Agent: Your Always-Available Operations Teammate (Bảo Phan - Nguyên Nguyễn)
![Session 3](../../images/4-EventParticipated/4.3-Event3/session3.jpg)

- **Nỗi đau của kỹ sư vận hành:** Phải truy xuất thủ công log/trace rải rác ở nhiều nơi, gây mất thời gian và dễ đứt đoạn ngữ cảnh khi xử lý lỗi.
- **Giải pháp DevOps Agent:** Hệ thống tự động phân loại, điều tra nguyên nhân gốc rễ (Root Cause), và vẽ ra sơ đồ Topology của hệ thống. 
- **Quy trình 4 bước tối ưu:** Phân loại -> Điều tra -> Đề xuất phương án sửa lỗi (Mitigation Plan) -> Cải thiện hệ thống, nhằm mục tiêu giảm thiểu tối đa thời gian phát hiện (MTTD) và thời gian phục hồi (MTTR).

#### 4. AI-Powered Productivity: Workforce Planning For Enterprise (Trường Trần - Minh Anh)
![Session 4](../../images/4-EventParticipated/4.3-Event3/session4.jpg)

- **Bài toán tuyển dụng thủ công:** HR tốn quá nhiều thời gian lọc CV, dễ đánh giá cảm tính và làm tuột mất những nhân tài (Key Talent).
- **Trợ lý AI Amazon Q:** Có khả năng đọc hiểu JD (Job Description), tự động trích xuất dữ liệu từ các CV định dạng PDF/Image và đối chiếu (match) chúng với nhau.
- **Đánh giá trực quan:** AI chấm điểm ứng viên dựa trên bộ tiêu chí chuẩn (benchmarks), tạo báo cáo (Talent Review Report) chỉ ra lý do đậu/rớt rõ ràng, giúp HR ra quyết định nhanh chóng và giảm tải công việc (workload).

#### 5. Building Secure Private MCP Connection with Amazon Quick (Hiếu Nghị - Toàn Nguyễn)
![Session 5](../../images/4-EventParticipated/4.3-Event3/session5.jpg)

- **Rủi ro Public Endpoint:** Khi Amazon Q (trên Cloud) cần truy xuất dữ liệu từ một MCP Server (chứa API của các bên thứ ba như Jira, Zalo, HR System), việc đi qua Public Internet có thể gây lộ lọt dữ liệu hoặc bị tấn công Man-in-the-Middle.
- **Giải pháp Private VPC Connection:** Đưa Amazon Q vào VPC nội bộ thông qua Interface Endpoint, kết hợp Application Load Balancer (ALB) và Route 53 Resolver.
- **Kết quả:** Tạo ra một đường hầm truyền tải dữ liệu nội bộ hoàn toàn khép kín, tuân thủ nghiêm ngặt các chính sách bảo mật Zero-Trust của doanh nghiệp.

### Tổng kết (Key Takeaways)

#### On AI Mindset & App Development
- **Giải quyết bài toán cốt lõi:** Giá trị của AI không nằm ở công nghệ hào nhoáng, mà ở khả năng tối ưu hóa những quy trình lặp đi lặp lại tốn kém (vận hành Cloud, lọc CV).
- **Kiểm soát & Tùy biến:** Đối với ngôn ngữ đặc thù như Tiếng Việt, việc bẻ nhỏ quy trình (ví dụ: chuyển Voice thành Text để LLM xử lý) quan trọng hơn là dùng một model end-to-end chưa hoàn thiện.

#### Technical Architecture
- **AI Security First:** Không bao giờ để dữ liệu nội bộ đi qua Internet công cộng khi tích hợp AI. Luôn sử dụng Private Link và VPC Connection.
- **Agentic Workflow:** Tương lai của vận hành là các Agent làm việc đa nhiệm, từ đọc Log hạ tầng đến review CV nhân sự.

### Applying to Work
- **Thử nghiệm Voice AI Architecture:** Triển khai mô hình Speech-to-Text kết hợp LLM cho các kịch bản tổng đài chăm sóc khách hàng nội bộ hoặc bot tư vấn.
- **Áp dụng AI vào HR/Admin:** Xây dựng các Agent đọc và trích xuất dữ liệu tự động từ các tài liệu, biểu mẫu, CV nhằm giảm bớt công việc tay chân cho bộ phận Back-office.
- **Rà soát kiến trúc bảo mật tích hợp:** Đảm bảo bất kỳ third-party AI nào khi cắm vào hệ thống dữ liệu công ty đều phải thông qua các kênh bảo mật nội bộ (VPC/ALB).

### Event Experience
Sự kiện FCAJ Community Day tháng 06/2026 mang lại một góc nhìn vô cùng thực tế về cách AI đang tái định hình mọi ngóc ngách của doanh nghiệp, từ phòng ban IT khô khan (DevOps, Cloud Infra) cho đến bộ phận chuyên làm việc với con người (HR). 

Điều gây ấn tượng mạnh nhất là cách các diễn giả chia sẻ những rào cản rất "đời thực" (như việc AI tổng đài bị chửi vì xưng hô sai giới tính, hay kỹ sư kiệt sức vì đọc log thủ công) và đưa ra giải pháp giải quyết triệt để bằng Agent. Ranh giới giữa một sản phẩm AI "demo cho vui" và "dùng để kiếm tiền" chính là khả năng kiểm soát dữ liệu (Guardrails) và kiến trúc bảo mật (Private Connection) đi kèm.

### Một số hình ảnh tại sự kiện

![Audience Photo 1](../../images/4-EventParticipated/4.3-Event3/event_27_06%281%29.jpg)

![Audience Photo 2](../../images/4-EventParticipated/4.3-Event3/event_27_06%282%29.jpg)

![Audience Photo 3](../../images/4-EventParticipated/4.3-Event3/event_27_06%283%29.jpg)
