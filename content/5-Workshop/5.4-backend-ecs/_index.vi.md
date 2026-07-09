---
title: "Triển khai Mạng & Compute Layer"
date: 2026-07-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Triển khai Cốt lõi Backend (Compute & Load Balancing)

Giai đoạn này là "trái tim" của toàn bộ hệ thống. Trong kiến trúc Enterprise này, chúng ta sẽ **không** sử dụng ECS Fargate (Serverless Compute). Việc này giúp tối ưu hóa chi phí khi hệ thống chạy 24/7 và đặc biệt là để hỗ trợ duy trì kết nối WebSocket ổn định với hiệu suất cao nhất. Thay vào đó, chúng ta sẽ quản lý một cụm máy chủ EC2 (Compute Layer) thông qua Auto Scaling và dùng Amazon ECS để điều phối các Docker container chạy trên đó.

Đồng thời, để bảo vệ hệ thống API khỏi các truy cập trái phép ngay từ "cửa ngõ", kiến trúc sẽ sử dụng **Amazon Cognito** kết hợp trực tiếp với **Application Load Balancer (ALB)**. 

Do sự ràng buộc chặt chẽ của kiến trúc, bạn **bắt buộc** phải triển khai theo đúng trình tự 3 bước sau:

1. **[Tạo Cognito & SSM Parameter Store](5.4.1-cognito-ssm/)**: Tạo hệ thống quản lý danh tính User trước, đồng thời lưu trữ các biến môi trường nhạy cảm.
2. **[Khởi tạo Hạ tầng Mạng, Compute Layer & ALB](5.4.2-alb/)**: Tạo VPC, Security Groups, cụm ECS/EC2 và cổng cân bằng tải ALB kết hợp quy tắc xác thực (Cognito).
3. **[Đẩy Image ECR & Khởi chạy Service](5.4.3-ecr-task/)**: Đóng gói mã nguồn Backend thành Docker, đẩy lên ECR và khởi chạy thành công trên cụm ECS.

