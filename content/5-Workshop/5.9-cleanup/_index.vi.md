---
title: "Dọn dẹp tài nguyên (Clean up)"
date: 2026-07-09
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---


Sau khi hoàn thành bài thực hành, việc dọn dẹp các tài nguyên AWS là cực kỳ quan trọng để **tránh phát sinh chi phí không mong muốn**.
Bạn cần xóa tài nguyên theo **thứ tự ngược lại** so với lúc tạo để tránh lỗi phụ thuộc (Dependency Error).

Hãy thực hiện lần lượt các bước dưới đây:

### 1. Xóa CloudFront Distribution
1. Truy cập **CloudFront** > **Distributions**.
2. Chọn Distribution `zopee-ecommerce` và bấm **Disable**. (Quá trình này mất khoảng 3-5 phút).
3. Sau khi trạng thái chuyển sang *Disabled*, chọn lại Distribution đó và bấm **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/CloudFront.png)

### 2. Xóa Amazon ElastiCache, CloudWatch và SNS
1. Truy cập **Amazon ElastiCache** → **Valkey caches** → Chọn Cache đã tạo → **Actions** → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Vakey.png)

2. Nếu đã tạo Subnet group riêng, truy cập **Subnet groups** → Chọn subnet group → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Subnetgroup.png)

3. Truy cập **Amazon CloudWatch** → **Alarms** → Chọn Alarm CPU đã tạo → **Actions** → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/CloudWatch.png)

4. Truy cập **Amazon SNS** → **Topics** → Chọn Topic `system-alerts` → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/SNS.png)

### 3. Xóa Amazon ECS & ECR
1. Truy cập **Amazon ECS** > **Clusters** > Chọn `my-app-cluster`.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/ECS.png)
2. Vào tab **Services**, chọn các Service đang chạy và bấm **Delete** (Hoặc Update Task = 0 trước khi xóa).
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Cluster.png)
3. Sau khi Service bị xóa, bấm nút **Delete Cluster** ở góc trên cùng bên phải.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Cluster.png)
4. Truy cập **Amazon ECR** > **Repositories**, chọn các repository Frontend/Backend và bấm **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/ECR.png)


### 4. Xóa Load Balancer & Auto Scaling Group
1. Truy cập **EC2** > **Load Balancers** > Chọn `my-app-alb` > **Actions** > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/ALB.png)

2. Chuyển sang **Target Groups** > Chọn `frontend-tg` và `backend-tg` > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/TG.png)

3. Chuyển sang **Auto Scaling Groups** (cuối menu trái) > Chọn ASG của ECS > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Auto.png)

4. Chuyển sang **Instances**, đảm bảo tất cả các EC2 đang bị *Terminated*.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/EC2.png)

### 5. Xóa Cognito & Route 53 (Tuỳ chọn)
1. Truy cập **Amazon Cognito** > **User Pools** > Chọn User Pool đã tạo > Bấm **Delete** (Xác nhận bằng cách gõ tên miền/ID).

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Cognito.png)

2. *(Nếu không giữ tên miền)* Truy cập **Route 53** > **Hosted Zones** > Chọn Zone `zopee.xyz` > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Route53.png)

### 6. Xóa VPC & Security Groups
1. Truy cập **VPC** > **NAT Gateways** > Chọn NAT Gateway > Bấm **Delete NAT gateway**. *(Chờ khoảng 2-3 phút để NAT Gateway bị xóa hoàn toàn)*.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Nat.png)

2. Truy cập **VPC** > **Elastic IPs** > Chọn EIP của NAT Gateway > **Actions** > **Release Elastic IP addresses**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Elastic_IPs.png)

3. Truy cập **VPC** > **Your VPCs** > Chọn `e-commerce-vpc` > Bấm **Actions** > **Delete VPC**. (Hành động này sẽ xóa luôn Subnets, Route Tables, Internet Gateway đi kèm).

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/VPC.png)

4. *(Nếu còn sót)* Truy cập **EC2** > **Security Groups** > Xóa `alb-sg` và `ecs-ec2-sg`,`redis-sg`

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/SG.png)


### 7. Xóa DynamoDB & S3 Buckets
1. Truy cập **DynamoDB** > **Tables** > Chọn bảng dữ liệu > Bấm **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/DynamoDB.png)

2. Truy cập **Amazon S3** > **Buckets**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/S3bucket.png)

3. Để xóa Bucket, bạn phải làm rỗng nó trước: Chọn Bucket > Bấm **Empty** (Gõ *permanently delete* để xác nhận).
4. Sau khi Bucket rỗng, chọn lại Bucket > Bấm **Delete**.


