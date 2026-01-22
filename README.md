# AWS High Availability 3-Tier Architecture with CI/CD

## Project Overview
This project demonstrates a production-grade **3-Tier Web Architecture** deployed on AWS. It includes a completely automated **CI/CD Pipeline** using AWS CodePipeline, CodeBuild, and CodeDeploy to deploy a **React (Frontend)** and **Node.js (Backend)** application with an **RDS MySQL** database.

## Architecture
![Architecture Diagram](architecture-diagram.jpg)

### Infrastructure Highlights
* **VPC Network:** Custom VPC with Public & Private Subnets across 2 Availability Zones (Multi-AZ).
* **Security:** Strict Security Group chaining (Bastion -> ALB -> Web -> App -> DB).
* **High Availability:** Auto Scaling Groups (ASG) and Application Load Balancers (ALB) at both Web and App tiers.
* **Database:** Amazon RDS MySQL with Multi-AZ standby for fault tolerance.

---

## CI/CD Pipeline Workflow
![Pipeline Diagram](pipeline-workflow.jpg)

The deployment is automated using **AWS Developer Tools**:
1.  **Source:** GitHub (Webhooks trigger the pipeline on commit).
2.  **Build:** AWS CodeBuild packages the source code.
3.  **Deploy:** AWS CodeDeploy pushes the artifacts to EC2 instances using the **In-Place Deployment** strategy.

---

## Deployment Instructions

### 1. Infrastructure Setup
* **VPC:** Create a VPC with CIDR `192.168.0.0/16`.
* **Subnets:** Provision 2 Public (Web) and 4 Private (App + DB) subnets.
* **NAT Gateway:** Attach to Public Subnet to allow private instances to reach the internet for updates.

### 2. Database Configuration
* Launch an **RDS MySQL** instance in the private subnet.
* Store credentials securely in **AWS Systems Manager (SSM) Parameter Store**:
    * `/nodeapp/db/hostname`
    * `/nodeapp/db/password`
    * `/nodeapp/db/user`

### 3. Launching Instances (User Data)
Use the provided shell scripts to bootstrap the EC2 instances.

**Frontend (Web Tier):**
* Use `frontend-launch-script.sh`.
* *Note:* Update the `APP_TIER_ALB_URL` variable inside the script with your Internal ALB DNS.

**Backend (App Tier):**
* Use `backend-launch-script.sh`.
* This script automatically installs Node.js, PM2, and the CloudWatch Agent.

### 4. CI/CD Pipeline Configuration
* **BuildSpec:** The repository includes `buildspec.yml` files for both frontend and backend.
* **AppSpec:** The `appspec.yml` defines the deployment hooks for AWS CodeDeploy.
* **Trigger:** The pipeline automatically detects changes in the `main` branch.
