---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# Nền tảng Thương mại Điện tử E-commer (Serverless E-commerce Platform)
## Hệ thống mua sắm trực tuyến kiến trúc Enterprise trên nền tảng AWS

### 1. Tóm tắt điều hành
Dự án **E-commer** là một nền tảng thương mại điện tử hiện đại, được xây dựng dựa trên kiến trúc Microservices và Serverless trên AWS. Hệ thống cung cấp trải nghiệm mua sắm toàn diện từ quản lý sản phẩm, giỏ hàng, thanh toán qua mã QR (PayOS) cho đến hệ thống gửi email tự động. Nền tảng tận dụng các dịch vụ mạnh mẽ của AWS như ECS, Lambda, DynamoDB, ElastiCache và CloudFront để đảm bảo hiệu suất cao, khả năng mở rộng tự động và tính bảo mật vượt trội cho người dùng cuối.

### 2. Tuyên bố vấn đề
*Vấn đề hiện tại*
Các hệ thống bán hàng truyền thống thường gặp khó khăn trong việc chịu tải vào những đợt mua sắm cao điểm. Kiến trúc monolithic nguyên khối khó nâng cấp, chi phí duy trì máy chủ tĩnh cao và thiếu sự linh hoạt trong việc tích hợp các cổng thanh toán nội địa hay xử lý các sự kiện bất đồng bộ như gửi email xác nhận.

*Giải pháp*
Dự án giải quyết các vấn đề trên bằng cách sử dụng kiến trúc AWS kết hợp giữa Containerization (ECS) và Serverless (Lambda). Frontend được phân phối tốc độ cao qua CloudFront và S3. Backend hoạt động mạnh mẽ với ECS kết hợp ALB, lưu trữ dữ liệu với DynamoDB và tăng tốc truy xuất qua ElastiCache (Redis). Quá trình thanh toán được tự động hóa qua PayOS, và các tác vụ nặng như gửi email xác nhận được xử lý bất đồng bộ qua kiến trúc Event-Driven với DynamoDB Streams và AWS SES. Amazon Cognito đảm bảo quản lý xác thực người dùng an toàn tuyệt đối.

*Lợi ích và hoàn vốn đầu tư (ROI)*
Kiến trúc linh hoạt giúp tối ưu chi phí theo lưu lượng sử dụng thực tế (Pay-as-you-go). Khả năng Auto Scaling giúp hệ thống không bị gián đoạn trong các dịp lễ tết. Quá trình triển khai hoàn toàn tự động qua CI/CD GitHub Actions giúp tiết kiệm hàng trăm giờ vận hành thủ công, mang lại lợi thế cạnh tranh lớn về tốc độ ra mắt tính năng mới.

### 3. Kiến trúc giải pháp
Nền tảng sử dụng kiến trúc phân tán với Frontend tĩnh lưu trên S3/CloudFront, gọi API tới Backend được triển khai bằng các container trên ECS qua Application Load Balancer. Dữ liệu phi cấu trúc lưu trữ trên DynamoDB với cơ chế Caching bằng Redis. Các sự kiện phát sinh (ví dụ: tạo đơn hàng) sẽ trigger Lambda qua DynamoDB Streams để xử lý nghiệp vụ gửi Email qua SES.

![E-Commerce Architecture](/images/2-Proposal/architecture.png)

*Các dịch vụ AWS cốt lõi được sử dụng (17 dịch vụ)*
- **Amazon VPC**: Thiết lập mạng lưới nội bộ, Subnets, Route Tables, Internet Gateway.
- **AWS IAM**: Quản lý định danh, Roles và Policies phân quyền truy cập an toàn.
- **Amazon EC2 & ALB**: Cung cấp năng lực tính toán và Application Load Balancer để phân tải giao thông mạng.
- **Amazon ECS**: Điều phối và vận hành các Docker containers cho Backend.
- **Amazon ECR**: Lưu trữ và quản lý các Docker Images an toàn.
- **AWS Lambda**: Hàm tính toán Serverless xử lý các tác vụ nền (Event-Driven).
- **Amazon DynamoDB**: Cơ sở dữ liệu NoSQL lưu trữ thông tin nghiệp vụ và kích hoạt Streams.
- **Amazon S3**: Lưu trữ ứng dụng Frontend tĩnh và tệp tin người dùng tải lên.
- **Amazon CloudFront**: Mạng phân phối nội dung (CDN) giúp tăng tốc độ tải trang toàn cầu.
- **Amazon API Gateway**: Xử lý và định tuyến các API Request an toàn.
- **Amazon Cognito**: Cung cấp tính năng đăng nhập, đăng ký và bảo mật xác thực người dùng.
- **Amazon SES**: Dịch vụ gửi email tự động (Transaction Emails).
- **Amazon SNS / SQS**: Hệ thống nhắn tin và hàng đợi phục vụ kiến trúc Event-Driven.
- **Amazon ElastiCache (Redis)**: Lưu trữ bộ đệm (Caching) giúp tăng tốc phản hồi cơ sở dữ liệu.
- **AWS Systems Manager (SSM)**: Lưu trữ bảo mật các biến môi trường và Secret Keys qua Parameter Store.
- **Amazon Route 53**: Quản lý DNS và định tuyến tên miền tùy chỉnh.
- **AWS Certificate Manager (ACM)**: Quản lý và cung cấp chứng chỉ SSL/TLS bảo mật đường truyền.

