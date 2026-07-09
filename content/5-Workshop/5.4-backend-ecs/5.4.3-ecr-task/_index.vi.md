---
title: "Tạo ECR & Triển khai Service"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

# Tạo ECR Repositories & Khởi chạy Service

Bước cuối cùng của việc khởi tạo hạ tầng Backend là chuẩn bị "kho chứa" mã nguồn và ra lệnh cho ECS khởi chạy các container trên cụm EC2 vừa tạo.

### 1. Tạo Kho chứa Docker (Amazon ECR)
Truy cập **Amazon ECR** (Elastic Container Registry) và tạo 2 kho chứa Private:
- `my-app-backend`

![ECR](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECR-be.png)

- `my-app-frontend`

![ECR](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECR-fe.png)

*(Ở phần CI/CD, GitHub Actions sẽ tự động build code của bạn thành các cục Docker Image và đẩy vào 2 kho chứa này).*

### 2. Định nghĩa Task (Task Definition)
Trong Amazon ECS, **Task Definition** là bản thiết kế (Blueprint) mô tả cách một hoặc nhiều container sẽ được triển khai và chạy trên ECS.

- Truy cập **Amazon ECS** → **Task Definitions** → **Create new task definition**.
- Chọn **Create new task definition with JSON**.
- Dán nội dung JSON của Task Definition Backend vào trình soạn thảo, sau đó chọn **Create**.

Task Definition Backend sẽ bao gồm các cấu hình chính sau:

- **Task family:** `my-app`
- **Launch type compatibility:** `EC2`
- **Network mode:** `bridge`
- **Task execution role:** `ecsTaskExecutionRole`
- **Task role:** `ecsTaskExecutionRole`

Khai báo 2 container:

- **Container 1 (Backend):**
  - **Container name:** `backend-container`
  - **Image:** Image URI từ Amazon ECR repository `my-app-backend`
  - **Port mapping:** `8080`

- **Container 2 (Frontend):**
  - **Container name:** `frontend-container`
  - **Image:** Image URI từ Amazon ECR repository `my-app-frontend`
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

Trong Amazon ECS, **Task Definition** là bản thiết kế (Blueprint) mô tả cách container sẽ được triển khai và chạy trên ECS.

- Truy cập **Amazon ECS** → **Task Definitions** → **Create new task definition**.
- Chọn **Create new task definition with JSON**.
- Dán nội dung JSON của Task Definition Frontend vào trình soạn thảo và chọn **Create**.

Task Definition Frontend sẽ bao gồm các cấu hình chính:

- **Task family:** `my-app-frontend`
- **Launch type compatibility:** `EC2`
- **Network mode:** `bridge`
- **CPU:** `256`
- **Memory:** `400`

Khai báo Container:

- **Container name:** `frontend-container`
- **Image:** Image URI từ Amazon ECR repository `my-app-frontend`
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

Sau khi hoàn tất, chọn **Create** để đăng ký Task Definition.
Task Definition sử dụng **ecsTaskExecutionRole** để ECS có quyền kéo Docker Image từ Amazon ECR và ghi log lên Amazon CloudWatch Logs.
### 3. Tạo ECS Services
Sau khi tạo Task Definition, cần tạo ECS Service để Amazon ECS duy trì số lượng Task luôn chạy theo cấu hình mong muốn và tích hợp với Application Load Balancer (ALB).

- **Truy cập Amazon ECS → Clusters → my-app-cluster → Services → Create.**
- **Service details**
  - **Task definition family:** Chọn Task Definition vừa tạo (my-app-backend hoặc my-app-frontend).
  - **Task definition revision:** Chọn Latest.
  - **Service name:** Đặt tên cho Service.
  - **Backend:** backend-service
  - **Frontend:** frontend-service

  ![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECS.png) 

  ![ECS](/images/5-Workshop/5.4-backend-ecs/5.4.3-ecr-task/ECS1.png)

  - **Compute configuration**
  - **Compute options:** Chọn Capacity provider strategy.
  - **Capacity provider:** Chọn Capacity Provider được Amazon ECS tự động tạo cùng Cluster (liên kết với Auto Scaling Group).
  - Nếu không sử dụng Capacity Provider, có thể chọn Launch type và chọn EC2.

- **Deployment configuration**
  - **Scheduling strategy:** Replica
  - **Desired tasks:** 2

Điều này giúp ECS luôn duy trì 2 Task đang chạy. Nếu một Task gặp sự cố, ECS sẽ tự động khởi tạo Task mới để thay thế.

- **Load balancing**
  - Bật **Use an existing load balancer** và cấu hình:
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
- **Các mục khác** (Giữ mặc định)
  - **Monitoring**
  - **Service Connect**
  - **Service Discovery**
  - **VPC Lattice**
  - **Service Auto Scaling**
  - **Volume**
  - **Tags**

- **Sau khi kiểm tra lại các thông tin, chọn Create để tạo ECS Service.**

Lưu ý: Bạn sẽ cần tạo 02 ECS Services riêng biệt:

**backend-service** sử dụng **Task Definition my-app-backend** và **Target Group backend-tg**.
**frontend-service** sử dụng **Task Definition my-app-frontend** và **Target Group frontend-tg**. Điều này giúp Frontend và Backend có thể được quản lý, triển khai và mở rộng độc lập.
