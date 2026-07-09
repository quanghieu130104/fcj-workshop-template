---
title: "Introduction"
date: 2026-07-08
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---


Before hands-on configuring any services, understanding the overall architecture is crucial.

The **E-commer Serverless Platform** project leverages the power of over 17 different AWS services to create a highly decoupled, secure, and cost-optimized system.

### 1. System Architecture Diagram

Below is the architecture diagram depicting the data flow of the entire system, from when a user accesses the website to when their order is processed and stored.

![System Architecture](/images/5-Workshop/5.1-Overview/architecture.png)


---

### 2. List of 17 AWS Services Deployed in this Workshop

The system is divided into 5 main architectural layers. Here is a summary of the services you will directly configure in this Workshop:

| Architecture Layer | AWS Service | Role in the E-commerce System |
| :--- | :--- | :--- |
| **Security & Network** | **Amazon VPC** | Builds the Virtual Private Cloud (Private/Public Subnets, NAT Gateway). |
| | **AWS IAM** | Manages access permissions for containers and services (Roles, Policies). |
| | **AWS Certificate Manager (ACM)** | Provides free SSL certificates for HTTPS encryption. |
| | **Amazon Route 53** | DNS service (if using custom domains). |
| **Storage & Database** | **Amazon DynamoDB** | Ultra-fast NoSQL database storing 13 data tables (User, Product, Order...). |
| | **Amazon S3** | Stores static Frontend source code (React/Next.js) and media assets. |
| **Compute & Backend** | **Amazon ECR** | Container registry storing Backend Docker Images. |
| | **Amazon ECS (Fargate)** | Runs Backend Containers entirely Serverless (no server management). |
| | **Application Load Balancer** | Distributes user traffic across ECS Tasks. |
| **Event-Driven Processing**| **AWS Lambda** | Serverless compute function triggered to handle background events. |
| | **DynamoDB Streams** | Captures data change events (e.g., someone just paid for an order). |
| | **Amazon SES** | Automatically sends payment/order confirmation emails to users. |
| | **Amazon SNS/SQS** | (Extended) Message queuing and asynchronous message processing. |
| **Performance & Caching**| **Amazon CloudFront** | Content Delivery Network (CDN) enabling fast global loading. |
| | **Amazon ElastiCache** | Redis server accelerating product list queries. |
| **Identity Management** | **Amazon Cognito** | Registration, Login, and user session management via JWT Tokens. |
| **Secure Configuration** | **AWS Systems Manager (SSM)** | SSM Parameter Store securely stores environment variables (API keys). |

---

### 3. Implementation Workflow

In the upcoming sections, we will configure each service using a Bottom-Up approach:
1. Build the Network & Database foundation (VPC, DynamoDB, S3).
2. Build the Backend core (Docker, ECR, ECS).
3. Integrate the Serverless ecosystem (Lambda, Redis, SES).
4. Expose the system externally (CloudFront, Cognito, Frontend).

