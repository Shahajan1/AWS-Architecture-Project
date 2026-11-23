# AWS Highly Available & Scalable Architecture
## Detailed Project Documentation

## 1. Project Overview

The objective of this project was to design and configure a scalable AWS network architecture using Amazon Web Services (AWS).

The infrastructure uses a custom Amazon Virtual Private Cloud (VPC), public and private subnets across two Availability Zones, an EC2 Auto Scaling Group, a Bastion Host, and an internet-facing Application Load Balancer (ALB).

The design separates public-facing components from private application servers and provides a foundation for high availability and automatic scaling.

### Project Objectives

- Create an isolated AWS network using Amazon VPC.
- Configure public and private subnets across two Availability Zones.
- Configure NAT Gateways for outbound internet access from private subnets.
- Configure an S3 Gateway VPC Endpoint.
- Deploy EC2 instances in private subnets using Auto Scaling.
- Configure a Bastion Host for SSH access to private instances.
- Configure an internet-facing Application Load Balancer.
- Distribute incoming HTTP traffic across backend EC2 instances.

---

## 2. AWS Services Used

| AWS Service / Component | Purpose |
|---|---|
| Amazon VPC | Creates an isolated virtual network. |
| Public Subnet | Hosts resources that need internet-facing connectivity. |
| Private Subnet | Hosts application servers without direct public IP access. |
| Availability Zones | Distributes resources across separate zones for resilience. |
| NAT Gateway | Allows private instances to initiate outbound internet connections. |
| S3 Gateway Endpoint | Provides private connectivity from the VPC to Amazon S3. |
| Amazon EC2 | Provides virtual servers. |
| Launch Template | Defines the configuration used to launch EC2 instances. |
| Auto Scaling Group | Maintains and adjusts the number of EC2 instances. |
| Bastion Host | Provides a controlled SSH entry point to private instances. |
| Application Load Balancer | Receives and distributes application traffic. |
| Target Group | Identifies the backend targets that receive traffic. |
| Security Groups | Control inbound and outbound network traffic. |

---

## 3. Architecture Summary

The intended architecture contains:

- One custom VPC.
- Two Availability Zones.
- Public and private subnets.
- NAT Gateway resources for private subnet egress.
- An S3 Gateway VPC Endpoint.
- An Auto Scaling Group with a desired capacity of two instances.
- A Bastion Host in a public subnet.
- An internet-facing Application Load Balancer in public subnets.
- A Target Group containing the private application instances.

> **Important:** Confirm the exact subnet and NAT Gateway counts in the AWS console before publishing this documentation. The original setup notes specify two Availability Zones and two subnets, but do not conclusively establish the final number of public and private subnets.

---

## 4. Implementation Steps

### Step 1: Create the VPC and Networking

Open the **VPC Dashboard** in the AWS Management Console and choose **Create VPC**. Select **VPC and more** to create the VPC and associated networking resources together.

Configuration recorded during the project:

- Project name: `Project VPC`
- Availability Zones: `2`
- Number of subnets: `2` (verify the final subnet layout)
- NAT Gateway: One per Availability Zone was selected.
- VPC Endpoint: S3 Gateway Endpoint.

Create the VPC and wait for AWS to provision the selected resources.

#### Why use two Availability Zones?

Distributing resources across two Availability Zones helps reduce dependence on a single zone. If one zone becomes unavailable, resources in the other zone may continue operating, depending on the application and its configuration.

#### Why use NAT Gateways?

Instances in private subnets generally should not receive direct inbound internet connections. A NAT Gateway allows instances to initiate outbound connections, for example to download operating-system updates, while remaining without direct public internet ingress.

For a highly available design, use a NAT Gateway in each Availability Zone and route each private subnet through the NAT Gateway in its own zone. This improves resilience but increases cost.

#### Why use an S3 Gateway Endpoint?

An S3 Gateway Endpoint lets resources in the VPC reach Amazon S3 using AWS network routing instead of requiring a NAT Gateway or internet gateway for that S3 traffic. The endpoint must be associated with the appropriate route tables.

---

### Step 2: Create an EC2 Launch Template

Open:

**EC2 → Launch Templates → Create launch template**

The Launch Template defines the configuration that the Auto Scaling Group uses when it launches EC2 instances.

Configuration recorded:

- Name: Project Launch Template
- Description: Project Launch Template
- AMI: Ubuntu (Quick Start)
- Instance type: `t3.micro`
- Key pair: Select the project key pair.
- Security group: Create or select the project security group.

Security Group details:

- Name: `proj-sg`
- Description: Project Security Group
- VPC: `Project VPC`

Configure only the inbound rules required by the architecture.

Common rules for this design:

| Type | Port | Recommended source |
|---|---:|---|
| HTTP | 80 | Application Load Balancer security group |
| SSH | 22 | Bastion Host security group |

The original notes mention ports 80 and 20. Verify the actual rules in the AWS console. Port 20 is associated with FTP data transfer and is not normally required for an HTTP web server. SSH uses port 22.

