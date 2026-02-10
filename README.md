# AWS EC2 Launch Lab

A **hands-on, end-to-end AWS infrastructure lab** demonstrating how to design, deploy, secure, and access an EC2 instance **from scratch using AWS best practices**.

This project focuses on **networking fundamentals, security controls, and real-world access challenges**, not just launching an instance.

---

## Project Overview

This lab builds a **fully functional EC2 environment** starting from a blank AWS account.

Every component is created deliberately to mirror **real cloud engineering workflows**, not shortcuts.

### Infrastructure Components Created

* **Virtual Private Cloud (VPC)** – Isolated AWS network
* **Public Subnet** – Controlled exposure to the internet
* **Internet Gateway (IGW)** – Enables outbound/inbound connectivity
* **Route Table** – Explicit traffic routing
* **Security Group** – Stateful firewall rules
* **EC2 Key Pair** – Secure SSH authentication
* **EC2 Instance** – Amazon Linux 2
* **AWS Systems Manager (SSM)** – Secure, SSH-less access

---

## Objective

**Purpose of this lab:**

* Demonstrate **strong AWS networking fundamentals**
* Launch EC2 instances **securely and intentionally**
* Apply **least-privilege security group rules**
* Handle **real-world access issues** (e.g., ISP SSH blocks)
* Use **SSM as a production-grade alternative to SSH**

This project is suitable for **learning, labs, and portfolio demonstration**.

---

## Architecture Summary

* Single VPC (`10.1.0.0/16`)
* Public subnet (`10.1.1.0/24`)
* Internet-facing EC2 instance
* SSH restricted to known IPs
* Optional access via AWS Systems Manager

(Architecture diagram available in `/architecture`)

---

## Key Implementation Steps

### 1. Create a VPC

```bash
aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]'
```

---

### 2. Create a Public Subnet

```bash
aws ec2 create-subnet \
  --vpc-id vpc-xxx \
  --cidr-block 10.1.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-subnet}]'
```

---

### 3. Create and Attach an Internet Gateway

```bash
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]'

aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxx \
  --vpc-id vpc-xxx
```

---

### 4. Create Route Table and Public Route

```bash
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
```

---

### 5. Create an EC2 Key Pair

```bash
aws ec2 create-key-pair \
  --key-name lab-key \
  --query 'KeyMaterial' \
  --output text > lab-key.pem

chmod 400 lab-key.pem
```

---

### 6. Create Security Group (Least Privilege)

```bash
aws ec2 create-security-group \
  --group-name lab-sg \
  --description "Security group for EC2 lab" \
  --vpc-id vpc-xxx
```

**Ingress rules:**

```bash
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
```

---

### 7. Launch EC2 Instance

```bash
aws ec2 run-instances \
  --image-id ami-xxx \
  --count 1 \
  --instance-type t3.micro \
  --key-name lab-key \
  --security-group-ids sg-xxx \
  --subnet-id subnet-xxx \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2-public}]'
```

---

### 8. Connect via SSH

```bash
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
```

---

## Optional: AWS Systems Manager (SSM)

SSM is used when:

* SSH is blocked by ISP
* No public IP is assigned
* Zero-trust access is preferred

```bash
aws ssm start-session --target i-xxx
```

---

## Observations

### Pros

* Full control over VPC and routing
* Secure, intentional EC2 deployment
* Free-tier compatible
* SSM removes SSH dependency
* Mirrors real cloud environments

### Challenges

* No public IP means no SSH
* ISP port 22 blocks are common
* SSM requires IAM configuration

---

## Security Best Practices Applied

* SSH restricted to known IPs
* No credentials committed to GitHub
* Least-privilege security groups
* Secure key management
* SSM for hardened remote access

---

## Key Takeaway

This project demonstrates **practical AWS skills**, **networking literacy**, and a **security-first cloud engineering mindset**.

---

## References

* AWS CLI Documentation
* # AWS EC2 Launch Lab

