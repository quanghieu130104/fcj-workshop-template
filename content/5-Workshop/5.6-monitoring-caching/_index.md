---
title: "Monitoring & Optimization"
date: 2026-07-09
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

**CloudWatch & SNS Alerts**
- 1. Go to SNS > Topics > Create topic > Type Standard. Name: system-alerts.
- 2. After creating, click Create subscription:
    - Protocol: Email.
    - Endpoint: Enter your personal email.
    - Remember to open your email and click Confirm.
 
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS1.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS2.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS3.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS4.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS5.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS6.png)
![SNS](/images/5-Workshop/5.6-monitoring-caching/SNS7.png)

**CloudWatch Alarm**
- 1. Go to CloudWatch > Alarms > Create alarm.
- 2. Click Select metric > Select ECS > Select CPUUtilization.

![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch1.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch2.png)
 
- After clicking, it will switch to the next screen. Do exactly as follows:
    - **Conditions Section**:
         - Select Static.
         - Select Greater/Equal (>=).
         - In the box to enter the number, type: 80 (meaning if CPU spikes over 80% it will alert).
         - Scroll to the bottom, click Next.

![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch3.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch4.png)

- **On the Configure actions screen**:
    - **In the Configure actions section**:
        - Select Send a notification to the following SNS topic.
        - In the search box below, click and select system-alerts (which you just created).
    - Click Next.
    - Name the alarm: Zopee-High-CPU-Alert. Click Next.
    - Scroll to the very bottom, click Create alarm.

![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch5.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch6.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch7.png)
![Cloud Watch](/images/5-Workshop/5.6-monitoring-caching/CloudWatch8.png)

**Speed Optimization with Amazon ElastiCache (Redis)**
- 1. Go to VPC > Security Groups > Create security group.
- 2. Name: redis-sg, description Security group for Redis Cluster.
- 3. Select the correct VPC of the E-commerce system.
- 4. In the Inbound rules section, add 1 rule:
    - Type: Custom TCP
    - Port range: 6379
    - Source: Click in the search box and select the Security Group of the EC2 Instances (ecs-sg)
- 5. Click Create security group.

![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache1.png)

**Access ElastiCache**
- 1. Go to ElastiCache > Redis clusters > Create Redis cluster.
- 2. Select Cluster cache.
- 3. Engine: Select Valkey (or Redis OSS is fine since it is fully compatible with the code).
- 4. Name: zopee-redis.
- 5. Node type: Scroll down and select cache.t4g.micro or cache.t3.micro (depending on your needs, this is the cheapest).
- 6. Number of Replicas: 0 (to save the most).
- 7. In Connectivity section: Select Create a new subnet group. Name it redis-subnet-group, select VPC, click Manage and tick only the 2 Private Subnets.
- 8. Scroll to the bottom and click Create

![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache2.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache3.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache4.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache5.png)
![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache6.png)
 
At Advanced settings
Selected security group: Select the security group created earlier - > next - > create

 ![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache7.png)
 
It will take about 5 minutes for this cluster to initialize

![ElastiCache](/images/5-Workshop/5.6-monitoring-caching/ElastiCache8.png)
 
1. After the Redis cluster shows Available status, click on its name.
2. Find the Primary endpoint line, copy that URL (Example: zopee-redis.xxxx.use1.cache.amazonaws.com:6379).
3. Access AWS Secrets Manager, open your ecommerce/production/secrets secret.
4. Click Retrieve secret value -> Edit.
5. Click Add row. In the Key box, enter REDIS_URL.
6. In the Value box, enter the following connection string: redis://[Paste the Primary Endpoint here] (example: redis://zopee-redis.xxxx.cache.amazonaws.com:6379).
7. Click Save.