Create the Launch Template.

---

### Step 3: Create the Auto Scaling Group

Open:

**EC2 → Auto Scaling Groups → Create Auto Scaling group**

1. Enter an Auto Scaling Group name.
2. Select the Launch Template created in Step 2.
3. Select `Project VPC`.
4. Select the required private subnets across both Availability Zones.
5. Configure health checks.
6. Configure the desired capacity and scaling policy.

#### Capacity configuration

| Setting | Value |
|---|---:|
| Desired capacity | 2 |
| Minimum capacity | 1 |
| Maximum capacity | 5 |

#### Scaling policy

Select a **Target tracking scaling policy** and use **Average CPU utilization** as the metric.

Set the target CPU utilization according to the project requirement. The target percentage was not recorded in the original setup notes, so confirm it in the Auto Scaling Group configuration.

#### Health checks

The project notes record enabling health-check behavior and a 30-second setting. Verify which health-check setting was configured. EC2 Auto Scaling health-check grace periods, health-check intervals, and load-balancer health-check thresholds are different settings and should not be described interchangeably.

Create the Auto Scaling Group.

**Expected result:** The Auto Scaling Group launches and maintains two desired EC2 instances in the selected private subnets, subject to successful instance launch and health checks.

---

### Step 4: Create and Configure the Bastion Host

A Bastion Host is a controlled entry point used by administrators to connect to instances in private subnets.

Open:

**EC2 → Instances → Launch instances**

Configuration recorded:

- Name: `bastion-host`
- AMI: Ubuntu
- Instance type: `t3.micro`
- VPC: `Project VPC`
- Subnet: Public subnet in Availability Zone B
- Auto-assign public IP: Enabled
- Security group: Select the appropriate Bastion Host security group.
- Key pair: Select the project key pair.

Launch the instance.

#### Connect to the Bastion Host

From your local computer, connect to the Bastion Host using SSH:

```bash
ssh -i Shahajan-key.pem ubuntu@BASTION_PUBLIC_IP
```

Change the hostname:

```bash
sudo hostnamectl set-hostname bastion-host
```

Update the package list:

```bash
sudo apt update
```

#### Transfer the private key to the Bastion Host

The project procedure used SCP to transfer the private key from the local computer to the Bastion Host:

```bash
scp -i Shahajan-key.pem Shahajan-key.pem ubuntu@BASTION_PUBLIC_IP:/home/ubuntu/
```

On the Bastion Host, restrict access to the key file:

```bash
chmod 400 Shahajan-key.pem
```

#### Connect to a private EC2 instance

From the Bastion Host:

```bash
ssh -i Shahajan-key.pem ubuntu@PRIVATE_INSTANCE_IP
```

Once connected to the private instance, update Ubuntu:

```bash
sudo apt update
```

Install Apache:

```bash
sudo apt install apache2 -y
```

Check the Apache service:

```bash
sudo systemctl status apache2
```

**Expected result:** The administrator can connect to a private EC2 instance through the Bastion Host and install or manage software.

> **Security note:** Copying a private SSH key onto a Bastion Host increases the risk if that host is compromised. For a production environment, consider AWS Systems Manager Session Manager, EC2 Instance Connect Endpoint, or SSH agent forwarding where appropriate. If a key must be copied, use strict file permissions, limit SSH access to trusted source IPs, and remove the key from the Bastion Host when it is no longer needed.

---

### Step 5: Create the Application Load Balancer

Open:

**EC2 → Load Balancers → Create Load Balancer**

Choose **Application Load Balancer**.

Configuration recorded:

- Name: Project ALB
- Scheme: Internet-facing
- IP address type: IPv4
- VPC: `Project VPC`
- Availability Zones: Select both Availability Zones.
- Subnets: Select the public subnet in each Availability Zone.
- Security group: Select the appropriate Load Balancer security group.

#### Why select public subnets for the Load Balancer?

The project uses an internet-facing Load Balancer. It must be reachable by clients on the internet, so it is placed in public subnets with routes to an Internet Gateway.

The backend application EC2 instances remain in private subnets. This separates the public entry point from the application servers.

The Load Balancer security group should allow inbound HTTP traffic on port 80 (and HTTPS on port 443 if TLS is configured). It should be allowed to send traffic to the backend instances on the application port.

---

### Step 6: Create the Target Group

During Load Balancer setup, create a Target Group for the backend application instances.

Configuration recorded:

- Target type: Instances
- Target Group name: Project Target Group
- Protocol: HTTP
- Port: `80`
- VPC: `Project VPC`

Select the private EC2 instances as targets, add them as registered targets, and create the Target Group.

Configure an appropriate health check, normally HTTP on the application port and a path served successfully by the application (for example, `/`).

Select the Target Group in the Application Load Balancer listener configuration and finish creating the Load Balancer.

#### Important Auto Scaling integration

For instances launched or replaced by Auto Scaling to be registered automatically with the Load Balancer, attach the Target Group to the Auto Scaling Group.