A **hands-on, end-to-end AWS infrastructure lab** demonstrating how to design, deploy, secure, and access an EC2 instance **from scratch using AWS best practices**.

This project focuses on **networking fundamentals, security controls, and real-world access challenges**, not just launching an instance.

---

## Project Overview

This lab builds a **fully functional EC2 environment** starting from a blank AWS account.

Every component is created deliberately to mirror **real cloud engineering workflows**, not shortcuts.

### Infrastructure Components Created

* **Virtual Private Cloud (VPC)** – Isolated AWS network
* **Public Subnet** – Controlled exposure to the internet
* **Internet Gateway (IGW)** – Enables outbound/inbound connectivity
* **Route Table** – Explicit traffic routing
* **Security Group** – Stateful firewall rules
* **EC2 Key Pair** – Secure SSH authentication
* **EC2 Instance** – Amazon Linux 2
* **AWS Systems Manager (SSM)** – Secure, SSH-less access

---

## Objective

**Purpose of this lab:**

* Demonstrate **strong AWS networking fundamentals**
* Launch EC2 instances **securely and intentionally**
* Apply **least-privilege security group rules**
* Handle **real-world access issues** (e.g., ISP SSH blocks)
* Use **SSM as a production-grade alternative to SSH**

This project is suitable for **learning, labs, and portfolio demonstration**.

---

## Architecture Summary

* Single VPC (`10.1.0.0/16`)
* Public subnet (`10.1.1.0/24`)
* Internet-facing EC2 instance
* SSH restricted to known IPs
* Optional access via AWS Systems Manager

(Architecture diagram available in `/architecture`)

---

## Key Implementation Steps

### 1. Create a VPC

```bash
aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]'
```

---

### 2. Create a Public Subnet

```bash
aws ec2 create-subnet \
  --vpc-id vpc-xxx \
  --cidr-block 10.1.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-subnet}]'
```

---

### 3. Create and Attach an Internet Gateway

```bash
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]'

aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxx \
  --vpc-id vpc-xxx
```

---

### 4. Create Route Table and Public Route

```bash
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
```

---

### 5. Create an EC2 Key Pair

```bash
aws ec2 create-key-pair \
  --key-name lab-key \
  --query 'KeyMaterial' \
  --output text > lab-key.pem

chmod 400 lab-key.pem
```

---

### 6. Create Security Group (Least Privilege)

```bash
aws ec2 create-security-group \
  --group-name lab-sg \
  --description "Security group for EC2 lab" \
  --vpc-id vpc-xxx
```

**Ingress rules:**

```bash
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
```

---

### 7. Launch EC2 Instance

```bash
aws ec2 run-instances \
  --image-id ami-xxx \
  --count 1 \
  --instance-type t3.micro \
  --key-name lab-key \
  --security-group-ids sg-xxx \
  --subnet-id subnet-xxx \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2-public}]'
```

---

### 8. Connect via SSH

```bash
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
```

---

## Optional: AWS Systems Manager (SSM)

SSM is used when:

* SSH is blocked by ISP
* No public IP is assigned
* Zero-trust access is preferred

```bash
aws ssm start-session --target i-xxx
```

---

## Observations

### Pros

* Full control over VPC and routing
* Secure, intentional EC2 deployment
* Free-tier compatible
* SSM removes SSH dependency
* Mirrors real cloud environments

### Challenges

* No public IP means no SSH
* ISP port 22 blocks are common
* SSM requires IAM configuration

---

## Security Best Practices Applied

* SSH restricted to known IPs
* No credentials committed to GitHub
* Least-privilege security groups
* Secure key management
* SSM for hardened remote access

---

## Key Takeaway

This project demonstrates **practical AWS skills**, **networking literacy**, and a **security-first cloud engineering mindset**.

---

## References

* AWS CLI Documentation
* Amazon EC2 User Guide
* AWS Systems Manager Session Manager


