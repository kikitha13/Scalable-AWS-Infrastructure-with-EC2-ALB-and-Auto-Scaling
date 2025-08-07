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

## 🧱 Step-by-Step Infrastructure Setup

### 1️⃣ Create a VPC and Public Subnet
- Create a new VPC with a CIDR block like `10.0.0.0/16`
- Add an Internet Gateway and attach to the VPC
- Create a Route Table and associate it with a public subnet
- Add a route for `0.0.0.0/0` to the Internet Gateway

### 2️⃣ Launch EC2 Instance (Amazon Linux)
- Use **Amazon Linux 2023 AMI**
- Configure security group to allow port 22 (SSH) and 80 (HTTP)
- User Data script to install Apache (httpd):
  ```bash
  #!/bin/bash
  yum update -y
  yum install httpd -y
  systemctl start httpd
  systemctl enable httpd
  echo "Hi Hello from EC2 instance" > /var/www/html/index.html
  ```

### 3️⃣ Allocate and Attach Elastic IP
- Go to EC2 > Elastic IPs > Allocate new address
- Associate the EIP to the above EC2 instance

### 4️⃣ Create a Launch Template
- Create a new Launch Template using the configuration from the EC2 instance
- Use the same AMI, security group, and instance type
- Add the above user data script in the template

### 5️⃣ Create a Target Group
- Protocol: HTTP, Port: 80
- Target type: Instance
- Health check path: `/`

### 6️⃣ Create an Application Load Balancer
- Type: Internet-facing, HTTP on port 80
- Assign public subnets in at least 2 AZs
- Register the Target Group

### 7️⃣ Create an Auto Scaling Group
- Attach the Launch Template
- Attach the Target Group
- Set desired capacity: 1, min: 1, max: 2

### 8️⃣ Test Setup
- Access the site via Elastic IP and ALB DNS
- Stop or terminate an instance to test Auto Scaling replacement

---

## 💡 Use Cases

- Hosting a **simple web application** with auto-recovery
- Learning **AWS Auto Scaling, ALB, and Launch Templates**
- Building **cost-effective fault-tolerant environments**
- **DevOps automation practice** for production setups

---

## ⚙️ Elastic IP — Use and Advantages

- Provides a **fixed public IP** to an EC2 instance.
- Avoids frequent IP changes on stop/start cycles.
- Ideal for DNS mapping, monitoring, and consistent SSH access.
- Can be **re-associated** quickly to new instances if needed.
- Helps in maintaining **reliable public endpoints** during Auto Scaling operations.

---

## 💸 Cost Optimization Tips

| Strategy | Benefit |
|---------|---------|
| **Use Amazon Linux** | Free-tier eligible and lightweight |
| **Set min=1 in ASG** | Only one instance running unless needed |
| **Scale-out only on demand** | Avoid over-provisioning |
| **Release unused EIP** | Avoid being charged when not in use |
| **Monitor with CloudWatch** | Detect abnormal usage patterns |

---

## 📌 Output

- Static HTML page (`Hi Hello`) is served on both:
  - Elastic IP (`http://<Elastic-IP>`)
  - ALB DNS (`http://<ALB-DNS>`)

- Auto Scaling ensures:
  - Fault tolerance (if an instance fails, another is launched)
  - Load distribution via ALB
  - Consistent user experience

---

## ✅ Conclusion

This project illustrates the power of **AWS Auto Scaling**, **Load Balancing**, and **Launch Templates** in building **robust, scalable web infrastructures**. With a clean separation of components, dynamic scaling, and cost-efficiency, it’s an ideal starting point for production-grade cloud applications.

---