### 4. Triển khai kỹ thuật
*Các giai đoạn triển khai*
Dự án được phát triển trong vòng 12 tuần với 4 giai đoạn chính:
1. *Nghiên cứu & Thiết kế*: Xây dựng nền tảng AWS, VPC, S3 và cấu hình IAM.
2. *Lập trình & Containerization*: Xây dựng API, kết nối DynamoDB, đóng gói Docker và cấu hình ECS.
3. *Frontend & Thanh toán*: Xây dựng UI, tích hợp xác thực Cognito và cổng thanh toán VietQR (PayOS).
4. *Tối ưu & Tự động hóa*: Triển khai CI/CD pipeline, tích hợp Redis Caching, cấu hình SSL/Domain và hệ thống Event-driven SES.

*Yêu cầu kỹ thuật*
- *Frontend*: React/Next.js tích hợp SDK của Cognito và PayOS Checkout.
- *Backend*: Node.js/Express hoặc Python đóng gói Docker, tối ưu hóa xử lý bất đồng bộ.
- *Hạ tầng*: Setup hoàn chỉnh GitHub Actions để tự động build image lên ECR và deploy ECS.

### 5. Lộ trình & Mốc triển khai

- **Giai đoạn 1: Xây dựng Hạ tầng AWS**
  - Thiết kế hạ tầng mạng với Amazon VPC, Public/Private Subnets, Internet Gateway, NAT Gateway và Route Tables.
  - Quản lý phân quyền với AWS IAM và cấu hình Security Groups.
  - Tạo Bucket Amazon S3 để lưu trữ Frontend tĩnh và hình ảnh sản phẩm.

- **Giai đoạn 2: Triển khai Ứng dụng E-commerce**
  - Đóng gói ứng dụng bằng Docker và lưu trữ Image trên Amazon ECR.
  - Triển khai Frontend và Backend bằng Amazon ECS (EC2 Launch Type).
  - Cấu hình Auto Scaling Group, Capacity Provider và Application Load Balancer (ALB).
  - Xây dựng cơ sở dữ liệu với Amazon DynamoDB và xác thực người dùng bằng Amazon Cognito.

- **Giai đoạn 3: Phân phối & Bảo mật Hệ thống**
  - Đăng ký tên miền và quản lý DNS bằng Amazon Route 53.
  - Cấu hình HTTPS với AWS Certificate Manager (ACM).
  - Phân phối nội dung thông qua Amazon CloudFront.
  - Bảo vệ ứng dụng bằng AWS WAF và các chính sách bảo mật của AWS.

- **Giai đoạn 4: Vận hành & Hoàn thiện Production**
  - Thiết lập quy trình CI/CD với GitHub Actions.
  - Giám sát hệ thống bằng Amazon CloudWatch và AWS CloudTrail.
  - Tối ưu hiệu năng với Amazon ElastiCache (Redis).
  - Tích hợp thanh toán PayOS và dịch vụ gửi Email/SMS bằng Amazon SES và Amazon SNS.
  - Kiểm thử, tối ưu hiệu năng và hoàn thiện hệ thống E-commerce trên môi trường Production.

## 6. Ước tính ngân sách

### Chi phí hạ tầng (Môi trường MVP)

| Dịch vụ | Cấu hình | Chi phí ước tính/tháng |
|---------|----------|-----------------------:|
| Amazon EC2 | 2 × t3.micro (ECS Cluster) | ~16 – 18 USD |
| Amazon EBS | 2 × 30 GB gp3 | ~5 USD |
| Amazon NAT Gateway | 1 NAT Gateway | ~33 – 38 USD |
| Application Load Balancer (ALB) | 1 ALB + lưu lượng thấp | ~16 – 20 USD |
| Amazon DynamoDB | On-Demand (PAY_PER_REQUEST) | ~1 – 5 USD |
| Amazon S3 | Frontend + Upload Images (~20 GB) | ~0.5 – 1 USD |
| Amazon CloudFront | Lưu lượng <100 GB | ~1 – 3 USD |
| Amazon ECR | Lưu trữ Docker Images | <1 USD |
| Amazon Route 53 | 1 Hosted Zone | ~0.50 USD |
| AWS Certificate Manager (ACM) | SSL/TLS Certificate | Miễn phí |
| Amazon Cognito | <50.000 MAU | Miễn phí |
| Amazon CloudWatch | Logs & Metrics cơ bản | ~1 – 3 USD |
| AWS CloudTrail | 1 Trail (Management Events) | Miễn phí |
| Amazon ElastiCache (Valkey) | cache.t4g.micro | ~10 – 13 USD |
| Amazon SES | ~1.000 email/tháng | <1 USD |
| Amazon SNS | Thông báo ít | <1 USD |

### Tổng chi phí ước tính

| Môi trường | Chi phí/tháng |
|------------|--------------:|
| MVP        | **~85 – 110 USD** |

### 7. Đánh giá rủi ro
*Ma trận rủi ro*
- Lỗi quá tải traffic đột ngột: Ảnh hưởng cao, xác suất trung bình.
- Rò rỉ cấu hình / Secret keys: Ảnh hưởng cực cao, xác suất thấp.
- Thanh toán lỗi từ phía đối tác: Ảnh hưởng cao, xác suất thấp.

*Chiến lược giảm thiểu*
- Cấu hình Auto Scaling cho ECS và dùng CloudFront/Redis để giảm tải Backend.
- Sử dụng AWS Systems Manager (Parameter Store) và GitHub Secrets để bảo mật thông tin.
- Tạo cơ chế tự động đối soát đơn hàng và webhook fallback.

### 8. Kết quả kỳ vọng
*Cải tiến kỹ thuật*: Một hệ thống E-commerce hoàn chỉnh, bảo mật cao, tự động hóa hoàn toàn từ code đến deploy (CI/CD), sẵn sàng chịu tải lớn.
*Giá trị dài hạn*: Trở thành nền tảng lõi vững chắc, dễ dàng tích hợp thêm các dịch vụ AI/Data Analytics trong tương lai để cá nhân hóa trải nghiệm mua sắm.