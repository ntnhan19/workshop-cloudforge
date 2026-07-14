---
title : "Kết nối Cognito và Frontend"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.11.3. </b> "
---

Để ứng dụng Frontend có thể giao tiếp với hệ thống Backend an toàn (API Gateway) và xác thực người dùng (Amazon Cognito) đã được xây dựng ở các chương trước, nhóm dự án tiến hành cung cấp các thông số kết nối thông qua Biến môi trường (Environment Variables) trên AWS Amplify.

Việc lưu trữ các thông số này trên AWS Amplify thay vì hard-code vào mã nguồn là một nguyên tắc bảo mật cốt lõi, giúp cô lập thông tin môi trường (Dev/Staging/Prod) khỏi kho lưu trữ GitHub.

#### Bước 1: Thiết lập Biến môi trường trên Amplify
Nhóm dự án tiến hành thu thập các giá trị thông số từ API Gateway và Amazon Cognito để khai báo vào Amplify.

1. Truy cập ứng dụng trên bảng điều khiển **AWS Amplify**.
2. Tại thanh điều hướng bên trái, dưới mục **Hosting**, chọn **Environment variables**.
3. Nhấn **Manage variables** và thêm các khóa (Key) cốt lõi sau:
   - `VITE_API_ENDPOINT`: Đường dẫn Invoke URL của **API Gateway** cộng thêm hậu tố `/api/v1` (Ví dụ: `https://xxxx.execute-api.ap-southeast-1.amazonaws.com/api/v1`).
   - `VITE_WS_URL`: Đường dẫn kết nối WebSocket thông qua **Application Load Balancer** (Ví dụ: `ws://cloudforge-alb-123456.ap-southeast-1.elb.amazonaws.com/api/v1/ingest/ws`).
   - `VITE_COGNITO_USER_POOL_ID`: Mã ID của Cognito User Pool (Ví dụ: `ap-southeast-1_xxxxxxxxx`).
   - `VITE_COGNITO_USER_POOL_CLIENT_ID`: Mã ID của App Client tương ứng.
4. Nhấn **Save** để lưu lại.

#### Bước 2: Bơm biến môi trường vào luồng Build (amplify.yml)
Đối với các framework tĩnh như React/Vite, biến môi trường chỉ có tác dụng nếu chúng được "bơm" (Inject) vào ứng dụng trong quá trình biên dịch mã nguồn. Do dự án được thiết kế theo cấu trúc **Monorepo**, các lệnh ghi file phải được thực thi chính xác tại thư mục ứng dụng Frontend.

1. Chuyển sang mục **Build settings** trong phần **App settings** ở menu bên trái.
2. Tìm đến phần cấu hình *App build specification* (nội dung tệp `amplify.yml`) và nhấn **Edit**.
3. Chỉnh sửa giai đoạn `preBuild` để tự động tạo tệp `.env` nội bộ trước khi chạy lệnh biên dịch:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - echo "VITE_API_ENDPOINT=$VITE_API_ENDPOINT" >> .env
        - echo "VITE_WS_URL=$VITE_WS_URL" >> .env
        - echo "VITE_COGNITO_USER_POOL_ID=$VITE_COGNITO_USER_POOL_ID" >> .env
        - echo "VITE_COGNITO_USER_POOL_CLIENT_ID=$VITE_COGNITO_USER_POOL_CLIENT_ID" >> .env
        - npm install
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: dist
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

4. Nhấn **Save** để cập nhật cấu hình luồng Build.

#### Bước 3: Tích hợp Component Xác thực trên mã nguồn React
Để giải quyết bài toán xây dựng giao diện quản lý người dùng trong thời gian ngắn, nhóm dự án tích hợp bộ thư viện giao diện có sẵn của AWS Amplify. Thư viện này tự động cung cấp toàn bộ các form Đăng nhập, Đăng ký, Quên mật khẩu và Xác thực OTP chuẩn Enterprise mà không cần lập trình thủ công từ đầu.

Cài đặt các thư viện bổ trợ tại thư mục mã nguồn Frontend:

```bash
npm install aws-amplify @aws-amplify/ui-react
```

Cấu hình khởi tạo và sử dụng Component `<Authenticator>` để bọc lấy luồng chính của ứng dụng trong tệp `src/App.jsx`:

```javascript
import { Amplify } from 'aws-amplify';
import { Authenticator } from '@aws-amplify/ui-react';
import '@aws-amplify/ui-react/styles.css';

// Cấu hình SDK kết nối với Amazon Cognito thông qua các biến môi trường được Amplify bơm vào
Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: import.meta.env.VITE_COGNITO_USER_POOL_ID,
      userPoolClientId: import.meta.env.VITE_COGNITO_USER_POOL_CLIENT_ID,
    }
  }
});

export default function App() {
  return (
    <Authenticator>
      {({ signOut, user }) => (
        <main style={{ padding: '20px' }}>
          <h1>Chào mừng {user?.username} đến với Smart Media Analytics!</h1>
          <button onClick={signOut}>Đăng xuất</button>
          
          {/* Giao diện Dashboard chính của ứng dụng sẽ hiển thị tại đây sau khi đăng nhập hợp lệ */}
          
        </main>
      )}
    </Authenticator>
  );
}
```

#### Bước 4: Kích hoạt triển khai lại (Redeploy)
Để toàn bộ cấu hình biến môi trường và mã nguồn tích hợp mới có hiệu lực trên môi trường Cloud, tiến trình Build cần được tái khởi động.

1. Trở lại màn hình **Deployments** của ứng dụng trên Amplify Console.
2. Bấm chọn vào nhánh **`main`**.
3. Nhìn lên góc phải trên cùng, nhấn chọn nút **Redeploy this version** để kích hoạt lại tiến trình CI/CD.
4. Chờ đợi hệ thống hoàn tất cả 4 giai đoạn và chuyển trạng thái sang **Deployed** màu xanh.

![Amplify Redeploy Success](/images/5-Workshop/5.11-Frontend-deployment/5.11.3-connect-cognito-frontend/5.11.3-amplify-success.png)

Lúc này, khi truy cập vào đường dẫn tên miền của ứng dụng, toàn bộ giao diện đã được bảo vệ bởi hệ thống xác thực. Người dùng bắt buộc phải đăng nhập hoặc tạo tài khoản thông qua Form UI tự động của Cognito trước khi có thể truy cập vào Dashboard chính.

![Cognito Login UI](/images/5-Workshop/5.11-Frontend-deployment/5.11.3-connect-cognito-frontend/5.11.3-login-ui.png)

***

***

**Bước tiếp theo:** Hệ thống Frontend đã kết nối thành công với Backend. Tuy nhiên, quy trình Build CI/CD hiện tại đang phụ thuộc hoàn toàn vào luồng cơ bản của Amplify. Ở chương tiếp theo ([**Chương 5.12: CI/CD Pipeline**](../../5.12-CICD/)), nhóm dự án sẽ chuẩn hóa quy trình phân phối liên tục (Continuous Delivery) cho toàn bộ hệ thống bằng AWS CodePipeline.
