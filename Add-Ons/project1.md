# AWS Three-Tier Architecture Setup Guide

## Overview
This guide walks through building a three-tier AWS application architecture with a VPC, application servers, load balancer, and managed database.

---

## 1. AWS Networking Fundamentals

### Key Concepts
- **VPC (Virtual Private Cloud)**: Your isolated network environment in AWS
- **Subnets**: Subdivisions within a VPC that group resources
- **Route Tables**: Control traffic flow between subnets
- **Internet Gateway (IGW)**: Enables communication with the internet
- **NAT Gateway**: Allows private resources to access the internet securely
- **Application Load Balancer (ALB)**: Distributes traffic across multiple servers

---

## 2. Create VPC

### Steps
1. Navigate to AWS VPC Dashboard
2. Click **Create VPC**
3. Configure settings:
   - **VPC Name**: `three-tier-vpc`
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - Leave DNS settings as default
4. Click **Create VPC**

### Why This CIDR?
The `/16` network provides 65,536 IP addresses for your infrastructure.

---

## 3. Create Subnets

### Subnet Strategy
Create 6 subnets across 2 availability zones for high availability:

#### Public Subnets (for IGW access)
| Subnet Name | Availability Zone | CIDR Block | Purpose |
|---|---|---|---|
| public-subnet-1 | us-east-1a | 10.0.1.0/24 | IGW, NAT Gateway, Jump Server |
| public-subnet-2 | us-east-1b | 10.0.2.0/24 | Alternate AZ |

#### Private Subnets (for application servers)
| Subnet Name | Availability Zone | CIDR Block | Purpose |
|---|---|---|---|
| private-subnet-1 | us-east-1a | 10.0.3.0/24 | PHP/Apache Servers |
| private-subnet-2 | us-east-1b | 10.0.4.0/24 | PHP/Apache Servers |

#### Database Subnets (for RDS)
| Subnet Name | Availability Zone | CIDR Block | Purpose |
|---|---|---|---|
| db-subnet-1 | us-east-1a | 10.0.5.0/24 | RDS Database |
| db-subnet-2 | us-east-1b | 10.0.6.0/24 | RDS Multi-AZ |

### Create Each Subnet
1. Go to **Subnets** in VPC Dashboard
2. Click **Create Subnet**
3. For each subnet:
   - Select your VPC
   - Enter subnet name
   - Select Availability Zone
   - Enter IPv4 CIDR block
   - Click **Create Subnet**

---

## 4. Create Route Tables

### Create Public Route Table

1. Navigate to **Route Tables**
2. Click **Create Route Table**
   - **Name**: `public-rt`
   - **VPC**: Select your three-tier-vpc
3. Click **Create Route Table**

### Create Private Route Tables

Create two private route tables (one per AZ for redundancy):
1. Create second route table named `private-rt-1a`
2. Create third route table named `private-rt-1b`

---

## 5. Route Table Subnet Associations

### Associate Public Subnets
1. Select `public-rt`
2. Go to **Subnet Associations** tab
3. Click **Edit Subnet Associations**
4. Select `public-subnet-1` and `public-subnet-2`
5. Click **Save Associations**

### Associate Private Subnets
1. Select `private-rt-1a`
2. Associate with `private-subnet-1`
3. Select `private-rt-1b`
4. Associate with `private-subnet-2`

### Associate Database Subnets
1. Create new route table `db-rt`
2. Associate with `db-subnet-1` and `db-subnet-2`

---

## 6. Create Internet Gateway (IGW)

### Steps
1. Go to **Internet Gateways**
2. Click **Create Internet Gateway**
   - **Name**: `three-tier-igw`
3. Click **Create Internet Gateway**
4. Select the newly created IGW
5. Click **Attach to VPC**
6. Select your VPC and confirm

### Add IGW Route
1. Go to **Route Tables** → `public-rt`
2. Click **Edit Routes**
3. Click **Add Route**:
   - **Destination**: `0.0.0.0/0` (all internet traffic)
   - **Target**: Select your Internet Gateway
4. Click **Save Routes**

---

## 7. Create NAT Gateway

### Allocate Elastic IP
1. Go to **Elastic IPs**
2. Click **Allocate Elastic IP Address**
3. Click **Allocate**

### Create NAT Gateway
1. Go to **NAT Gateways**
2. Click **Create NAT Gateway**
   - **Subnet**: Select `public-subnet-1`
   - **Elastic IP Allocation ID**: Select the IP you just created
3. Click **Create NAT Gateway**
4. Wait for state to change to "Available"

---

## 8. Add Routes for IGW and NAT

### Update Public Route Table
Already completed in step 6 - public subnets route to IGW.

### Update Private Route Tables
1. Select `private-rt-1a`
2. Click **Edit Routes**
3. Click **Add Route**:
   - **Destination**: `0.0.0.0/0`
   - **Target**: NAT Gateway (select the one in us-east-1a)
