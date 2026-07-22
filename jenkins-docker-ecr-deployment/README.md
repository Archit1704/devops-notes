# CI/CD Pipeline: GitHub → Jenkins → Docker → Amazon ECR → AWS EC2

An automated, production-style CI/CD pipeline that builds a Docker image once and deploys it consistently across environments — following the **"Build Once, Deploy Anywhere"** principle.

Every push to the main branch triggers Jenkins to build, test, containerize, and ship the application to AWS — with zero manual intervention.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Pipeline Stages in Detail](#pipeline-stages-in-detail)
- [Why Amazon ECR?](#why-amazon-ecr)
- [Why Amazon EC2?](#why-amazon-ec2)
- [Security](#security)
- [Project Structure](#project-structure)
- [Traditional Deployment vs. This Pipeline](#traditional-deployment-vs-this-pipeline)
- [Key Takeaways](#key-takeaways)
- [What I Learned](#what-i-learned)
- [Outcome](#outcome)

---

## Overview

Manually deploying code — SSHing into a server, pulling the latest changes, and rebuilding the app on the spot — is slow, error-prone, and hard to reproduce. This project solves that problem.

Instead of building the application directly on the production server, **Jenkins builds the Docker image exactly once** and publishes it to **Amazon Elastic Container Registry (ECR)**. The deployment server then simply pulls that pre-built, already-tested image and runs it.

The result is a pipeline that is faster, safer, and far easier to roll back than a traditional deploy-by-hand workflow.

---

## Architecture

> Replace the path below with the actual image location in your repository.

![CI/CD Pipeline](images/ci-cd-pipeline.png)

---

## How It Works

```
Developer
   │  git push
   ▼
GitHub Repository
   │  webhook trigger
   ▼
Jenkins Pipeline
   ├─ Clone repository
   ├─ Build Docker image
   ├─ Authenticate with Amazon ECR
   ├─ Push image to Amazon ECR
   └─ SSH into deployment EC2 instance
          │
          ▼
   Deployment EC2 Instance
   ├─ Authenticate with Amazon ECR
   ├─ Pull latest Docker image
   ├─ Stop old container
   ├─ Remove old container
   └─ Start new container
          │
          ▼
   Application is live 🚀
```

**In short:** a code push is all it takes. GitHub notifies Jenkins, Jenkins builds and ships the image, and EC2 just runs it.

---

## Tech Stack

| Technology | Role in the Pipeline |
|---|---|
| Git | Version control |
| GitHub | Source code repository |
| GitHub Webhooks | Automatically triggers Jenkins on push |
| Jenkins | CI/CD automation server |
| Docker | Application containerization |
| Amazon ECR | Private Docker image registry |
| Amazon EC2 | Deployment / production server |
| AWS IAM | Secure authentication and authorization |
| AWS CLI | Communication with AWS services |
| SSH | Secure remote deployment |
| Ubuntu Linux | Server operating system |

---

## Pipeline Stages in Detail

### 1. Code Push
The developer commits and pushes changes to the private GitHub repository:

```bash
git add .
git commit -m "Update application"
git push origin main
```

### 2. GitHub Webhook
GitHub automatically notifies Jenkins the moment new code lands — no manual trigger required.

### 3. Jenkins Pipeline
Jenkins picks up the webhook event and runs through its stages: clone, build, authenticate, push, deploy.

### 4. Docker Build
Jenkins builds a Docker image containing the application code, runtime, dependencies, and configuration — guaranteeing the exact same environment runs everywhere.

### 5. Authentication with Amazon ECR
Jenkins authenticates using an IAM role or IAM credentials. AWS issues a short-lived authorization token, which Docker uses to log in to ECR securely — no long-lived secrets involved.

### 6. Image Push
The newly built image is tagged and pushed to ECR, e.g.:

```
project:v1
project:v2
project:latest
```

### 7. Deployment
Jenkins connects to the deployment EC2 instance over SSH, where it:

- Authenticates with Amazon ECR
- Pulls the latest image
- Stops and removes the old container
- Starts the new one

The application is now live, running the newly built image.

---

## Why Amazon ECR?

ECR is the single source of truth for Docker images in this pipeline. Rather than rebuilding on the production server, Jenkins builds once and stores the result centrally — servers just download what's already been tested.

**Benefits:**
- Centralized, versioned image storage
- "Build once, deploy anywhere" workflow
- Simple rollback (just point to a previous tag)
- Secure, IAM-based authentication
- Native integration with the rest of AWS

---

## Why Amazon EC2?

EC2's job in this architecture is intentionally narrow — it only pulls images and runs containers. It never builds application code.

That separation matters: production servers spend their CPU serving traffic, not compiling code, which makes deployments faster and more predictable.

---

## Security

This pipeline follows standard AWS security practices:

- Private GitHub repository
- IAM-based authentication (no hardcoded credentials)
- Short-lived, temporary ECR authorization tokens
- SSH-based deployment
- Private Docker image registry

---

## Project Structure

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

## Traditional Deployment vs. This Pipeline

### Traditional Deployment

```
Developer → SSH into server → git pull → docker build → docker run
```

**Drawbacks:**
- Fully manual
- Builds happen on the production server (high CPU load)
- Slower deployments
- Rollback is difficult
- Higher risk of deployment failure

### This Pipeline

```
Developer → GitHub → Jenkins → Docker Build → Amazon ECR → EC2 → Live
```

**Advantages:**
- Fully automated, triggered by a single push
- Image is built once, deployed anywhere
- Production server only ever runs pre-tested images
- Faster, more consistent deployments
- Simple rollback via image tags

---

## Key Takeaways

- Jenkins builds the Docker images.
- Amazon ECR stores them securely and centrally.
- EC2 only pulls and runs — it never builds.
- Separating build and deployment environments makes the system more reliable and easier to scale.

---

## What I Learned

Building this pipeline end-to-end gave me hands-on experience with:

- Git & GitHub workflows
- Jenkins pipeline configuration
- Docker & Dockerfile authoring
- Amazon EC2 & Amazon ECR
- AWS IAM and the AWS CLI
- SSH-based deployment automation
- Production deployment strategies and rollback planning
- Debugging real CI/CD failures

---

## Outcome

A working, production-style CI/CD pipeline that automatically builds Docker images, stores them securely in Amazon ECR, and deploys the latest version to Amazon EC2 — with deployments that are reliable, repeatable, and easy to scale.

---

⭐ **If this project was useful to you, consider giving it a star.**
