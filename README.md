# 📌 AWS EC2 Auto Scaling with ALB and Launch Template

> **Author:** Ravilla Kikitha  
> **Use Case:** DevOps | AWS Infrastructure | Web Hosting | Auto Scaling  
> **Level:** Beginner to Intermediate  

---

## 📖 Description

This project demonstrates how to build a **highly available, scalable web infrastructure** on AWS using:

- **EC2 with Amazon Linux and HTTPD (Apache Web Server)**
- **Application Load Balancer (ALB)**
- **Launch Template**
- **Auto Scaling Group**
- **Elastic IP**
- **VPC with Subnets**
- **Target Groups with Health Checks**

A sample HTTP message ("Hi Hello") is displayed when accessing the public IP or Load Balancer URL, showcasing that the auto-scaled infrastructure is up and running.

---

## 🛠️ AWS Services Used

| Service | Purpose |
|--------|---------|
| **EC2** | To host the web application (Amazon Linux + HTTPD) |
| **Amazon Linux AMI** | Used for EC2; lightweight, stable, and cost-effective |
| **HTTPD (Apache)** | To serve a sample web page (`Hi Hello`) |
| **Launch Template** | Blueprint to launch EC2 instances with pre-configured settings |
| **Auto Scaling Group** | Automatically scale EC2 instances (min: 1, max: 2) |
| **Application Load Balancer (ALB)** | Distributes incoming traffic to healthy instances |
| **Target Group** | Manages health checks and routes traffic to EC2 |
| **Elastic IP (EIP)** | Static public IP for consistent access to the EC2 instance |
| **VPC** | Isolated network environment for resources |
| **Subnets (Public)** | Allows internet-facing communication for EC2 and ALB |

---

## 🌐 Why This Project?

This project is a **practical use case of real-world DevOps infrastructure**, showing how to:
- Achieve **high availability** with Auto Scaling
- Ensure **reliability** with Load Balancing and Health Checks
- Maintain **consistent access** with Elastic IP
- Prepare for **disaster recovery** and **cost-efficient scaling**

---

## 🔁 Project Workflow (Visual)

```
Internet
   │
   ▼
[Application Load Balancer]
   │
   ▼
[Target Group]
   │
   ▼
[Auto Scaling Group]
   │
   ├── EC2 Instance 1 (Amazon Linux + HTTPD)
   └── EC2 Instance 2 (optional, on demand)
   │
   ▼
[Elastic IP (bound to initial instance for direct access)]
```

---

🧱 Step-by-Step Infrastructure Setup
🧰 Optional: Custom VPC Setup (Beginners Can Skip This)
If you're already using the default VPC, you may skip this section.
However, creating a custom VPC helps in understanding how AWS networking components work together.

🧱 Steps to Set Up a Custom VPC:
Create a VPC

CIDR block: 10.0.0.0/16

Name: MyVPC or similar

Create a Public Subnet

CIDR block: 10.0.1.0/24

Availability Zone: Choose one (e.g., ap-south-1a)

Create and Attach an Internet Gateway

Go to VPC → Internet Gateways

Create a new gateway and attach it to your VPC

Create a Route Table

Add a route: 0.0.0.0/0 → Internet Gateway

Associate the route table with the public subnet

Enable Auto-assign Public IP

When launching EC2 in this subnet, ensure public IP assignment is enabled

🔍 Why Use a Custom VPC?
Full control over your network (IP ranges, routing, firewall)

Practice real-world cloud architectures

Segregate environments (dev, test, prod)

✅ This setup enables EC2 instances in the public subnet to access the internet.

---

2️⃣ Launch an EC2 Instance (Amazon Linux)
AMI: Use Amazon Linux 2023 (free tier eligible)

Instance Type: t2.micro (or any desired type)

Security Group:

Inbound: Allow SSH (22) and HTTP (80)

User Data Script (automates Apache install & web content):

bash
Copy
Edit
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "Hi Hello from EC2 instance" > /var/www/html/index.html
Verify EC2 is accessible via its public IP on port 80
---

3️⃣ Allocate and Attach Elastic IP
Go to EC2 → Elastic IPs → Allocate Address

Allocate a new Elastic IP

Associate this IP to the EC2 instance

🔁 Why?

Keeps the public IP constant across reboots

Useful for DNS, remote access, and monitoring consistency

Can be quickly reassigned if the instance fails or is replaced
---

4️⃣ Create a Launch Template
Navigate to EC2 → Launch Templates → Create New

Use configuration from the previous EC2:

Same AMI, instance type, security group

Paste the same User Data Script

Enable instance monitoring (optional)

🔄 This template will be used by the Auto Scaling Group to launch identical EC2s.
---

5️⃣ Create a Target Group
Navigate to EC2 → Target Groups → Create

Target Type: Instance

Protocol: HTTP

Port: 80

Health check path: /

Register your test EC2 instance to verify health check logic

💡 Health checks keep track of instance health and remove unhealthy instances from load balancer routing.
---

6️⃣ Create an Application Load Balancer (ALB)
Navigate to Load Balancers → Create ALB

Type: Application Load Balancer

Scheme: Internet-facing

Listeners: HTTP on port 80

AZs: Select 2+ public subnets in different AZs

Attach to the Target Group from previous step

🌐 ALB automatically distributes incoming traffic across healthy EC2s in the group.
---

7️⃣ Create an Auto Scaling Group
Navigate to Auto Scaling Groups → Create ASG

Select the Launch Template

Attach the Target Group

Set:

Desired capacity: 1

Minimum: 1

Maximum: 2

Enable health check replacement and scaling policies (optional)

Select both public subnets for instance placement


🔁 ASG will launch, terminate, and replace instances as needed for high availability.

8️⃣ Test and Validate Setup
Access the Elastic IP in the browser → should show:
Hi Hello from EC2 instance

Access the ALB DNS name in the browser → same response

Manually terminate instance from EC2 dashboard:

Auto Scaling Group should automatically launch a replacement

Watch health status in Target Group
---
🖼️ Project Architecture Diagram
pgsql
Copy
Edit
User Request
     ↓
+-------------------------+
| Application Load Balancer |
+-------------------------+
       ↓
  Target Group
       ↓
 Auto Scaling Group
   ↓        ↓
EC2-1     EC2-2
 (Amazon Linux + Apache)
   ↓
Public Subnet → VPC → Internet Gateway
💡 Use Cases
Hosting lightweight or production-grade web applications

Automated recovery using health checks and scaling

Building reliable, fault-tolerant AWS architectures

DevOps/CI-CD pipelines for deploying infrastructure-as-code

⚙️ Elastic IP – Use & Advantages
🔹 Fixed Public IP Address – Useful for consistent external access (e.g., browser or SSH)

🔹 DNS Mapping – Avoids need to update records after stop/start

🔹 Auto Scaling Recovery – Can be manually or programmatically re-associated with new instance

🔹 Reliable Monitoring – Monitoring agents/tools can use fixed IPs

💸 Cost Optimization Tips
Use Amazon Linux 2023 AMI (free tier eligible)

Choose t2.micro or t3.micro (within free tier or low cost)

Keep min=1 and max=2 for Auto Scaling

Terminate test resources when not in use

Set up CloudWatch alarms to monitor usage and costs

Use Savings Plans or Reserved Instances for long-term deployments

✅ Final Output
Web server accessible via Elastic IP and ALB DNS

Automatic instance replacement when failed

Load balancing across instances if scaled up

Easily replicable and extensible architecture
