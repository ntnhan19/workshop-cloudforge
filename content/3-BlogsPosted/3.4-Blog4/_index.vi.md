---
title: "Blog 4"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 3.4. </b> "
---
# Deep Dive vào AWS Bedrock & Vector Embedding – Cái gì biến văn bản thô thành "Tọa độ" cho AI?

Chào mọi người trong cộng đồng AWS Study Group VN.

Từ góc độ của những thành viên trước giờ chỉ quen cắm mặt vào code các luồng xử lý Backend truyền thống bằng Java hay Node.js, dạo gần đây khi bắt đầu tìm hiểu sâu hơn về Cloud AI và kiến trúc RAG (Retrieval-Augmented Generation) cũng như làm 1 dự án thật sự phải tối ưu hóa kiến trúc này, nhóm chúng mình nhận ra một sự thật khá thú vị: AI thực chất không hề biết đọc chính xác chữ nghĩa user cung cấp!

Chúng ta thường nghe về việc đưa file tài liệu cho AI đọc rồi hỏi đáp. Nhưng đằng sau đó, hệ thống không hề đọc chuỗi String như cách con người đọc sách. Nó yêu cầu một bước chuyển đổi cực kỳ quan trọng gọi là Embedding (Nhúng).

Hôm nay, nhóm muốn mổ xẻ quá trình sử dụng Amazon Bedrock để biến dữ liệu thô thành các Vector – cốt lõi của mọi hệ thống AI hiện đại.

![AWS Bedrock Embedding](/images/3-BlogsPosted/3.4-Blog4/blog4.jpg)

Để xây dựng một hệ thống RAG hoàn chỉnh trên AWS, dữ liệu của bạn sẽ đi qua một luồng xử lý (pipeline) mà dân làm Backend chúng mình rất quen thuộc:

### 1. Chunking (Băm nhỏ dữ liệu)
Bạn không thể ném nguyên một file PDF 100 trang vào Bedrock và bảo nó "Nhúng đi". Nó giống như việc bạn cố nhét nguyên một con gà vào miệng vậy. Chúng ta phải dùng code tách tài liệu ra thành từng đoạn nhỏ (chunk) khoảng vài ba câu hoặc 1 đoạn văn. Việc cắt nhỏ này giúp AI sau này tìm kiếm thông tin chính xác hơn thay vì bị loãng ngữ cảnh.

### 2. Gọi Amazon Bedrock
Sau khi có các đoạn text nhỏ, hệ thống Backend sẽ gọi API gửi đoạn text đó lên Amazon Bedrock (sử dụng các model chuyên biệt như Titan Text Embeddings hoặc Cohere). Bedrock đóng vai trò như một "chiếc hộp đen diệu kỳ". Nó nhận vào một chuỗi String (ví dụ: "Chính sách đổi trả trong vòng 7 ngày"), và trả về một mảng số thực khổng lồ (ví dụ: `[0.012, -0.453, 0.887, ...]`).

### 3. Lưu trữ vào Vector Database
Mảng số (Vector) này về lý thuyết vẫn có thể nhét vào một cột kiểu JSON hoặc BLOB trong MySQL hay PostgreSQL thông thường và không có gì cấm cả. Vấn đề nằm ở chỗ khác: khi cần tìm kiếm tương đồng (similarity search) trên hàng triệu vector, database truyền thống không có cơ chế index phù hợp (như HNSW hay IVFFlat), nên sẽ phải quét tuần tự toàn bộ dữ liệu — rất chậm và tốn tài nguyên.

Vì vậy trong thực tế, người ta thường dùng các Database chuyên dụng cho không gian nhiều chiều như Amazon OpenSearch Serverless, hoặc gắn thêm extension chuyên trách như pgvector vào PostgreSQL để có index ANN (Approximate Nearest Neighbor), giúp việc tìm kiếm nhanh hơn đáng kể so với quét toàn bộ. Mỗi bản ghi lúc này sẽ gồm: `[Đoạn Text gốc] + [Mảng Vector]`.

