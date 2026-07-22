# 🚀 CI/CD Pipeline using GitHub, Jenkins, Docker, Amazon ECR & AWS EC2

> **Automated CI/CD pipeline implementing the Build Once, Deploy Anywhere principle using Docker, Jenkins, Amazon ECR, and AWS EC2.**

---

## 📖 Overview

This project demonstrates a complete production-style **Continuous Integration and Continuous Deployment (CI/CD)** pipeline.

The pipeline automatically builds, packages, stores, and deploys a Dockerized application whenever new code is pushed to GitHub.

Instead of building the application directly on the production server, Jenkins builds the Docker image once, publishes it to **Amazon Elastic Container Registry (ECR)**, and the deployment server pulls the latest tested image from ECR.

This architecture improves deployment reliability, scalability, security, and consistency while following industry best practices.

---

# 🏗️ Architecture

> Replace the image path below with your repository image path.

```md
![CI/CD Pipeline](images/ci-cd-pipeline.png)
```

---

# 🔄 Complete Workflow

```text
Developer
     │
     ▼
Git Push
     │
     ▼
GitHub Repository
     │
     ▼
GitHub Webhook
     │
     ▼
Jenkins Pipeline
     │
     ├── Clone Repository
     ├── Build Docker Image
     ├── Authenticate with Amazon ECR
     ├── Push Docker Image to Amazon ECR
     └── SSH into Deployment EC2
                     │
                     ▼
Deployment EC2
     ├── Authenticate with Amazon ECR
     ├── Pull Latest Docker Image
     ├── Stop Old Container
     ├── Remove Old Container
     └── Run New Container
                     │
                     ▼
            🚀 Application Live
```

---

# 🎯 Project Objective

The objective of this project is to automate the software deployment lifecycle.

Instead of manually deploying code to a production server, the entire process is automated through Jenkins.

Every code push automatically:

- Builds the application
- Creates a Docker image
- Pushes the image to Amazon ECR
- Deploys the latest version to the production server

This eliminates manual deployment, reduces errors, and ensures consistent deployments.

---

# 🛠️ Technologies Used

| Technology | Purpose |
|------------|---------|
| Git | Version Control |
| GitHub | Source Code Repository |
| GitHub Webhook | Automatically triggers Jenkins |
| Jenkins | CI/CD Automation Server |
| Docker | Containerization |
| Dockerfile | Docker Image Build Instructions |
| Amazon ECR | Private Docker Image Registry |
| Amazon EC2 | Deployment Server |
| AWS IAM | Secure Authentication & Authorization |
| AWS CLI | AWS Service Communication |
| SSH | Secure Remote Deployment |
| Ubuntu Linux | Server Operating System |

---

# ⚙️ Pipeline Execution

## 1. Developer Pushes Code

The developer commits and pushes code to the private GitHub repository.

```bash
git add .
git commit -m "Update application"
git push origin main
```

---

## 2. GitHub Webhook

GitHub automatically sends a webhook event to Jenkins whenever new code is pushed.

No manual build trigger is required.

---

## 3. Jenkins Pipeline

Jenkins receives the webhook and starts the pipeline.

Pipeline stages include:

- Clone Repository
- Build Docker Image
- Authenticate with AWS
- Push Image to Amazon ECR
- Deploy to EC2

---

## 4. Docker Build

Jenkins builds a Docker image using the Dockerfile.

The image contains:

- Application Code
- Runtime Environment
- Dependencies
- Configuration

This ensures identical execution across environments.

---

## 5. Authentication with Amazon ECR

Jenkins authenticates using an IAM Role (or IAM credentials).

AWS returns a temporary authorization token.

Docker uses this token to securely log in to Amazon ECR.

---

## 6. Push Docker Image

The newly built Docker image is tagged and pushed to Amazon ECR.

Example:

```
project:v1
project:v2
project:latest
```

---

## 7. Deployment

Jenkins connects to the deployment EC2 instance through SSH.

Deployment server:

- Authenticates with Amazon ECR
- Pulls latest Docker image
- Stops old container
- Removes old container
- Starts new container

Application becomes live.

---

# 📦 Why Amazon ECR?

Amazon ECR acts as the centralized Docker image registry.

Instead of building Docker images directly on the production server, Jenkins builds the image once and stores it in ECR.

Deployment servers simply download the already-tested image.

Benefits include:

- Centralized Docker image storage
- Build once, deploy anywhere
- Easy image versioning
- Simplified rollback
- Secure IAM authentication
- Better scalability
- Native AWS integration

---

# 🖥️ Why Amazon EC2?

Amazon EC2 provides the compute environment where the application runs.

Its responsibility is only to:

- Pull Docker images
- Run containers
- Serve application traffic

The production server never builds application code.

This reduces CPU usage and improves deployment reliability.

---

# 🔒 Security

The project follows AWS security best practices.

- Private GitHub Repository
- IAM-based authentication
- Temporary ECR authorization tokens
- SSH-based deployment
- Private Docker image registry
- No hardcoded AWS credentials

---

# 📂 Project Structure

```
.
├── Dockerfile
├── Jenkinsfile
├── package.json
├── src/
├── images/
│   └── ci-cd-pipeline.png
└── README.md
```

---

# ✨ Features

- Automated CI/CD Pipeline
- Dockerized Application
- GitHub Webhook Integration
- Jenkins Automation
- Amazon ECR Integration
- Secure IAM Authentication
- Automatic Deployment
- Docker Image Versioning
- Easy Rollback
- Production-Ready Workflow

---

# 🚀 Key Benefits

✅ Fully automated deployment

✅ Zero manual deployment

✅ Build Once, Deploy Anywhere

✅ Faster deployments

✅ Reduced production server load

✅ Consistent deployments

✅ Version-controlled Docker images

✅ Simplified rollback

✅ Production-ready architecture

---

# 💡 Traditional Deployment vs CI/CD

## Traditional Deployment

```
Developer

↓

SSH into Server

↓

git pull

↓

docker build

↓

docker run
```

### Problems

- Manual deployment
- High CPU usage
- Slow deployments
- Build on production server
- Difficult rollback
- Higher risk of deployment failures

---

## CI/CD Deployment (This Project)

```
Developer

↓

GitHub

↓

Jenkins

↓

Docker Build

↓

Amazon ECR

↓

Deployment EC2

↓

Application Live
```

### Advantages

- Automated deployment
- Build only once
- Production server only runs tested images
- Faster deployments
- Better scalability
- Easy rollback
- Consistent deployments

---

# 🧠 What I Learned

Through this project, I gained hands-on experience with:

- Git & GitHub
- Jenkins Pipelines
- Docker
- Dockerfile
- Amazon EC2
- Amazon ECR
- AWS IAM
- AWS CLI
- SSH Automation
- Production Deployment Strategies
- CI/CD Best Practices
- Troubleshooting Deployment Issues

---

# 📌 Key Takeaways

- Jenkins is responsible for building Docker images.
- Amazon ECR stores Docker images securely.
- EC2 retrieves and runs Docker containers.
- Production servers should never build application code.
- Docker images provide consistent deployments.
- Separating build and deployment environments improves reliability and scalability.

---

# 🏆 Outcome

Successfully implemented a production-style CI/CD pipeline that automatically:

- Builds Docker images
- Stores images securely in Amazon ECR
- Deploys the latest version to Amazon EC2
- Keeps deployments reliable, repeatable, and scalable

---

## ⭐ If you found this project helpful, consider giving it a Star!
