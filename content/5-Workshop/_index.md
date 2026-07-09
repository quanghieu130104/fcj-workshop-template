---
title: "Workshop"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Serverless E-commerce Platform
## Enterprise-scale Online Shopping System Architecture on AWS

#### Overview

Welcome to the **Serverless E-commerce System** deployment workshop on AWS! This is not just a standard tutorial, but a comprehensive journey that simulates the deployment of an Enterprise-level software system in a real-world environment.

In this Workshop, we will solve the core challenge: *How do we build an online shopping platform capable of auto-scaling, handling high traffic during Flash Sales, securing payment data, and optimizing infrastructure costs (Pay-as-you-go)?*

Through a detailed step-by-step guide, you will manually design and connect over **17 core AWS services** across multiple architectural layers:
- **Infrastructure Layer**: Build a secure network with Amazon VPC (Public/Private Subnets) and enforce strict access control via AWS IAM.
- **Database Layer**: Deploy a high-performance NoSQL database using 12 Amazon DynamoDB tables and static storage on S3.
- **Compute Layer**: Utilize Containerization (Docker) to package the Backend API and operate it on Amazon ECS (Fargate) behind an Application Load Balancer.
- **Serverless & Event-Driven Layer**: Integrate AWS Lambda, DynamoDB Streams, Amazon SES (email delivery), and ElastiCache Redis (caching) to handle complex background asynchronous tasks.
- **DevOps & Automation Layer**: Set up a complete CI/CD Pipeline using GitHub Actions for automated building and deployment.

The ultimate goal of this Workshop is to help you master Cloud-native design thinking, become proficient with modern AWS tools, and confidently operate large-scale real-world projects. Let's get started!



#### Content

1. [Overview](5.1-Overview/)
2. [Configuring IAM Access](5.2-IAM/)
3. [DynamoDB & S3 Buckets Initialization](5.3-database-storage/)
4. [Network & Compute Layer Deployment](5.4-backend-ecs/)
5. [Serverless & Event-Driven Integration](5.5-serverless/)
6. [System Monitoring & Caching](5.6-monitoring-caching/)
7. [Global Content Delivery (CloudFront)](5.7-cloudfront/)
8. [API Gateway Integration (Upcoming)](#)
9. [Resource Cleanup](5.9-cleanup/)