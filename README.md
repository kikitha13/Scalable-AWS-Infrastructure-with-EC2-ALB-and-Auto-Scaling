#  AWS EC2 Auto Scaling with ALB and Launch Template

> **Author**: Ravilla Kikitha  
> **Use Case**: DevOps | AWS Infrastructure | Load Balancing | Auto Scaling | Cost Optimization  
> **Level**: Intermediate to Advanced

---

##  Overview

This project demonstrates how to deploy a **highly available**, **fault-tolerant**, and **scalable** web infrastructure on AWS using EC2, Launch Template, Auto Scaling Group (ASG), and Application Load Balancer (ALB). The infrastructure ensures minimal downtime and automatic recovery in case of instance failures, with cost efficiency in mind.

---

## AWS Services Used and Their Purpose

|  Service |  Purpose |
|-----------|------------|
| **Amazon EC2** | Hosts the Apache HTTPD web server that displays a basic "Hi Hello" webpage. |
| **Amazon Linux 2 AMI** | Free-tier eligible prebuilt image used to launch EC2 instances. |
| **Launch Template** | Stores instance configuration used for consistent and repeatable deployments via Auto Scaling. |
| **Auto Scaling Group (ASG)** | Automatically scales EC2 instances between 1 and 2 based on health status or demand. |
| **Application Load Balancer (ALB)** | Distributes incoming traffic across healthy EC2 instances. |
| **Target Group** | Checks health of EC2 instances and ensures ALB forwards traffic only to healthy ones. |
| **Elastic IP (EIP)** | Optional static public IP address; useful when consistent IP is needed. |
| **VPC (Virtual Private Cloud)** | Isolated network environment to host all AWS resources securely. |
| **Public Subnets** | Enable internet access for ALB and EC2 instances. |
| **Security Groups** | Firewall rules that allow HTTP (port 80) and SSH (port 22) access. |

---

##  Is Elastic IP Mandatory?

>  **No, Elastic IP is not mandatory.**

- **ALB** already provides a DNS name (`http://<ALB-DNS>`) to access the application.
- **EIP** is optional and only necessary if:
  - You directly access an EC2 instance
  - You need a static IP for domain mapping (without Route 53)
- **Best Practice**: Use **ALB DNS** to avoid extra EIP charges when not required.

---

##  Workflow and Steps (with Explanations)

###  Step 1: Create VPC and Subnets
- Create a **VPC** (e.g., `10.0.0.0/16`).
- Add **2 Public Subnets** in different Availability Zones.
- Attach **Internet Gateway** to the VPC.
- Update **Route Table**: `0.0.0.0/0` via Internet Gateway.
- Enable **auto-assign public IP** for subnets.

 *Purpose:* Provides isolated and controlled networking for your AWS resources.

---

###  Step 2: Launch EC2 Instance (Web Server)
- AMI: **Amazon Linux 2**
- Instance type: `t2.micro` (Free Tier eligible)
- Security Group:
  - Allow **HTTP (80)** and **SSH (22)**
- Add User Data (startup script):
 I In bash
  #!/bin/bash
  yum update -y
  yum install httpd -y
  systemctl start httpd
  systemctl enable httpd
  echo "Hi Hello " > /var/www/html/index.html


