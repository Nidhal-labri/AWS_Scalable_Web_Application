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


### ✅ Step 1 – Create VPC, Subnets, Route Table & Internet Gateway

I created a VPC named `my-vpc-01` where the IPv4 CIDR is `10.10.0.0/16`.

Then I created **three public subnets**:

- `Public-Subnet-1` → `10.10.1.0/24` → `us-east-1a`
- `Public-Subnet-2` → `10.10.2.0/24` → `us-east-1b`
- `Public-Subnet-3` → `10.10.3.0/24` → `us-east-1c`

Next, I created an **Internet Gateway** named `my-internet-gateway` and attached it to the VPC.

Even though having an Internet Gateway now enables our resources to communicate with the Internet, we still need to define a route table and associate it with our subnets. I gave my route table of my VPC the name `my-route-table`. I verified the subnet association. Last but not least, I added a default route for `0.0.0.0/0` to my Internet Gateway.


📷 **Resource Map**  
<img width="1655" height="556" alt="image_2025-07-26_01-10-09" src="https://github.com/user-attachments/assets/aa3dae12-7467-473a-b72d-deea330d7dd0" />

---

### 🔐 Step 2 – Define Security Groups

Security is paramount, so I created two security groups:

- `my-load-balancer-security-group`: Allowed HTTP traffic from the internet (`0.0.0.0/0`).  
- `my-web-server-security-group`: Restricted HTTP traffic to only come from the ALB.

---

### 🧩 Step 3 – Create Launch Template

On this step, we will create a Launch Template named `my-launch-template` that will be used to define the EC2 instances we deploy in our Auto Scaling Group.

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

I configured an Application Load Balancer (ALB) named `my-load-balancer` that serves as internet-facing to distribute traffic across the instances across different subnets and Availability Zones. I attached `my-load-balancer-security-group` as a security group to allow traffic from the internet.

The ALB was tied to a Target Group named `my-project` that I created to register the instances automatically.

---

### 📈 Step 5 – Create Auto Scaling Group (ASG)

I've created an Auto Scaling Group named `my-asg` that used the previously created launch template and linked to my Load Balancer. I configured the desired capacity as 2 instances, with a minimum desired capacity of 2 and maximum of 5. The scaling policy is set to maintain average CPU utilization at 70%.

For monitoring, I enabled group metrics collection within CloudWatch.

I also enabled notifications for the events (launch, terminate, fail to launch, fail to terminate) by creating an SNS Topic `my-sns-topic` that sends an email to `nidhal.labri@gmail.com` if one of those events is triggered.

📬 Here is an example of the received mail when a new EC2 instance was launched:  
<img width="1518" height="677" alt="55" src="https://github.com/user-attachments/assets/de44a8ab-fb39-4e3f-9c78-3c50b83d0c3c" />

---

### 🔍 Step 6 – Test Functionality

Finally! We can now access our web application through the ALB DNS name:  
**`my-load-balancer-2072421467.us-east-1.elb.amazonaws.com`**

<img width="1919" height="1003" alt="31" src="https://github.com/user-attachments/assets/cdf24ce6-98cd-480a-84b1-46b3fd902bab" />


✅ We can see that the web application displays that the traffic is being distributed between the 2 desired instances via the Load Balancer. Each tab shows a different EC2 instance (different Instance IDs and Availability Zones), confirming that the Load Balancer is correctly balancing the traffic across multiple instances in the Auto Scaling Group.

---

## ⚠️ Problem Faced: 502 Bad Gateway

I faced a **502 Bad Gateway** error when I was trying to access my web application through the ALB. I saw in the Target Group that it showed the existing two EC2 instances as unhealthy.

After some debugging to one of those EC2 instances via SSH, I realized the issue was with the user data script — it didn’t fully set up the web server before the ALB health checks kicked in. I updated the script to correctly install NGINX and generate a proper HTML page, and it worked for that machine.
So I updated the user data in the Launch Template with the new script and recreated the Auto Scaling Group with the new Launch Template version and everything worked perfectly ✅ — the ALB routed traffic correctly, and no manual intervention was needed anymore.

---

## 💡 Note

I originally planned to deploy a Flask application connected to an RDS PostgreSQL Multi-AZ DB cluster (a primary DB instance with two readable standbys in separate Availability Zones), but unfortunately, the Multi-AZ RDS is not Free Tier eligible, so I have excluded it for now.


---

## ✍️ Author

**Made with 💻 by Nidhal Labri**  
🔗 [LinkedIn](https://www.linkedin.com/in/nidhal-labri/)
