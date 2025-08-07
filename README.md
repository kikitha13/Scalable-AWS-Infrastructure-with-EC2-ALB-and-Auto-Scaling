#  AWS EC2 Auto Scaling with ALB and Launch Template

> **Author**: Ravilla Kikitha  
> **Level**: Beginner to Intermediate  
> **Use Case**: DevOps | Infrastructure as Code | High Availability | Auto Recovery  

---

##  Project Overview

This project demonstrates a scalable and cost-optimized architecture on AWS that:
- Uses **Amazon EC2 instances** with **Auto Scaling**
- Serves traffic via an **Application Load Balancer (ALB)**
- Uses a **Launch Template** to auto-deploy new instances with Apache (`httpd`)
- Assigns a **static Elastic IP** to the ALB for consistent public access
- Operates within a custom **VPC and Subnets** for better network control

---

##  Services Used

| Service | Purpose |
|--------|---------|
| **Amazon EC2** | Launch virtual servers to host the web app |
| **Amazon VPC** | Create isolated network with custom IP ranges |
| **Subnets** | Host EC2s and ALB across availability zones |
| **Internet Gateway** | Provide internet access to public subnet |
| **Route Tables** | Route traffic between subnets and the internet |
| **Security Groups** | Control inbound/outbound traffic |
| **Elastic IP** | Assign a static public IP to Load Balancer |
| **Launch Template** | Define EC2 config for Auto Scaling |
| **Application Load Balancer (ALB)** | Distribute incoming traffic |
| **Target Groups** | Monitor and route to healthy EC2s |
| **Auto Scaling Group** | Automatically add/remove EC2s |

---
##  Why This Project?

This project is a **practical use case of real-world DevOps infrastructure**, showing how to:
- Achieve **high availability** with Auto Scaling
- Ensure **reliability** with Load Balancing and Health Checks
- Maintain **consistent access** with Elastic IP
- Prepare for **disaster recovery** and **cost-efficient scaling**

---

##  Project Workflow (Visual)

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

##  Step-by-Step Infrastructure Setup

###  Optional: Custom VPC Setup (Beginners Can Skip This)
If you're already using the default VPC, you may skip this section.
However, creating a custom VPC helps in understanding how AWS networking components work together.
## Steps to Set Up a Custom VPC:
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

### Why Use a Custom VPC?
Full control over your network (IP ranges, routing, firewall)

Practice real-world cloud architectures

Segregate environments (dev, test, prod)
---
### Step:1 Launch EC2 Instance (Amazon Linux)
 EC2 Instance Creation (Amazon Linux)
In this step, we launch a base EC2 instance that will later serve as the foundation for creating a Launch Template and an Auto Scaling Group.

 Step-by-Step:
