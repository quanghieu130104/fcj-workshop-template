---
title: "Create ECR & Deploy Service"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

# Create ECR Repositories & Launch ECS Service

The final step in establishing the Backend infrastructure is preparing the "vault" for your source code and commanding ECS to launch containers on your newly created EC2 cluster.

### 1. Create Docker Registries (Amazon ECR)
Navigate to **Amazon ECR** (Elastic Container Registry) and create 2 Private repositories:
- `my-app-backend`

![ECR](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECR-be.png)

- `my-app-frontend`

![ECR](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECR-fe.png)

*(In the CI/CD section, GitHub Actions will automatically build your code into Docker Images and push them into these repositories).*

### 2. Task Definition
In Amazon ECS, a **Task Definition** is the blueprint describing how one or more containers will be deployed and run on ECS.

- Navigate to **Amazon ECS** → **Task Definitions** → **Create new task definition**.
- Select **Create new task definition with JSON**.
- Paste the JSON content for the Backend Task Definition into the editor, then select **Create**.

The Backend Task Definition includes the following main configurations:

- **Task family:** `my-app`
- **Launch type compatibility:** `EC2`
- **Network mode:** `bridge`
- **Task execution role:** `ecsTaskExecutionRole`
- **Task role:** `ecsTaskExecutionRole`

Declare 2 containers:

- **Container 1 (Backend):**
  - **Container name:** `backend-container`
  - **Image:** Image URI from the Amazon ECR repository `my-app-backend`
  - **Port mapping:** `8080`

- **Container 2 (Frontend):**
  - **Container name:** `frontend-container`
  - **Image:** Image URI from the Amazon ECR repository `my-app-frontend`
  - **Port mapping:** `3000`

```json
{
  "taskDefinitionArn": "arn:aws:ecs:ap-southeast-1:405872562581:task-definition/my-app-backend:36",
  "containerDefinitions": [
    {
      "name": "backend-container",
      "image": "405872562581.dkr.ecr.ap-southeast-1.amazonaws.com/my-app-backend:5682acda3c73c584b0f07c3c180d0d7c478b5378",
      "cpu": 256,
      "memory": 512,
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080,
          "protocol": "tcp",
          "name": "backend-container-8080-tcp",
          "appProtocol": "http"
        }
      ],
      "essential": true,
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "mountPoints": [],
      "volumesFrom": [],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app-backend",
          "awslogs-create-group": "true",
          "awslogs-region": "ap-southeast-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "systemControls": []
    }
  ],
  "family": "my-app-backend",
  "taskRoleArn": "arn:aws:iam::405872562581:role/ecsTaskExecutionRole",
  "executionRoleArn": "arn:aws:iam::405872562581:role/ecsTaskExecutionRole",
  "networkMode": "bridge",
  "revision": 36,
  "volumes": [],
  "status": "ACTIVE",
  "requiresAttributes": [
    {
      "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
    },
    {
      "name": "ecs.capability.execution-role-awslogs"
    },
    {
      "name": "com.amazonaws.ecs.capability.ecr-auth"
    },
    {
      "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
    },
    {
      "name": "com.amazonaws.ecs.capability.task-iam-role"
    },
    {
      "name": "ecs.capability.execution-role-ecr-pull"
    },
    {
      "name": "com.amazonaws.ecs.capability.docker-remote-api.1.29"
    }
  ],
  "placementConstraints": [],
  "compatibilities": [
    "EC2"
  ],
  "runtimePlatform": {
    "cpuArchitecture": "X86_64",
    "operatingSystemFamily": "LINUX"
  },
  "requiresCompatibilities": [
    "EC2"
  ],
  "cpu": "256",
  "memory": "512",
  "registeredAt": "2026-07-07T11:51:38.198Z",
  "registeredBy": "arn:aws:iam::405872562581:user/e-commerce-deployer",
  "tags": []
}
```

- Navigate to **Amazon ECS** → **Task Definitions** → **Create new task definition**.
- Select **Create new task definition with JSON**.
- Paste the JSON content for the Frontend Task Definition into the editor, then select **Create**.

The Frontend Task Definition includes the following main configurations:

- **Task family:** `my-app-frontend`
- **Launch type compatibility:** `EC2`
- **Network mode:** `bridge`
- **CPU:** `256`
- **Memory:** `400`

Declare the Container:

