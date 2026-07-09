---
title: "Resource Cleanup"
date: 2026-07-09
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---


After completing the workshop, cleaning up AWS resources is extremely important to **avoid unexpected charges**.
You must delete resources in the **reverse order** of creation to prevent dependency errors.

Follow the steps below sequentially:

### 1. Delete CloudFront Distribution
1. Navigate to **CloudFront** > **Distributions**.
2. Select the `zopee-ecommerce` distribution and click **Disable**. (This takes about 3-5 minutes).
3. Once the status changes to *Disabled*, select it again and click **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/CloudFront.png)

### 2. Delete Amazon ElastiCache, CloudWatch and SNS
1. Navigate to **Amazon ElastiCache** → **Valkey caches** → Select the created Cache → **Actions** → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Vakey.png)

2. If you created a separate Subnet group, navigate to **Subnet groups** → Select the subnet group → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Subnetgroup.png)

3. Navigate to **Amazon CloudWatch** → **Alarms** → Select the created CPU Alarm → **Actions** → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/CloudWatch.png)

4. Navigate to **Amazon SNS** → **Topics** → Select the `system-alerts` Topic → **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/SNS.png)

### 3. Delete Amazon ECS & ECR
1. Navigate to **Amazon ECS** > **Clusters** > Select `my-app-cluster`.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/ECS.png)
2. Go to the **Services** tab, select the running Services, and click **Delete** (Or update desired tasks to 0 before deleting).
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Cluster.png)
3. After the Services are deleted, click the **Delete Cluster** button in the top right corner.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Cluster.png)
4. Navigate to **Amazon ECR** > **Repositories**, select the Frontend/Backend repositories, and click **Delete**.
![CloudFront Delete](/images/5-Workshop/5.9-cleanup/ECR.png)


### 4. Delete Load Balancer & Auto Scaling Group
1. Navigate to **EC2** > **Load Balancers** > Select `my-app-alb` > **Actions** > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/ALB.png)

2. Go to **Target Groups** > Select `frontend-tg` and `backend-tg` > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/TG.png)

3. Go to **Auto Scaling Groups** (at the bottom of the left menu) > Select the ECS ASG > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Auto.png)

4. Check **Instances** to ensure all EC2 instances are *Terminated*.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/EC2.png)

### 5. Delete Cognito & Route 53 (Optional)
1. Navigate to **Amazon Cognito** > **User Pools** > Select your User Pool > Click **Delete** (Confirm by typing the requested text).

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Cognito.png)

2. *(If you don't want to keep the domain)* Navigate to **Route 53** > **Hosted Zones** > Select the `zopee.xyz` zone > **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Route53.png)

### 6. Delete VPC & Security Groups
1. Navigate to **VPC** > **NAT Gateways** > Select your NAT Gateway > Click **Delete NAT gateway**. *(Wait 2-3 minutes for it to be fully deleted)*.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Nat.png)

2. Navigate to **VPC** > **Elastic IPs** > Select the EIP attached to the NAT Gateway > **Actions** > **Release Elastic IP addresses**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/Elastic_IPs.png)

3. Navigate to **VPC** > **Your VPCs** > Select `e-commerce-vpc` > Click **Actions** > **Delete VPC**. (This automatically deletes associated Subnets, Route Tables, and Internet Gateway).

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/VPC.png)

4. *(If any remain)* Navigate to **EC2** > **Security Groups** > Delete `alb-sg`, `ecs-ec2-sg`, and `redis-sg`.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/SG.png)


### 7. Delete DynamoDB & S3 Buckets
1. Navigate to **DynamoDB** > **Tables** > Select your table > Click **Delete**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/DynamoDB.png)

2. Navigate to **Amazon S3** > **Buckets**.

![CloudFront Delete](/images/5-Workshop/5.9-cleanup/S3bucket.png)

3. To delete a bucket, you must empty it first: Select the Bucket > Click **Empty** (Type *permanently delete* to confirm).
4. Once empty, select the Bucket again > Click **Delete**.



