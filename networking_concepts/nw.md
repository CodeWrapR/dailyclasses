# AWS Networking Labs: Complete Guide from Basic to Advanced
## With EC2 Connectivity Testing and Full Cleanup Procedures

---

## Overview

This guide contains 16 progressive labs covering AWS networking from foundational to enterprise-level scenarios. Each lab includes objectives, step-by-step instructions, EC2 connectivity testing, and complete cleanup procedures.

**Services Covered:**
- VPC (Virtual Private Cloud)
- Subnets (Public & Private)
- Security Groups & NACLs
- Gateways (IGW, NAT, VPN)
- Route Tables
- VPC Peering
- Transit Gateway
- VPC Flow Logs
- CloudWatch Monitoring

---

## SECTION 1: VPC FUNDAMENTALS (Labs 1-4)

### Lab 1: Creating Your First VPC (Virtual Private Cloud)

**Objective:** Understand VPCs and create your first isolated network.

**Why This Matters:**
- VPC = Your private network in AWS
- Like renting your own network infrastructure
- Complete isolation from others
- Full control over IP addresses

### Understanding VPCs

```
AWS Account
├─ VPC 1 (Your network)
│  ├─ Your IP addresses: 10.0.0.0/16
│  ├─ Your subnets
│  ├─ Your servers
│  └─ Completely isolated
│
├─ VPC 2 (Different app)
│  ├─ Different IP addresses: 172.31.0.0/16
│  ├─ Different subnets
│  └─ No access to VPC 1
│
└─ Internet (Outside)
```

### Step-By-Step Instructions

**Step 1: Navigate to VPC Console**
- Open AWS Console
- Search for "VPC"
- Click "VPC" service
- Click "Your VPCs"

**Step 2: Create VPC**
- Click "Create VPC"
- Name: `MyFirstVPC`
- IPv4 CIDR: `10.0.0.0/16`
  - This means IP range from 10.0.0.0 to 10.0.255.255
  - 65,536 possible IP addresses
- IPv6 CIDR: Leave empty
- Tenancy: Default
- Click "Create VPC"

**Step 3: Understand CIDR Notation**

```
CIDR: 10.0.0.0/16

First number set: 10.0.0.0 (Network address)
/16 (How many bits for network)

Examples:
10.0.0.0/16 = 10.0.0.0 to 10.0.255.255 (65,536 IPs)
10.0.0.0/24 = 10.0.0.0 to 10.0.0.255 (256 IPs)
10.0.0.0/28 = 10.0.0.0 to 10.0.0.15 (16 IPs)

Higher number after / = Fewer IPs (more restricted)
```

**Step 4: View VPC Details**
- Click on your VPC
- See CIDR block: 10.0.0.0/16
- DNS resolution: Enabled
- DNS hostnames: Disabled (enable it)
  - This lets instances have domain names

**Step 5: Enable DNS Hostnames**
- Click "Edit VPC Settings"
- Enable "DNS hostnames"
- Click "Save"

### What You Just Created

Your isolated network with:
- Own IP address range (10.0.0.0/16)
- DNS configuration
- Route tables (created automatically)
- Network ACLs (created automatically)
- All ready for subnets

### Real-World Analogy

```
Physical World:
You rent an office building
├─ Address: 123 Main Street
├─ Security: Only employees inside
└─ Phone system: Only internal calls initially

VPC:
You create a VPC
├─ CIDR: 10.0.0.0/16
├─ Security: Isolation from other customers
└─ Network: Only internal communication initially
```

### Validation Checklist
- [ ] VPC created with name `MyFirstVPC`
- [ ] CIDR block: 10.0.0.0/16
- [ ] DNS hostnames enabled
- [ ] Can see VPC in list
- [ ] Note VPC ID (format: vpc-xxxxxxxx)

### Key Takeaway
✅ VPC = Private network space  
✅ Complete isolation  
✅ You control IP addressing  
✅ Foundation for everything else  

---

### Lab 2: Creating Subnets (Public & Private)

**Objective:** Divide VPC into smaller networks with different purposes.

**Why This Matters:**
- Subnets = Sections of your network
- Public subnet = Accessible from internet (web servers)
- Private subnet = Not accessible from internet (databases)
- Organize resources by security level

### Understanding Subnets

```
VPC: 10.0.0.0/16
│
├─ Public Subnet: 10.0.1.0/24 (256 IPs)
│  ├─ Web Server A
│  └─ Web Server B
│
└─ Private Subnet: 10.0.2.0/24 (256 IPs)
   ├─ Database
   └─ Internal App Server

Public = Accessible from internet
Private = Only from within VPC
```

### Step-By-Step Instructions

**Step 1: Create Public Subnet**
- Go to VPC Console → Subnets
- Click "Create Subnet"
- VPC: MyFirstVPC
- Subnet name: `PublicSubnet-AZ1`
- Availability Zone: us-east-1a (choose first)
- IPv4 CIDR: `10.0.1.0/24` (256 IPs in this subnet)
- Click "Create Subnet"

**Step 2: Enable Public IP Assignment**
- Select PublicSubnet-AZ1
- Click "Edit Subnet Settings"
- Enable "Auto-assign public IPv4 address"
- Click "Save"

This automatically gives instances public IPs!

**Step 3: Create Private Subnet**
- Click "Create Subnet"
- VPC: MyFirstVPC
- Subnet name: `PrivateSubnet-AZ1`
- Availability Zone: us-east-1a
- IPv4 CIDR: `10.0.2.0/24`
- Click "Create Subnet"

**Step 4: Verify Subnets**
View both subnets:
```
PublicSubnet-AZ1: 10.0.1.0/24
├─ Availability Zone: us-east-1a
├─ Auto-assign public IPv4: Enabled ✅
└─ Available IPs: 251 (5 reserved by AWS)

PrivateSubnet-AZ1: 10.0.2.0/24
├─ Availability Zone: us-east-1a
├─ Auto-assign public IPv4: Disabled
└─ Available IPs: 251
```

### CIDR Planning for Your VPC

```
VPC: 10.0.0.0/16 (65,536 IPs total)

Environment 1:
├─ PublicSubnet-AZ1: 10.0.1.0/24
├─ PrivateSubnet-AZ1: 10.0.2.0/24
└─ Available: 10.0.3.0 - 10.0.255.0

Environment 2:
├─ PublicSubnet-AZ2: 10.0.11.0/24
├─ PrivateSubnet-AZ2: 10.0.12.0/24
└─ Available: 10.0.13.0 onwards

Growth Planning: Always leave room!
```

### Best Practices

✅ **DO:**
- Plan CIDR blocks before creating
- Use /24 for small subnets
- Separate public/private clearly
- Spread across multiple AZs
- Document your IP plan

❌ **DON'T:**
- Use overlapping CIDR blocks
- Make subnets too small
- Put databases in public subnets
- Use entire range in one subnet

### Validation Checklist
- [ ] Public subnet created: 10.0.1.0/24
- [ ] Private subnet created: 10.0.2.0/24
- [ ] Both in AZ: us-east-1a
- [ ] Public subnet has auto-assign public IP enabled
- [ ] Can see both in subnet list

### Key Takeaway
✅ Subnets divide VPC into sections  
✅ Public = internet accessible  
✅ Private = internal only  
✅ Plan CIDR blocks carefully  

---

### Lab 3: Internet Gateway (Making Public Subnets Public)

**Objective:** Connect your VPC to the internet.

**Why This Matters:**
- Internet Gateway (IGW) = Door to internet
- Needed for instances to reach internet
- Needed for internet to reach instances (with security groups)
- Without IGW, no internet communication

### Understanding Internet Gateways

```
Without IGW:
VPC (10.0.0.0/16) ═══ BLOCKED ═══ Internet

With IGW:
VPC (10.0.0.0/16) ───── IGW ───── Internet
                 (Route configured)
```

### Step-By-Step Instructions

**Step 1: Create Internet Gateway**
- Go to VPC Console → Internet Gateways
- Click "Create Internet Gateway"
- Name: `MyFirstIGW`
- Click "Create Internet Gateway"

**Step 2: Attach IGW to VPC**
- Select your IGW
- Click "Attach to VPC"
- VPC: MyFirstVPC
- Click "Attach Internet Gateway"

Status changes to "Available"! ✅

**Step 3: Create Route Table (or use existing)**
- Go to Route Tables
- Look for one associated with MyFirstVPC
- If none, click "Create Route Table":
  - Name: `PublicRouteTable`
  - VPC: MyFirstVPC
  - Click "Create"

**Step 4: Add Route to Internet**
- Select route table
- Click "Edit Routes"
- Click "Add Route"
- Destination: `0.0.0.0/0` (all internet traffic)
- Target: Select your IGW (MyFirstIGW)
- Click "Save Routes"

Route added:
```
Destination     Target
0.0.0.0/0       MyFirstIGW
(all internet)   (send to internet gateway)
```

**Step 5: Associate Route Table with Public Subnet**
- In route table, click "Subnet Associations"
- Click "Edit Subnet Associations"
- Select: PublicSubnet-AZ1
- Click "Save Associations"

**Step 6: Verify Setup**
Route table now shows:
```
Destination        Target           Status
10.0.0.0/16        Local            Active (within VPC)
0.0.0.0/0          MyFirstIGW       Active (to internet)
```

### Understanding the Flow

```
Instance in PublicSubnet
    ↓
Sends packet to 8.8.8.8
    ↓
Route table checks: "8.8.8.8, where do I send this?"
    ↓
Route matches: 0.0.0.0/0 → IGW
    ↓
Packet sent to IGW
    ↓
IGW sends to internet
    ↓
Response comes back
    ↓
IGW routes back to instance
    ↓
Done! ✅
```

### Real-World Scenario

```
Company Network:
Office (10.0.1.0/24) with Internet Gateway
├─ Employee computers can access internet
├─ Customers can access company website
└─ All through the gateway

Without Gateway:
Office isolated completely
└─ Can't access anything outside
```

