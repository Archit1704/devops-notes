# 🚀 Production-Ready CI/CD Pipeline using GitHub, Jenkins, Docker, Amazon ECR & AWS EC2

<p align="center">
  <img src="https://img.shields.io/badge/AWS-EC2%20%7C%20ECR-orange?style=for-the-badge" alt="AWS">
  <img src="https://img.shields.io/badge/Docker-Container-blue?style=for-the-badge" alt="Docker">
  <img src="https://img.shields.io/badge/Jenkins-CI%2FCD-red?style=for-the-badge" alt="Jenkins">
  <img src="https://img.shields.io/badge/GitHub-Webhook-black?style=for-the-badge" alt="GitHub">
  <img src="https://img.shields.io/badge/Ubuntu-24.04-E95420?style=for-the-badge" alt="Ubuntu">
</p>

<p align="center">
  <strong>Automated CI/CD pipeline implementing the Build Once, Deploy Anywhere principle using Docker, Jenkins, Amazon ECR, and Amazon EC2.</strong>
</p>

---

## 📑 Table of Contents

- [📖 Overview](#overview)
- [🎯 Project Objective](#project-objective)
- [🏗️ Architecture](#architecture)
- [🔄 End-to-End Workflow](#end-to-end-workflow)
- [🛠️ Technology Stack](#technology-stack)
- [⚙️ Pipeline Stages](#pipeline-stages)
- [📦 Why Amazon ECR?](#why-amazon-ecr)
- [🖥️ Why Amazon EC2?](#why-amazon-ec2)
- [🔒 Security Considerations](#security-considerations)
- [⚖️ Traditional Deployment vs This Pipeline](#traditional-deployment-vs-this-pipeline)
- [📂 Project Structure](#project-structure)
- [💼 What This Project Demonstrates](#what-this-project-demonstrates)
- [📌 Key Takeaways](#key-takeaways)
- [🚀 Future Improvements](#future-improvements)
- [🔗 Application Source Repository](#application-source-repository)
- [🏆 Outcome](#outcome)

---

## 📖 Overview

This project demonstrates a production-style **Continuous Integration and Continuous Deployment (CI/CD)** workflow for a Dockerized application using **GitHub**, **Jenkins**, **Docker**, **Amazon Elastic Container Registry (ECR)**, and **Amazon EC2**.

In many beginner or manually managed deployments, developers connect to the production server through SSH, pull the latest source code, rebuild the application on the same machine, and then restart the service. That process works for small experiments, but it introduces several practical problems:

- The production server spends CPU and memory building the application.
- Deployments depend on manual execution and are easy to break.
- Rollbacks are slower because the artifact is not managed centrally.
- Environment drift becomes more likely over time.
- Different servers may end up running slightly different builds.

This repository documents a better approach.

Instead of building directly on the production server, **Jenkins builds the Docker image once**, pushes it to **Amazon ECR**, and the deployment server simply **pulls and runs the already-built image**. This creates a more consistent, scalable, and professional deployment model.

---

## 🎯 Project Objective

The main objective of this project is to implement a reliable deployment pipeline based on clear separation of responsibilities:

- **GitHub** stores the application source code.
- **Jenkins** handles automation and image creation.
- **Docker** packages the application into a portable artifact.
- **Amazon ECR** stores the versioned image securely.
- **Amazon EC2** runs the final container in production.

This structure follows the **Build Once, Deploy Anywhere** principle.

That principle matters because the same Docker image that Jenkins builds is the exact image that gets deployed. There is no rebuild on the EC2 server, which reduces inconsistency and makes the release process easier to trust.

---

## 🏗️ Architecture

![CI/CD Pipeline Architecture](architecture.png)

The architecture is intentionally simple and production-oriented.

A code push to GitHub triggers Jenkins. Jenkins then builds a Docker image, authenticates with Amazon ECR, and pushes the image to the private registry. After that, Jenkins connects to the EC2 instance through SSH so the server can pull the latest image and start the new container.

This design keeps the production server focused on **running containers**, not building them.

---

## 🔄 End-to-End Workflow

```text
Developer
   |
   v
Git Push
   |
   v
GitHub Repository
   |
   v
GitHub Webhook
   |
   v
Jenkins Pipeline
   |
   +--> Clone repository
   +--> Build Docker image
   +--> Authenticate to Amazon ECR
   +--> Push image to Amazon ECR
   +--> Connect to EC2 over SSH
                         |
                         v
                  Deployment EC2
                   +--> Authenticate to ECR
                   +--> Pull latest image
                   +--> Stop old container
                   +--> Remove old container
                   +--> Run new container
                         |
                         v
                  Application Live
```

### ✅ Workflow Summary

1. Developer pushes code changes to GitHub.
2. GitHub webhook notifies Jenkins automatically.
3. Jenkins starts the CI/CD pipeline.
4. Jenkins builds a Docker image from the latest source code.
5. Jenkins pushes the image to Amazon ECR.
6. Jenkins connects to the EC2 server using SSH.
7. EC2 pulls the latest image and starts the updated container.
8. The application becomes live with the new version.

---

## 🛠️ Technology Stack

| Technology | Role in the Project |
|------------|---------------------|
| Git | Version control |
| GitHub | Source code hosting |
| GitHub Webhooks | Automatic Jenkins trigger |
| Jenkins | CI/CD orchestration |
| Docker | Application packaging and runtime consistency |
| Amazon ECR | Private container image registry |
| Amazon EC2 | Deployment target |
| AWS IAM | Access control for AWS resources |
| AWS CLI | Command-line interaction with AWS |
| SSH | Secure remote deployment |
| Ubuntu Linux | Base operating system |

### 💡 Why this stack works well

This combination is practical because it separates source management, build automation, artifact storage, and runtime hosting into distinct layers. That reduces operational confusion and makes the pipeline easier to maintain, debug, and extend later.

---

## ⚙️ Pipeline Stages

### 1. 📤 Source Update

A developer pushes changes to the main branch of the GitHub repository.

```bash
git add .
git commit -m "Update application"
git push origin main
```

This push acts as the starting event for the entire workflow.

### 2. 🪝 GitHub Webhook Trigger

GitHub sends a webhook event to Jenkins whenever new code is pushed. This removes the need for someone to log into Jenkins and click a manual build button.

That matters because automation should start from the source of truth: the repository.

### 3. 🏗️ Jenkins Build Stage

Jenkins checks out the latest source code and builds a Docker image that contains:

- Application code
- Runtime environment
- Dependencies
- Startup configuration

Because the image is created in one controlled environment, every deployment uses the same build artifact.

### 4. 🔐 Amazon ECR Authentication Stage

Before Jenkins can push the image, it must authenticate with AWS and Amazon ECR.

This is typically handled using:

- IAM users with scoped permissions, or
- IAM roles where available

AWS provides a temporary authentication token, and Docker uses that token to log in to the ECR registry securely.

### 5. 📦 Amazon ECR Push Stage

Once the image is built and authentication is complete, Jenkins tags and pushes the image to Amazon ECR.

Typical tag examples:

```text
project:v1
project:v2
project:latest
```

This creates a registry-backed deployment artifact that can be reused, tracked, and rolled back if necessary.

### 6. 🚚 Deployment Stage on EC2

After the image is available in ECR, Jenkins connects to the EC2 instance over SSH.

The EC2 server then performs the runtime deployment steps:

- Authenticates with Amazon ECR
- Pulls the latest image
- Stops the old running container
- Removes the previous container
- Starts a new container from the latest image

This makes the deployment server responsible only for **pulling and running** the container, which is exactly what a production host should do.

---

## 📦 Why Amazon ECR?

Amazon ECR serves as the centralized private registry for Docker images in this project.

Instead of rebuilding images on every server, Jenkins builds the image once and stores it in ECR. Any deployment target can then pull the exact same artifact.

### ✅ Benefits of Amazon ECR

- Centralized Docker image storage
- Better version control for deployment artifacts
- Easier rollback to previous image tags
- Secure authentication using AWS IAM
- Native integration with other AWS services
- Cleaner separation between CI and production runtime

ECR is an important part of making this pipeline repeatable and production-friendly.

---

## 🖥️ Why Amazon EC2?

Amazon EC2 acts as the deployment target where the application actually runs.

Its responsibilities are intentionally limited to:

- Pulling Docker images
- Running containers
- Serving application traffic
- Replacing old containers during deployments

The EC2 instance is not used as a build machine. That design decision reduces resource waste and avoids coupling production runtime behavior with build-time logic.

### ✅ Benefits of using EC2 in this setup

- Easy to provision and manage
- Full control over the server environment
- Works well for SSH-driven deployments
- Good fit for learning and intermediate production-style setups

---

## 🔒 Security Considerations

This project follows common AWS and CI/CD security practices.

### 🔐 Security practices applied

- Use a private GitHub repository when appropriate.
- Avoid hardcoded AWS credentials in source code.
- Store secrets in Jenkins credentials.
- Use IAM users or IAM roles with limited permissions.
- Use temporary ECR authentication tokens.
- Restrict SSH access to trusted hosts and users.
- Keep the container registry private.

### ⚠️ Why this matters

A CI/CD pipeline is not just about automation. It is also a privileged path into production. If Jenkins credentials, SSH access, or AWS permissions are handled poorly, the deployment system becomes a major security risk. That is why secret handling and permission scoping are part of the architecture, not an afterthought.

---

## ⚖️ Traditional Deployment vs This Pipeline

### 🧱 Traditional Deployment

```text
Developer
   |
   v
SSH into server
   |
   v
git pull
   |
   v
docker build
   |
   v
docker run
```

### ❌ Limitations of the traditional model

- Manual operational steps
- Build workload on the production server
- Higher risk of mistakes during deployment
- Slower and less repeatable releases
- Harder rollback process
- More environment inconsistency over time

### 🚀 This CI/CD Pipeline

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   v
Docker Build
   |
   v
Amazon ECR
   |
   v
Deployment EC2
   |
   v
Application Live
```

### ✅ Advantages of this pipeline

- Automated end-to-end deployment
- Same image used from build to production
- Faster and more consistent releases
- Easier version control and rollback
- Cleaner separation of concerns
- Reduced operational load on the production host

---

## 📂 Project Structure

```text
.
├── README.md
├── architecture.png
└── ChatGPT Image Jul 22, 2026, 12_50_08 PM.png
```

### 📝 Notes on structure

- `README.md` contains the full project explanation.
- `architecture.png` contains the main architecture diagram used in this documentation.
- `ChatGPT Image Jul 22, 2026, 12_50_08 PM.png` is an additional image asset currently present in the repository.

---

## 💼 What This Project Demonstrates

This project demonstrates practical DevOps and deployment concepts such as:

- GitHub to Jenkins webhook integration
- Docker image creation in CI
- Secure image storage in Amazon ECR
- Remote deployment to EC2 using SSH
- Separation of build and runtime responsibilities
- Repeatable container-based release workflows
- Production-style CI/CD thinking

It is a strong example of how to move from manual deployment habits to a cleaner and more maintainable release process.

---

## 📌 Key Takeaways

- Jenkins is responsible for building the application artifact.
- Docker makes the deployment artifact portable and consistent.
- Amazon ECR acts as the private registry for image storage.
- Amazon EC2 is responsible for running, not building, the application.
- Separating CI from runtime improves reliability and operational clarity.
- Centralized image storage makes rollback and version tracking easier.

---

## 🚀 Future Improvements

This project can be extended further with more advanced DevOps practices:

- Add automated test stages before publishing images.
- Add linting and static analysis in Jenkins.
- Add image vulnerability scanning.
- Manage infrastructure with Terraform.
- Introduce monitoring using Prometheus and Grafana.
- Add blue-green or canary deployments.
- Move to Kubernetes for orchestration when scale requires it.
- Add notifications for deployment success and failure.

These improvements would make the pipeline even more production-ready and closer to enterprise delivery workflows.

---

## 🔗 Application Source Repository

The application source code used with this deployment workflow is maintained separately:

`https://github.com/Archit1704/project-repo`

If needed later, this section can be expanded with repository details such as the application stack, branch strategy, or environment-specific deployment notes.

---

## 🏆 Outcome

This project successfully demonstrates a professional CI/CD pipeline where:

- Code is pushed to GitHub
- Jenkins automatically builds the Docker image
- Amazon ECR stores the image securely
- Amazon EC2 pulls and runs the latest version

The final result is a deployment process that is **repeatable**, **cleaner to operate**, **less manual**, and **closer to real production practices** than traditional source-based deployment on the server.

It clearly shows not only **what was built**, but also **why this architecture is useful** in modern DevOps workflows.
