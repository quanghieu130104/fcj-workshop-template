---
title: "Tạo Cognito & SSM"
date: 2026-07-08
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

# 1. Khởi tạo Amazon Cognito User Pool

Amazon Cognito sẽ đóng vai trò là "bảo vệ" xác thực người dùng trước khi họ được phép gọi vào API của Backend. Chúng ta sẽ tích hợp Cognito trực tiếp vào Load Balancer ở bước sau.

### Bước 1: Tạo User Pool
- Tại màn hình **Amazon Cognito**, chọn **User pools**.
- Bấm **Create user pool**.

![Create User Pool](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito0.png)

### Bước 2: Cấu hình thông tin cơ bản
Điền các thông số như sau:
- **Application type:** Chọn **Traditional web application**.
- **Name your application:** Đặt tên tùy ý, ví dụ `My web app`.
- **Options for sign-in identifiers:** Chọn **Email** và **Username**.
- **Self-registration:** Tích chọn **Enable self-registration**.
- **Required attributes for sign-up:** Chọn **Email** và **name**.
- Cuối cùng bấm **Create user directory**.

![Cognito Step 1](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito1.png)

![Cognito Step 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito2.png)

### Bước 3: Setup Resources
- Tại màn hình **Set up resources for your application**.
- Ở phần **Quick setup guide**, chọn ngôn ngữ là **NodeJS** -> Sau đó bấm **Go to overview**.

![Cognito Setup Resources 1](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito3.png)

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito4.png)

### Bước 4: Cấu hình App Client & Hosted UI
Để tích hợp được với Application Load Balancer (ALB), bạn phải cấu hình thêm Hosted UI.
Truy cập vào User Pool vừa tạo, chuyển sang tab **App integration**, cuộn xuống phần **App clients**
Chọn **Create app client**  
- **Application type:** Chọn **Traditional web application**.
- **Name your application:** Đặt tên tùy ý, ví dụ `My web app`.
- **Add a return URL**: Nhập `Tên miền của bạn`
- Chọn **Create app client**

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito5.png)

click vào App Client của bạn -> **Edit**:

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito6.png)

- Ở mục **Auth flows**, đảm bảo đã chọn đủ 3 quyền: `ALLOW_USER_PASSWORD_AUTH`, `ALLOW_REFRESH_TOKEN_AUTH`, `ALLOW_USER_SRP_AUTH`.


![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito7.png)

### Bước 5: Tạo Cognito Domain
- Vẫn ở tab **App integration**, tìm mục **Domain**.
- Đăng ký một **Domain Name** mặc định của Cognito (hoặc dùng tên miền riêng của bạn).


---

# 2. Lưu trữ biến môi trường (SSM Parameter Store)

Thay vì hardcode các thông tin nhạy cảm (như mã bí mật JWT, API Key thanh toán) vào thẳng trong Source code của Backend, chúng ta sẽ lưu chúng an toàn trên **AWS Systems Manager (SSM) Parameter Store**.

Truy cập vào **Systems Manager > Parameter Store**, và bấm **Create parameter** để tạo lần lượt các biến môi trường sau:

*Lưu ý: Bắt buộc chọn Type là **SecureString** để AWS mã hóa nội dung.*

1. Name: `/app/env/jwt_secret` | Type: `SecureString` | Value: *(Nhập mã bí mật tự bịa của bạn)*
2. Name: `/app/env/PAYOS_CLIENT_ID` | Type: `SecureString` | Value: *(Key của cổng thanh toán PayOS)*
3. Name: `/app/env/PAYOS_API_KEY` | Type: `SecureString` | Value: *(API Key của cổng thanh toán PayOS)*
4. Name: `/app/env/PAYOS_CHECKSUM_KEY` | Type: `SecureString` | Value: *(Checksum Key của PayOS)*

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/SSM.png)

*💡 Nhờ có Role `ecsInstanceRole` (cùng Policy `Zopee-Backend-Permission`) mà chúng ta đã setup ở bài 5.2, mã nguồn Node.js của Backend sẽ có quyền tự động vào đây để lấy các biến môi trường này lúc khởi động!*
