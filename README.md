# ECS Cluster with MongoDB Atlas and Fargate

This project sets up a **MongoDB Atlas** cluster integrated with an **ECS cluster** using **Fargate** as the launch type. It includes two separate services for a React frontend and a Node.js backend, each behind its own Application Load Balancer (ALB). The Node.js backend connects to the MongoDB Atlas cluster for database operations.

## Table of Contents
- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Environment Configuration](#environment-configuration)
- [Docker Images](#docker-images)
- [Setup Steps](#setup-steps)
  - [1. MongoDB Atlas Setup](#1-mongodb-atlas-setup)
  - [2. Environment Variables Setup](#2-environment-variables-setup)
  - [3. Push Docker Images to DockerHub](#3-push-docker-images-to-dockerhub)
  - [4. ECS Cluster and Fargate Setup](#4-ecs-cluster-and-fargate-setup)
  - [5. Application Load Balancer Setup](#5-application-load-balancer-setup)
- [License](#license)

## Architecture Overview
This architecture consists of:
1. **React Frontend**: Hosted in an ECS service with Fargate behind an Application Load Balancer (ALB).
2. **Node.js Backend API**: Hosted in another ECS service with Fargate behind its own ALB. The backend communicates with MongoDB Atlas.
3. **MongoDB Atlas Cluster**: A cloud-hosted MongoDB database used by the backend API for data storage.

### Key Components:
- **ECS (Fargate Launch Type)**: For serverless container orchestration.
- **Application Load Balancers (ALB)**: Two ALBs are used, one for routing frontend requests and another for backend API requests.
- **MongoDB Atlas**: Cloud-hosted MongoDB database.
- **DockerHub**: The backend and frontend Docker images are pushed to DockerHub.

## Prerequisites
- AWS account with IAM permissions for ECS, Fargate, ALB, and VPC creation.
- MongoDB Atlas account and cluster setup.
- DockerHub account for hosting the Docker images.

## Environment Configuration

Create an `env` folder containing the following environment files:

### 1. `backend.env`
This file will store the backend environment variables, including MongoDB credentials:

```bash
MONGODB_USERNAME=<your-mongodb-username>
MONGODB_PASSWORD=<your-mongodb-password>
MONGODB_URL=<your-mongodb-atlas-cluster-url>
MONGODB_NAME=<dev-or-prod>
```

### 2. `mongodb.env`
This file can store any MongoDB-specific environment variables (if needed), although the `backend.env` will generally suffice for this purpose.

## Docker Images

Both the React frontend and Node.js backend will be containerized and pushed to DockerHub.

- **React Frontend Image**: This image will be built from your React app and pushed to DockerHub.
- **Node.js Backend Image**: The backend service will be containerized and pushed to DockerHub as well.

## Setup Steps

### 1. MongoDB Atlas Setup
- Sign up for **[MongoDB Atlas](https://www.mongodb.com/cloud/atlas)**.
- Create a new **MongoDB cluster** and configure IP whitelisting to allow your ECS services to access the database.
- Obtain the connection string and replace the `<your-mongodb-atlas-cluster-url>` placeholder in `backend.env`.

### 2. Environment Variables Setup
- Create the `env` directory at the root of your project.
- Add the `backend.env` and `mongodb.env` files inside the `env` folder.
- Make sure the `backend.env` file contains your MongoDB credentials.

### 3. Push Docker Images to DockerHub
1. **Build and push the React frontend** image to DockerHub:
   ```bash
   # Navigate to your React app directory
   docker build -t <your-dockerhub-username>/react-frontend .
   docker push <your-dockerhub-username>/react-frontend
   ```

2. **Build and push the Node.js backend** image to DockerHub:
   ```bash
   # Navigate to your backend directory
   docker build -t <your-dockerhub-username>/node-backend .
   docker push <your-dockerhub-username>/node-backend
   ```

### 4. ECS Cluster and Fargate Setup
1. **Create an ECS Cluster** with **Fargate** as the launch type.
2. Define two **Task Definitions** in the ECS cluster:
   - **Frontend Task**: Use the Docker image from DockerHub for the React app.
   - **Backend Task**: Use the Docker image from DockerHub for the Node.js API.

3. **Create ECS Services**:
   - **React Frontend Service**: Use the frontend task definition, ensure it is deployed in a public subnet.
   - **Node.js Backend Service**: Use the backend task definition, deploy it in a private subnet (with outbound access to MongoDB Atlas).

### 5. Application Load Balancer Setup
1. **Create an ALB for the React Frontend**:
   - This ALB will route all traffic for the React frontend.
   - Configure appropriate listener rules to route traffic to the ECS frontend service.
   
2. **Create an ALB for the Node.js Backend**:
   - This ALB will handle all incoming API requests for the backend.
   - Ensure the ALB forwards requests to the backend ECS service, which in turn communicates with the MongoDB Atlas cluster.

### 6. Security Considerations
- **Security Groups**: Ensure that your security groups allow necessary inbound traffic to your ALBs and restrict access appropriately for your backend service to communicate only with MongoDB Atlas.
- **VPC Setup**: Configure your VPC with public subnets for the frontend and private subnets for the backend.
