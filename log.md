AWS EC2 Launch Tutorial - Logs

Note: All sensitive values (AWS IDs, account numbers, etc.) are replaced with xxx. Replace them with your real values when following this tutorial.


1. Create VPC
Command:
aws ec2 create-vpc \
    --cidr-block 10.1.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]'
Output:
{
    "Vpc": {
        "VpcId": "vpc-xxx",
        "State": "pending",
        "CidrBlock": "10.1.0.0/16",
        "OwnerId": "xxx",
        "Tags": [{"Key": "Name", "Value": "lab-vpc"}],
        "DhcpOptionsId": "dopt-xxx"
    }
}
Note: VpcId is referenced in later steps. Replace vpc-xxx with your actual VPC ID.



2. Create Subnet
Command:
aws ec2 create-subnet \
    --vpc-id vpc-xxx \
    --cidr-block 10.1.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-subnet}]'
Output:
{
    "Subnet": {
        "SubnetId": "subnet-xxx",
        "VpcId": "vpc-xxx",
        "CidrBlock": "10.1.1.0/24",
        "AvailabilityZone": "us-east-1a",
        "State": "available",
        "Tags": [{"Key": "Name", "Value": "lab-subnet"}]
    }
}
Note: Keep SubnetId for route table association.


3. Create Internet Gateway (IGW)
Command:
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]'
Output:
{
    "InternetGateway": {
        "InternetGatewayId": "igw-xxx",
        "Tags": [{"Key": "Name", "Value": "lab-igw"}],
        "Attachments": []
    }
}



4. Attach IGW to VPC
Command:
aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-xxx \
    --vpc-id vpc-xxx
Output (success or already attached):
# If successful: no output
# If already attached:
An error occurred (Resource.AlreadyAssociated) when calling the AttachInternetGateway operation: resource igw-xxx is already attached to network vpc-xxx
Note: It’s fine if you see Resource.AlreadyAssociated. It means IGW is already attached.



5. Create Route Table
Command:
aws ec2 create-route-table \
    --vpc-id vpc-xxx \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=lab-rt}]'
Output:
{
    "RouteTable": {
        "RouteTableId": "rtb-xxx",
        "VpcId": "vpc-xxx",
        "Routes": [{"DestinationCidrBlock": "10.1.0.0/16", "GatewayId": "local"}],
        "Tags": [{"Key": "Name", "Value": "lab-rt"}]
    }
}



6. Add Route to IGW (for internet access)
Command:
aws ec2 create-route \
    --route-table-id rtb-xxx \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-xxx
Output:
{
    "Return": true
}



7. Associate Route Table with Subnet
Command:
aws ec2 associate-route-table \
    --subnet-id subnet-xxx \
    --route-table-id rtb-xxx
Output:
{
    "AssociationId": "rtbassoc-xxx",
    "AssociationState": {"State": "associated"}
}


8. Create Key Pair
Command:
aws ec2 create-key-pair \
    --key-name lab-key \
    --query 'KeyMaterial' \
    --output text > lab-key.pem
Output:
Creates a private key file: lab-key.pem
Set proper permissions for SSH:
chmod 400 lab-key.pem
Note: Keep lab-key.pem secure. Never share it publicly.



9. Create Security Group
Command:
aws ec2 create-security-group \
    --group-name lab-sg \
    --description "Security group for my first EC2" \
    --vpc-id vpc-xxx
Output:
{
    "GroupId": "sg-xxx",
    "SecurityGroupArn": "arn:aws:ec2:us-east-1:xxx:security-group/sg-xxx"
}



10. Add Ingress Rules to Security Group
Allow SSH (port 22) from your IP:
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR_PUBLIC_IP/32
Output:
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-xxx",
            "GroupId": "sg-xxx",
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "xxx/32"
        }
    ]
}
Allow HTTP (port 80) from anywhere:
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
Output:
Success: returns true
If duplicate:
An error occurred (InvalidPermission.Duplicate): the specified rule already exists
Note: Only add rules you need. Replace YOUR_PUBLIC_IP with your real public IP.



