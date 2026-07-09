---
title: "Create Infrastructure & Compute Layer"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# Initialize Infrastructure, Compute Layer & ALB

This lesson will guide you through initializing the entire core infrastructure to run the Backend, including preparing the network (VPC), security groups, EC2 cluster (Compute Layer), and finally the Application Load Balancer (ALB).

### Step 1: Initialize Network (VPC)
Before creating any network resources like a Load Balancer or Target Group, you must have a private network space (VPC).
- Navigate to **VPC Dashboard > Click Create VPC**.
- Select the **VPC and more** option (Automatically create VPC with Subnets):
  - **Name tag auto-generation:** Name it, e.g., `e-commerce-vpc`.
  - **IPv4 CIDR block:** `10.0.0.0/16`.
  - **Number of Availability Zones (AZs):** `2`.
  - **Number of public subnets:** `2` (Used for ALB and NAT Gateway).
  - **Number of private subnets:** `2` (Used for EC2 Backend and Database for security).
  - **NAT gateways:** Select `Zonal`, Select `1 AZ` or `2 AZ` (To save costs for the Lab, you can select 1 AZ).
  - Click **Create VPC**.

![VPC](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/VPC.png)

![VPC](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/VPC1.png)

### Step 2: Create Security Groups
We need to create 2 Security Groups to control traffic flow:
1. **`alb-sg` (For Load Balancer):** Allow Inbound HTTP (80) and HTTPS (443) from `0.0.0.0/0` (Internet).

![Security group](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Security-group.png)

2. **`ecs-ec2-sg` (For EC2 Backend):** Allow Inbound from `alb-sg` on ports `8080` (Backend) and `3000` (Frontend). Allow SSH (22) from your IP if needed.

![Security group](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Security-group1.png)

### Step 3: Initialize ECS Cluster & Auto Scaling Group
- **ECS Cluster:** Navigate to the **Amazon ECS** service, create a new cluster and name it `my-app-cluster`.
- **Infrastructure - advanced:**
  - **Auto Scaling group (ASG):** Select Create a new Auto Scaling group - advanced
  - **AMI:** Select **ECS-optimized Amazon Linux 2023** (OS pre-installed with Docker and ECS Agent).
  - **EC2 instance type:** Select `t3.micro`.
  - **EC2 instance role:** Select the **`ecsInstanceRole`** created earlier.
    - **Desired capacity:** Minimum 0, Maximum 2.
  - **Root EBS volume size:** 30.

![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS.png)

![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS1.png)

  - **SSH Key pair:** Select Create a new key pair:
    - **Name:** ecs-ec2-key
    - **Key pair type:** RSA
    - **Private key file format:** .pem
    - **Create key pair.**

![SSH Key pair](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS_key.png)

- **Network settings:**
  - **VPC:** Select `e-commerce-vpc` created in Step 1.
  - **Subnets:** Select 2 **Private Subnets**.
  - **Security Group:** Select `ecs-ec2-sg` created in Step 2.

![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ECS2.png)

- **Auto Scaling Group (ASG):**
  - After creating the ECS Cluster, **Amazon ECS will automatically create** an **Auto Scaling Group (ASG)** and **Launch Template** based on the configured parameters (AMI, EC2 Instance Type, Desired Capacity, Root EBS Volume, Network Settings, etc.).
  - The ASG is configured to deploy EC2 instances in the `e-commerce-vpc` and the 2 selected **Private Subnets**.
  - Check the **Amazon EC2 → Auto Scaling Groups** service to confirm the ASG was created successfully.

### Step 4: Create Target Groups
Tell the system where (which port) the Backend and Frontend are running.
Navigate to **EC2 > Target Groups** and create the following 2 groups (Ensure you select `e-commerce-vpc`):

1. **`backend-tg`**:
   - Target type: `Instances`
   - Protocol/Port: `HTTP` / `8080`
   - Health check path: `/api/health`
- **Register targets - recommended**: Select 2 EC2 instances from the newly created cluster.

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target1.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target2.png)

2. **`frontend-tg`**:
   - Target type: `Instances`
   - Protocol/Port: `HTTP` / `3000`
   - Health check path: `/`
- **Register targets - recommended**: Select 2 EC2 instances from the newly created cluster.

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target3.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target4.png)

![Register targets](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Target2.png)

### Step 5: Initialize Application Load Balancer (ALB)
Navigate to **EC2 > Load Balancers** > Click **Create Load Balancer** > Select **Application Load Balancer** > **Create**.
- **Load balancer name:** `my-app-alb`.
- **Scheme:** `Internet-facing` (IPv4).
- **Network mapping:** Select `e-commerce-vpc` and check 2 **Public Subnets**.
- **Security Groups:** Select the `alb-sg` Security Group created in Step 2.

![Load balancer name](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ALB.png)

![Load balancer name](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/ALB1.png)

### Step 6: Configure Listeners & Rules
This is where you direct the data flow (Routing):

**Listener HTTP (80)**
- Protocol: HTTP
- Port: 80
- Default action: Redirect to URL
- Protocol: HTTPS
- Port: 443
- Status code: HTTP_301

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
  - You must attach an SSL Certificate from AWS Certificate Manager (ACM).
  - Add the following Rules in priority order:
    1. **Cognito Auth Rule:** If `Path` is `/checkout/*` or `/api/orders/*` -> Action 1: **Authenticate** (Select the Cognito User Pool created earlier) -> Action 2: Forward to `backend-tg`.
    2. **Normal API Rule:** If `Path` is `/api/*` -> Forward to `backend-tg`.
    3. **Default Rule:** Forward all remaining traffic to `frontend-tg` (For the Frontend server to render the UI).

- **Security policy**: Default.

- **Certificate source**: Your SSL/TLS certificate created for your domain in AWS Certificate Manager (ACM).

![Listener](/images/5-Workshop/5.4-backend-ecs/5.4.2-alb/Listeners2.png)
