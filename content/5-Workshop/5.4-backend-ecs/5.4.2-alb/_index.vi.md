---
title: "Tạo Hạ tầng Mạng & Compute Layer"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# Khởi tạo Hạ tầng Mạng, Compute Layer & ALB

Bài này sẽ hướng dẫn bạn khởi tạo toàn bộ hạ tầng cốt lõi để chạy Backend, bao gồm việc chuẩn bị mạng (VPC), nhóm bảo mật (Security Groups), cụm máy chủ EC2 (Compute Layer) và cuối cùng là bộ cân bằng tải (ALB).

### Bước 1: Khởi tạo Mạng (VPC)
Trước khi tạo bất kỳ tài nguyên mạng nào như Load Balancer hay Target Group, bạn bắt buộc phải có một không gian mạng riêng biệt (VPC).
- Truy cập **VPC Dashboard > Click Create VPC**.
- Chọn tùy chọn **VPC and more** (Tạo VPC tự động kèm Subnets):
  - **Name tag auto-generation:** Đặt tên, ví dụ `e-commerce-vpc`.
  - **IPv4 CIDR block:** `10.0.0.0/16`.
  - **Number of Availability Zones (AZs):** `2`.
  - **Number of public subnets:** `2` (Dùng cho ALB và NAT Gateway).
  - **Number of private subnets:** `2` (Dùng cho EC2 Backend và Database để bảo mật).
  - **NAT gateways:** Chọn `Zonal`, Chọn `1 AZ` hoặc `2 AZ` (Để tiết kiệm chi phí, bạn có thể chọn 1 AZ).
  - Bấm **Create VPC**.

![VPC](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/VPC.png)

![VPC](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/VPC1.png)

### Bước 2: Tạo Security Groups
Chúng ta cần tạo 2 Security Groups để kiểm soát luồng truy cập:
1. **`alb-sg` (Dành cho Load Balancer):** Cho phép Inbound HTTP (80) và HTTPS (443) từ `0.0.0.0/0` (Internet).

![Security group](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Security-group.png)

2. **`ecs-ec2-sg` (Dành cho EC2 Backend):** Cho phép Inbound từ `alb-sg` ở các port `8080` (Backend) và `3000` (Frontend). Cho phép SSH (22) từ IP của bạn (nếu cần).

![Security group](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Security-group1.png)

### Bước 3: Khởi tạo Cụm máy chủ ECS & Auto Scaling Group
- **ECS Cluster:** Truy cập dịch vụ **Amazon ECS**, tạo một cụm (Cluster) mới và đặt tên là `my-app-cluster`.
- **Infrastructure - advanced:**
  - **Auto Scaling group (ASG):** Chọn Create a new Auto Scaling group - advanced
  - **AMI:** Chọn **ECS-optimized Amazon Linux 2023** (Hệ điều hành đã cài sẵn Docker và ECS Agent).
  - **EC2 instance type:** Chọn `t3.micro`.
  - **EC2 instance role:** Chọn **`ecsInstanceRole`** đã tạo.
    - **Desired capacity:** Minimum 0 , Maximum 2.
  - **Root EBS volume size:** 30.

![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS.png)

![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS1.png)

  - **SSH Key pair:** Chọn Create a new key pair:
    - **Name:** ecs-ec2-key
    - **Key pair type:** RSA
    - **Private key file format:** .pem
    - **Create key pair.**

![SSH Key pair](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS_key.png)

- **Network settings:**
  - **VPC:** Chọn `e-commerce-vpc` vừa tạo ở Bước 1.
  - **Subnets:** Chọn 2 **Private Subnets**.
  - **Security Group:** Chọn `ecs-ec2-sg` vừa tạo ở Bước 2.

![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS2.png)

- **Auto Scaling Group (ASG):**
  - Sau khi tạo ECS Cluster, **Amazon ECS sẽ tự động tạo** một **Auto Scaling Group (ASG)** cùng **Launch Template** dựa trên các thông số đã cấu hình (AMI, EC2 Instance Type, Desired Capacity, Root EBS Volume, Network Settings,...).
  - ASG được cấu hình để triển khai các EC2 trong `e-commerce-vpc` và 2 **Private Subnets** đã chọn.
  - Kiểm tra trong dịch vụ **Amazon EC2 → Auto Scaling Groups** để xác nhận ASG đã được tạo thành công.

### Bước 4: Tạo Target Groups (Nhóm mục tiêu)
Khai báo cho hệ thống biết Backend và Frontend đang chạy ở đâu (port nào).
Truy cập **EC2 > Target Groups** và tạo 2 nhóm sau (Ở mục VPC, phải chọn đúng `e-commerce-vpc`):

1. **`backend-tg`**:
   - Target type: `Instances`
   - Protocol/Port: `HTTP` / `8080`
   - Health check path: `/api/health`
- **Register targets - recommended**: Chọn 2 instances EC2 của cluster vừa tạo.

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target1.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target2.png)

2. **`frontend-tg`**:
   - Target type: `Instances`
   - Protocol/Port: `HTTP` / `3000`
   - Health check path: `/`
- **Register targets - recommended**: Chọn 2 instances EC2 của cluster vừa tạo.

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target3.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target4.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target2.png)

### Bước 5: Khởi tạo Application Load Balancer (ALB)
Truy cập **EC2 > Load Balancers** > Bấm **Create Load Balancer** > Chọn **Application Load Balancer** > **Create**.
- **Load balancer name:** `my-app-alb`.
- **Scheme:** `Internet-facing` (IPv4).
- **Network mapping:** Chọn VPC `e-commerce-vpc` và tích chọn 2 **Public Subnets**.
- **Security Groups:** Chọn Security Group `alb-sg` đã tạo ở Bước 2.

![Load balancer name](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ALB.png)

![Load balancer name](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ALB1.png)

### Bước 6: Cấu hình Listeners & Rules
Đây là nơi điều hướng dòng chảy dữ liệu (Routing):

**Listener HTTP (80)**
- Protocol: HTTP
- Port: 80
- Default action: Redirect to URL
- Protocol: HTTPS
- Port: 443
Status code: HTTP_301

![Listener](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Listeners.png)


**Listener HTTPS (443)**
- Protocol: HTTPS
- Port: 443
- Default action: 
- Pre-routing action: No pre-routing action
- Routing action: Forward to target groups
- Target group: frontend-tg

![Listener](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Listeners1.png)

**Secure listener settings**

- **Listener HTTPS (Port 443):**
  - Bắt buộc gắn SSL Certificate từ AWS Certificate Manager (ACM).
  - Thêm các Rules sau theo thứ tự ưu tiên:
    1. **Rule Xác thực Cognito:** Nếu `Path` là `/checkout/*` hoặc `/api/orders/*` -> Action 1: **Authenticate** (Chọn Cognito User Pool vừa tạo) -> Action 2: Forward tới `backend-tg`.
    2. **Rule gọi API thường:** Nếu `Path` là `/api/*` -> Forward tới `backend-tg`.
    3. **Default Rule (Mặc định):** Forward tất cả traffic còn lại về `frontend-tg` (Để máy chủ Frontend render nội dung giao diện).


- **Security policy**: Mặc định.

- **Certificate source**: chứng chỉ SSL/TLS được tạo cho tên miền (domain) của bạn trong AWS Certificate Manager (ACM).

![Listener](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Listeners2.png)