11. Launch EC2 Instance
Initial attempt (non-free tier):
aws ec2 run-instances \
    --image-id ami-xxx \
    --count 1 \
    --instance-type t2.micro \
    --key-name lab-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2}]'
Output:
An error occurred (InvalidParameterCombination): The specified instance type is not eligible for Free Tier.
Check free-tier eligible instances:
aws ec2 describe-instance-types \
    --filters Name=free-tier-eligible,Values=true \
    --query 'InstanceTypes[*].InstanceType' \
    --output table
Output Example:
+---------------------+
|  t3.micro           |
|  t4g.micro          |
|  t3.small           |
+---------------------+
Launch instance with free-tier type:
aws ec2 run-instances \
    --image-id ami-xxx \
    --count 1 \
    --instance-type t3.micro \
    --key-name lab-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2}]'
Output (abridged):
{
    "Instances": [
        {
            "InstanceId": "i-xxx",
            "State": {"Name": "pending"},
            "InstanceType": "t3.micro",
            "KeyName": "lab-key",
            "SubnetId": "subnet-xxx",
            "VpcId": "vpc-xxx",
            "PrivateIpAddress": "10.1.1.xxx",
            "SecurityGroups": [{"GroupId": "sg-xxx", "GroupName": "lab-sg"}]
        }
    ]
}
Note:
InstanceId and private IP will be unique; replace xxx with your real values.
At this point, your EC2 instance is launched in the subnet, using the security group and key pair you created.


12. Attempt to SSH into EC2 Instance
Command:
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
Output / Observation:
ssh: connect to host <PUBLIC_IP> port 22: Operation timed out
Problem Encountered:
The instance was running, but SSH connection timed out.
describe-instances showed PublicIpAddress as null initially.
Debugging Steps:
Verified instance state:
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].State.Name'
Output:
"running"
Checked if instance had a public IP:
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].PublicIpAddress'
Output:
null
Realized the instance was launched without associating a public IP.
Recommendation / Fix:
When launching EC2 in a public subnet, use --associate-public-ip-address to ensure the instance gets a public IP:
aws ec2 run-instances \
    --image-id ami-xxx \
    --count 1 \
    --instance-type t3.micro \
    --key-name lab-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --associate-public-ip-address \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2-public}]'
Verify public IP:
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=lab-ec2-public" \
    --query 'Reservations[0].Instances[0].PublicIpAddress'
Output example:
"44.193.xxx.xxx"


13. Open SSH and HTTP Ports in Security Group
Command (SSH from your IP only):
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR_PUBLIC_IP/32
Output:
{
    "Return": true,
    "SecurityGroupRules": [...]
}
Command (HTTP from anywhere):
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
Output:
{
    "Return": true
}
Problems Encountered:
Duplicate rule errors:
InvalidPermission.Duplicate
Fix:
Ignore if the rule already exists. AWS prevents adding duplicate rules.



14. SSH into the Public EC2 Instance
Command:
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
Output (success with phone hotspot):
The authenticity of host '<PUBLIC_IP>' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|
[ec2-user@ip-10-1-1-103 ~]$
Problem Encountered:
SSH works with phone hotspot but fails from home Wi-Fi.
Debugging Steps:
Verified instance was running and public IP was correct:
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].State.Name'
Output:
"running"
Tested port 22 with nc (netcat):
nc -vz <PUBLIC_IP> 22
Output:
nc: connectx to <PUBLIC_IP> port 22 (tcp) failed: Operation timed out
Checked Network ACLs and confirmed default ACL allows all traffic.
Conclusion / Recommendation:
Issue is with ISP blocking outbound SSH (port 22).
Verified by connecting via phone hotspot — SSH succeeded.
Recommendation if encountering this:
Test SSH from another network (hotspot, VPN, or cloud network).
Consider using port 443 tunneling or AWS Systems Manager if persistent ISP blocks.
Ensure security group is configured to allow SSH from your connecting IP.



