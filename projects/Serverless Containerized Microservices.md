# Project 4: Serverless Containerized Microservices (AWS Fargate & Docker)

## 🚀 Project Overview
In this project, I architected and deployed a highly available, serverless containerized web application using **Docker, Amazon ECR, and AWS ECS Fargate**. 

This project marks the culmination of my manual AWS CLI provisioning training. It demonstrates advanced proficiency in Linux system administration, container network isolation, IAM Zero-Trust security, and AWS VPC networking. By utilizing AWS Fargate instead of AWS Lambda, I provisioned a "Marathon Runner" compute environment capable of continuous, 24/7 HTTP listening without the 15-minute execution limits of traditional serverless functions.

## 🏗️ Architecture Diagram
*(This diagram illustrates the Zero-Trust data flow, container lifecycle, and network isolation of the architecture).*


<img src="https://github.com/officialhunter2007-netizen/cloud-projects/blob/main/projects/Screenshot%202026-07-22%20102050.png" width="800" alt="Architecture Diagram">





## 🧠 Technologies & Concepts Mastered
*   **Container Network Namespaces:** Leveraged Docker to isolate application environments, utilizing virtual network interfaces and ephemeral ports.
*   **Amazon ECR (Elastic Container Registry):** Managed private container image repositories and handled secure Docker daemon authentication via AWS STS.
*   **AWS ECS Fargate:** Provisioned serverless compute engines (`awsvpc` network mode, 0.25 vCPU, 512MB RAM) to run containers without managing underlying EC2 instances.
*   **Defense in Depth (Security):** Implemented dual-layer security by combining **Security Groups** (TCP/IP network traffic control) with **IAM Roles** (AWS API permission control).
*   **Linux System Administration:** Managed host memory allocation and Swap space to protect production servers from the Kernel OOM (Out of Memory) Killer.

## 📋 Step-by-Step Implementation

1. **Host Safety & Dockerization:** Ran diagnostic checks on a live Linux VPS. To protect existing production applications from the Linux OOM Killer during heavy image builds, I manually provisioned a 2GB Swap file. Wrote a Python Flask application and a highly optimized `Dockerfile` (`python:3.12-slim`).
2. **Local Isolated Testing:** Built the Docker image and ran it locally. Utilized explicit port mapping (`-p 9090:80`) to drill a secure hole through the container's network namespace without conflicting with the host's existing web servers.
3. **The Cloud Harbor (Amazon ECR):** Provisioned an ECR repository via the AWS CLI. Authenticated the local Docker engine, tagged the image with the AWS account's routing address, and pushed the layers to the cloud.
4. **Zero-Trust Security (IAM):** Forged an IAM Task Execution Role. Wrote a strict Trust Policy allowing only `ecs-tasks.amazonaws.com` to assume the role, granting Fargate the legal authority to pull the image and stream logs to CloudWatch.
5. **Networking & Fargate Deployment:** Queried the default VPC and Subnets. Provisioned a Security Group to act as the network bouncer (allowing inbound Port 80). Registered the ECS Task Definition (The Blueprint) and launched the Fargate task, assigning it a dedicated Public IP address.
<img src="https://github.com/officialhunter2007-netizen/cloud-projects/blob/main/projects/Screenshot%202026-07-22%20225917.png" width="800" alt="Architecture Diagram">

<img src="https://github.com/officialhunter2007-netizen/cloud-projects/blob/main/projects/Screenshot%202026-07-22%20230021.png" width="800" alt="Architecture Diagram">

## 🛑 Deep Architectural Challenges & Solutions

### 1. Bypassing Local Firewalls (Packet Loss)
*   **The Problem:** Initially attempted to build the environment on a local Windows machine using WSL2. Encountered strict corporate/antivirus firewall rules dropping ICMP and TCP packets (`100% packet loss`), preventing the AWS CLI from downloading.
*   **The Solution:** Pivoted to a remote Linux VPS via SSH, demonstrating adaptability, remote server management, and an understanding of NAT/Virtual Switch failures in Windows environments.

### 2. Managing the Blast Radius (Port Conflicts)
*   **The Problem:** The remote VPS was already hosting a complex, production web application on Port 80. Running the Docker container locally on Port 80 would have caused a fatal port collision.
*   **The Solution:** Utilized Docker's port mapping (`-p 9090:80`) to completely isolate the container's network namespace from the host's production traffic, ensuring zero downtime for the existing application.

## ⏭️ Next Steps: The Transition to IaC
This project represents my final manual deployment using the AWS CLI. Having mastered the underlying API mechanics, JSON configurations, and networking rules, **my next phase of development will focus entirely on Infrastructure as Code (IaC) using Terraform.** Moving forward, all infrastructure will be declarative, state-managed, and version-controlled.