4. Save Routes
5. Repeat for `private-rt-1b` if you created a NAT in 1b

---

## 9. Create Jump Server (Bastion Host)

### Launch Instance
1. Go to **EC2 Dashboard**
2. Click **Launch Instances**
3. Configure:
   - **Name**: `jump-server`
   - **AMI**: Amazon Linux 2
   - **Instance Type**: `t2.micro`
   - **VPC**: Select your three-tier-vpc
   - **Subnet**: `public-subnet-1`
   - **Auto-assign Public IP**: Enable
4. Create or select a key pair for SSH access
5. Create security group:
   - **Name**: `jump-server-sg`
   - **Inbound Rule**: SSH (port 22) from your IP (0.0.0.0/0 for testing)
6. Click **Launch Instance**

### Access Jump Server
```bash
ssh -i your-key.pem ec2-user@<jump-server-public-ip>
```

---

## 10. Create PHP/Apache Servers

### Create Security Group for App Servers
1. Go to **Security Groups**
2. Click **Create Security Group**
   - **Name**: `app-servers-sg`
   - **VPC**: Select your VPC
3. Add inbound rules:
   - HTTP (80): From your ALB security group
   - HTTPS (443): From your ALB security group
   - SSH (22): From jump-server-sg
4. Click **Create Security Group**

### Launch PHP/Apache Servers
1. Launch first instance:
   - **Name**: `php-server-1`
   - **AMI**: Amazon Linux 2
   - **Instance Type**: `t2.micro`
   - **VPC**: your three-tier-vpc
   - **Subnet**: `private-subnet-1`
   - **Security Group**: `app-servers-sg`
   - **Auto-assign Public IP**: Disable
2. Click **Launch Instance**
3. Repeat to create `php-server-2` in `private-subnet-2`

---

## 11. Install PHP and Apache

### SSH into Server via Jump Server
```bash
# From your local machine, SSH into jump server first
ssh -i your-key.pem ec2-user@<jump-server-ip>

# From jump server, SSH into PHP server (using private IP)
ssh -i your-key.pem ec2-user@10.0.3.10  # Private IP of php-server-1
```

### Install Apache and PHP
```bash
# Update system
sudo yum update -y

# Install Apache
sudo yum install httpd -y

# Install PHP
sudo yum install php php-mysqli -y

# Enable and start Apache
sudo systemctl enable httpd
sudo systemctl start httpd

# Verify Apache is running
sudo systemctl status httpd
```

### Create Test PHP File
```bash
sudo nano /var/www/html/index.php
```

Add content:
```php
<?php
  echo "Server: " . gethostname() . "<br>";
  echo "IP: " . $_SERVER['SERVER_ADDR'] . "<br>";
?>
```

Save (Ctrl+O, Enter, Ctrl+X)

---

## 12. Install phpMyAdmin

### Install phpMyAdmin
```bash
# Install additional dependencies
sudo yum install php-xml -y

# Download phpMyAdmin
cd /var/www/html
sudo wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.tar.gz

# Extract
sudo tar -xzf phpMyAdmin-5.1.1-all-languages.tar.gz
sudo mv phpMyAdmin-5.1.1-all-languages phpmyadmin

# Set permissions
sudo chown -R apache:apache phpmyadmin
sudo chmod -R 755 phpmyadmin
```

### Configure phpMyAdmin
```bash
cd /var/www/html/phpmyadmin
sudo cp config.sample.inc.php config.inc.php
```

---

## 13. Create and Configure Application Load Balancer

### Create Target Group
1. Go to **Target Groups** (EC2 Dashboard)
2. Click **Create Target Group**
3. Configure:
   - **Type**: Instances
   - **Name**: `php-target-group`
   - **Protocol**: HTTP
   - **Port**: 80
   - **VPC**: Select your VPC
4. Under health checks:
   - **Health Check Path**: `/index.php`
   - **Matcher**: 200
5. Click **Create Target Group**

### Register Targets
1. Go to your target group
2. Click **Register Targets**
3. Select both PHP servers and click **Include as Pending Below**
4. Click **Register Targets**

### Create Load Balancer
1. Go to **Load Balancers**
2. Click **Create Load Balancer** → **Application Load Balancer**
3. Configure:
   - **Name**: `app-lb`
   - **Scheme**: Internet-facing
   - **IP Address Type**: IPv4
4. Under Network Mapping:
   - Select your VPC
   - Select `public-subnet-1` and `public-subnet-2`
5. Create security group:
   - **Name**: `alb-sg`
   - **Inbound**: HTTP (80) from 0.0.0.0/0, HTTPS (443) from 0.0.0.0/0
6. Create listener:
   - **Protocol**: HTTP
   - **Port**: 80
   - **Target Group**: `php-target-group`
7. Click **Create Load Balancer**

### Verify Load Balancer
Wait for state to change to "Active", then access via the DNS name provided.

---

## 14. Create RDS Instance