15. Adjust Security Group as Needed
To remove wide SSH access (0.0.0.0/0) and restrict to your home IP:
aws ec2 revoke-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
Output:
{
    "Return": true
}
Add your public IP back:
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR_PUBLIC_IP/32
Output:
InvalidPermission.Duplicate  # Ignore if rule already exists

Notes / Observations:
Always restrict SSH to known IPs for security.
Verify connectivity from allowed IP after changes.

✅ At this point:
VPC, subnet, IGW, route table, security group, key pair, and EC2 instance are all set up.
Instance is accessible via SSH from networks that allow outbound port 22.
Issues were mainly due to missing public IP and ISP restrictions on SSH.


AWS EC2 Launch Tutorial - Logs

Note: All sensitive values (AWS IDs, account numbers, etc.) are replaced with xxx. Replace them with your real values when following this tutorial.


1. Create VPC
Command:
aws ec2 create-vpc \
    --cidr-block 10.1.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]'
Output:
{
    "Vpc": {
        "VpcId": "vpc-xxx",
        "State": "pending",
        "CidrBlock": "10.1.0.0/16",
        "OwnerId": "xxx",
        "Tags": [{"Key": "Name", "Value": "lab-vpc"}],
        "DhcpOptionsId": "dopt-xxx"
    }
}
Note: VpcId is referenced in later steps. Replace vpc-xxx with your actual VPC ID.



2. Create Subnet
Command:
aws ec2 create-subnet \
    --vpc-id vpc-xxx \
    --cidr-block 10.1.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-subnet}]'
Output:
{
    "Subnet": {
        "SubnetId": "subnet-xxx",
        "VpcId": "vpc-xxx",
        "CidrBlock": "10.1.1.0/24",
        "AvailabilityZone": "us-east-1a",
        "State": "available",
        "Tags": [{"Key": "Name", "Value": "lab-subnet"}]
    }
}
Note: Keep SubnetId for route table association.


3. Create Internet Gateway (IGW)
Command:
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]'
Output:
{
    "InternetGateway": {
        "InternetGatewayId": "igw-xxx",
        "Tags": [{"Key": "Name", "Value": "lab-igw"}],
        "Attachments": []
    }
}



4. Attach IGW to VPC
Command:
aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-xxx \
    --vpc-id vpc-xxx
Output (success or already attached):
# If successful: no output
# If already attached:
An error occurred (Resource.AlreadyAssociated) when calling the AttachInternetGateway operation: resource igw-xxx is already attached to network vpc-xxx
Note: It’s fine if you see Resource.AlreadyAssociated. It means IGW is already attached.



5. Create Route Table
Command:
aws ec2 create-route-table \
    --vpc-id vpc-xxx \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=lab-rt}]'
Output:
{
    "RouteTable": {
        "RouteTableId": "rtb-xxx",
        "VpcId": "vpc-xxx",
        "Routes": [{"DestinationCidrBlock": "10.1.0.0/16", "GatewayId": "local"}],
        "Tags": [{"Key": "Name", "Value": "lab-rt"}]
    }
}



6. Add Route to IGW (for internet access)
Command:
aws ec2 create-route \
    --route-table-id rtb-xxx \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-xxx
Output:
{
    "Return": true
}



7. Associate Route Table with Subnet
Command:
aws ec2 associate-route-table \
    --subnet-id subnet-xxx \
    --route-table-id rtb-xxx
Output:
{
    "AssociationId": "rtbassoc-xxx",
    "AssociationState": {"State": "associated"}
}


8. Create Key Pair
Command:
aws ec2 create-key-pair \
    --key-name lab-key \
    --query 'KeyMaterial' \
    --output text > lab-key.pem
Output:
Creates a private key file: lab-key.pem
Set proper permissions for SSH:
chmod 400 lab-key.pem
Note: Keep lab-key.pem secure. Never share it publicly.



9. Create Security Group
Command:
aws ec2 create-security-group \
    --group-name lab-sg \
    --description "Security group for my first EC2" \
    --vpc-id vpc-xxx
Output:
{
    "GroupId": "sg-xxx",
    "SecurityGroupArn": "arn:aws:ec2:us-east-1:xxx:security-group/sg-xxx"
}



