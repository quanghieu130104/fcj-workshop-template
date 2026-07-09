---
title: "IAM Permissions và Roles"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Thiết lập IAM Permissions và IAM Roles

Trong hệ thống E-commerce của chúng ta, bảo mật và phân quyền là ưu tiên hàng đầu. Để các dịch vụ trên AWS có thể giao tiếp với nhau một cách an toàn mà không bị lộ dữ liệu nhạy cảm, chúng ta cần sử dụng **AWS IAM Roles**.

Dựa vào kiến trúc thực tế bạn triển khai, hệ thống sẽ xoay quanh **5 Role cốt lõi**. Dưới đây là hướng dẫn chi tiết cách cấu hình 3 Role quan trọng nhất mà bạn phải tự tạo.

### IAM permissions and roles

**1. `ecsTaskExecutionRole`**
Đây là Role cực kỳ quan trọng được gắn cho nền tảng Amazon ECS. Trong kiến trúc của chúng ta, nó không chỉ hỗ trợ nền tảng kéo Image mà còn đóng vai trò cấp quyền cho Container truy cập các tài nguyên lưu trữ khác.
- **Cách tạo:** Tạo Role với Use case là `Elastic Container Service Task`. Sau đó, tìm và tích chọn **4 Policies** có sẵn sau đây:
  - `AmazonECSTaskExecutionRolePolicy` (Để ECS kéo Image và ghi log)
  - `AmazonDynamoDBFullAccess` (Cấp toàn quyền thao tác trên Database)
  - `AmazonSSMReadOnlyAccess` (Quyền đọc các tham số cấu hình)
  - `SecretsManagerReadWrite` (Quyền đọc/ghi các thông tin mật khẩu, API key)

![Permissions policies](/images/5-Workshop/5.2-IAM/Permissions%20policies.png)

![ecsTaskExecutionRole](/images/5-Workshop/5.2-IAM/ecsTaskExecutionRole.png)

**2. `lambda-order-email-role`**
Role này mang nghiệp vụ cốt lõi của hệ thống Serverless, giúp xử lý gửi email bất đồng bộ.
- **Cách tạo:** Tạo Role cho dịch vụ `Lambda`.
- **Gắn Policy có sẵn:** Tìm và tích chọn `AWSLambdaBasicExecutionRole` (Quyền ghi log).
- **Gắn Policy tự viết:** Tạo một Inline Policy mới (đặt tên là `lambda-dynamodb-ses-policy`) với đoạn mã JSON sau để cấp quyền đọc DynamoDB Streams và gửi Email qua SES:
  ```json 
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "dynamodb:DescribeStream",
                  "dynamodb:GetRecords",
                  "dynamodb:GetShardIterator",
                  "dynamodb:ListStreams"
              ],
              "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/order_table/stream/*"
          },
          {
              "Effect": "Allow",
              "Action": "ses:SendEmail",
              "Resource": "*"
          }
      ]
  }
  ```

**3. `ecsInstanceRole`**
Role này được gắn trực tiếp cho các máy chủ EC2 nằm trong cụm ECS Cluster của bạn (hỗ trợ cho kiểu chạy EC2 bên cạnh Fargate).
- **Cách tạo:** Tạo Role với Use case là `EC2`.
- **Gắn Policy có sẵn:** Tìm và tích chọn 2 Policies:
  - `AmazonEC2ContainerServiceforEC2Role`
  - `AmazonSSMManagedInstanceCore`
- **Gắn Policy tự viết:** Tạo một Inline Policy mới (đặt tên là `Zopee-Backend-Permission`) với đoạn JSON dưới đây. Đoạn mã này cấp quyền cho Backend truy cập Secrets Manager, tải ảnh lên S3, và dùng SQS để xử lý hàng đợi đơn hàng:
  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "secretsmanager:GetSecretValue"
              ],
              "Resource": "arn:aws:secretsmanager:ap-southeast-1:*:secret:ecommerce/production/secrets-*"
          },
          {
              "Effect": "Allow",
              "Action": [
                  "s3:PutObject",
                  "s3:GetObject"
              ],
              "Resource": "arn:aws:s3:::my-app-uploads/*"
          },
          {
              "Effect": "Allow",
              "Action": [
                  "sqs:SendMessage",
                  "sqs:ReceiveMessage",
                  "sqs:DeleteMessage",
                  "sqs:GetQueueAttributes"
              ],
              "Resource": "arn:aws:sqs:ap-southeast-1:*:order-processing-queue"
          }
      ]
  }
  ```
