AWS EC2 Launch Lab
A hands-on, end-to-end AWS infrastructure lab demonstrating how to design, deploy, secure, and access an EC2 instance from scratch using AWS best practices.
This project focuses on networking fundamentals, security controls, and real-world access challenges, not just launching an instance.
Project Overview
This lab builds a fully functional EC2 environment starting from a blank AWS account.
Every component is created deliberately to mirror real cloud engineering workflows, not shortcuts.
Infrastructure Components Created
Virtual Private Cloud (VPC) – Isolated AWS network
Public Subnet – Controlled exposure to the internet
Internet Gateway (IGW) – Enables outbound/inbound connectivity
Route Table – Explicit traffic routing
Security Group – Stateful firewall rules
EC2 Key Pair – Secure SSH authentication
EC2 Instance – Amazon Linux 2
AWS Systems Manager (SSM) – Secure, SSH-less access
Objective
Purpose of this lab:
Demonstrate deep understanding of AWS networking
Launch EC2 instances securely and intentionally
Control traffic using least-privilege security groups
Handle real-world issues like ISP SSH blocking
Use SSM as a production-grade alternative to SSH
This is designed for learning, labs, and portfolio demonstration.
Architecture Summary
Single VPC (10.1.0.0/16)
Public subnet (10.1.1.0/24)
Internet-facing EC2 instance
SSH restricted to known IPs
Optional access via AWS Systems Manager
(Architecture diagram included in /architecture)
Key Implementation Steps
1. Create a VPC
aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]'
2. Create a Public Subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxx \
  --cidr-block 10.1.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-subnet}]'
3. Create and Attach an Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]'

aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxx \
  --vpc-id vpc-xxx
4. Create Route Table and Public Route
aws ec2 create-route-table \
  --vpc-id vpc-xxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=lab-rt}]'

aws ec2 create-route \
  --route-table-id rtb-xxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxx

aws ec2 associate-route-table \
  --subnet-id subnet-xxx \
  --route-table-id rtb-xxx
5. Create an EC2 Key Pair
aws ec2 create-key-pair \
  --key-name lab-key \
  --query 'KeyMaterial' \
  --output text > lab-key.pem

chmod 400 lab-key.pem
6. Create Security Group (Least Privilege)
aws ec2 create-security-group \
  --group-name lab-sg \
  --description "Security group for EC2 lab" \
  --vpc-id vpc-xxx
Ingress rules:
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp \
  --port 22 \
  --cidr YOUR_PUBLIC_IP/32

aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
7. Launch EC2 Instance
aws ec2 run-instances \
  --image-id ami-xxx \
  --count 1 \
  --instance-type t3.micro \
  --key-name lab-key \
  --security-group-ids sg-xxx \
  --subnet-id subnet-xxx \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2-public}]'
8. Connect via SSH
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
Optional: AWS Systems Manager (SSM) Access
Used when:
SSH is blocked by ISP
No public IP is assigned
Zero-trust access is preferred
SSM Setup
Create IAM Role
Attach AmazonSSMManagedInstanceCore
Associate instance profile with EC2
Start Session
aws ssm start-session --target i-xxx
Observations & Trade-offs
Pros
Full control over networking stack
Clear separation of responsibilities
Secure SSH access model
Free-tier compatible
SSM eliminates SSH dependency
Real-world troubleshooting exposure
Challenges
No public IP = no SSH access
ISP port 22 blocks are common
SSM requires correct IAM configuration
Duplicate SG rules may trigger warnings
Reproducible Setup Instructions
Configure AWS CLI:
aws configure
Create resources in this order:
VPC → Subnet → IGW → Route Table → Security Group → Key Pair → EC2
Verify public IP:
aws ec2 describe-instances \
  --instance-ids i-xxx \
  --query 'Reservations[0].Instances[0].PublicIpAddress'
Connect via SSH or SSM
Security Best Practices Applied
SSH restricted to known IPs
No hardcoded credentials
Private keys never committed
Security groups follow least privilege
SSM used for secure remote management
Public exposure only where required
Key Takeaways
This lab demonstrates:
Strong AWS networking fundamentals
Security-first EC2 deployment
Real-world troubleshooting
Professional cloud engineering mindset
References
AWS CLI Documentation
Amazon EC2 User Guide
AWS Systems Manager Session Manager
