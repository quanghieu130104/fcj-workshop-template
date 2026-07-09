---
title: "Giám sát & Tối ưu Hệ thống"
date: 2026-07-09
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

**Cảnh báo CloudWatch & SNS**
- 1.	Vào SNS > Topics > Create topic > Loại Standard. Tên: system-alerts.
- 2.	Tạo xong, bấm Create subscription:
    - Protocol: Email.
    - Endpoint: Nhập email cá nhân của bạn.
    - Nhớ mở email lên và bấm Confirm.
 
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS1.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS2.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS3.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS4.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS5.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS6.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS7.png)

**CloudWatch Alarm**
- 1.	Vào CloudWatch > Alarms > Create alarm.
- 2.	Bấm Select metric > Chọn ECS  > Chọn CPUUtilization.

![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch1.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch2.png)
 
- Sau khi bấm xong, nó sẽ chuyển sang màn hình tiếp theo. Bạn làm y hệt thế này nhé:
    - **Phần Conditions (Điều kiện)**:
         - Chọn Static.
         - Chọn Greater/Equal (>=).
         - Ở cái ô vuông để nhập số, bạn gõ: 80 (ý là nếu CPU vọt qua 80% thì réo chuông).
         - Kéo xuống dưới cùng, bấm Next.

![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch3.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch4.png)

- **Sang màn hình Configure actions**:
    - **Ở phần Configure actions**:
        - Chọn Send a notification to the following SNS topic.
        - Ở ô tìm kiếm phía dưới, nhấp vào và chọn cái system-alerts (mà bạn vừa tạo lúc nãy).
    - Bấm Next.
    - Đặt tên báo động là: Zopee-High-CPU-Alert. Bấm Next.
    - Kéo tuốt xuống dưới cùng, bấm Create alarm.


![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch5.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch6.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch7.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch8.png)

**Tối ưu Tốc độ bằng Amazon ElastiCache (Redis)**
- 1.  Vào VPC > Security Groups > Create security group.
- 2.  Đặt tên: redis-sg, mô tả Security group for Redis Cluster.
- 3.  Chọn đúng VPC của hệ thống E-commerce.
- 4.  Ở phần Inbound rules, thêm 1 rule:
    - Type: Custom TCP
    - Port range: 6379
    - Source: Nhấn vào ô tìm kiếm và chọn Security Group của các EC2 Instance (ecs-sg)
- 5.  Bấm Create security group.

![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache1.png)

**Truy cập ElastiCach**
- 1.  Vào ElastiCache > Redis clusters > Create Redis cluster.
- 2.  Chọn Cluster cache.
- 3.  Engine: Chọn Valkey (hoặc Redis OSS đều được vì nó hoàn toàn tương thích với code).
- 4.  Đặt tên: zopee-redis.
- 5.  Node type: Kéo xuống chọn cache.t4g.micro hoặc cache.t3.micro (tùy nhu cầu, đây là loại rẻ nhất).
- 6.  Số Replicas: 0 (để tiết kiệm nhất).
- 7.  Ở mục Connectivity: Chọn Create a new subnet group. Đặt tên redis-subnet-group, chọn VPC, bấm Manage và chỉ tick chọn 2 cái Private Subnets.
- 8.  Kéo xuống dưới cùng và bấm Create

![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache2.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache3.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache4.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache5.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache6.png)
 
Tại Advanced settings
Selected security group: Chọn security group đã tạo trước đó - > next - > create
 
 ![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache7.png)

Sẽ mất khoảng 5 phút để cụm này khởi tạo

![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache8.png)
 
1.	Sau khi cụm Redis báo trạng thái Available, bấm vào tên của nó.
2.	Tìm dòng Primary endpoint, copy cái URL đó (Ví dụ: zopee-redis.xxxx.use1.cache.amazonaws.com:6379).
3.	Truy cập vào AWS Secrets Manager, mở secret ecommerce/production/secrets của bạn.
4.	Bấm Retrieve secret value -> Edit.
5.	Bấm Add row. Ở ô khóa (Key), điền REDIS_URL.
6.	Ở ô giá trị (Value), điền chuỗi kết nối sau: redis://[Dán cái Primary Endpoint vào đây] (ví dụ: redis://zopee-redis.xxxx.cache.amazonaws.com:6379).
7.	Bấm Save.
