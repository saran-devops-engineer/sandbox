# AWS Infrastructure Setup and Deployment Guide

## 1. High-Level Architectural Overview

This guide describes a scalable, highly available infrastructure setup on AWS, featuring:

- A Virtual Private Cloud (VPC) spanning two Availability Zones (AZs)
- Segregated public and private subnets for security and routing
- Auto Scaling Group for dynamic app server provisioning
- A bastion (jump) host for secure SSH access to private servers
- An Application Load Balancer (ALB) for routing external traffic
- Health checks and deployment of a simple static web app

---

## 2. Key Components and Their Purposes

| Component              | Purpose                                                                 |
|------------------------|-------------------------------------------------------------------------|
| **VPC**                | Provides a logically isolated network environment                       |
| **Public Subnets**     | Host internet-facing resources (e.g., bastion host, load balancer)      |
| **Private Subnets**    | Host internal application servers                                       |
| **Internet Gateway**   | Enables internet access for public subnets                              |
| **NAT Gateways**       | Allow outbound internet access from private subnets                     |
| **Route Tables**       | Direct traffic between subnets and internet                             |
| **Launch Template**    | Template for EC2 instance configuration in Auto Scaling Group           |
| **Auto Scaling Group** | Manages EC2 instances based on demand                                   |
| **Bastion Host**       | Enables SSH access to private EC2 instances from the internet           |
| **Application Load Balancer** | Distributes incoming traffic to app servers                  |
| **Target Group**       | Contains the EC2 instances behind the Load Balancer                     |
| **Static Web App**     | Simple HTML app served via Python HTTP server                           |

---

## 3. Infrastructure Breakdown

### VPC and Networking

- **VPCs**: 1
- **Availability Zones**: 2 (e.g., us-east-1a and us-east-1b)
- **Subnets**: 4
  - 2 Public (1 per AZ)
  - 2 Private (1 per AZ)
- **Route Tables**:
  - 1 Public (routes 0.0.0.0/0 to IGW)
  - 2 Private (1 per AZ, route 0.0.0.0/0 to respective NAT Gateway)
- **Internet Gateway**: 1
- **NAT Gateways**: 2 (one per public subnet)

### Auto Scaling and EC2

1. **Launch Template**
   - Specify AMI (use latest one)
   - Configure instance type, SSH key, security group

2. **Auto Scaling Group**
   - Select launch template
   - Use **private subnets in 2 AZs**
   - No load balancer at creation
   - Define group size (min, max, desired)

3. **Verification**
   - Go to EC2 dashboard to verify that instances are deployed in selected AZs

### Bastion/Jump Server

- Deployed in **public subnet**
- Enable **Auto-assign Public IP**
- Use for SSH access to private EC2 instances
- Copy `.pem` key files to Bastion using `scp`:

```bash
scp -i path/aws-login.pem your-key.pem ubuntu@<bastion-ip>:/home/ubuntu/
