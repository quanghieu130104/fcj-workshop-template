---
title: "IAM Permissions and Roles"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Configuring IAM Permissions and IAM Roles

In our E-commerce system, security and access control are top priorities. For AWS services to communicate with each other securely without exposing sensitive data, we must use **AWS IAM Roles**.

Based on your actual deployment architecture, the system revolves around **5 core Roles**. Below is a detailed guide on how to configure the 3 most important Roles that you must create manually.

### IAM permissions and roles

**1. `ecsTaskExecutionRole`**
This is a critical Role attached to the Amazon ECS platform. In our architecture, it not only pulls Images but also grants the Container access to other storage resources.
- **How to create:** Create a Role with the `Elastic Container Service Task` Use case. Then, find and attach the following **4 managed Policies**:
  - `AmazonECSTaskExecutionRolePolicy` (For ECS to pull images and write logs)
  - `AmazonDynamoDBFullAccess` (Grants full access to the Database)
  - `AmazonSSMReadOnlyAccess` (Permission to read configuration parameters)
  - `SecretsManagerReadWrite` (Permission to read/write password and API key information)

![Permissions policies](/images/5-Workshop/5.2-IAM/ecsTaskExecutionRole.png)

![ecsTaskExecutionRole](/images/5-Workshop/5.2-IAM/ecsTaskExecutionRole.png)

**2. `lambda-order-email-role`**
This Role carries the core business logic of the Serverless system, handling asynchronous email delivery.
- **How to create:** Create a Role for the `Lambda` service.
- **Attach managed Policy:** Find and attach `AWSLambdaBasicExecutionRole` (Logging permissions).
- **Attach custom Policy:** Create a new Inline Policy (named `lambda-dynamodb-ses-policy`) with the following JSON code to grant permission to read DynamoDB Streams and send Emails via SES:
  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "dynamodb:DescribeStream",
                  "dynamodb:GetRecords",
                  "dynamodb:GetShardIterator",
                  "dynamodb:ListStreams"
              ],
              "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/order_table/stream/*"
          },
          {
              "Effect": "Allow",
              "Action": "ses:SendEmail",
              "Resource": "*"
          }
      ]
  }
  ```

**3. `ecsInstanceRole`**
This Role is attached directly to the EC2 instances within your ECS Cluster (supporting the EC2 launch type alongside Fargate).
- **How to create:** Create a Role with the `EC2` Use case.
- **Attach managed Policies:** Find and attach 2 Policies:
  - `AmazonEC2ContainerServiceforEC2Role`
  - `AmazonSSMManagedInstanceCore`
- **Attach custom Policy:** Create a new Inline Policy (named `Zopee-Backend-Permission`) with the JSON snippet below. This grants the Backend permission to access Secrets Manager, upload images to S3, and use SQS to process order queues:
  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "secretsmanager:GetSecretValue"
              ],
              "Resource": "arn:aws:secretsmanager:ap-southeast-1:*:secret:ecommerce/production/secrets-*"
          },
          {
              "Effect": "Allow",
              "Action": [
                  "s3:PutObject",
                  "s3:GetObject"
              ],
              "Resource": "arn:aws:s3:::my-app-uploads/*"
          },
          {
              "Effect": "Allow",
              "Action": [
                  "sqs:SendMessage",
                  "sqs:ReceiveMessage",
                  "sqs:DeleteMessage",
                  "sqs:GetQueueAttributes"
              ],
              "Resource": "arn:aws:sqs:ap-southeast-1:*:order-processing-queue"
          }
      ]
  }
  ```