### Create DB Subnet Group
1. Go to **RDS Dashboard**
2. Click **DB Subnet Groups**
3. Click **Create DB Subnet Group**
   - **Name**: `three-tier-db-subnet`
   - **VPC**: Select your VPC
   - **Subnets**: Select `db-subnet-1` and `db-subnet-2`
4. Click **Create**

### Create Security Group for RDS
1. Go to **Security Groups**
2. Click **Create Security Group**
   - **Name**: `rds-sg`
   - **VPC**: Select your VPC
3. Add inbound rule:
   - **Type**: MySQL/Aurora (3306)
   - **Source**: `app-servers-sg` (from your app server security group)
4. Click **Create Security Group**

### Create RDS Instance
1. Go to **RDS Dashboard** → **Databases**
2. Click **Create Database**
3. Configure:
   - **Engine**: MySQL
   - **Version**: MySQL 8.0
   - **Templates**: Free tier
   - **DB Instance Identifier**: `three-tier-db`
   - **Master Username**: `admin`
   - **Master Password**: Create strong password
4. Under Connectivity:
   - **VPC**: Select your VPC
   - **DB Subnet Group**: `three-tier-db-subnet`
   - **Public Accessibility**: No
   - **VPC Security Groups**: `rds-sg`
5. Under Backup:
   - **Backup Retention Period**: 7 days
   - **Multi-AZ Deployment**: Enable (for production)
6. Click **Create Database**

---

## 15. Configure phpMyAdmin with RDS

### Get RDS Endpoint
1. Go to **RDS Databases**
2. Select your database instance
3. Copy the **Endpoint** (e.g., three-tier-db.xxxxxxx.us-east-1.rds.amazonaws.com)

### Configure phpMyAdmin
```bash
# SSH into your PHP server via jump server
ssh -i your-key.pem -J ec2-user@<jump-server-ip> ec2-user@10.0.3.10

# Edit phpMyAdmin config
sudo nano /var/www/html/phpmyadmin/config.inc.php
```

Update the configuration:
```php
<?php
$i = 0;
$i++;

$cfg['Servers'][$i]['host'] = 'three-tier-db.xxxxxxx.us-east-1.rds.amazonaws.com';
$cfg['Servers'][$i]['port'] = '3306';
$cfg['Servers'][$i]['user'] = 'admin';
$cfg['Servers'][$i]['password'] = 'your-rds-password';
$cfg['Servers'][$i]['auth_type'] = 'config';

$cfg['blowfish_secret'] = 'your-random-secret-key-here';
```

### Restart Apache
```bash
sudo systemctl restart httpd
```

### Access phpMyAdmin
Navigate to: `http://your-load-balancer-dns/phpmyadmin`

---

## 16. Configure Session Stickiness

### Enable Sticky Sessions on Load Balancer
1. Go to **Target Groups**
2. Select `php-target-group`
3. Click **Edit Attributes**
4. Check **Stickiness** and set:
   - **Duration**: 86400 seconds (24 hours)
   - **Cookie**: Leave as default
5. Click **Save**

### Why Stickiness?
Ensures user sessions persist on the same PHP server, preventing session loss during page navigations.

---

## 17. Services Recap

### Architecture Components

**Networking Layer**
- VPC (10.0.0.0/16): Isolated network environment
- Public Subnets: Host jump server and load balancer
- Private Subnets: Host PHP application servers
- Database Subnets: Host RDS instance

**Security Layer**
- Security Groups: Control inbound/outbound traffic
- Jump Server: Bastion host for secure server access
- NAT Gateway: Secure outbound internet for private servers

**Application Layer**
- Application Load Balancer: Distributes HTTP traffic
- PHP/Apache Servers: Process application logic
- phpMyAdmin: Database management interface

**Data Layer**
- RDS MySQL: Managed relational database
- Multi-AZ: Automatic failover for high availability

---

## Cleanup

To avoid unnecessary charges:

1. Delete Load Balancer
2. Terminate EC2 Instances
3. Delete RDS Database
4. Release Elastic IPs
5. Delete NAT Gateway
6. Detach and Delete Internet Gateway
7. Delete Subnets
8. Delete VPC
9. Delete Security Groups
10. Delete Key Pairs

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Cannot SSH to private servers | Ensure jump server is in public subnet and has public IP |
| PHP servers unreachable | Check security group rules allow traffic from ALB |
| RDS connection fails | Verify RDS security group allows port 3306 from app servers |
| phpMyAdmin login fails | Confirm RDS credentials and endpoint in config.inc.php |
| ALB shows unhealthy targets | Check health check path and ensure Apache is running |

---

## Best Practices

- Use multiple AZs for high availability
- Keep databases in private subnets
- Use security groups as firewalls
- Enable RDS backups and Multi-AZ
- Use jump server instead of public IPs for servers
- Implement session stickiness for stateful applications
- Monitor CloudWatch metrics regularly
- Use Auto Scaling for production workloads
