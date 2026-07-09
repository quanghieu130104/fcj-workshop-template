---
title: "Network & Compute Layer Deployment"
date: 2026-07-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Core Backend Deployment (Compute & Load Balancing)

This phase is the "heart" of the entire system. In this Enterprise architecture, we will **not** use ECS Fargate (Serverless Compute). This choice optimizes costs for 24/7 operations and, crucially, supports stable, high-performance WebSocket connections. Instead, we will manage a cluster of EC2 instances (Compute Layer) using Auto Scaling and let Amazon ECS orchestrate the Docker containers running on them.

Simultaneously, to protect the API system from unauthorized access right at the "front door", the architecture integrates **Amazon Cognito** directly with the **Application Load Balancer (ALB)**.

Due to strict architectural dependencies, you **must** deploy this in the following precise order:

1. **[Create Cognito & SSM Parameter Store](5.4.1-cognito-ssm/)**: Set up the User identity management system first and store sensitive environment variables.
2. **[Initialize Infrastructure, Compute Layer & ALB](5.4.2-alb/)**: Create VPC, Security Groups, the ECS/EC2 cluster, and the ALB with authentication rules (Cognito).
3. **[Push ECR Image & Run Service](5.4.3-ecr-task/)**: Package the Backend source code into a Docker container, push it to ECR, and successfully launch it on the ECS cluster.