- **Container name:** `frontend-container`
- **Image:** Image URI from the Amazon ECR repository `my-app-frontend`
- **Port mapping:** `3000`
- **Log driver:** `awslogs`
  - **Log group:** `/ecs/my-app-frontend`
  - **Region:** `ap-southeast-1`
  - **Stream prefix:** `ecs`

```json
{
  "taskDefinitionArn": "arn:aws:ecs:ap-southeast-1:405872562581:task-definition/my-app-frontend:33",
  "containerDefinitions": [
    {
      "name": "frontend-container",
      "image": "405872562581.dkr.ecr.ap-southeast-1.amazonaws.com/my-app-frontend:6064759f281011df35a96bfbf4f1c643d4bab89e",
      "cpu": 256,
      "memory": 400,
      "portMappings": [
        {
          "containerPort": 3000,
          "hostPort": 3000,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "environment": [],
      "mountPoints": [],
      "volumesFrom": [],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app-frontend",
          "awslogs-create-group": "true",
          "awslogs-region": "ap-southeast-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "systemControls": []
    }
  ],
  "family": "my-app-frontend",
  "networkMode": "bridge",
  "revision": 33,
  "volumes": [],
  "status": "ACTIVE",
  "requiresAttributes": [
    {
      "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
    },
    {
      "name": "com.amazonaws.ecs.capability.ecr-auth"
    },
    {
      "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
    },
    {
      "name": "com.amazonaws.ecs.capability.docker-remote-api.1.29"
    }
  ],
  "placementConstraints": [],
  "compatibilities": [
    "EXTERNAL",
    "EC2"
  ],
  "requiresCompatibilities": [
    "EC2"
  ],
  "cpu": "256",
  "memory": "400",
  "registeredAt": "2026-07-07T12:08:57.716Z",
  "registeredBy": "arn:aws:iam::405872562581:user/e-commerce-deployer",
  "tags": []
}
```

Once complete, select **Create** to register the Task Definition.
The Task Definition uses **ecsTaskExecutionRole** so ECS has permission to pull Docker Images from Amazon ECR and push logs to Amazon CloudWatch Logs.

### 3. Create ECS Services
After creating the Task Definition, you need to create an ECS Service to instruct Amazon ECS to maintain the desired number of running tasks and integrate with the Application Load Balancer (ALB).

- **Navigate to Amazon ECS → Clusters → my-app-cluster → Services → Create.**
- **Service details**
  - **Task definition family:** Select the Task Definition you just created (`my-app-backend` or `my-app-frontend`).
  - **Task definition revision:** Select Latest.
  - **Service name:** Name your Service.
  - **Backend:** `backend-service`
  - **Frontend:** `frontend-service`

  ![ECS1](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECS.png)

  ![ECS1](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECS1.png)

- **Compute configuration**
  - **Compute options:** Select Capacity provider strategy.
  - **Capacity provider:** Select the Capacity Provider automatically created by Amazon ECS with the Cluster (linked to the Auto Scaling Group).
  - If you are not using a Capacity Provider, you can select Launch type and choose EC2.

- **Deployment configuration**
  - **Scheduling strategy:** Replica
  - **Desired tasks:** 2

This tells ECS to always maintain 2 running tasks. If one fails, ECS will automatically launch a replacement.

- **Load balancing**
  - Enable **Use an existing load balancer** and configure:
  - **Backend Service**
    - **Load Balancer:** my-app-alb
    - **Target Group:** backend-tg
    - **Container:** backend-container
    - **Container Port:** 8080
  - **Frontend Service**
    - **Load Balancer:** my-app-alb
    - **Target Group:** frontend-tg
    - **Container:** frontend-container
    - **Container Port:** 3000
- **Other sections** (Leave default)
  - **Monitoring**
  - **Service Connect**
  - **Service Discovery**
  - **VPC Lattice**
  - **Service Auto Scaling**
  - **Volume**
  - **Tags**

- **After reviewing the information, select Create to create the ECS Service.**

Note: You will need to create 02 separate ECS Services:

**backend-service** using **Task Definition my-app-backend** and **Target Group backend-tg**.
**frontend-service** using **Task Definition my-app-frontend** and **Target Group frontend-tg**. This ensures Frontend and Backend can be managed, deployed, and scaled independently.

**Congratulations! 🎉** 
You have successfully established the entire core architecture of the E-commerce Backend. Your containers will now automatically run on EC2 instances; internet traffic will flow through the ALB, be authenticated by Cognito, and then be routed to your Backend.

Next, proceed to **Section 5.5** to integrate the Serverless Event-Driven ecosystem (DynamoDB Streams, Lambda, SES).