Launch EC2 Instance
![WhatsApp Image 2025-08-06 at 17 02 03_6d4a2616](https://github.com/user-attachments/assets/d6b72bca-3b3c-49f0-9fd5-daabfdffcf7d)


AMI: Amazon Linux 2023 (Free Tier eligible)

Instance Type: t2.micro or t3.micro (cost-efficient)

Key Pair: Select an existing key or create a new one for SSH access

Network Settings:

VPC: Use default or custom

Subnet: Choose a public subnet

Auto-assign public IP: Enable

Security Group: Create or select one with the following rules:

Type	Protocol	Port Range	Source
SSH	TCP	22	My IP
HTTP	TCP	80	Anywhere

User Data Script (for Apache Web Server)
Add the following script in the Advanced details > User data section:

bash
```
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "Hi Hello " > /var/www/html/index.html
Launch and Verify
```
![WhatsApp Image 2025-08-06 at 17 01 34_44675fae](https://github.com/user-attachments/assets/6674c76d-a98a-431c-8db3-2a031c2ecc03)


After instance starts, copy its Public IP

Paste into browser → You should see: Hi Hello
![WhatsApp Image 2025-08-06 at 17 02 24_44d5e940](https://github.com/user-attachments/assets/e066f18d-6f27-4226-ac77-83daa27c5074)



---

### STEP:2 Create a Launch Template
- Launch Template Creation (Using Existing EC2 Instance Configuration)
After setting up your EC2 instance and verifying that it's working as expected (e.g., running Apache and responding via HTTP), you can create a Launch Template directly from it. This will allow Auto Scaling to use the same configuration for future instances.

 Steps to Create the Launch Template:
Go to EC2 Dashboard

Navigate to Instances

Select your existing EC2 instance

Click Actions > Create template from instance

Basic Configuration

Template name: Choose a meaningful name like webserver-template

Version description: Example — base-config-version

AMI: It will automatically use the Amazon Linux AMI from your instance

Instance type: This will inherit your current instance type (e.g., t2.micro)

Key pair: Same as the instance (used for SSH access)

Network and Security
![WhatsApp Image 2025-08-06 at 17 34 13_478b3734](https://github.com/user-attachments/assets/c1ea76a6-0d12-4881-8df1-67920ba93167)


The template will inherit the Security Group from the instance (with open ports 22 and 80)

Subnet/network settings are not needed in the template — they are handled in the Auto Scaling Group

Storage

The EBS volume configuration will be copied from the instance
![WhatsApp Image 2025-08-06 at 17 25 21_2287207a](https://github.com/user-attachments/assets/704005fa-515c-46a3-9b55-2605b85f4cf3)






Create the Template

Click Create launch template

### Benefits of This Approach
 Reusability: No need to manually recreate configurations for each new instance

 Consistency: Ensures identical settings for all Auto Scaling instances

 Efficiency: Saves time by using a working EC2 as a base

Reliable Security: Inherits tested security group rules
---
## Step:3 Target Group Setup
A Target Group is used to route traffic from the ALB to the EC2 instances. It helps in monitoring instance health and distributing load.
### Steps to Create a Target Group:
Go to EC2 > Target Groups > Create target group

Target type: Choose Instance

Protocol: HTTP

Port: 80

VPC: Select the same VPC used by your EC2 instance

Health checks:

Protocol: HTTP

Path: / (default root path)

Click Next → Don’t register targets now (Auto Scaling will handle this)

Click Create target group



---
## Step 4:
### Application Load Balancer (ALB)
The ALB distributes incoming HTTP traffic across multiple EC2 instances in the Target Group. It also detects unhealthy instances and routes traffic only to healthy ones.

🔧 Steps to Create the ALB:
Go to EC2 > Load Balancers > Create Load Balancer

Select Application Load Balancer

Name: e.g., web-alb

Scheme: Internet-facing

Listeners: HTTP on port 80
![WhatsApp Image 2025-08-06 at 17 12 49_ccd455ba](https://github.com/user-attachments/assets/30b7c548-1cf9-43b3-bad5-ceb968df4dab)


Availability Zones:

Choose at least 2 public subnets in different AZs

Security Groups:

Attach the security group allowing inbound HTTP (port 80)

Target Group:

Choose “Existing target group” → select the one created earlier

Click Create Load Balancer
![WhatsApp Image 2025-08-06 at 17 13 29_5447c825](https://github.com/user-attachments/assets/da14c3a5-169f-46bf-ad11-55466725705a)


 Once created, your ALB will have a DNS name (like web-alb-12345678.us-east-1.elb.amazonaws.com) to access your application.
 ---
 ## STEP 5:

 Auto Scaling Group (ASG)
The Auto Scaling Group will ensure that the application scales based on demand and recovers from instance failure.

🔧 Steps to Create an Auto Scaling Group:
Go to EC2 > Auto Scaling Groups > Create Auto Scaling group

Name: e.g., web-asg

Launch Template:

Select the template you created earlier

Network:

Choose your VPC and select at least 2 public subnets

Load Balancing:
![WhatsApp Image 2025-08-06 at 17 26 23_691ce7cc](https://github.com/user-attachments/assets/9b4cff3f-395b-4702-9e62-c2efd5387f09)


Attach the previously created Target Group

Group Size:

Desired Capacity: 1

Minimum Capacity: 1

Maximum Capacity: 2
![WhatsApp Image 2025-08-06 at 17 27 27_cbaff1f5](https://github.com/user-attachments/assets/21c4b695-2bb7-42d8-b550-43767be7bdb7)


Health Check Type:

Choose EC2 or ELB (ELB recommended for Load Balancer integration)

Scaling Policies:

Select “Target tracking scaling policy”

Example: Keep the average CPU utilization at 50%

Click Create Auto Scaling group
![WhatsApp Image 2025-08-06 at 17 32 02_38ad0c35](https://github.com/user-attachments/assets/8959e3f0-bd98-4076-94f8-ef53d6640404)


 Test by terminating an instance manually – ASG should auto-launch a replacement.
 ![WhatsApp Image 2025-08-06 at 17 34 45_d8b38600](https://github.com/user-attachments/assets/7cb1c147-e9b2-4959-ae28-f019811d4684)

---

 ## STEP 6: Elastic IP Address (EIP)
Elastic IP provides a static public IP address for a specific EC2 instance. Even after stop/start cycles, the public IP remains consistent.

 Why Use EIP?
Eliminates changing public IPs after restarts

Ensures consistent access and DNS mapping

Useful for monitoring, SSH, and long-term access

Can be re-associated quickly to new instances if needed

🔧 Steps to Allocate and Attach EIP:
Go to EC2 > Elastic IPs > Allocate Elastic IP address
![WhatsApp Image 2025-08-06 at 17 35 31_debb15eb](https://github.com/user-attachments/assets/c4b261e0-7a0f-477f-aefe-3f5474505d73)

Click Allocate

Select the IP and choose Actions > Associate

Attach it to your EC2 instance (can also be used with a NAT or Bastion if needed)

 Elastic IPs are free when associated with a running instance. AWS charges for idle EIPs.

### Test the Setup
- Access your app using:
  - ALB DNS name (e.g., `http://my-alb-123.us-west-2.elb.amazonaws.com`)
  - Or the **Elastic IP** you attached to the ALB
  - ![WhatsApp Image 2025-08-06 at 17 39 39_063172be](https://github.com/user-attachments/assets/fb5ae55c-d5e5-4f9e-8450-868b847d5b32)

- Terminate EC2 manually to observe **Auto Scaling** launching a new instance
- Ensure **Health Checks** show healthy status in Target Group

---

##  Elastic IP – Use & Advantages
- Assigns a **fixed public IP** to Load Balancer (or EC2)
- Useful for:
  - DNS records
  - Firewall whitelisting
  - Monitoring
  - Persistent public access
- Reassign quickly to new resources for high availability

---

##  Use Cases
- Hosting static/dynamic web apps with high availability
- Auto-recovery from EC2 failure using Auto Scaling
- Zero-downtime deployment with Load Balancer
- Hands-on learning of production-grade AWS infrastructure

---

##  Cost Optimization Tips
- Use **Amazon Linux** for free-tier eligible instances
- Set **Auto Scaling max** to a reasonable number (e.g., 2)
- Avoid idle EIPs or unused Load Balancers (they incur cost)
- Clean up resources after testing to avoid charges

---




## 📎 Conclusion
This project effectively demonstrates a highly available, auto-scaling web setup on AWS. Using a combination of EC2, Launch Templates, Load Balancer, and Elastic IP ensures scalability, reliability, and consistent access. It's an ideal setup for production-ready deployments and DevOps learning environments.

---

**Feel free to fork, star, and contribute!** 🚀
