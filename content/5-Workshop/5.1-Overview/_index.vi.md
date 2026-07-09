---
title: "Giới thiệu"
date: 2026-07-08
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

Trước khi bắt tay vào cấu hình bất kỳ dịch vụ nào, việc hiểu rõ bức tranh kiến trúc tổng thể là yếu tố vô cùng quan trọng. 

Dự án **E-commer Serverless Platform** ứng dụng sức mạnh của 17 dịch vụ AWS khác nhau để tạo nên một hệ thống liên kết chặt chẽ, an toàn và tối ưu chi phí.

### 1. Sơ đồ Kiến trúc Hệ thống

Dưới đây là sơ đồ kiến trúc mô tả luồng dữ liệu (Data flow) của toàn bộ hệ thống từ khi người dùng truy cập trang web cho đến khi đơn hàng được xử lý và lưu trữ.

![Kiến trúc hệ thống](/images/5-Workshop/5.1-Overview/architecture.png)

---

### 2. Danh sách 17 Dịch vụ AWS triển khai trong Workshop

Hệ thống được chia thành 5 nhóm kiến trúc chính. Dưới đây là bảng tóm tắt các dịch vụ bạn sẽ được thao tác trực tiếp trong Workshop này:

| Phân lớp Kiến trúc | Dịch vụ AWS | Vai trò trong hệ thống E-commerce |
| :--- | :--- | :--- |
| **Bảo mật & Mạng** | **Amazon VPC** | Xây dựng không gian mạng riêng ảo (Private/Public Subnets, NAT Gateway). |
| | **AWS IAM** | Quản lý quyền truy cập cho các container và dịch vụ (Roles, Policies). |
| | **AWS Certificate Manager (ACM)** | Cung cấp chứng chỉ SSL miễn phí mã hóa HTTPS. |
| | **Amazon Route 53** | Dịch vụ DNS (nếu có sử dụng tên miền tùy chỉnh). |
| **Lưu trữ & Dữ liệu** | **Amazon DynamoDB** | Database NoSQL siêu tốc lưu trữ 13 bảng dữ liệu (User, Product, Order...). |
| | **Amazon S3** | Lưu trữ mã nguồn Frontend (React/Next.js) và tài nguyên hình ảnh. |
| **Máy tính & Xử lý** | **Amazon ECR** | Kho lưu trữ các Docker Image chứa mã nguồn Backend. |
| | **Amazon ECS (Fargate)** | Vận hành Backend Container hoàn toàn Serverless (không quản lý máy chủ). |
| | **Application Load Balancer** | Phân tải lượng truy cập từ người dùng vào các ECS Task. |
| **Xử lý Sự kiện (Event)** | **AWS Lambda** | Hàm tính toán không máy chủ, được trigger để xử lý các sự kiện ngầm. |
| | **DynamoDB Streams** | Bắt các sự kiện thay đổi dữ liệu (ví dụ: có người vừa thanh toán đơn hàng). |
| | **Amazon SES** | Dịch vụ tự động gửi Email xác nhận thanh toán/đơn hàng cho người dùng. |
| | **Amazon SNS/SQS** | (Mở rộng) Hàng đợi thông báo và xử lý bản tin bất đồng bộ. |
| **Hiệu năng & Tối ưu** | **Amazon CloudFront** | Mạng phân phối nội dung (CDN) giúp trang web tải nhanh trên toàn cầu. |
| | **Amazon ElastiCache** | Máy chủ Redis tăng tốc độ truy vấn danh sách sản phẩm. |
| **Quản lý danh tính** | **Amazon Cognito** | Hệ thống Đăng ký, Đăng nhập và quản lý phiên người dùng bằng JWT Token. |
| **Cấu hình bảo mật** | **AWS Systems Manager (SSM)** | SSM Parameter Store lưu trữ biến môi trường bảo mật (API keys, DB endpoints). |

---

### 3. Quy trình thực hành

Trong các phần tiếp theo, chúng ta sẽ lần lượt thiết lập từng dịch vụ theo quy trình từ dưới lên trên (Bottom-Up):
1. Xây dựng nền móng Mạng & Database (VPC, DynamoDB, S3).
2. Xây dựng phần lõi Backend (Docker, ECR, ECS).
3. Gắn kết hệ sinh thái Serverless (Lambda, Redis, SES).
4. Phơi bày hệ thống ra bên ngoài (CloudFront, Cognito, Frontend).

