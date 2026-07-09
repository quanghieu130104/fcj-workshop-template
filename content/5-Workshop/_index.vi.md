---
title: "Workshop"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Nền tảng Thương mại Điện tử E-commer (Serverless E-commerce Platform)
## Hệ thống mua sắm trực tuyến kiến trúc Enterprise trên nền tảng AWS
#### Tổng quan

Chào mừng bạn đến với Workshop triển khai **Hệ thống Thương mại Điện tử (E-commerce Serverless)** trên nền tảng AWS! Đây không chỉ là một bài hướng dẫn thông thường, mà là một hành trình mô phỏng chính xác quy trình triển khai hệ thống phần mềm cấp độ doanh nghiệp (Enterprise) trong môi trường thực tế.

Trong Workshop này, chúng ta sẽ cùng nhau giải quyết bài toán: *Làm thế nào để xây dựng một nền tảng bán hàng trực tuyến có khả năng tự động mở rộng (Auto-scaling), chịu tải cao trong các đợt Flash Sale, bảo mật tuyệt đối dữ liệu thanh toán và tối ưu chi phí hạ tầng (Pay-as-you-go)?*

Thông qua hướng dẫn chi tiết từng thao tác (Step-by-Step), bạn sẽ tự tay thiết kế và kết nối **17 dịch vụ AWS cốt lõi**, trải dài qua các tầng kiến trúc:
- **Tầng Hạ tầng (Infrastructure)**: Xây dựng mạng lưới an toàn với Amazon VPC (Public/Private Subnets) và phân quyền chặt chẽ qua AWS IAM.
- **Tầng Dữ liệu (Database)**: Triển khai cơ sở dữ liệu NoSQL tốc độ cao với 12 bảng Amazon DynamoDB và lưu trữ tĩnh trên S3.
- **Tầng Xử lý (Compute)**: Ứng dụng Containerization (Docker) để đóng gói Backend API và vận hành trên Amazon ECS (Fargate) thông qua Application Load Balancer.
- **Tầng Serverless & Tối ưu (Event-Driven)**: Tích hợp AWS Lambda, DynamoDB Streams, Amazon SES (gửi email), ElastiCache Redis (bộ nhớ đệm) để xử lý các sự kiện ngầm phức tạp.
- **Tầng Tự động hóa (DevOps)**: Thiết lập CI/CD Pipeline hoàn chỉnh với GitHub Actions để tự động build và deploy.

Mục tiêu của Workshop là giúp bạn làm chủ tư duy thiết kế Cloud-native, thành thạo các công cụ hiện đại và tự tin vận hành các dự án AWS thực tế. Hãy cùng bắt đầu!


#### Nội dung

1. [Tổng quan](5.1-Overview/)
2. [Thiết lập quyền truy cập IAM](5.2-IAM/)
3. [Khởi tạo DynamoDB và S3 Buckets](5.3-database-storage/)
4. [Triển khai Mạng & Compute Layer](5.4-backend-ecs/)
5. [Tích hợp Serverless & Sự kiện](5.5-serverless/)
6. [Giám sát hệ thống & Caching](5.6-monitoring-caching/)
7. [Phân phối nội dung toàn cầu (CloudFront)](5.7-cloudfront/)
8. [Tích hợp API Gateway (Sắp tới)](#)
9. [Dọn dẹp tài nguyên](5.9-cleanup/)