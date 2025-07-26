# 🚀 AWS_Scalable_Web_Application

**AWS EC2-based Scalable Web Application with Load Balancer and Auto Scaling**

This project demonstrates how i deployed a simple web application on AWS using EC2 instances, ensuring high availability and scalability with Elastic Load Balancing (ALB) and Auto Scaling Groups (ASG). The setup follows best practices for compute scalability, security, and cost optimization.

## 🗺️ Architecture Diagram

<img width="3113" height="2063" alt="Arcitecture" src="https://github.com/user-attachments/assets/a53b13bb-4ba9-4cef-8adc-20429eb1a3e3" />

---

## 🧱 Key AWS Services Used

- **EC2**: Launch and host web application instances  
- **Application Load Balancer (ALB)**: Distributes traffic evenly across instances  
- **Auto Scaling Group (ASG)**: Automatically adjusts the number of EC2 instances based on demand  
- **IAM**: Role-based access to EC2 and AWS resources  
- **CloudWatch & SNS**: Monitors system performance and sends alerts

---

## 🛠️ Deployment Steps

---

### ✅ Step 1 – Create VPC, Subnets, Route Table & Internet Gateway

I created a VPC named `my-vpc-01` with the following IPv4 CIDR block:

- `10.10.0.0/16`

Then I created **three public subnets**:

- `Public-Subnet-1` → `10.10.1.0/24` → `us-east-1a`
- `Public-Subnet-2` → `10.10.2.0/24` → `us-east-1b`
- `Public-Subnet-3` → `10.10.3.0/24` → `us-east-1c`

Next, I created an **Internet Gateway** named `my-internet-gateway` and attached it to the VPC.

To enable routing:

- Created a route table named `my-route-table`
- Associated it with the public subnets
- Added a default route `0.0.0.0/0` pointing to the Internet Gateway

📷 **Resource Map**  
*Image to be added here*

---

### 🔐 Step 2 – Define Security Groups

Created two security groups:

- `my-load-balancer-security-group`: Allowed **HTTP traffic from anywhere** (`0.0.0.0/0`)
- `my-web-server-security-group`: Allowed **HTTP traffic only from the ALB**

---

### 🧩 Step 3 – Create Launch Template

Launch Template name: `my-launch-template`

Configuration used:

- **AMI**: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type  
- **Instance type**: `t2.micro`  
- **Key pair**: `MyWebAppKeyPair`  
- **Security Group**: `my-web-server-security-group`  
- **IAM Role**: `EC2WebAppProfile` (to allow access to CloudWatch, etc.)  
- **Hostname type**: IP name  
- **User Data**: Bash script to install and configure the web server on boot  

---

### ⚖️ Step 4 – Create Application Load Balancer (ALB)

Configured an **internet-facing** ALB named `my-load-balancer` that:

- Spans across the three public subnets in different Availability Zones
- Uses the security group `my-load-balancer-security-group`

The ALB is connected to a **Target Group** named `my-project` that registers EC2 instances automatically.

---

### 📈 Step 5 – Create Auto Scaling Group (ASG)

Created an ASG named `my-asg`:

- Based on the `my-launch-template`
- Linked to the Load Balancer and Target Group
- Desired capacity: **2**
- Minimum: **2**, Maximum: **5**
- Scaling policy: Maintain average CPU utilization at **70%**

✅ Enabled **CloudWatch group metrics**  
📧 Configured an **SNS Topic** (`my-sns-topic`) to notify `nidhal.labri@gmail.com` for events:
- EC2 instance launch
- EC2 termination
- Launch failure
- Termination failure

📬 **Example Notification**  
*Image to be added here*

---

### 🔍 Step 6 – Test Functionality

Access the application via the ALB DNS name:  
**`my-load-balancer-2072421467.us-east-1.elb.amazonaws.com`**

📷 *Screenshot showing instance IDs / AZs in browser*  
*Image to be added here*

Confirmed successful traffic distribution across instances — each tab loads a different EC2 instance (with unique Instance ID and AZ), confirming the ALB is working properly.

---

## ⚠️ Problem Faced: 502 Bad Gateway

I encountered a **502 Bad Gateway** error when accessing the application via the ALB. The Target Group showed the EC2 instances as **unhealthy**.

Upon SSH access and debugging, I found that the **user data script did not finish setting up NGINX** before the health checks began.

### 🔧 Solution:

- Fixed the user data script to correctly install NGINX and serve a valid HTML page
- Updated the launch template with the fixed script
- Recreated the Auto Scaling Group using the updated launch template version

✅ Everything worked perfectly after that — the ALB routed traffic without any manual intervention.

---

## 💡 Future Improvements (RDS Plan)

I originally planned to deploy a **Flask application connected to an RDS PostgreSQL Multi-AZ DB cluster**, including:

- 1 primary DB instance
- 2 readable standbys in separate AZs

However, **Multi-AZ RDS is not free tier eligible**, so I excluded it for now.

---

## ✍️ Author

**Made with 💻 by Nidhal Labri**  
🔗 [LinkedIn](https://www.linkedin.com/in/nidhal-labri/)
