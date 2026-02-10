# AWS EC2 Launch Lab

A step-by-step lab guide for launching and managing an EC2 instance on AWS, including networking setup, security configurations, and remote access methods. This tutorial demonstrates best practices for creating a secure and accessible AWS environment suitable for labs and learning purposes.

---

## Project Overview

This project walks through building a complete AWS EC2 environment from scratch. The following components were created:

- **VPC & Subnet**: Isolated network for the instance.
- **Internet Gateway & Route Table**: Enable internet connectivity.
- **Security Group**: Controls inbound/outbound traffic.
- **Key Pair**: Secure SSH access.
- **EC2 Instance**: Amazon Linux 2 server.
- **AWS Systems Manager (SSM)**: Alternative remote access without relying on SSH.

**Purpose:**  
To provide a hands-on tutorial for launching EC2 instances securely, managing network configurations, and troubleshooting common access issues (e.g., ISP SSH blocks).

---

## Key Steps

1. **Create a VPC**
```bash
aws ec2 create-vpc --cidr-block 10.1.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]'

Create a Subnet

aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.1.1.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-subnet}]'

Create and Attach Internet Gateway (IGW)

aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]'
aws ec2 attach-internet-gateway --internet-gateway-id igw-xxx --vpc-id vpc-xxx


Create Route Table and Add Routes

aws ec2 create-route-table --vpc-id vpc-xxx --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=lab-rt}]'
aws ec2 create-route --route-table-id rtb-xxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxx
aws ec2 associate-route-table --subnet-id subnet-xxx --route-table-id rtb-xxx


Create Key Pair for SSH Access

aws ec2 create-key-pair --key-name lab-key --query 'KeyMaterial' --output text > lab-key.pem
chmod 400 lab-key.pem


Create Security Group and Configure Ingress

aws ec2 create-security-group --group-name lab-sg --description "Security group for my first EC2" --vpc-id vpc-xxx
aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr YOUR_PUBLIC_IP/32
aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 80 --cidr 0.0.0.0/0


Launch EC2 Instance
aws ec2 run-instances \
    --image-id ami-xxx \
    --count 1 \
    --instance-type t3.micro \
    --key-name lab-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --associate-public-ip-address \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2-public}]'


Connect via SSH
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>


Optional: Configure AWS Systems Manager (SSM)


Create IAM role & instance profile
Attach AmazonSSMManagedInstanceCore policy
Associate instance profile with EC2
Start SSM session:
aws ssm start-session --target i-xxx
Observations / Pros and Cons
Pros:
Full control over VPC, subnet, IGW, route tables, and security groups.
Free-tier EC2 instance is sufficient for labs.
SSM provides remote access even if SSH is blocked by ISP.
Tightened security by restricting SSH to known IPs.
Cons / Challenges:
Instances launched without a public IP cannot be accessed via SSH.
ISP port 22 blocks can prevent SSH access from home networks.
Duplicate security group rules may cause warnings (ignore if already applied).
Initial SSM setup requires IAM configuration.
Setup Instructions (Reproducible)
Configure AWS CLI with your credentials:
aws configure
Follow Key Steps in the order listed above to create:
VPC → Subnet → IGW → Route Table → Security Group → Key Pair → EC2 Instance.
Verify public IP before attempting SSH:
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].PublicIpAddress'
Connect via SSH:
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
Optional: Configure SSM for remote access without SSH.
Security Recommendations
Restrict SSH Access: Always limit inbound SSH (port 22) to known IPs.
Use SSM for Remote Management: Avoid dependency on public IPs and ISP blocks.
Security Group Hygiene: Avoid overly permissive rules (0.0.0.0/0) unless required.
Key Management: Keep private keys secure and never commit them to GitHub.
Monitor Network ACLs & Route Tables: Ensure only required traffic is allowed.
Free-tier Compliance: Verify instance types eligible for free-tier usage.
Troubleshoot ISP Restrictions: Test connectivity from alternate networks if port 22 is blocked.
References
AWS CLI Documentation
Amazon EC2 User Guide
AWS Systems Manager Session Manager