### Khi User đặt câu hỏi, AI tìm kiếm bằng cách nào?
Đây là phần thú vị nhất của kiến trúc RAG. Khi người dùng gõ vào khung chat: "Tôi muốn bộ đồ ABCXYZ". Hệ thống sẽ làm các bước sau:
1. Gửi câu hỏi này lên Bedrock để biến nó thành một Vector (tạo ra một tọa độ mới trên bản đồ).
2. Cầm tọa độ này, đi vào Vector Database và chạy một thuật toán gọi là Cosine Similarity (đo góc giữa hai vector, chứ không phải đo khoảng cách thẳng thông thường). Nói theo ngôn ngữ bình dân: Database sẽ dùng thước đo góc để xem trên bản đồ hàng ngàn chiều đó, những tài liệu nào nằm ở gần tọa độ của câu hỏi nhất.
3. Lấy ra top 5 kết quả gần nhất (chính là những đoạn text nói về áo cộc tay, quần đùi) và đẩy đống text đó cho mô hình ngôn ngữ lớn (LLM) để nó tổng hợp lại thành câu trả lời tự nhiên nhất.

Tất cả diễn ra chỉ trong vài trăm mili-giây!

### Tại sao lại là AWS Bedrock mà không tự chạy Model?
Đứng từ góc độ kiến trúc hệ thống, việc tự host các mô hình Embedding bằng thư viện mã nguồn mở trên EC2 là một cực hình. Bạn phải lo cài đặt GPU, cấu hình môi trường, và đặc biệt là giải quyết bài toán Auto-scaling khi lượng request tăng vọt.

Với Amazon Bedrock, phần hạ tầng gần như hoàn toàn được AWS quản lý (managed/serverless). Bạn chỉ việc cấu hình SDK trong Java hay Node.js, gọi API, và AWS sẽ tự lo phần scale hạ tầng bên dưới trong giới hạn quota tài khoản. Hết bao nhiêu request thì trả tiền bấy nhiêu. Điều này giúp các lập trình viên tập trung hoàn toàn vào việc xây dựng luồng nghiệp vụ thay vì trở thành thợ sửa ống nước cho cơ sở hạ tầng.

### Những "vết sẹo" kinh nghiệm khi tích hợp Bedrock Embedding
- **Lỗi Rate Limit (Throttling)**: Khi bạn viết một script để đọc hàng ngàn trang tài liệu và liên tục gọi API Bedrock để chuyển thành Vector, rất dễ dính lỗi giới hạn API (quota theo tài khoản/khu vực). Luôn nhớ implement cơ chế Retry (thử lại sau vài giây) hoặc bỏ vào hàng đợi (Amazon SQS) để gọi từ từ.
- **Chiến lược Chunking quyết định sự sống còn**: Cắt text quá ngắn thì AI bị mất ngữ cảnh (không hiểu đang nói về cái gì). Cắt quá dài thì lúc tìm kiếm bị lẫn lộn thông tin. Kinh nghiệm là nên để các đoạn text "overlap" (chồng lấp) lên nhau một chút.
- **Vector DB cũng tốn tiền**: Lưu trữ Vector tốn nhiều dung lượng RAM hơn lưu text thuần rất nhiều. Hãy tính toán kỹ chi phí khi chọn dịch vụ lưu trữ.
- **Chi phí gọi API cũng cần tính toán**: Ngoài chi phí lưu trữ Vector DB, việc gọi Bedrock để embedding hàng loạt tài liệu (nhất là khi tái embedding do đổi chiến lược chunking) cũng tính phí theo token xử lý. Với dữ liệu lớn, khoản này cộng dồn không hề nhỏ, nên cân nhắc kỹ trước khi chạy batch trên toàn bộ tài liệu nhiều lần.

### Kết luận
Amazon Bedrock không chỉ đơn thuần là nơi chứa các mô hình xịn sò như Claude hay Llama, mà nó còn cung cấp công cụ Embedding cực kỳ mạnh mẽ để chúng ta thổi hồn vào dữ liệu thô. Việc nắm vững quá trình Chunking → Embedding → Vector Search chính là chìa khóa để một Backend Developer có thể tự tin xây dựng các tính năng AI xịn sò mà không cần phải là một chuyên gia toán học hay khoa học dữ liệu.

Mọi người ở đây đã ai từng thử build hệ thống RAG và gặp vấn đề khi tối ưu kết quả tìm kiếm từ Vector DB chưa? Rất mong được nghe chia sẻ thực tế từ các anh em đi trước!