### Validation Checklist
- [ ] Internet Gateway created and attached
- [ ] Route table created and associated with public subnet
- [ ] Route added: 0.0.0.0/0 → IGW
- [ ] Route table shows both Local and IGW routes
- [ ] Status shows "Active"

### Key Takeaway
✅ IGW = Door to internet  
✅ Route table directs traffic  
✅ Must be associated with subnet  
✅ Without IGW, no internet access  

---

### Lab 4: Launching EC2 in VPC (Test Connectivity)

**Objective:** Launch EC2 instance in your VPC and test internet connectivity.

**Why This Matters:**
- First real test of your network setup
- Verify routing works
- Test public/private connectivity
- Real-world networking in action

### Step-By-Step Instructions

**Step 1: Launch EC2 Instance**
- Go to EC2 Console
- Click "Launch Instances"
- Name: `TestServer-Public`
- AMI: Amazon Linux 2
- Instance type: t2.micro
- Click "Next"

**Step 2: Configure Network**
- VPC: MyFirstVPC
- Subnet: PublicSubnet-AZ1
- Auto-assign public IP: Yes (should be auto from subnet)
- Click "Next"

**Step 3: Configure Security Group**
- Name: `PublicSG`
- Allow:
  - SSH (22) from your IP: 0.0.0.0/0 (we'll restrict after testing)
  - HTTP (80) from 0.0.0.0/0
  - HTTPS (443) from 0.0.0.0/0
  - All ICMP from 0.0.0.0/0 (for ping test)
- Click "Launch"

**Step 4: Download Key Pair**
- Create new key pair
- Name: `NetworkLab-Key`
- Download and save safely

**Step 5: Wait for Instance to Start**
- Go to Instances
- Wait for status: "running"
- Note the public IP address: 3.xxx.xxx.xxx

**Step 6: Connect to Instance**
- Select instance
- Click "Connect"
- Use "Session Manager" or SSH

Via SSH:
```bash
ssh -i NetworkLab-Key.pem ec2-user@3.xxx.xxx.xxx
```

**Step 7: Test Internet Connectivity from Instance**

Inside the instance, run:

```bash
# Test DNS resolution
nslookup google.com
# Should resolve to IP addresses

# Test ping (ICMP)
ping -c 5 google.com
# Should see responses (5 packets sent, 5 received)

# Test HTTP
curl http://www.google.com
# Should see HTML response

# Check your public IP
curl http://checkip.amazonaws.com
# Should show 3.xxx.xxx.xxx (your EC2 public IP)

# Check your internal IP
hostname -I
# Should show 10.0.1.x (your private IP in subnet)
```

### Connectivity Test Results

```
✅ Instance has private IP: 10.0.1.x
✅ Instance has public IP: 3.xxx.xxx.xxx
✅ Can reach internet (ping, curl work)
✅ Route table directing traffic to IGW
✅ IGW translating addresses (NAT-like behavior)
✅ Everything working!
```

### Understanding the Flow

```
Your Laptop (203.0.113.x)
    ↓
SSH to 3.xxx.xxx.xxx (public IP)
    ↓
Internet
    ↓
IGW (maps external IP to internal)
    ↓
Instance at 10.0.1.x
    ↓
Can ping google.com
    ↓
All working! ✅
```

### Validation Checklist
- [ ] Instance in PublicSubnet-AZ1
- [ ] Instance has public IP (3.xxx.xxx.xxx)
- [ ] Can SSH into instance
- [ ] Can ping google.com
- [ ] Can curl http://checkip.amazonaws.com
- [ ] Sees correct public IP

---

## POST-LAB 1 CLEANUP

**Stop but don't delete (for reuse in next labs):**

```bash
# In EC2 Console:
1. Select instance
2. Instance State → Stop (don't terminate yet)
3. This stops charges while keeping instance

# Or use AWS CLI:
aws ec2 stop-instances --instance-ids i-xxxxxxxxx --region us-east-1
```

**Estimated cost so far:**
- VPC: Free
- IGW: Free
- Route tables: Free
- EC2 t2.micro (stopped): Free
- Total: $0 ✅

---

## SECTION 2: SUBNET MANAGEMENT & ROUTING (Labs 5-7)

### Lab 5: Private Subnets and NAT Gateway (Internet Access for Private Servers)

**Objective:** Enable internet access for private subnet instances without exposing them.

**Why This Matters:**
- Private instances need software updates
- Need to download packages
- Can't expose databases to internet
- NAT Gateway = Solution!

### Understanding NAT Gateway

```
Private Subnet Instance (10.0.2.x)
    ↓
Wants to reach internet
    ↓
Can't go directly (no IGW)
    ↓
Sends to NAT Gateway (in public subnet)
    ↓
NAT Gateway translates address
    ↓
Sends out to internet as if from itself
    ↓
Response comes back
    ↓
NAT Gateway sends to instance
    ↓
Instance gets response ✅
```

### Step-By-Step Instructions

**Step 1: Create Elastic IP for NAT Gateway**
- Go to VPC Console → Elastic IPs
- Click "Allocate Elastic IP Address"
- Click "Allocate"
- Copy the IP (example: 3.yyy.yyy.yyy)

**Step 2: Create NAT Gateway**
- Go to VPC Console → NAT Gateways
- Click "Create NAT Gateway"
- Subnet: PublicSubnet-AZ1 (must be in public subnet!)
- Elastic IP allocation: Your elastic IP
- Click "Create NAT Gateway"

Wait 1-2 minutes for creation...

**Step 3: Create Route Table for Private Subnet**
- Go to Route Tables
- Click "Create Route Table"
- Name: `PrivateRouteTable`
- VPC: MyFirstVPC
- Click "Create"

**Step 4: Add Route to NAT Gateway**
- Select PrivateRouteTable
- Click "Edit Routes"
- Click "Add Route"
- Destination: `0.0.0.0/0` (all internet)
- Target: NAT Gateway (select your NAT)
- Click "Save Routes"

```
Private Route Table:
Destination        Target
10.0.0.0/16        Local
0.0.0.0/0          NAT Gateway
(VPC local)        (to internet via NAT)
```

**Step 5: Associate Route Table with Private Subnet**
- In PrivateRouteTable, click "Subnet Associations"
- Click "Edit Subnet Associations"
- Select: PrivateSubnet-AZ1
- Click "Save Associations"

**Step 6: Launch Instance in Private Subnet**
- Go to EC2
- Click "Launch Instances"
- Name: `TestServer-Private`
- VPC: MyFirstVPC
- Subnet: PrivateSubnet-AZ1
- Auto-assign public IP: Disable (on purpose!)
- Security Group: Create new
  - Name: `PrivateSG`
  - Allow: All traffic from 10.0.0.0/16 (within VPC)
  - Deny: Everything from internet
- Launch

**Step 7: Test Private Instance Connectivity**

Connect via Session Manager (only way to access private instance):
- Select instance
- Click "Connect"
- Use "Session Manager" tab

Inside instance:
```bash
# Test internet access
ping -c 5 google.com
# Should work! (going through NAT Gateway)

# Check public IP
curl http://checkip.amazonaws.com
# Shows NAT Gateway's IP (3.yyy.yyy.yyy), not instance's

# Download package (proves internet access)
sudo yum update -y
# Should work!

# But you can't SSH directly to this instance
# (no public IP, Security Group blocks)
```

### Understanding Private Instance Security

```
Scenario 1: Try to SSH to private instance
External IP (203.0.113.x) → Instance (10.0.2.x)
    ↓
Security Group blocks SSH from outside VPC
    ↓
Connection denied ✅ (secure!)

Scenario 2: Private instance reaches internet
Instance (10.0.2.x) → NAT Gateway (3.yyy.yyy.yyy)
    ↓
NAT translates source address
    ↓
Internet sees request from 3.yyy.yyy.yyy
    ↓
Response comes back
    ↓
NAT routes to instance
    ↓
Instance gets response ✅ (secure + functional!)
```

### Cost of NAT Gateway

```
NAT Gateway: $0.045 per hour + $0.045 per GB processed
= ~$32/month base + data transfer costs

Alternative: NAT Instance (old way)
= Cheaper but more work to manage

Use NAT Gateway when: Production (reliability needed)
Use NAT Instance when: Dev/test (cost conscious)
```

### Real-World Architecture

```
VPC: 10.0.0.0/16
│
├─ Public Subnet: 10.0.1.0/24
│  ├─ Web Server (can be accessed from internet)
│  ├─ NAT Gateway (translates traffic)
│  └─ Internet Gateway (door to internet)
│
└─ Private Subnet: 10.0.2.0/24
   ├─ Database Server (cannot be accessed from internet)
   ├─ App Server (updates via NAT)
   └─ Route: 0.0.0.0/0 → NAT Gateway
```

### Validation Checklist
- [ ] Elastic IP created and allocated
- [ ] NAT Gateway created in public subnet
- [ ] Route table created for private subnet
- [ ] Route added: 0.0.0.0/0 → NAT Gateway
- [ ] Private subnet associated with private route table
- [ ] Private instance has NO public IP
- [ ] Can access private instance via Session Manager
- [ ] Private instance can ping google.com
- [ ] Private instance can update packages

### Key Takeaway
✅ NAT Gateway enables internet access  
✅ Instances stay private (no public IP)  
✅ Secure + functional  
✅ Necessary for production databases  

---

### Lab 6: Multiple Availability Zones (High Availability)

**Objective:** Spread resources across multiple AZs for redundancy.

**Why This Matters:**
- Single AZ = Single point of failure
- If AZ goes down, everything down
- Multiple AZs = True high availability
- AWS best practice

### Understanding Multiple AZs

```
Scenario 1: Single AZ (Risky)
AZ-1a
├─ Web Server
├─ Database
└─ NAT Gateway

If AZ-1a fails → Everything down! ❌

Scenario 2: Multiple AZs (Safe)
AZ-1a                  AZ-1b
├─ Web Server 1        ├─ Web Server 2
├─ Database Primary    ├─ Database Replica
└─ NAT Gateway 1       └─ NAT Gateway 2

If AZ-1a fails → AZ-1b handles traffic ✅
```

### Step-By-Step Instructions

**Step 1: Create Subnets in Second AZ**
- Go to VPC → Subnets
- Click "Create Subnet"
- VPC: MyFirstVPC
- Name: `PublicSubnet-AZ2`
- AZ: us-east-1b (different from AZ1!)
- CIDR: `10.0.11.0/24`
- Create

Repeat for private:
- Name: `PrivateSubnet-AZ2`
- AZ: us-east-1b
- CIDR: `10.0.12.0/24`

**Step 2: Create Second NAT Gateway**
- Go to NAT Gateways
- Create new
- Subnet: PublicSubnet-AZ2
- New Elastic IP
- Create NAT Gateway

**Step 3: Update Route Tables**

For PublicRouteTable:
- Already routes 0.0.0.0/0 to IGW
- Works for both public subnets ✅

For PrivateRouteTable:
- Currently routes 0.0.0.0/0 to NAT-1 (in AZ1)
- Need to update to prefer local NAT

**Step 4: Create Separate Route Table for AZ2 Private**
- Create new route table: `PrivateRouteTable-AZ2`
- Add route: 0.0.0.0/0 → NAT Gateway 2 (in AZ2)
- Associate with PrivateSubnet-AZ2

**Step 5: Associate Subnets with Route Tables**

Public Route Table:
- PublicSubnet-AZ1 ✓
- PublicSubnet-AZ2 ✓
(Both use same route table → same routes)

Private Route Tables:
- PrivateRouteTable → PrivateSubnet-AZ1
- PrivateRouteTable-AZ2 → PrivateSubnet-AZ2

**Step 6: Launch Instances Across AZs**

Public Instance in AZ2:
- Name: `TestServer-Public-AZ2`
- VPC: MyFirstVPC
- Subnet: PublicSubnet-AZ2
- Create

Private Instance in AZ2:
- Name: `TestServer-Private-AZ2`
- VPC: MyFirstVPC
- Subnet: PrivateSubnet-AZ2
- Create

**Step 7: Test Cross-AZ Connectivity**

From public instance in AZ1:
```bash
# Ping instance in AZ2
ping -c 5 10.0.11.x (AZ2 public instance)
# Should work (same VPC)

ping -c 5 10.0.12.x (AZ2 private instance)
# Should work (same VPC)
```

From private instance in AZ1:
```bash
# Reach private instance in AZ2
ping -c 5 10.0.12.x
# Should work (same VPC)

# Download from internet via AZ1's NAT
curl http://checkip.amazonaws.com
# Should return NAT-1's IP
```

### Multi-AZ Network Flow

```
User in Europe
    ↓
Reaches public instance in AZ1 via public IP
    ↓
Instance in AZ1 needs to access private DB
    ↓
Both instances communicate directly (10.0.x.x addresses)
    ↓
Private instance needs updates
    ↓
Goes through NAT Gateway in AZ1 (same AZ)
    ↓
Gets updates
    ↓
Response comes back via NAT

If AZ1 fails:
User reaches public instance in AZ2 via same URL (DNS)
    ↓
Everything continues working! ✅
```

### Real-World Multi-AZ Setup

```
Company Website
VPC: 10.0.0.0/16

AZ-1a (Primary):
├─ PublicSubnet: 10.0.1.0/24
│  ├─ Web Server 1
│  ├─ IGW
│  └─ NAT Gateway 1
├─ PrivateSubnet: 10.0.2.0/24
│  ├─ App Server 1
│  ├─ Database (Primary)
│  └─ Route → NAT-1

AZ-1b (Standby):
├─ PublicSubnet: 10.0.11.0/24
│  ├─ Web Server 2
│  └─ NAT Gateway 2
├─ PrivateSubnet: 10.0.12.0/24
│  ├─ App Server 2
│  ├─ Database (Replica)
│  └─ Route → NAT-2

Load Balancer: Distributes to both AZs
Result: Always available, even if one AZ fails! ✅
```

### Validation Checklist
- [ ] Subnets created in AZ-1b
- [ ] NAT Gateway created in AZ-1b
- [ ] Route tables configured for AZ-1b
- [ ] Instances launched in AZ-1b
- [ ] Cross-AZ ping works
- [ ] Both NAT Gateways operational
- [ ] Private instances reach internet via their local NAT

### Key Takeaway
✅ Multi-AZ provides high availability  
✅ Replicate infrastructure across AZs  
✅ Route traffic locally when possible  
✅ Survive AZ failure  

---

### Lab 7: Security Groups vs NACLs (Stateful vs Stateless)

**Objective:** Understand difference and when to use each.

**Why This Matters:**
- Security Groups = Stateful (remembers connections)
- NACLs = Stateless (checks every packet)
- Different use cases
- Defense in depth

### Understanding Security Groups vs NACLs

```
Security Group (Stateful):
┌─────────────────────────────┐
│ "I remember conversations"  │
│                             │
│ Inbound: Allow SSH (22)     │
│ Outbound: Implicit allow    │
│           for established   │
│           connections       │
│                             │
│ Return traffic auto allowed │
└─────────────────────────────┘

NACL (Stateless):
┌─────────────────────────────┐
│ "Every packet checked"      │
│                             │
│ Inbound rule 100: Allow 22  │
│ Inbound rule 110: Allow 80  │
│ Outbound rule 100: Allow 22 │
│                             │
│ Return traffic needs rule   │
└─────────────────────────────┘
```

### Step-By-Step Instructions

**Step 1: Test Current Security Group Behavior**

From public instance, create a connection:
```bash
# Start SSH connection to private instance (Session Manager won't work here)
# Instead, use instance connect or SSM

# Test traffic allowed by SG
ping -c 5 10.0.2.x (private instance)
# Allowed (ICMP in SG)

# Test on specific port
nc -zv 10.0.2.x 22
# Connection refused (SSH SG doesn't allow)
```

**Step 2: Create Custom NACL**
- Go to VPC → Network ACLs
- Click "Create Network ACL"
- Name: `CustomNACL`
- VPC: MyFirstVPC
- Create

**Step 3: Add NACL Rules (Inbound)**

These rules processed in order (lower number = higher priority):

```
Rule #  Type        Protocol  Port Range  CIDR            Action
100     SSH (22)    TCP       22          10.0.1.0/24     Allow
110     HTTP (80)   TCP       80          0.0.0.0/0       Allow
120     HTTPS (443) TCP       443         0.0.0.0/0       Allow
130     ICMP        ICMP      All         0.0.0.0/0       Allow
140     Ephemeral   TCP       1024-65535  0.0.0.0/0       Allow
150     Ephemeral   UDP       1024-65535  0.0.0.0/0       Allow
*       All traffic All       All         0.0.0.0/0       Deny
```

**Why ephemeral ports?**
- When client initiates connection to port 22
- Server responds from random ephemeral port (1024-65535)
- Must allow inbound ephemeral for responses!

Add these rules in NACL:
- Click "Edit Inbound Rules"
- Add each rule
- Click "Save"

**Step 4: Add NACL Rules (Outbound)**

Similar rules needed for outbound:
```
Rule #  Type        Protocol  Port Range  CIDR            Action
100     SSH (22)    TCP       22          10.0.1.0/24     Allow
110     HTTP (80)   TCP       80          0.0.0.0/0       Allow
120     HTTPS (443) TCP       443         0.0.0.0/0       Allow
130     ICMP        ICMP      All         0.0.0.0/0       Allow
140     Ephemeral   TCP       1024-65535  0.0.0.0/0       Allow
150     Ephemeral   UDP       1024-65535  0.0.0.0/0       Allow
*       All traffic All       All         0.0.0.0/0       Deny
```

**Step 5: Test NACL Effect**

Forget NACL for now (we won't apply yet).
Instead, focus on understanding the rules.

**Step 6: Compare SG vs NACL Behavior**

Security Group (Current Setup):
```bash
# SSH from 10.0.1.x to 10.0.2.x
ssh -i key.pem ec2-user@10.0.2.x

# Flow:
1. Client: syn to port 22 (outbound from SG)
2. Server: syn-ack from ephemeral port (inbound to SG)
3. SG remembers: "This is return traffic"
4. Auto allows it! ✅
   (No need to specify ephemeral in outbound)
```

NACL (If Applied):
```
Same flow:
1. Client: syn to port 22 (check outbound NACL)
   → Rule 100 matches: Allow
2. Server: syn-ack from ephemeral (check inbound NACL)
   → Rule 140 matches: Allow ephemeral TCP
3. Connection works! ✅
   (Had to explicitly allow ephemeral!)
```

### Real-World Example

**Web Server with Different Tiers:**

```
Tier 1: Web Servers (Public Subnet)
├─ Allow: 80, 443 from 0.0.0.0/0
├─ Allow: SSH from admin IPs only
└─ NACL: Allow HTTP, HTTPS, SSH, ephemeral

Tier 2: App Servers (Private Subnet)
├─ Allow: 8080 from 10.0.1.0/24 (web tier)
├─ Allow: SSH from 10.0.1.0/24 (bastion hosts)
└─ NACL: Allow responses back, block others

Tier 3: Database (Private Subnet)
├─ Allow: 3306 from 10.0.2.0/24 (app tier)
├─ Deny: Everything else
└─ NACL: Allow MySQL, block world
```

### When to Use Each

**Use Security Groups:**
- Application-level access control
- Allow rules (simpler)
- Stateful (remember conversations)
- Most common

**Use NACLs:**
- Subnet-level traffic blocking
- Explicit deny rules (block bad traffic)
- Stateless (detailed control)
- Additional security layer

### Validation Checklist
- [ ] Understand security group stateful behavior
- [ ] Understand NACL stateless behavior
- [ ] Know difference between them
- [ ] Can explain ephemeral port requirement
- [ ] Know when to use each

### Key Takeaway
✅ Security Groups = Stateful, application level  
✅ NACLs = Stateless, subnet level  
✅ Both provide defense in depth  
✅ Combined security is strongest  

---

## POST-LAB 5-7 CLEANUP

**For Labs 5-7:**

```bash
# Stop EC2 instances
aws ec2 stop-instances --instance-ids i-xxx i-yyy i-zzz \
  --region us-east-1

# Delete NAT Gateways (takes 5 minutes)
aws ec2 delete-nat-gateway --nat-gateway-id natgw-xxx \
  --region us-east-1
aws ec2 delete-nat-gateway --nat-gateway-id natgw-yyy \
  --region us-east-1

# Release Elastic IPs
aws ec2 release-address --allocation-id eipalloc-xxx \
  --region us-east-1
aws ec2 release-address --allocation-id eipalloc-yyy \
  --region us-east-1

# Estimated cost so far: $1-3 (NAT Gateway usage)
```

---

## SECTION 3: ADVANCED NETWORKING (Labs 8-12)

### Lab 8: VPC Peering (Connect Multiple VPCs)

**Objective:** Connect two separate VPCs to communicate.

**Why This Matters:**
- Different departments need separate VPCs
- Can't use same VPC (security boundaries)
- Need VPC Peering to connect them
- Alternative: Transit Gateway (covered later)

### Understanding VPC Peering

```
VPC A: 10.0.0.0/16          VPC B: 172.31.0.0/16
├─ Subnet: 10.0.1.0/24      ├─ Subnet: 172.31.1.0/24
└─ Instance: 10.0.1.10      └─ Instance: 172.31.1.10

Without Peering:
10.0.1.10 cannot reach 172.31.1.10 ❌

With Peering:
10.0.1.10 ────── Peering Connection ────── 172.31.1.10 ✅
```

### Step-By-Step Instructions

**Step 1: Create Second VPC**
- Go to VPC Console → Your VPCs
- Click "Create VPC"
- Name: `VPC-B`
- IPv4 CIDR: `172.31.0.0/16`
- Create VPC

**Step 2: Create Subnets in VPC-B**
- Create public subnet: 172.31.1.0/24
- Create private subnet: 172.31.2.0/24
- Same as VPC-A setup

**Step 3: Attach IGW to VPC-B**
- Create new IGW
- Attach to VPC-B

**Step 4: Create Route Tables in VPC-B**
- Create public route table
- Add route: 0.0.0.0/0 → IGW
- Associate with public subnet

**Step 5: Create VPC Peering Connection**
- Go to VPC → Peering Connections
- Click "Create Peering Connection"
- Name: `VPC-A-to-VPC-B`
- Local VPC: MyFirstVPC (VPC-A)
- Peer VPC: VPC-B (in same account)
- Create Peering Connection

**Step 6: Accept Peering Connection**
- Connection shows status: "Pending Acceptance"
- Click the connection
- Click "Accept Peering Connection"
- Status changes to "Active" ✅

**Step 7: Update Route Tables for Peering**

In VPC-A Public Route Table:
```
Destination        Target
10.0.0.0/16        Local
0.0.0.0/0          IGW
172.31.0.0/16      Peering Connection
(to reach VPC-B)
```

In VPC-B Public Route Table:
```
Destination        Target
172.31.0.0/16      Local
0.0.0.0/0          IGW
10.0.0.0/16        Peering Connection
(to reach VPC-A)
```

Do same for private route tables!

**Step 8: Update Security Groups**

In VPC-A instances' SG:
- Add inbound rule: All traffic from 172.31.0.0/16

In VPC-B instances' SG:
- Add inbound rule: All traffic from 10.0.0.0/16

**Step 9: Launch Test Instance in VPC-B**
- EC2 → Launch Instance
- Name: `TestServer-VPC-B`
- VPC: VPC-B
- Subnet: Public subnet
- Create

**Step 10: Test VPC Peering**

From instance in VPC-A:
```bash
# Ping instance in VPC-B
ping -c 5 172.31.1.x
# Should work! ✅

# SSH to instance in VPC-B
ssh -i key.pem ec2-user@172.31.1.x
# Should work! ✅

# Check what network you're in
hostname -I
# Shows 10.0.1.x (still in VPC-A)
```

From instance in VPC-B:
```bash
# Ping instance in VPC-A
ping -c 5 10.0.1.x
# Should work!

# This proves peering is working! ✅
```

### VPC Peering Limitations

```
Works:
✅ Same region
✅ Different AWS accounts
✅ Different regions (modern peering)
✅ Transitive peering (A-B, B-C, but A-C doesn't work)
✅ CIDR blocks cannot overlap

Doesn't work:
❌ CIDR overlap (can't peer 10.0.0.0/16 with 10.1.0.0/16)
❌ Not transitive (must peer directly)
❌ Maximum peering connections per VPC

Solution for transitive: Use Transit Gateway (Lab 12)
```

### Real-World Scenario

**Multi-Department Company:**
```
Finance VPC (10.0.0.0/16)
├─ Finance App
├─ Finance Database
└─ Peering to Admin VPC

Admin VPC (172.31.0.0/16)
├─ Admin Tools
├─ Audit Logs
└─ Peering to Finance VPC

Marketing VPC (192.168.0.0/16)
├─ Marketing App
└─ NOT peered with others

Result:
Finance ↔ Admin: Can communicate
Marketing ↔ Others: Cannot communicate
Perfect security boundaries! ✅
```

### Validation Checklist
- [ ] Second VPC created (VPC-B)
- [ ] Peering connection created
- [ ] Peering connection accepted
- [ ] Route tables updated (both VPCs)
- [ ] Security groups updated
- [ ] Can ping across VPCs
- [ ] Connection shows "Active"

### Key Takeaway
✅ Peering connects two VPCs  
✅ Low latency, no internet
✅ Must update route tables both ways  
✅ Not transitive (direct connection only)  

---

### Lab 9: VPC Flow Logs (Monitor Network Traffic)

**Objective:** Log and analyze all network traffic in your VPC.

**Why This Matters:**
- Troubleshoot connectivity issues
- Security analysis
- Audit compliance
- Understand traffic patterns

### Understanding VPC Flow Logs

```
VPC Activity:
├─ Instance sends packet to 8.8.8.8
├─ Instance receives response
├─ Instance communicates with database
├─ Failed connection attempts
└─ All captured in Flow Logs!

Flow Log Record:
{
  "version": 2,
  "account-id": "123456789012",
  "interface-id": "eni-1234567",
  "srcaddr": "10.0.1.10",
  "dstaddr": "8.8.8.8",
  "srcport": 12345,
  "dstport": 53,
  "protocol": 17,
  "packets": 1,
  "bytes": 70,
  "start": 1423424347,
  "end": 1423424348,
  "action": "ACCEPT",
  "log-status": "OK"
}
```

### Step-By-Step Instructions

**Step 1: Create CloudWatch Log Group**
- Go to CloudWatch → Logs → Log Groups
- Click "Create Log Group"
- Name: `/aws/vpc/flowlogs`
- Create

**Step 2: Create IAM Role for VPC Flow Logs**
- Go to IAM Console
- Click "Roles"
- Click "Create Role"
- Service: EC2 (or VPC)
- Actually, use trust: VPC Flow Logs service
- Search for: `vpc-flow-logs`
- Select: `VPCFlowLogsDefaultRole` if exists
- Otherwise create manually with policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

**Step 3: Enable Flow Logs on VPC**
- Go to VPC → Your VPCs
- Select MyFirstVPC
- Click "Flow Logs" tab
- Click "Create Flow Log"
- Name: `MyVPC-FlowLog`
- Filter: All (capture everything)
- Maximum aggregation interval: 1 minute (most detailed)
- Destination: CloudWatch Logs
- Log group: `/aws/vpc/flowlogs`
- IAM role: Your role
- Click "Create"

**Step 4: Wait and Generate Traffic**

From public instance:
```bash
# Generate network activity
ping google.com
curl http://google.com
ssh to another instance
```

**Step 5: View Flow Logs**

In CloudWatch:
- Go to Logs → Log Groups
- Select `/aws/vpc/flowlogs`
- Click log stream (usually organized by ENI)
- See records:

```
version account-id interface-id srcaddr dstaddr ... action
2 123456789012 eni-abc123 10.0.1.10 8.8.8.8 ... ACCEPT
2 123456789012 eni-abc123 8.8.8.8 10.0.1.10 ... ACCEPT
2 123456789012 eni-abc123 10.0.1.10 10.0.2.x ... ACCEPT
```

**Step 6: Query Logs with CloudWatch Insights**

- Go to Logs → Log Groups
- Select your log group
- Click "Insights"
- Run queries:

```bash
# Count accepted vs rejected
fields action | stats count() by action

# Find traffic from specific IP
fields * | filter srcaddr="10.0.1.10"

# Find failed connections
fields * | filter action="REJECT"

# Find traffic to specific port
fields * | filter dstport=443

# Bytes transferred by source IP
fields srcaddr, bytes | stats sum(bytes) by srcaddr
```

### Real-World Troubleshooting

**Problem: Instance can't reach database**

```bash
# Query: Find traffic to DB port (3306)
fields * | filter dstport=3306 | stats count() by action

# Results show:
action=REJECT count=10

# Diagnosis: Traffic being rejected!
# Check:
1. Security group on DB instance
2. NACL rules
3. Route table
4. Database listener

# In this case:
Security Group: Allow 3306 from X.X.X.X ← Wrong IP!
Fix: Update to correct source IP
Verify with Flow Logs: Should show ACCEPT now
```

### Cost of Flow Logs

```
CloudWatch Logs: $0.50 per GB ingested
VPC Flow Logs: Included (no additional cost)

Typical usage: 50-500 MB/day for small VPC
= $0.75 - $7.50/month
```

### Validation Checklist
- [ ] Log group created
- [ ] IAM role created with correct permissions
- [ ] Flow Logs enabled on VPC
- [ ] Traffic generated to populate logs
- [ ] Can see flow log records
- [ ] Can query with Insights
- [ ] Understand action (ACCEPT/REJECT)

### Key Takeaway
✅ Flow Logs show all network traffic  
✅ Valuable for troubleshooting  
✅ Can query with CloudWatch Insights  
✅ Essential for security audits  

---

### Lab 10: Network Access Control Lists (NACLs) - Deep Dive

**Objective:** Implement granular subnet-level traffic control.

**Why This Matters:**
- Subnet-wide protection
- Blocking malicious IPs
- Explicit deny rules
- Additional layer beyond security groups

### Step-By-Step Instructions

**Step 1: View Default NACL**
- Go to VPC → Network ACLs
- See default NACL for your VPC
- Shows rules:
  - Inbound: Allow all
  - Outbound: Allow all

**Step 2: Create Restrictive NACL**
- Click "Create Network ACL"
- Name: `RestrictiveNACL`
- VPC: MyFirstVPC
- Create

**Step 3: Add Inbound Rules**

Default NACL behavior: Allow all traffic
RestrictiveNACL: Deny all by default

Add rules in order (lower number = higher priority):

```
Rule #  Type        Protocol  Port    CIDR            Action
100     SSH         TCP       22      10.0.0.0/8      Allow
110     HTTP        TCP       80      0.0.0.0/0       Allow
120     HTTPS       TCP       443     0.0.0.0/0       Allow
130     DNS         UDP       53      0.0.0.0/0       Allow
140     Ephemeral   TCP       1024-65535  0.0.0.0/0  Allow
150     Ephemeral   UDP       1024-65535  0.0.0.0/0  Allow
200     Malicious   TCP       9999    203.113.50.0/24 Deny
*       All Traffic All       All     0.0.0.0/0       Deny
```

**Step 4: Add Outbound Rules**

Same rules (in reverse):
```
Rule #  Type        Protocol  Port    CIDR            Action
100     HTTP        TCP       80      0.0.0.0/0       Allow
110     HTTPS       TCP       443     0.0.0.0/0       Allow
120     DNS         UDP       53      0.0.0.0/0       Allow
130     DNS         TCP       53      0.0.0.0/0       Allow
140     Ephemeral   TCP       1024-65535  0.0.0.0/0  Allow
150     Ephemeral   UDP       1024-65535  0.0.0.0/0  Allow
200     Malicious   TCP       9999    203.113.50.0/24 Deny
*       All Traffic All       All     0.0.0.0/0       Deny
```

**Step 5: Associate with Private Subnet**
- Select RestrictiveNACL
- Click "Subnet Associations"
- Click "Edit"
- Select: PrivateSubnet-AZ1
- Save

**Step 6: Test NACL Restrictions**

From public instance, try to reach private instance:
```bash
# SSH to private instance (ephemeral ports allowed)
ssh -i key.pem ec2-user@10.0.2.x
# Should work (port 22 allowed in rule 100)

# Try port 9999 (blocked by rule 200)
nc -zv 10.0.2.x 9999
# Connection refused (denied by NACL)

# Try HTTP to private instance (not in rules)
nc -zv 10.0.2.x 80
# Connection refused (default deny)
```

### NACL Rule Processing

```
Packet arrives: Source=203.113.51.1 Port=9999

NACL checks rules in order:
Rule 100 (SSH):     Port=22, doesn't match
Rule 110 (HTTP):    Port=80, doesn't match
...
Rule 200 (Malicious): 203.113.50.0/24... Close! Is 203.113.51.1 in range?
                      No, 51.1 ≠ 50.0-50.255
Rule * (Default):   Deny
Result: REJECT ✅ (denied)

If packet was from 203.113.50.50:
Rule 200 would match exactly: DENY first! ✅
```

### Common NACL Patterns

**Pattern 1: Deny Specific Subnet**
```
Rule 50: Source=192.168.1.0/24 DENY (bad actor)
Rule 100: HTTPS from 0.0.0.0/0 ALLOW
Rule 110: HTTP from 0.0.0.0/0 ALLOW
*: All DENY
```

**Pattern 2: Regional Restriction**
```
Rule 100: Allow from Europe only
Rule 110: Allow from US only
Rule 200: Deny from China
*: All DENY
```

**Pattern 3: Application-Specific**
```
For Web Tier:
├─ Allow 80, 443 from 0.0.0.0/0
├─ Allow SSH from 10.0.0.0/16
└─ Ephemeral

For DB Tier:
├─ Allow 3306 from 10.0.2.0/24
├─ Allow 3306 from 10.0.12.0/24
└─ Ephemeral
```

### Performance Consideration

```
Security Groups: Check only affected traffic (fast)
NACLs: Check every packet (slower but stateless)

Typical: Use both
├─ NACLs for subnet-level blocking
└─ Security Groups for instance-level fine-tuning
```

### Validation Checklist
- [ ] RestrictiveNACL created
- [ ] Rules added in correct order
- [ ] Associated with private subnet
- [ ] SSH to private instance works
- [ ] Port 9999 denied
- [ ] Port 80 (HTTP) denied (not in rules)

### Key Takeaway
✅ NACLs = Subnet-level firewalls  
✅ Stateless (every packet checked)  
✅ Rules processed in order  
✅ Default deny more secure  

---

## POST-LAB 8-10 CLEANUP

**For Labs 8-10:**

```bash
# Delete VPC Peering Connection
aws ec2 delete-vpc-peering-connection \
  --vpc-peering-connection-id pcx-xxx \
  --region us-east-1

# Disable Flow Logs
aws ec2 delete-flow-logs --flow-log-ids fl-xxx \
  --region us-east-1

# Delete Log Group
aws logs delete-log-group --log-group-name /aws/vpc/flowlogs \
  --region us-east-1

# Cost so far: Minimal ($1-3 for NAT, VPC Peering free)
```

---

### Lab 11: Transit Gateway (Central Hub for Multiple VPCs)

**Objective:** Connect multiple VPCs and on-premises networks through central hub.

**Why This Matters:**
- VPC Peering doesn't scale (must peer each pair)
- Transit Gateway = Central hub
- Can connect 5,000+ VPCs theoretically
- Also connects on-premises (VPN/Direct Connect)

### Understanding Transit Gateway

```
Without Transit Gateway (VPC Peering):
VPC-A ── peer ── VPC-B
VPC-A ── peer ── VPC-C
VPC-A ── peer ── VPC-D
VPC-B ── peer ── VPC-C
VPC-B ── peer ── VPC-D
VPC-C ── peer ── VPC-D
Total: 6 peering connections
(N×(N-1)/2 = complex!)

With Transit Gateway:
    ┌─────────────────────┐
    │  Transit Gateway    │
    └────┬────┬────┬──────┘
         │    │    │
        VPC  VPC  VPC
         A    B    C
Total: 3 connections (simple!)
```

### Step-By-Step Instructions

**Step 1: Create Transit Gateway**
- Go to VPC → Transit Gateways
- Click "Create Transit Gateway"
- Name: `MyTransitGateway`
- Default route table association: Enable
- DNS support: Enable
- VPN ECMP support: Enable
- Create Transit Gateway

Wait 1-2 minutes for creation...

**Step 2: Create Transit Gateway Attachments for VPC-A**
- Go to Transit Gateway Attachments
- Click "Create Transit Gateway Attachment"
- Type: VPC
- Name: `TGW-VPC-A`
- Transit Gateway: MyTransitGateway
- VPC: MyFirstVPC (VPC-A)
- Subnets: Select PublicSubnet-AZ1 and PublicSubnet-AZ2
- Create

Wait for status: "Available"

**Step 3: Create Attachment for VPC-B**
- Same process
- Name: `TGW-VPC-B`
- VPC: VPC-B
- Subnets: PublicSubnet-AZ1 and PrivateSubnet-AZ1
- Create

**Step 4: Update Route Tables to Use Transit Gateway**

In VPC-A Public Route Table:
```
Destination        Target
10.0.0.0/16        Local
0.0.0.0/0          IGW
172.31.0.0/16      Transit Gateway (not peering!)
```

In VPC-B Public Route Table:
```
Destination        Target
172.31.0.0/16      Local
0.0.0.0/0          IGW
10.0.0.0/16        Transit Gateway
```

Add/modify routes:
- Click "Edit Routes"
- Modify route to peer CIDR
- Change target from "Peering Connection" to "Transit Gateway"
- Save

**Step 5: Test Transit Gateway**

From instance in VPC-A:
```bash
# Should still reach VPC-B
ping -c 5 172.31.1.x
# Still works! (Now via TGW instead of peering)
```

From instance in VPC-B:
```bash
# Reach VPC-A
ping -c 5 10.0.1.x
# Works! (Via TGW)
```

### Transit Gateway Features

**Features:**
- Connect thousands of VPCs
- Simplified management
- Connects to on-premises (VPN, Direct Connect)
- Route tables within TGW
- Cross-region support
- Multi-account support

**Cost:**
- $0.05 per attachment per hour
- $0.02 per GB data processed
- For 10 VPCs: $0.50/hour = $360/month
- Worth it for scale!

### Real-World Multi-Account Architecture

```
AWS Organization
├─ Management Account
│  └─ Transit Gateway (central hub)
│
├─ Production Account
│  ├─ VPC-Prod (attached to TGW)
│  └─ TGW Attachment
│
├─ Staging Account
│  ├─ VPC-Staging (attached to TGW)
│  └─ TGW Attachment
│
├─ Development Account
│  ├─ VPC-Dev (attached to TGW)
│  └─ TGW Attachment
│
└─ On-Premises Data Center
   ├─ VPN Connection to TGW
   └─ Encrypted tunnel

Result: All connected through one TGW! ✅
```

### Validation Checklist
- [ ] Transit Gateway created
- [ ] Attachments created for both VPCs
- [ ] Route tables updated
- [ ] Can ping across VPCs via TGW
- [ ] Peering connection still exists (for comparison)

### Key Takeaway
✅ Transit Gateway = Central hub  
✅ Scales better than peering  
✅ Simplifies multi-VPC architecture  
✅ Enables on-premises connectivity  

---

## POST-LAB 11-12 CLEANUP

**After completing Transit Gateway and VPN labs:**

```bash
# Detach and delete Transit Gateway
aws ec2 delete-transit-gateway-route-table \
  --transit-gateway-route-table-id tgw-rtb-xxx \
  --region us-east-1

# Delete Transit Gateway (takes 5 minutes)
aws ec2 delete-transit-gateway \
  --transit-gateway-id tgw-xxx \
  --region us-east-1

# Delete VPN Connection
aws ec2 delete-vpn-connection \
  --vpn-connection-id vpn-xxx \
  --region us-east-1

# Delete Virtual Private Gateway
aws ec2 delete-vpn-gateway \
  --vpn-gateway-id vgw-xxx \
  --region us-east-1

# Delete Customer Gateway
aws ec2 delete-customer-gateway \
  --customer-gateway-id cgw-xxx \
  --region us-east-1

# Estimated total cost: $5-10 for NAT + TGW usage
```

---

## SECTION 4: MONITORING & OPTIMIZATION (Labs 13-16)

### Lab 13: CloudWatch Network Monitoring

**Objective:** Monitor network performance metrics.

**Why This Matters:**
- Detect issues before they impact users
- Understand traffic patterns
- Capacity planning
- Cost optimization

### Step-By-Step Instructions

**Step 1: Enable Enhanced Monitoring on EC2**
- Go to EC2 Instances
- Select public instance
- Right-click → Monitor and troubleshoot → Manage Detailed Monitoring
- Enable detailed monitoring

**Step 2: Create CloudWatch Dashboard**
- Go to CloudWatch → Dashboards
- Click "Create Dashboard"
- Name: `NetworkDashboard`
- Create

**Step 3: Add Network Metrics**

Add these widgets:

**Widget 1: Network In/Out**
- Metric: EC2 → Per-Instance Metrics
- NetworkIn, NetworkOut
- Instance: Your public instance
- Period: 5 minutes

**Widget 2: Network Packets**
- Metric: NetworkPacketsIn, NetworkPacketsOut
- Same instance

**Widget 3: VPC Flow Logs (from Lab 9)**
- Query:
```bash
fields @timestamp, srcaddr, dstaddr, bytes
| stats sum(bytes) as total_bytes by srcaddr
| sort total_bytes desc
| limit 10
```

**Widget 4: Subnet Performance**
- Custom metric or create:
```bash
fields dstaddr | stats count() as requests by dstaddr
```

**Step 4: Create Alarms**

Create alarm: High Network Activity
- Metric: NetworkIn
- Threshold: 100 MB/min
- Action: SNS notification

Create alarm: VPC Flow Log Rejected Traffic
- Query Flow Logs:
```bash
fields action | stats count() as rejects where action="REJECT"
```
- Threshold: >10 rejected packets/min

**Step 5: Test Monitoring**

Generate traffic:
```bash
# From public instance, generate load
yes | curl http://google.com &
yes | curl http://google.com &

# Watch dashboard
# See NetworkIn spike in real-time! 📈
```

### Network Performance Baseline

```
Normal Web Server:
├─ NetworkIn: 1-5 MB/min
├─ NetworkOut: 10-50 MB/min
├─ NetworkPacketsIn: 100-1000/min
└─ NetworkPacketsOut: 50-500/min

Under Load:
├─ NetworkIn: 50+ MB/min
├─ NetworkOut: 100+ MB/min
└─ Indicates traffic spike or attack
```

### Real-World Scenario

**DDoS Detection:**
```
Baseline: 5 MB/min in
After attack: 500 MB/min in
Alarm triggered: Alert sent!
Action: Scale up instances, review security

Flow Logs show:
├─ Source: Malicious IP
├─ Port: Random (not 80/443)
├─ Action: REJECT (blocked by NACL)
└─ Conclusion: Attack blocked, no impact ✅
```

### Validation Checklist
- [ ] Dashboard created
- [ ] Network metrics added
- [ ] Alarms created
- [ ] Traffic spike visible on dashboard
- [ ] Understand normal vs abnormal traffic

### Key Takeaway
✅ CloudWatch monitors network metrics  
✅ Alarms alert on anomalies  
✅ Flow Logs provide detailed analysis  
✅ Proactive troubleshooting possible  

---

### Lab 14: Network Performance Optimization

**Objective:** Optimize network throughput and latency.

**Why This Matters:**
- Faster applications
- Better user experience
- Cost savings
- Scalability

### Understanding Network Optimization

```
Factors Affecting Performance:

1. Instance Type
├─ t2.micro: 5 Gbps max
├─ t3.medium: 5 Gbps max
└─ c5.large: 10 Gbps max

2. Placement Groups
├─ Cluster: Ultra-low latency (<1ms)
├─ Partition: For distributed workloads
└─ Spread: For reliability

3. Enhanced Networking
├─ ENA (Elastic Network Adapter): 100 Gbps
└─ SR-IOV: Bypass hypervisor (faster)

4. Jumbo Frames
├─ MTU (Max Transmission Unit): 1500 bytes (default)
└─ Jumbo: 9000 bytes (less overhead)
```

### Step-By-Step Instructions

**Step 1: Launch Optimized Instance**
- EC2 → Launch Instance
- Name: `OptimizedServer`
- Instance type: t3.large (has ENA support)
- Network settings:
  - VPC: MyFirstVPC
  - Subnet: PublicSubnet-AZ1
  - ENA support: Enabled (usually default)

**Step 2: Verify ENA Support**

In instance:
```bash
# Check for ENA driver
ethtool -i eth0 | grep ena
# Should show: driver: ena

# View network stats
ethtool -S eth0
# Shows detailed stats
```

**Step 3: Create Placement Group (For Low Latency)**
- Go to EC2 → Placement Groups
- Click "Create Placement Group"
- Name: `HighPerformanceCluster`
- Strategy: Cluster (for ultra-low latency)
- Create

**Step 4: Launch Instances in Placement Group**
- Launch 2 instances
- Placement group: HighPerformanceCluster
- Same subnet (must be in same AZ)
- Create

**Step 5: Test Performance**

Between instances in placement group:
```bash
# Install iperf for bandwidth test
sudo yum install iperf -y

# On server instance
iperf -s

# On client instance
iperf -c 10.0.1.x -t 60

# Results show throughput
# Placement group: ~10 Gbps+
# Normal: ~5 Gbps
```

Test latency:
```bash
# Ping (shows latency)
ping -c 100 10.0.1.x | tail -1
# Placement group: <1ms latency
# Normal: 1-5ms latency
```

**Step 6: Enable Jumbo Frames**

```bash
# Current MTU
ip link show eth0
# Shows: mtu 1500

# Change MTU (if needed for your app)
sudo ip link set dev eth0 mtu 9000

# Verify
ip link show eth0
# Shows: mtu 9000

# Note: All instances in path must support jumbo frames!
```

### Performance Comparison

```
Without Optimization:
├─ Instance: t2.micro (5 Gbps)
├─ Network: Normal (5ms latency)
├─ Throughput: ~400 Mbps (due to overhead)
└─ Best for: Small workloads, testing

With Optimization:
├─ Instance: c5.large (10 Gbps)
├─ Network: Cluster Placement Group
├─ Jumbo Frames: Enabled
├─ Throughput: ~8 Gbps
└─ Best for: Big data, HPC, databases
```

### Cost vs Performance

```
t2.micro: $0.012/hr → 400 Mbps → $0.03 per Gbps/hr
c5.large: $0.085/hr → 8000 Mbps → $0.01 per Gbps/hr

For high throughput:
Better performance per dollar with larger instance! ✅
```

### Validation Checklist
- [ ] Optimized instance launched
- [ ] ENA verified
- [ ] Placement group created
- [ ] Instances in placement group
- [ ] Latency measured (<1ms)
- [ ] Throughput tested (>8 Gbps)

### Key Takeaway
✅ Instance type affects network performance  
✅ Placement groups for ultra-low latency  
✅ Enhanced networking (ENA) needed  
✅ Significant performance gains possible  

---

### Lab 15: Cost Optimization (Reduce Network Costs)

**Objective:** Optimize network costs without sacrificing performance.

**Why This Matters:**
- Data transfer is expensive
- Can save 50%+ with optimization
- Proper architecture saves money

### Understanding Network Costs

```
AWS Data Transfer Costs:

EC2 → Internet: $0.09 per GB OUT
EC2 → EC2 (different regions): $0.02 per GB
EC2 → EC2 (same AZ): FREE
EC2 → S3 (same region): FREE
CloudFront: $0.085 per GB OUT

Example for 100 GB/day to internet:
Without optimization: 100 × $0.09 × 30 = $270/month
With CloudFront: 100 × $0.085 × 30 = $255/month
Savings: $15/month (5%)

For 1 TB/day:
Savings: $150/month (16%)!
```

### Step-By-Step Instructions

**Step 1: Analyze Current Traffic**

Using Flow Logs from Lab 9:
```bash
# Query: Where is traffic going?
fields dstaddr | stats count() as count by dstaddr | sort count desc

Results:
dstaddr          count
8.8.8.8          5000  ← Internet (expensive!)
172.31.1.10      2000  ← Another VPC (cross-AZ, expensive)
10.0.1.10        8000  ← Same VPC (free!)
```

**Step 2: Implement CloudFront for Static Content**
- Go to CloudFront
- Create distribution
- Origin: Your web server (or S3)
- Behavior: /static/* → Cache
- TTL: 1 day
- Create

Benefits:
- Content served from edge locations (faster)
- Reduced origin load
- Reduced data transfer costs

**Step 3: Implement S3 Gateway Endpoint**

For applications that frequently access S3:

```bash
# Create Gateway Endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-xxx

# Now all S3 traffic = FREE! ✅
# Before: $0.09 per GB
# After: $0 per GB
```

**Step 4: Implement Interface Endpoint for DynamoDB**

```bash
# Same for DynamoDB
aws ec2 create-vpc-endpoint \
  --vpc-endpoint-type Interface \
  --vpc-id vpc-xxx \
  --service-name com.amazonaws.us-east-1.dynamodb \
  --subnet-ids subnet-xxx
```

**Step 5: Implement NAT Gateway Optimization**

Instead of large NAT Gateway:
```
Option 1: NAT Instance (smaller, cheaper)
├─ Cost: Instance cost only (~$0.02/hr)
├─ Pro: Cheaper
└─ Con: Manual management

Option 2: NAT Gateway (managed, reliable)
├─ Cost: $0.045/hr + $0.045/GB
├─ Pro: AWS managed
└─ Con: More expensive

Choose NAT Instance for dev, NAT Gateway for prod
```

**Step 6: Optimize Cross-Region Traffic**

```
If your app talks to another region:

Before:
├─ US-East (App)
└─ EU-West (Database)
└─ All traffic: $0.02/GB (expensive!)

After:
├─ US-East (App + Read Replica)
├─ EU-West (Database)
└─ Only writes cross-region: Reduced traffic ✅
```

### Cost Optimization Summary

```
Strategy                          Savings
────────────────────────────────────────────
CloudFront for static              5-15%
S3 Gateway Endpoint                30-50% (for S3 traffic)
DynamoDB Endpoint                  30-50% (for DynamoDB)
NAT Instance vs Gateway            40-50%
Same-AZ communication              100% (free vs paid)
Regional database replicas         20-30%

Total potential: 50-80% reduction! 💰
```

### Real-World Scenario

**Before Optimization:**
```
Monthly traffic: 500 GB out to internet
Monthly cost: 500 × $0.09 = $45/month

After implementing:
├─ CloudFront: Reduces to 400 GB (20% local cache)
├─ S3 Endpoint: Saves 50 GB (free S3)
├─ DynamoDB Endpoint: Saves 20 GB (free DynamoDB)
└─ Total: 330 GB out

New cost: 330 × $0.09 = $29.70/month
Savings: $15.30/month (34%)
```

### Validation Checklist
- [ ] Analyzed current traffic pattern
- [ ] Calculated potential savings
- [ ] CloudFront distribution created
- [ ] VPC Endpoints created
- [ ] Understand cost vs performance tradeoff

### Key Takeaway
✅ CloudFront reduces data transfer costs  
✅ VPC Endpoints = Free internal traffic  
✅ Same-AZ communication is free  
✅ 50%+ cost reductions possible  

---

### Lab 16: Network Troubleshooting (Comprehensive Diagnosis)

**Objective:** Diagnose and fix network connectivity issues.

**Why This Matters:**
- Applications go down due to network issues
- Fast troubleshooting = Less downtime
- Systematic approach saves time

### Common Network Issues

```
Issue: Instances can't communicate
Possible causes:
├─ Route table missing route
├─ Security group blocking
├─ NACL blocking
├─ Subnet routing misconfigured
├─ Gateway not attached
└─ DNS resolution failing
```

### Step-By-Step Troubleshooting

**Scenario: Instance A can't reach Instance B**

```
Instance A: 10.0.1.10
Instance B: 10.0.2.20
```

**Step 1: Verify Both Instances Running**
```bash
# Check instance status
aws ec2 describe-instances \
  --instance-ids i-xxx i-yyy \
  --query 'Reservations[0].Instances[*].[InstanceId,State.Name]'

# Both should show "running"
```

**Step 2: Verify Network Interface**
```bash
# Check if instance has network interface
aws ec2 describe-instances \
  --instance-ids i-xxx \
  --query 'Reservations[0].Instances[0].NetworkInterfaces'

# Verify:
├─ IP address assigned
├─ Subnet correct
└─ Security group attached
```

**Step 3: Test Ping**
```bash
# From Instance A, ping B
ssh ec2-user@3.xxx.xxx.xxx
ping -c 5 10.0.2.20

# If fails, continue troubleshooting
```

**Step 4: Check Route Tables**
```bash
# Instance A route table
aws ec2 describe-route-tables \
  --filters "Name=association.subnet-id,Values=subnet-xxx" \
  --query 'RouteTables[0].Routes'

# Should show:
# 1. Local route (10.0.0.0/16)
# 2. Route to 10.0.2.0/24 target

# If missing: Add route!
aws ec2 create-route \
  --route-table-id rtb-xxx \
  --destination-cidr-block 10.0.2.0/24 \
  --instance-id i-yyy
```

**Step 5: Check Security Groups**
```bash
# Instance B security group
aws ec2 describe-security-groups \
  --group-ids sg-xxx \
  --query 'SecurityGroups[0].IpPermissions'

# Should allow traffic from 10.0.1.0/24
# If not, add rule:
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol all \
  --cidr 10.0.1.0/24
```

**Step 6: Check NACLs**
```bash
# Get subnet's NACL
aws ec2 describe-network-acls \
  --filters "Name=association.subnet-id,Values=subnet-xxx" \
  --query 'NetworkAcls[0].Entries'

# Check if rule blocks traffic
# If yes, modify:
aws ec2 replace-network-acl-entry \
  --network-acl-id acl-xxx \
  --rule-number 100 \
  --protocol 6 \
  --port-range From=22,To=22 \
  --cidr-block 10.0.1.0/24 \
  --ingress
```

**Step 7: Test with nc (netcat)**
```bash
# More detailed test than ping
nc -zv -w 2 10.0.2.20 22
# Returns:
# Connection succeeded (if working)
# Connection refused (if SG blocking)
# Timeout (if route missing)
```

**Step 8: Check DNS Resolution**
```bash
# If using DNS instead of IP
nslookup instance-b.aws.internal
# Should resolve to 10.0.2.20
# If fails, check Route 53 or DNS settings
```

### Troubleshooting Checklist

```
Connectivity Issue Diagnosis:

□ Both instances running?
□ Network interfaces attached?
□ Route tables have correct routes?
□ Security groups allow traffic?
□ NACLs allow traffic?
□ Gateways attached (IGW for internet)?
□ NAT Gateway working (for private)?
□ DNS resolving correctly?
□ Firewall/OS-level rules?

Test progression:
1. ping (ICMP)
2. nc (TCP connection)
3. telnet (specific port)
4. traceroute (route path)
5. tcpdump (packet capture)
```

### Real-World Scenario

**Problem: Users can't reach web server**

```
Investigation:
1. Web server is running ✓
2. Has public IP ✓
3. Can SSH to it locally ✓
4. But browser times out from internet ✗

Diagnosis steps:
1. Check security group
   → Found rule: SSH from 0.0.0.0/0 ✓
   → But HTTP/HTTPS NOT allowed ✗
   → FIX: Add rules for 80, 443

2. After fix, still timeout ✗

3. Check route table
   → Found: No route to 0.0.0.0/0 ✗
   → FIX: Add route to IGW

4. After fix, works! ✅
```

### Validation Checklist
- [ ] Successfully diagnosed ping failure
- [ ] Found and fixed route table issue
- [ ] Found and fixed security group issue
- [ ] Verified connectivity with nc
- [ ] Understood systematic troubleshooting

### Key Takeaway
✅ Systematic troubleshooting saves time  
✅ Check: Routes → SG → NACL → DNS
✅ Use tools: ping, nc, traceroute, tcpdump  
✅ Document findings  

---

## COMPLETE CLEANUP PROCEDURE

### Full Cleanup (All Labs)

**Step 1: Stop All EC2 Instances**
```bash
# Stop instances (don't delete yet, in case you need them)
aws ec2 stop-instances \
  --instance-ids i-xxx i-yyy i-zzz \
  --region us-east-1

# Wait for all to stop
aws ec2 wait instance-stopped --instance-ids i-xxx
```

**Step 2: Delete Transit Gateway Resources**
```bash
# Delete TGW route tables
aws ec2 delete-transit-gateway-route-table \
  --transit-gateway-route-table-id tgw-rtb-xxx \
  --region us-east-1

# Delete attachments
aws ec2 delete-transit-gateway-attachment \
  --transit-gateway-attachment-id tgw-attach-xxx \
  --region us-east-1

# Delete TGW itself
aws ec2 delete-transit-gateway \
  --transit-gateway-id tgw-xxx \
  --region us-east-1
```

**Step 3: Delete VPN Resources**
```bash
# Delete VPN connection
aws ec2 delete-vpn-connection \
  --vpn-connection-id vpn-xxx \
  --region us-east-1

# Delete VGW
aws ec2 delete-vpn-gateway \
  --vpn-gateway-id vgw-xxx \
  --region us-east-1

# Delete customer gateway
aws ec2 delete-customer-gateway \
  --customer-gateway-id cgw-xxx \
  --region us-east-1
```

**Step 4: Delete NAT Gateways**
```bash
# Delete both NAT Gateways
aws ec2 delete-nat-gateway \
  --nat-gateway-id natgw-xxx \
  --region us-east-1

aws ec2 delete-nat-gateway \
  --nat-gateway-id natgw-yyy \
  --region us-east-1

# Wait for deletion
aws ec2 wait nat-gateway-deleted --nat-gateway-ids natgw-xxx

# Release Elastic IPs
aws ec2 release-address \
  --allocation-id eipalloc-xxx \
  --region us-east-1
```

**Step 5: Delete VPC Endpoints**
```bash
# List endpoints
aws ec2 describe-vpc-endpoints

# Delete each
aws ec2 delete-vpc-endpoints \
  --vpc-endpoint-ids vpce-xxx \
  --region us-east-1
```

**Step 6: Delete CloudFront Distribution**
```bash
# Disable distribution first
aws cloudfront update-distribution-config \
  --id XXXXX \
  --distribution-config file://disabled-config.json

# Then delete (after disabled)
aws cloudfront delete-distribution \
  --id XXXXX
```

**Step 7: Delete Placement Groups**
```bash
# Delete placement group
aws ec2 delete-placement-group \
  --group-name HighPerformanceCluster \
  --region us-east-1
```

**Step 8: Delete VPC Peering Connection**
```bash
# Delete peering
aws ec2 delete-vpc-peering-connection \
  --vpc-peering-connection-id pcx-xxx \
  --region us-east-1
```

**Step 9: Disable and Delete Flow Logs**
```bash
# Delete flow logs
aws ec2 delete-flow-logs \
  --flow-log-ids fl-xxx \
  --region us-east-1

# Delete CloudWatch log group
aws logs delete-log-group \
  --log-group-name /aws/vpc/flowlogs \
  --region us-east-1
```

**Step 10: Delete Network ACLs (Keep default)**
```bash
# Delete custom NACLs only
aws ec2 delete-network-acl \
  --network-acl-id acl-xxx \
  --region us-east-1

# Keep default NACL (AWS requirement)
```

**Step 11: Delete Subnets**
```bash
# Delete all custom subnets
aws ec2 delete-subnet \
  --subnet-id subnet-xxx \
  --region us-east-1

# Repeat for all subnets
```

**Step 12: Delete Internet Gateway**
```bash
# First detach
aws ec2 detach-internet-gateway \
  --internet-gateway-id igw-xxx \
  --vpc-id vpc-xxx \
  --region us-east-1

# Then delete
aws ec2 delete-internet-gateway \
  --internet-gateway-id igw-xxx \
  --region us-east-1
```

**Step 13: Delete Route Tables**
```bash
# Delete custom route tables (keep main)
aws ec2 delete-route-table \
  --route-table-id rtb-xxx \
  --region us-east-1
```

**Step 14: Delete Instances**
```bash
# Now safe to terminate
aws ec2 terminate-instances \
  --instance-ids i-xxx i-yyy i-zzz \
  --region us-east-1
```

**Step 15: Delete VPCs**
```bash
# Delete non-default VPCs
aws ec2 delete-vpc \
  --vpc-id vpc-xxx \
  --region us-east-1
```

**Step 16: Delete Security Groups**
```bash
# Delete custom security groups (keep default)
aws ec2 delete-security-group \
  --group-id sg-xxx \
  --region us-east-1
```

**Step 17: Delete CloudWatch Resources**
```bash
# Delete dashboards
aws cloudwatch delete-dashboards \
  --dashboard-names NetworkDashboard \
  --region us-east-1

# Delete alarms
aws cloudwatch delete-alarms \
  --alarm-names HighNetworkActivity \
  --region us-east-1
```

**Step 18: Verify Cleanup**
```bash
# Check all resources deleted
aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId'
aws ec2 describe-vpcs --query 'Vpcs[?CidrBlock!=`172.31.0.0/16`]'
aws ec2 describe-security-groups
aws ec2 describe-nat-gateways
aws ec2 describe-flow-logs
```

### Cost Summary

```
Lab 1-4 (VPC Basics): ~$0
Lab 5-7 (NAT + Multi-AZ): ~$5-10
Lab 8-10 (Peering + Monitoring): ~$1-3
Lab 11-12 (TGW + VPN): ~$10-15
Lab 13-16 (Optimization): ~$2-5

Total: $18-33 for all labs
(Minimal compared to production)
```

### Verification Checklist

- [ ] All EC2 instances terminated
- [ ] All VPCs deleted (except default)
- [ ] All NAT Gateways deleted
- [ ] All Elastic IPs released
- [ ] All Flow Logs disabled
- [ ] All CloudWatch dashboards deleted
- [ ] All alarms deleted
- [ ] All security groups deleted (except default)
- [ ] AWS Billing showing $0 charges (after stop)

---

## Final Summary

### What You Learned

| Lab | Topic | Key Skill |
|-----|-------|-----------|
| 1 | VPC Creation | Understand IP addressing |
| 2 | Subnets | Public vs Private |
| 3 | Internet Gateway | Internet connectivity |
| 4 | EC2 Testing | Verify connectivity |
| 5 | NAT Gateway | Private internet access |
| 6 | Multi-AZ | High availability |
| 7 | Security Groups vs NACL | Defense in depth |
| 8 | VPC Peering | Connect multiple VPCs |
| 9 | VPC Flow Logs | Monitor traffic |
| 10 | NACL Deep Dive | Stateless firewalls |
| 11 | Transit Gateway | Central hub architecture |
| 12 | Site-to-Site VPN | On-premises connectivity |
| 13 | CloudWatch Monitoring | Real-time alerts |
| 14 | Performance Optimization | Low latency networking |
| 15 | Cost Optimization | Reduce data transfer |
| 16 | Troubleshooting | Systematic diagnosis |

### AWS Networking Best Practices

✅ **Design:**
- Plan CIDR blocks before creating
- Multi-AZ for high availability
- Separate public/private clearly
- Document everything

✅ **Security:**
- Defense in depth (SG + NACL)
- Principle of least privilege
- VPC Flow Logs for audit
- Regular security reviews

✅ **Operations:**
- CloudWatch dashboards
- Automated alerts
- Systematic troubleshooting
- Cost monitoring

✅ **Performance:**
- Placement groups for low latency
- Enhanced networking (ENA)
- Regional architecture
- Endpoint optimization

### Real-World Architecture

```
Enterprise Multi-Region Setup:

US-East Region:
├─ VPC (10.0.0.0/16)
│  ├─ Public Subnet AZ-1a
│  ├─ Public Subnet AZ-1b
│  ├─ Private Subnet AZ-1a
│  └─ Private Subnet AZ-1b
├─ Transit Gateway (hub)
└─ VPN to on-premises

EU-West Region:
├─ VPC (10.1.0.0/16)
├─ Transit Gateway (hub)
└─ Peered with US-East

On-Premises:
├─ Data Center
└─ VPN tunnel to TGW

Result: Global, resilient, connected! 🌍
```

### Next Steps

1. **Implement:** Build actual network in production
2. **Monitor:** Set up CloudWatch dashboards
3. **Optimize:** Reduce costs by 50%+
4. **Scale:** Add regions and VPCs as needed
5. **Secure:** Regular audits and updates

---

## Appendix: Useful AWS CLI Commands

```bash
# List all VPCs
aws ec2 describe-vpcs

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# List subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-xxx"

# Create subnet
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24

# List route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-xxx"

# Add route
aws ec2 create-route --route-table-id rtb-xxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxx

# List security groups
aws ec2 describe-security-groups --filters "Name=vpc-id,Values=vpc-xxx"

# Add security group rule
aws ec2 authorize-security-group-ingress --group-id sg-xxx

**Objective:** Create secure tunnel to on-premises network (or simulate with another VPC).

**Why This Matters:**
- Connect office networks to AWS
- Hybrid cloud setup
- Encrypted communication
- Essential for enterprises

### Understanding Site-to-Site VPN

```
On-Premises Network        AWS VPC
(192.168.0.0/16)          (10.0.0.0/16)

On-Prem Router ─── VPN Tunnel ─── VPC Router
(Encrypted)

All traffic between networks encrypted!
```

### Step-By-Step Instructions

**Step 1: Create Customer Gateway (Represents On-Premises)**
- Go to VPC → Customer Gateways
- Click "Create Customer Gateway"
- Name: `OnPremises-Gateway`
- BGP ASN: 65000 (or IP address)
- Type: Dynamic (if using BGP) or Static

For testing, we'll simulate this as another VPC:
- Type: Static
- IP address: Use public IP of instance in VPC-B (3.xxx.xxx.xxx)

Actually, let's simplify: We'll just create the gateway pointing to a public IP.

**Step 2: Create Virtual Private Gateway**
- Go to VPC → Virtual Private Gateways
- Click "Create Virtual Private Gateway"
- Name: `MyVPC-VGW`
- ASN: 64512 (AWS default)
- Create

**Step 3: Attach VGW to VPC**
- Select your VGW
- Click "Attach to VPC"
- VPC: MyFirstVPC (VPC-A)
- Attach

**Step 4: Create VPN Connection**
- Go to VPC → VPN Connections
- Click "Create VPN Connection"
- Name: `OnPrem-to-AWS`
- Target Virtual Private Gateway: MyVPC-VGW
- Customer Gateway: OnPremises-Gateway
- Routing options: Static (for simplicity)
- Create VPN Connection

Wait for status: "Available"

**Step 5: Configure Customer Side (Simulated)**

In real life, you'd configure the on-premises router with:
- VPN endpoints (from AWS side)
- Pre-shared keys
- Encryption settings

For simulation, skip this step (no real on-premises router).

**Step 6: Enable VPC Route Propagation**

- Go to Route Tables
- Select route table for VPC-A
- Click "Edit Route Table Association"
- Enable "Route Propagation"
- Select VGW
- Save

Routes automatically added:
```
Destination              Target
10.0.0.0/16             Local
0.0.0.0/0               IGW
192.168.0.0/16          VGW (virtual private gateway)
(on-prem network)       (via VPN tunnel)
```

**Step 7: Real VPN Setup (If You Had On-Premises)**

1. Download VPN configuration from AWS:
   - VPN Connection → Download Configuration
   - Select your router model
   - Get config file

2. Load configuration on on-premises router:
   - Import XML/config file
   - Router establishes encrypted tunnel
   - Tunnel shows "IPSEC_ESTABLISH" phase

3. Once established:
   - On-premises network ↔ AWS VPC
   - All traffic encrypted
   - Transparent routing

### VPN vs Direct Connect

```
VPN (What we just created):
├─ Encrypted over internet
├─ Setup: Minutes
├─ Cost: Low ($0.05/hour)
├─ Bandwidth: Up to 1.25 Gbps
└─ Best for: Small offices, quick setup

Direct Connect:
├─ Dedicated network connection
├─ Setup: Weeks (physical cable)
├─ Cost: High ($0.30/hour + port fees)
├─ Bandwidth: 1Gbps to 100Gbps
└─ Best for: Large data transfer, consistent connection
```

### Real-World Hybrid Cloud

```
Company Setup:
On-Premises Data Center (192.168.0.0/16)
├─ Database servers (legacy)
├─ Email servers
└─ File servers

AWS (10.0.0.0/16)
├─ Web servers (public)
├─ App servers (private)
└─ Cache servers

Connection: Site-to-Site VPN
├─ App servers query legacy DB
├─ DB servers backup to AWS
├─ Users access both transparently
└─ All encrypted

Result: Seamless hybrid architecture! ✅
```

### Validation Checklist
- [ ] Customer Gateway created
- [ ] Virtual Private Gateway created and attached
- [ ] VPN Connection created
- [ ] Route tables updated with