If the Target Group was created after the Auto Scaling Group, edit the Auto Scaling Group's load-balancing settings and attach the Target Group. Confirm that the instances appear as registered and healthy targets.

---

### Step 7: Verify Routing and Troubleshoot Connectivity

During the project, a routing issue was encountered. The VPC route tables and NAT Gateway configuration were inspected to identify the problem.

Use the following checks.

#### Public subnet route table

The route table associated with each public subnet should contain:

| Destination | Target |
|---|---|
| VPC CIDR | local |
| `0.0.0.0/0` | Internet Gateway |

#### Private subnet route table

A private subnet that needs IPv4 internet egress should have a route similar to:

| Destination | Target |
|---|---|
| VPC CIDR | local |
| `0.0.0.0/0` | NAT Gateway |

For a design with one NAT Gateway per Availability Zone, route each private subnet through the NAT Gateway in the same Availability Zone where practical.

#### NAT Gateway checks

1. Open **VPC → NAT Gateways**.
2. Confirm that the NAT Gateway state is **Available**.
3. Confirm that it is deployed in a public subnet.
4. Confirm that it has an Elastic IP address.
5. Confirm that the public subnet has a working default route to the Internet Gateway.
6. Confirm that the private subnet route table points to the NAT Gateway.

#### Security Group checks

- The Load Balancer security group allows client traffic on the listener port.
- The application instance security group allows the application port from the Load Balancer security group.
- The Bastion Host security group allows SSH only from trusted administrator IP addresses.
- The private instance security group allows SSH from the Bastion Host security group if SSH administration is used.

---

## 5. Application Traffic Flow

The intended request flow is:

1. A user sends an HTTP request over the internet.
2. The internet-facing Application Load Balancer receives the request.
3. The Load Balancer forwards the request to a healthy registered target.
4. The private EC2 instance receives the request on the application port.
5. Apache processes the request and returns a response.
6. The response travels back through the Load Balancer to the user.

### Private Server Administration Flow

1. An administrator connects to the Bastion Host using SSH.
2. The Bastion Host provides the route to the private network.
3. The administrator connects to a private EC2 instance.
4. The administrator performs server-management tasks, such as installing Apache.

### Private Subnet Internet Egress Flow

1. A private EC2 instance initiates an outbound internet connection.
2. Its subnet route table sends the traffic to the NAT Gateway.
3. The NAT Gateway forwards the traffic through the Internet Gateway.
4. Return traffic is routed back to the private instance.

---

## 6. Verification Checklist

Use this checklist before considering the deployment complete.

- [ ] VPC is created successfully.
- [ ] Public and private subnet layout is correct.
- [ ] Subnets are distributed across two Availability Zones.
- [ ] Internet Gateway is attached to the VPC.
- [ ] Public route tables route internet traffic through the Internet Gateway.
- [ ] NAT Gateway status is Available.
- [ ] Private route tables use the correct NAT Gateway for outbound IPv4 internet access.
- [ ] S3 Gateway Endpoint is associated with the required route tables.
- [ ] Launch Template uses the intended Ubuntu AMI and instance type.
- [ ] Auto Scaling Group has desired capacity of two.
- [ ] Auto Scaling Group uses the intended private subnets.
- [ ] Bastion Host is reachable through SSH from an authorized source.
- [ ] Private EC2 instances are reachable through the Bastion Host.
- [ ] Apache is installed and running on the backend instances.
- [ ] Application Load Balancer is active and deployed in public subnets.
- [ ] Target Group is attached to the Auto Scaling Group.
- [ ] Backend targets are registered and healthy.
- [ ] The application can be reached through the Load Balancer DNS name.

---

## 7. Key Learnings

Through this project, I gained practical experience in:

- Creating and configuring Amazon VPC.
- Understanding public and private subnet architecture.
- Working with multiple Availability Zones.
- Configuring NAT Gateways and S3 Gateway Endpoints.
- Creating EC2 Launch Templates.
- Deploying and managing instances with Auto Scaling.
- Connecting to private EC2 instances through a Bastion Host.
- Using SSH and SCP for remote administration.
- Installing Apache Web Server on Ubuntu.
- Configuring an internet-facing Application Load Balancer.
- Creating Target Groups and understanding health checks.
- Troubleshooting route tables, security groups, and connectivity.

---

## 8. Conclusion

This project provided hands-on experience designing and configuring an AWS network architecture using core cloud infrastructure services.

It demonstrates practical knowledge of VPC networking, public and private subnets, multi-AZ design, EC2 Auto Scaling, Bastion Host access, NAT Gateway routing, and Application Load Balancing.

The architecture provides a foundation for hosting web applications on AWS with separated network tiers and scalable backend compute resources.

---

## 9. Project Author

**Shahajan Shaikh**  
DevOps & AWS Cloud Enthusiast

- GitHub: [Shahajan1](https://github.com/Shahajan1)
- LinkedIn: [Shahajan Shaikh](https://www.linkedin.com/in/shahajanshaikh/)

