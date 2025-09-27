# 🛠️ Ollama Translator Infrastructure Transition Plan

This document outlines the high-level plan for migrating the **Ollama Internal Translator** from its current monolithic EC2 setup to a scalable, production-ready architecture on AWS.

## 1. Current State (Bootstrap)

| Component | Status | Model/Instance |
| :--- | :--- | :--- |
| **Hosting** | Single EC2 Instance | `g4dn.xlarge` (GPU) |
| **Services** | Nginx (Frontend) and Ollama API (Backend) | Co-located on the same instance. |
| **Model** | **`gemma3n`** | Stored on the instance's EBS volume. |
| **Access** | Direct IP/Hostname access. | Secured only by Security Group/Network ACLs. |
| **Scalability** | **None** | Single Point of Failure (SPOF). |

***

## 2. Target State (Production)

The target architecture is designed for **security, high availability, and auto-scaling** for internal use only.

### 2.1. Frontend Decoupling

| Action | AWS Service | Rationale |
| :--- | :--- | :--- |
| **Separate UI** | **S3 & CloudFront** | Host the static HTML/JS UI (including the character count logic) on S3. Distribute via CloudFront for global caching and **mandatory HTTPS**. |

### 2.2. Backend Decoupling & Scaling

| Action | AWS Service | Rationale |
| :--- | :--- | :--- |
| **Load Balancing** | **Application Load Balancer (ALB)** | Distribute traffic to multiple Ollama EC2 instances and handle **TLS Termination**. |
| **Auto Scaling** | **Auto Scaling Group (ASG)** | Automatically launch/terminate EC2 `g4dn.xlarge` instances based on demand (e.g., CPU utilization) for high availability and cost optimization. |
| **Network Isolation**| **VPC & Private Subnets** | Place all Ollama instances in private subnets, ensuring they are **inaccessible directly** from the public internet. |

### 2.3. Security Implementation

1.  **Restrict ALB:** Configure the ALB Security Group to only allow inbound traffic on **Port 443 (HTTPS)** from your internal **VPN IP range or whitelisted corporate IPs**.
2.  **Internal SG:** Configure the Ollama Instance Security Group to only accept traffic on **Port 11434 (Ollama)** from the **ALB's Security Group**.
3.  **CORS:** Ensure the **`OLLAMA_ORIGINS`** environment variable is set to the CloudFront/custom domain URL to prevent unauthorized API calls.

***

## 3. Deployment Steps Checklist

| Phase | Task | Status |
| :--- | :--- | :--- |
| **VPC Prep** | ⬜ Create VPC with Public and Private Subnets. | |
| **Frontend** | ⬜ Upload UI files to S3. | |
| **Frontend** | ⬜ Configure CloudFront Distribution with HTTPS and CNAME (if applicable). | |
| **Backend** | ⬜ Create a 'Golden AMI' or Launch Template for EC2 instances (including Ollama, Nginx configuration, and model pull script). | |
| **Backend** | ⬜ Set up the **ALB** to listen on 443 and forward traffic to the ASG Target Group. | |
| **Backend** | ⬜ Configure the **ASG** with the Launch Template and desired scaling policies. | |
| **Security** | ⬜ Apply strict **Security Group rules** (as detailed above). | |