10. Add Ingress Rules to Security Group
Allow SSH (port 22) from your IP:
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR_PUBLIC_IP/32
Output:
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-xxx",
            "GroupId": "sg-xxx",
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "xxx/32"
        }
    ]
}
Allow HTTP (port 80) from anywhere:
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
Output:
Success: returns true
If duplicate:
An error occurred (InvalidPermission.Duplicate): the specified rule already exists
Note: Only add rules you need. Replace YOUR_PUBLIC_IP with your real public IP.



11. Launch EC2 Instance
Initial attempt (non-free tier):
aws ec2 run-instances \
    --image-id ami-xxx \
    --count 1 \
    --instance-type t2.micro \
    --key-name lab-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2}]'
Output:
An error occurred (InvalidParameterCombination): The specified instance type is not eligible for Free Tier.
Check free-tier eligible instances:
aws ec2 describe-instance-types \
    --filters Name=free-tier-eligible,Values=true \
    --query 'InstanceTypes[*].InstanceType' \
    --output table
Output Example:
+---------------------+
|  t3.micro           |
|  t4g.micro          |
|  t3.small           |
+---------------------+
Launch instance with free-tier type:
aws ec2 run-instances \
    --image-id ami-xxx \
    --count 1 \
    --instance-type t3.micro \
    --key-name lab-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2}]'
Output (abridged):
{
    "Instances": [
        {
            "InstanceId": "i-xxx",
            "State": {"Name": "pending"},
            "InstanceType": "t3.micro",
            "KeyName": "lab-key",
            "SubnetId": "subnet-xxx",
            "VpcId": "vpc-xxx",
            "PrivateIpAddress": "10.1.1.xxx",
            "SecurityGroups": [{"GroupId": "sg-xxx", "GroupName": "lab-sg"}]
        }
    ]
}
Note:
InstanceId and private IP will be unique; replace xxx with your real values.
At this point, your EC2 instance is launched in the subnet, using the security group and key pair you created.


12. Attempt to SSH into EC2 Instance
Command:
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
Output / Observation:
ssh: connect to host <PUBLIC_IP> port 22: Operation timed out
Problem Encountered:
The instance was running, but SSH connection timed out.
describe-instances showed PublicIpAddress as null initially.
Debugging Steps:
Verified instance state:
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].State.Name'
Output:
"running"
Checked if instance had a public IP:
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].PublicIpAddress'
Output:
null
Realized the instance was launched without associating a public IP.
Recommendation / Fix:
When launching EC2 in a public subnet, use --associate-public-ip-address to ensure the instance gets a public IP:
aws ec2 run-instances \
    --image-id ami-xxx \
    --count 1 \
    --instance-type t3.micro \
    --key-name lab-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --associate-public-ip-address \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab-ec2-public}]'
Verify public IP:
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=lab-ec2-public" \
    --query 'Reservations[0].Instances[0].PublicIpAddress'
Output example:
"44.193.xxx.xxx"


13. Open SSH and HTTP Ports in Security Group
Command (SSH from your IP only):
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR_PUBLIC_IP/32
Output:
{
    "Return": true,
    "SecurityGroupRules": [...]
}
Command (HTTP from anywhere):
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
Output:
{
    "Return": true
}
Problems Encountered:
Duplicate rule errors:
InvalidPermission.Duplicate
Fix:
Ignore if the rule already exists. AWS prevents adding duplicate rules.



14. SSH into the Public EC2 Instance
Command:
ssh -i lab-key.pem ec2-user@<PUBLIC_IP>
Output (success with phone hotspot):
The authenticity of host '<PUBLIC_IP>' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|
[ec2-user@ip-10-1-1-103 ~]$
Problem Encountered:
SSH works with phone hotspot but fails from home Wi-Fi.
Debugging Steps:
Verified instance was running and public IP was correct:
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].State.Name'
Output:
"running"
Tested port 22 with nc (netcat):
nc -vz <PUBLIC_IP> 22
Output:
nc: connectx to <PUBLIC_IP> port 22 (tcp) failed: Operation timed out
Checked Network ACLs and confirmed default ACL allows all traffic.
Conclusion / Recommendation:
Issue is with ISP blocking outbound SSH (port 22).
Verified by connecting via phone hotspot — SSH succeeded.
Recommendation if encountering this:
Test SSH from another network (hotspot, VPN, or cloud network).
Consider using port 443 tunneling or AWS Systems Manager if persistent ISP blocks.
Ensure security group is configured to allow SSH from your connecting IP.



