---
title: "Proposal"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

In this section, you need to summarize the contents of the workshop that you **plan** to conduct.

# Serverless E-commerce Platform
## Enterprise-grade Online Shopping System on AWS

### 1. Executive Summary
The **E-commer** project is a modern e-commerce platform built on a Microservices and Serverless architecture on AWS. The system provides a comprehensive shopping experience from product management, shopping cart, and QR code payments (PayOS) to automated email notifications. The platform leverages powerful AWS services such as ECS, Lambda, DynamoDB, ElastiCache, and CloudFront to ensure high performance, automatic scalability, and superior security for end users.

### 2. Problem Statement
*Current Challenges*
Traditional e-commerce systems often struggle to handle traffic spikes during peak shopping seasons. Monolithic architectures are hard to upgrade, incur high static server costs, and lack the flexibility to integrate local payment gateways or handle asynchronous events like confirmation emails.

*The Solution*
The project addresses these issues using an AWS architecture that combines Containerization (ECS) and Serverless (Lambda). The frontend is delivered at high speed via CloudFront and S3. The backend is powered by ECS and ALB, storing data in DynamoDB and accelerating read access with ElastiCache (Redis). The checkout process is automated via PayOS, and heavy tasks like sending confirmation emails are handled asynchronously via an Event-Driven architecture using DynamoDB Streams and AWS SES. Amazon Cognito ensures robust user authentication.

*Benefits and Return on Investment (ROI)*
The flexible architecture optimizes costs based on actual traffic (Pay-as-you-go). Auto Scaling ensures the system remains highly available during holidays. The fully automated deployment process via CI/CD (GitHub Actions) saves hundreds of hours of manual operations, providing a competitive edge in releasing new features quickly.

### 3. Solution Architecture
The platform utilizes a distributed architecture with a static Frontend hosted on S3/CloudFront, making API calls to Backend containers deployed on ECS via an Application Load Balancer. Unstructured business data is stored on DynamoDB, cached by Redis. System events (e.g., order creation) trigger Lambda functions via DynamoDB Streams to handle business logic like sending emails through SES.

![E-Commerce Architecture](/images/2-Proposal/architecture.png)

*Core AWS Services Used (17 Services)*
- **Amazon VPC**: Establishes secure internal networking, Subnets, Route Tables, and Internet Gateway.
- **AWS IAM**: Manages identities, Roles, and Policies for secure access control.
- **Amazon EC2 & ALB**: Provides compute capacity and Application Load Balancer for traffic distribution.
- **Amazon ECS**: Orchestrates and runs Docker containers for the Backend.
- **Amazon ECR**: Securely stores and manages Docker container images.
- **AWS Lambda**: Serverless compute functions for handling background tasks (Event-Driven).
- **Amazon DynamoDB**: NoSQL database for business data storage and triggering Streams.
- **Amazon S3**: Hosts static Frontend application and user-uploaded files.
- **Amazon CloudFront**: Content Delivery Network (CDN) to accelerate global page load speeds.
- **Amazon API Gateway**: Securely processes and routes API requests.
- **Amazon Cognito**: Provides user sign-up, sign-in, and authentication security.
- **Amazon SES**: Automated transactional email sending service.
- **Amazon SNS / SQS**: Messaging and queuing systems supporting the Event-Driven architecture.
- **Amazon ElastiCache (Redis)**: In-memory caching to accelerate database response times.
- **AWS Systems Manager (SSM)**: Securely stores environment variables and Secret Keys via Parameter Store.
- **Amazon Route 53**: Manages DNS and custom domain routing.
- **AWS Certificate Manager (ACM)**: Provisions and manages SSL/TLS certificates for secure communication.

### 4. Technical Implementation
*Implementation Phases*
The project was developed over 12 weeks through 4 main phases:
1. *Research & Design*: Setting up AWS foundation, VPC, S3, and IAM.
2. *Programming & Containerization*: Building APIs, connecting DynamoDB, Docker packaging, and ECS setup.
3. *Frontend & Payments*: Building the UI, integrating Cognito authentication, and VietQR payment gateway (PayOS).
4. *Optimization & Automation*: Deploying the CI/CD pipeline, integrating Redis caching, configuring SSL/Domains, and the Event-driven SES system.

*Technical Requirements*
- *Frontend*: React/Next.js integrating Cognito SDK and PayOS Checkout.
- *Backend*: Node.js/Express or Python containerized with Docker, optimized for async processing.
- *Infrastructure*: Complete GitHub Actions setup to automatically build images to ECR and deploy to ECS.

### 5. Timeline & Milestones
- *Phase 1 (Weeks 1-4)*: Core infrastructure setup (VPC, EC2, S3, IAM, CloudFront, DB).
- *Phase 2 (Weeks 5-8)*: Serverless/ECS Backend development, Docker integration, and Cognito authentication.
- *Phase 3 (Weeks 9-10)*: In-depth architecture design, programming core E-commerce modules.
- *Phase 4 (Weeks 11-12)*: Finalizing CI/CD, Event-Driven SES, PayOS payment, and Caching optimization.

### 6. Budget Estimation

### Infrastructure Costs (MVP Environment)

| Service | Configuration | Estimated Cost/Month |
|---------|----------|-----------------------:|
| Amazon EC2 | 2 × t3.micro (ECS Cluster) | ~$16 – $18 |
| Amazon EBS | 2 × 30 GB gp3 | ~$5 |
| Amazon NAT Gateway | 1 NAT Gateway | ~$33 – $38 |
| Application Load Balancer (ALB) | 1 ALB + low traffic | ~$16 – $20 |
| Amazon DynamoDB | On-Demand (PAY_PER_REQUEST) | ~$1 – $5 |
| Amazon S3 | Frontend + Upload Images (~20 GB) | ~$0.5 – $1 |
| Amazon CloudFront | Traffic <100 GB | ~$1 – $3 |
| Amazon ECR | Docker Images storage | <$1 |
| Amazon Route 53 | 1 Hosted Zone | ~$0.50 |
| AWS Certificate Manager (ACM) | SSL/TLS Certificate | Free |
| Amazon Cognito | <50,000 MAU | Free |
| Amazon CloudWatch | Basic Logs & Metrics | ~$1 – $3 |
| AWS CloudTrail | 1 Trail (Management Events) | Free |
| Amazon ElastiCache (Valkey) | cache.t4g.micro | ~$10 – $13 |
| Amazon SES | ~1,000 emails/month | <$1 |
| Amazon SNS | Low notifications | <$1 |

### Total Estimated Cost

| Environment | Cost/Month |
|------------|--------------:|
| MVP        | **~$85 – $110** |

### 7. Risk Assessment
*Risk Matrix*
- Traffic Spikes: High impact, medium probability.
- Config/Secret Leaks: Extreme impact, low probability.
- Payment Gateway Failures: High impact, low probability.

*Mitigation Strategies*
- Configure Auto Scaling for ECS and use CloudFront/Redis to offload the Backend.
- Use AWS Systems Manager (Parameter Store) and GitHub Secrets to secure credentials.
- Implement automated order reconciliation and webhook fallback mechanisms.

### 8. Expected Outcomes
*Technical Improvements*: A complete, highly secure E-commerce system, fully automated from code to deployment (CI/CD), and ready to handle high loads.
*Long-term Value*: Serves as a solid core platform, easily allowing the integration of future AI/Data Analytics services to personalize the shopping experience.