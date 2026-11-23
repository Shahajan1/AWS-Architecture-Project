# 🚀 AWS Highly Available & Scalable Architecture

A hands-on AWS project demonstrating how to design and deploy a scalable, highly available, and secure cloud infrastructure using Amazon Web Services.

The architecture uses a custom VPC, public and private subnets across two Availability Zones, EC2 Auto Scaling, a Bastion Host, and an internet-facing Application Load Balancer.

## 🏗️ Architecture Overview

The infrastructure is designed to separate public-facing components from private application servers while supporting scalability and availability.

**Architecture Components:**
- Custom Amazon VPC
- Public and Private Subnets
- Two Availability Zones
- NAT Gateways
- S3 Gateway VPC Endpoint
- EC2 Auto Scaling Group
- Bastion Host
- Application Load Balancer
- Target Groups
- Security Groups

> 📌 Add your AWS architecture diagram or screenshot here.

## 🛠️ AWS Services & Technologies

| Service | Implementation |
|---|---|
| Amazon VPC | Custom virtual network |
| Subnets | Public and Private |
| Availability Zones | Two AZs |
| NAT Gateway | Private subnet internet access |
| S3 Gateway Endpoint | Private S3 connectivity |
| Amazon EC2 | Ubuntu instances |
| Launch Template | EC2 configuration |
| Auto Scaling | Automatic instance management |
| Bastion Host | Secure SSH access |
| Application Load Balancer | HTTP traffic distribution |
| Target Group | Backend instance registration |
| Security Groups | Network traffic control |

## ⚙️ Project Implementation

### 1. VPC & Networking
- Created a custom VPC named `Project VPC`.
- Configured networking across two Availability Zones.
- Created public and private subnets.
- Configured NAT Gateways.
- Added an S3 Gateway VPC Endpoint.

### 2. EC2 Auto Scaling
- Created an Ubuntu-based EC2 Launch Template.
- Configured the `proj-sg` security group.
- Created an Auto Scaling Group.
- Deployed instances across private subnets.

**Scaling Configuration:**
- Desired Capacity: 2
- Minimum Capacity: 1
- Maximum Capacity: 5
- Scaling Policy: Target Tracking – Average CPU Utilization.

### 3. Bastion Host
- Launched an Ubuntu EC2 instance in a public subnet.
- Configured SSH access.
- Transferred the private key using SCP.
- Connected to private EC2 instances through SSH.
- Installed and configured Apache Web Server.

### 4. Application Load Balancer
- Created an internet-facing Application Load Balancer.
- Selected public subnets across both Availability Zones.
- Created an EC2 Target Group.
- Registered private EC2 instances as backend targets.
- Configured HTTP traffic distribution.

## 🔄 Architecture Workflow

1. Users send HTTP requests through the internet.
2. The Application Load Balancer receives incoming requests.
3. The Load Balancer forwards requests to healthy private EC2 instances.
4. Apache Web Server processes the requests.
5. Responses are returned to users through the Load Balancer.

Administrators access private EC2 instances through the Bastion Host using SSH.

## 🔐 Security & Networking

- Application servers are deployed in private subnets.
- Bastion Host provides controlled SSH access.
- Security Groups regulate inbound and outbound traffic.
- NAT Gateways provide outbound internet connectivity for private instances.
- S3 Gateway Endpoint enables private connectivity to Amazon S3.

## 📚 Key Learnings

Through this project, I gained practical experience in:

- AWS VPC and subnet architecture.
- Public and private networking.
- Multi-AZ infrastructure design.
- NAT Gateway and VPC Endpoint configuration.
- EC2 Auto Scaling and Launch Templates.
- Bastion Host and SSH connectivity.
- Application Load Balancer configuration.
- Target Groups and health checks.
- AWS networking troubleshooting.

## 📖 Project Documentation

For detailed implementation steps, configurations, and explanations, refer to:

**[AWS Architecture Project – Detailed Documentation](./AWS-Architecture-Documentation.md)**

## 👨‍💻 Author

**Shahajan Shaikh**

DevOps & AWS Cloud Enthusiast

- GitHub: [Shahajan1](https://github.com/Shahajan1)
- LinkedIn: [Shahajan Shaikh](https://www.linkedin.com/in/shahajanshaikh/)

---

⭐ If you find this project useful, feel free to explore the repository!