15. Adjust Security Group as Needed
To remove wide SSH access (0.0.0.0/0) and restrict to your home IP:
aws ec2 revoke-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
Output:
{
    "Return": true
}
Add your public IP back:
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 22 \
    --cidr YOUR_PUBLIC_IP/32
Output:
InvalidPermission.Duplicate  # Ignore if rule already exists

Notes / Observations:
Always restrict SSH to known IPs for security.
Verify connectivity from allowed IP after changes.

✅ At this point:
VPC, subnet, IGW, route table, security group, key pair, and EC2 instance are all set up.
Instance is accessible via SSH from networks that allow outbound port 22.
Issues were mainly due to missing public IP and ISP restrictions on SSH.


16. Configure AWS Systems Manager (SSM) for EC2 access without SSH

Step 1: Create IAM Role for SSM
Command:
aws iam create-role \
    --role-name lab-ec2-ssm-role \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "ec2.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'
Output:
{
    "Role": {
        "RoleName": "lab-ec2-ssm-role",
        "Arn": "arn:aws:iam::xxx:role/lab-ec2-ssm-role"
    }
}

Step 2: Attach AmazonSSMManagedInstanceCore policy
Command:
aws iam attach-role-policy \
    --role-name lab-ec2-ssm-role \
    --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
Output:
{}

Step 3: Create IAM Instance Profile for EC2
Command:
aws iam create-instance-profile \
    --instance-profile-name lab-ec2-ssm-profile
Output:
{
    "InstanceProfile": {
        "InstanceProfileName": "lab-ec2-ssm-profile",
        "Arn": "arn:aws:iam::xxx:instance-profile/lab-ec2-ssm-profile"
    }
}

Step 4: Add role to instance profile
aws iam add-role-to-instance-profile \
    --instance-profile-name lab-ec2-ssm-profile \
    --role-name lab-ec2-ssm-role
Output:
{}



Step 5: Associate instance profile with EC2 instance
aws ec2 associate-iam-instance-profile \
    --instance-id i-xxx \
    --iam-instance-profile Name=lab-ec2-ssm-profile
Output:
{
    "IamInstanceProfileAssociation": {
        "InstanceId": "i-xxx",
        "IamInstanceProfile": {
            "Arn": "arn:aws:iam::xxx:instance-profile/lab-ec2-ssm-profile"
        },
        "State": "associating"
    }
}



Step 6: Verify SSM agent on the instance
Problem Encountered:
Amazon Linux 2 usually comes with amazon-ssm-agent pre-installed.
For some AMIs, the agent may not be installed or systemctl/status commands may fail.
On Mac SSH terminal, systemctl isn’t available; you can use:
sudo amazon-ssm-agent -version
or check installation with:
ps aux | grep amazon-ssm-agent

Step 7: Start SSM Session
Command:
aws ssm start-session --target i-xxx

Problem Encountered:
Output: TargetNotConnected
Cause: SSM agent wasn’t running or instance not fully connected yet.

Recommendation / Fix:
Ensure the EC2 has the correct IAM instance profile attached.

Make sure the SSM agent is installed and running.

Verify network connectivity (port 443 for HTTPS).

SSM avoids port 22 entirely, so it works even if ISP blocks SSH.

Observation / Benefit:
Once configured, you can SSH over Session Manager (HTTPS) from any network.

No need to update security groups for each new IP.

Perfect for lab practice, real deployments, and DevSecOps workflows.

✅ At this point:
EC2 is fully accessible via SSH (port 22) or SSM (HTTPS, port 443).
Security rules are tightened, and public IP dependency is minimized.
ISP SSH restrictions no longer block management access.