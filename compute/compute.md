# AWS Compute Labs: Complete Guide from Basic to Advanced

## Overview
This guide contains 18 progressive labs covering AWS Compute services from foundational to enterprise-level scenarios. Each lab includes objectives, prerequisites, step-by-step instructions, and real-world applications.

**Compute Services Covered:**
- EC2 (Elastic Compute Cloud) - Virtual Machines
- Elastic Beanstalk - Easy Web App Deployment
- Lambda - Serverless Computing
- ECS & EKS - Container Orchestration
- Auto Scaling - Automatic Scaling
- Load Balancing - Traffic Distribution

---

## SECTION 1: FOUNDATIONAL CONCEPTS (Labs 1-4)

### Lab 1: Understanding Compute Services (What They Are)

**Objective:** Understand different AWS compute options and when to use each.

**Why This Matters:**
AWS offers many compute services. Choosing the right one is like choosing the right transportation:
- **EC2** = Buying a car (full control, you maintain it)
- **Elastic Beanstalk** = Using a taxi (someone else drives, you just sit)
- **Lambda** = Calling an Uber (pay per ride, no car ownership)
- **ECS/EKS** = Shipping containers (for complex apps)

### Lab Steps:

**Step 1: The Compute Spectrum**

```
┌─────────────────────────────────────────────────────────────┐
│                    COMPUTE SPECTRUM                         │
├─────────────────────────────────────────────────────────────┤
│  More Control ──────────────────────→ Less Work            │
│                                                              │
│  ┌──────────┐  ┌───────────┐  ┌────────┐  ┌────────┐      │
│  │   EC2    │→ │ Elastic   │→ │  ECS/  │→ │ Lambda │      │
│  │          │  │ Beanstalk │  │  EKS   │  │        │      │
│  └──────────┘  └───────────┘  └────────┘  └────────┘      │
│                                                              │
│  Full VM      App Platform  Containers  Serverless         │
│  Management   (Java, Python) (Docker)    (Events)          │
└─────────────────────────────────────────────────────────────┘
```

**Step 2: Quick Comparison Table**

| Service | Control | Effort | Cost Model | Best For |
|---------|---------|--------|-----------|----------|
| **EC2** | Full | High | Per hour | Traditional apps, games, databases |
| **Beanstalk** | Medium | Low | Per hour | Web apps, REST APIs |
| **Lambda** | Low | Minimal | Per execution | Scheduled tasks, event triggers |
| **ECS** | High | Medium | Per hour | Containerized microservices |
| **EKS** | High | High | Per hour | Complex orchestration |

**Step 3: Decision Tree**

Ask yourself:
```
Do you need a full operating system?
├─ YES → Use EC2
└─ NO
    ├─ Is this a web application?
    │  ├─ YES → Use Elastic Beanstalk
    │  └─ NO
    │     └─ Does it run based on events?
    │        ├─ YES → Use Lambda
    │        └─ NO → Use ECS/EKS
```

**Real-World Examples:**
1. **E-commerce Website:** Elastic Beanstalk
2. **Mobile App Backend:** Lambda (events trigger functions)
3. **Game Server:** EC2 (full OS control)
4. **File Processing:** Lambda (triggered on upload)
5. **Microservices:** ECS/EKS

### Key Takeaway
✅ EC2 = Full control, more responsibility  
✅ Beanstalk = Less control, easier  
✅ Lambda = No servers, pay per use  
✅ ECS/EKS = Multiple containers  

---

## SECTION 2: EC2 - VIRTUAL MACHINES (Labs 2-6)

### Lab 2: Launching Your First EC2 Instance

**Objective:** Create and connect to a simple EC2 instance.

**Prerequisites:**
- AWS account with free tier eligibility
- SSH client available

**What is EC2?**
Think of EC2 as renting a computer from AWS:
- You choose the size (small/large)
- You choose OS (Linux/Windows)
- You pay for time used
- Full control of everything

### Step-By-Step Instructions

**Step 1: Navigate to EC2 Dashboard**
- Open AWS Console
- Search for "EC2"
- Click "EC2" service
- Click "Launch Instances"

**Step 2: Choose Operating System (AMI)**
- Look for: "Amazon Linux 2 AMI (Free tier eligible)"
- This is simple and cheap
- Click "Select"

**Step 3: Choose Instance Type**
- Look for: "t2.micro" (Free tier eligible)
- This is small but sufficient for testing
- Click "Next: Configure Instance Details"

**Step 4-6: Configure & Storage**
- Default settings are fine
- Click through next steps
- Add storage: Default 8GB is fine

**Step 7: Add Tags**
- Add tag: Key = "Name", Value = "MyFirstServer"
- Helps identify your instance

**Step 8: Configure Security Group (Firewall)**
- Type: SSH
- Source: My IP (only your computer)
- Click "Review and Launch"

**Step 9: Create Key Pair**
- Click "Create a new key pair"
- Name it: "my-first-key"
- Download (save safely!)
- Click "Launch Instances"

**Step 10: Connect**
- Wait for instance to show "running"
- Click instance → "Connect"
- Choose "Session Manager"
- Click "Connect"

Success! You're inside your virtual computer! 🎉

### Testing
Type these commands:
```bash
whoami         # See who you are
pwd            # Current directory
ls             # List files
uname -a       # System info
```

### Validation Checklist
- [ ] Instance created and running
- [ ] Key pair downloaded
- [ ] Connected successfully
- [ ] Can run commands
- [ ] Instance has public IP

---

### Lab 3: Managing EC2 Instances (Cost Optimization)

**Objective:** Learn to stop/start instances to save money.

**Why This Matters:**
EC2 charges while running. Stop when not needed!

- **Stop** = Off, data saved (very cheap)
- **Start** = Turn back on (same IP usually)
- **Terminate** = Delete forever (careful!)

### Step-By-Step Instructions

**Step 1: Stop Your Instance**
- Select instance
- Instance State → Stop
- Confirm "Yes, Stop"
- Status changes to "stopped"
- Compute charges STOP

**Step 2: Check Billing Impact**
- Hourly charge stops
- Only storage costs tiny amount

**Step 3: Start Again**
- Select instance
- Instance State → Start
- Instance comes back online
- Charges resume

**Step 4: Create Image (Backup)**
- Select instance
- Image → Create Image
- Name: "MyServerBackup"
- This saves everything
- Takes a few minutes

**Step 5: Understand Cost Savings**

```
Running 24/7: 0.0116/hr × 730 hrs = $8.47/month
Used 8 hrs/day: 0.0116 × 240 hrs = $2.78/month
Savings: $5.69/month (67% less!)
```

### Real-World Scenario

**Development Team:**
- Start servers: 8:00 AM
- Work during day
- Stop servers: 5:00 PM
- Save: $4,000+/year

### Key Takeaway
✅ Stop instances when not using  
✅ Don't terminate unless sure  
✅ Create images for backup  
✅ Plan shutdown strategy  

---

### Lab 4: Security Groups (Firewall)

**Objective:** Control who can connect to your instance.

**Why This Matters:**
Without firewall rules, hackers can attack. Security groups protect it.

### Understanding Security Groups

```
Your Instance
     ↓
┌──────────────────┐
│ Security Group   │
│ (Firewall Rules) │
│                  │
│ Allow: SSH (22)  │ ← Only this open
│ Allow: HTTP (80) │ ← Web traffic
│ Deny: Everything │ ← Everything else blocked
└──────────────────┘
     ↓
Outside Internet
```

### Step-By-Step Instructions

**Step 1: View Current Security Group**
- Go to EC2 Instances
- Click on your instance
- Find "Security Groups"
- Click the group name

**Step 2: Add HTTP Rule**
- Click "Edit inbound rules"
- Click "Add rule"
- Type: HTTP
- Port: 80
- Source: 0.0.0.0/0 (anyone)
- Save

**Step 3: Add HTTPS Rule**
- Click "Add rule"
- Type: HTTPS
- Port: 443
- Source: 0.0.0.0/0
- Save

**Step 4: Verify SSH**
- You should see SSH rule (port 22)
- Port 22 lets you remote in

### Common Rules

| Service | Port | Who | Use Case |
|---------|------|-----|----------|
| SSH | 22 | Your IP | Remote login |
| HTTP | 80 | Everyone | Website |
| HTTPS | 443 | Everyone | Secure website |
| RDP | 3389 | Your IP | Windows remote |
| MySQL | 3306 | Internal | Database |

### Security Best Practices

✅ **DO:**
- SSH from your IP only
- HTTP/HTTPS from everyone (for public websites)
- Most restrictive possible
- Document rules

❌ **DON'T:**
- SSH from 0.0.0.0/0 (anyone can try)
- Leave unused ports open
- Database ports on internet
- Overly permissive rules

### Key Takeaway
✅ Security groups are firewalls  
✅ Always restrict SSH to your IP  
✅ Think about what each port needs  
✅ Default deny is safest  

---

### Lab 5: Installing Software on EC2 (Web Server)

**Objective:** Install and run a web server on your instance.

**Why This Matters:**
EC2 is empty. Install software to do useful work.

### Step-By-Step Instructions

**Step 1: Connect to Instance**
- EC2 Dashboard → Your Instance
- Click "Connect"
- Choose "Session Manager"
- Click "Connect"

**Step 2: Update System**
```bash
sudo yum update -y
# Takes 1-2 minutes
```

**Step 3: Install Apache Web Server**
```bash
# Install Apache
sudo yum install httpd -y

# Start it
sudo systemctl start httpd

# Auto-start on reboot
sudo systemctl enable httpd
```

**Step 4: Create Website**
```bash
cd /var/www/html
sudo nano index.html
```

Type:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My AWS Server</title>
</head>
<body>
    <h1>Hello from EC2! 🎉</h1>
    <p>This website is running on AWS</p>
</body>
</html>
```

Press: Ctrl+X → Y → Enter (save)

**Step 5: Verify It Works**
```bash
sudo systemctl status httpd
# Should show: Active (running)
```

**Step 6: Access Website**
- Get public IP from instance details
- Open browser
- Type: http://YOUR-PUBLIC-IP
- You see your website! 🌐

### Validation Checklist
- [ ] Apache installed
- [ ] Service running
- [ ] Website created
- [ ] Can access from browser
- [ ] See custom HTML

---

### Lab 6: Elastic IP (Fixed Address)

**Objective:** Assign permanent IP to instance.

**Why This Matters:**
EC2 instances get new IP when stopped/started.
Elastic IP stays the same.

### Understanding Elastic IP

```
Without Elastic IP:
Stop → New IP → Links broken 😞

With Elastic IP:
Stop → Same IP → Links work! 😊
```

### Step-By-Step Instructions

**Step 1: Allocate Elastic IP**
- EC2 Dashboard → Elastic IPs
- Click "Allocate Elastic IP Address"
- Select region
- Click "Allocate"
- Copy the new IP

**Step 2: Associate IP**
- Select Elastic IP
- Click "Associate Elastic IP Address"
- Choose: Instance = your instance
- Click "Associate"

**Step 3: Test It**
- Access website with Elastic IP
- Stop instance
- Start instance
- Access website again
- IP is the same!

### Cost
- Free if associated with running instance
- $0.005/hour if unused (charges!)

**Lesson:** Release unused Elastic IPs!

### Key Takeaway
✅ Elastic IPs don't change  
✅ Use for production websites  
✅ Release unused ones  
✅ Allows fixed DNS pointing  

---

## SECTION 3: ELASTIC BEANSTALK (Labs 7-9)

### Lab 7: Deploy Web App with Beanstalk

**Objective:** Deploy application without managing servers directly.

**Why This Matters:**
Beanstalk handles:
- Creating EC2 instances
- Configuring load balancers
- Setting up auto-scaling
- Managing updates

You just upload code!

### Understanding Beanstalk

```
Manual EC2 Way:
Buy EC2 → Install OS → Install web server → Deploy → Manage

Beanstalk Way:
Write code → Upload → Done! ✅
```

### Step-By-Step Instructions

**Step 1: Create Sample App**

Create folder with index.html:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Beanstalk App</title>
    <style>
        body {
            font-family: Arial;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background: white;
            padding: 40px;
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 10px 25px rgba(0,0,0,0.2);
        }
        button {
            background: #667eea;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello from Elastic Beanstalk! 🚀</h1>
        <p>This app is automatically deployed and scaled</p>
        <button onclick="alert('You clicked!')">Click Me</button>
    </div>
</body>
</html>
```

**Step 2: Go to Elastic Beanstalk**
- Open AWS Console
- Search "Elastic Beanstalk"
- Click "Create Application"

**Step 3: Configure**
- Application name: "MyFirstBeanstalkApp"
- Platform: "Node.js"
- Application code: "Sample application"
- Click "Create Application"

Wait 2-3 minutes for deployment...

**Step 4: Access Your App**
- Once deployed, see URL
- Click URL to see your app live!
- Example: `myfirstbeanstalkapp-env.us-east-1.elasticbeanstalk.com`

**Step 5: Understand What Was Created**
Beanstalk automatically created:
- EC2 instance(s)
- Load balancer
- Auto Scaling group
- Security groups
- Monitoring and logs

All automatically!

### Real-World Use
- Deploy web apps
- Deploy APIs
- Deploy microservices
- Zero server management

### Cost
- Same as EC2 underneath
- Usually $8-15/month for small apps
- No Beanstalk markup fee

### Key Takeaway
✅ Automates server management  
✅ Same cost as EC2  
✅ Perfect for web apps  
✅ Built-in scaling  

---

### Lab 8: Auto Scaling for Growth

**Objective:** Make app handle more users automatically.

**Why This Matters:**
Your app works with 100 users.
What about 1,000 users?
→ Auto Scaling adds servers automatically!

### Understanding Auto Scaling

```
Without Scaling:
100 users → Works
1000 users → Crashes 💥

With Auto Scaling:
100 users → 1 server
500 users → 3 servers (added automatically)
1000 users → 5 servers (added automatically)
```

### Step-By-Step Instructions

**Step 1: View Configuration**
- Go to Beanstalk app
- Click "Configuration"
- Find "Capacity"

**Step 2: Enable Auto Scaling**
- Click "Edit" next to Capacity
- Find "Auto Scaling Group":
  - Min instances: 2
  - Max instances: 4
- Scaling trigger (CPU):
  - Scale up when > 70%
  - Scale down when < 30%
- Click "Apply"

**Step 3: Create Load Test**
```bash
# Install Apache Bench
brew install httpd  # Mac
# OR apt-get install apache2-utils  # Linux

# Run test
ab -n 10000 -c 100 http://YOUR-BEANSTALK-URL/

# -n 10000 = 10,000 requests
# -c 100 = 100 concurrent
```

**Step 4: Watch It Scale**
- Run load test
- Go to Beanstalk console
- Watch "Capacity" tab
- See new instances being added! 📈

### Real-World Scenario

**E-commerce During Holiday:**
```
Normal: 2 servers handle traffic
Holiday rush: 10x traffic
→ Auto Scaling detects high CPU
→ Automatically adds 8 more servers
→ Website stays fast
→ After sale: Extra servers removed, costs down
```

### Key Takeaway
✅ Handles traffic spikes  
✅ Prevents outages  
✅ Reduces costs when slow  
✅ Fully automatic  

---

### Lab 9: Custom Domain with Beanstalk

**Objective:** Point custom domain to Beanstalk app.

**Why This Matters:**
`myfirstbeanstalkapp-env.us-east-1.elasticbeanstalk.com` is ugly.
`www.mycompany.com` is professional.

### Prerequisites
- Custom domain

### Step-By-Step Instructions

**Step 1: Get Beanstalk URL**
- Go to Beanstalk app
- Copy the URL (example: `myapp-env.us-east-1.elasticbeanstalk.com`)

**Step 2: Update Domain DNS**
- Go to domain registrar (GoDaddy, Namecheap, etc.)
- Find "DNS Settings"
- Create CNAME record:
  - Name: www
  - Points to: Your Beanstalk URL
  - TTL: 3600
- Save

**Step 3: Wait for DNS**
- Takes 5-30 minutes usually
- Check: `nslookup www.yourdomain.com`

**Step 4: Test**
- Open browser
- Type: http://www.yourdomain.com
- Your app loads! ✅

### Alternative: AWS Route53
1. Buy domain in Route53
2. Create Alias record to Beanstalk
3. Instant propagation (no wait!)

### Cost
- Beanstalk: Free (pay for EC2)
- Route53: $0.50/month per zone
- Domain: $10-15/year

### Key Takeaway
✅ Custom domains are professional  
✅ Easy DNS setup  
✅ Route53 is simpler for AWS  
✅ No extra cost  

---

## SECTION 4: LAMBDA - SERVERLESS (Labs 10-12)

### Lab 10: Your First Lambda Function

**Objective:** Create and run serverless code.

**Why This Matters:**
Lambda is magical:
- No servers to manage
- Pay only for execution
- Auto-scales infinitely
- Perfect for events

### Understanding Lambda

```
Traditional App:
Your Code runs 24/7 → Costs always → You manage

Lambda:
Code runs when event happens → Cost only then → AWS manages
```

### Step-By-Step Instructions

**Step 1: Go to Lambda Console**
- Search "Lambda"
- Click "Create Function"

**Step 2: Create Function**
- Name: `MyFirstFunction`
- Runtime: Python 3.11
- Click "Create Function"

**Step 3: Write Code**
Replace default code:
```python
def lambda_handler(event, context):
    name = event.get('name', 'World')
    
    message = f'Hello, {name}! You triggered Lambda!'
    
    return {
        'statusCode': 200,
        'body': message
    }
```

**Step 4: Test Function**
- Click "Test"
- Create test event:
```json
{
  "name": "Alice"
}
```
- Click "Test"
- See output:
```
Response:
{
  "statusCode": 200,
  "body": "Hello, Alice! You triggered Lambda!"
}
```

Success! 🎉

**Step 5: Check Details**
- Execution duration: milliseconds
- Memory used: 128 MB
- See logs

### Real-World Uses
- Process uploaded files
- Send emails when triggered
- Resize images
- Generate reports
- Run scheduled tasks

### Cost
```
First 1 million requests: FREE
Next million: $0.20
Usually FREE tier covers you!
```

### Key Takeaway
✅ No servers to manage  
✅ Pay only for execution  
✅ Auto-scales infinitely  
✅ Often free  

---

### Lab 11: Lambda Triggered by S3 Upload

**Objective:** Run code automatically when file uploaded.

**Why This Matters:**
Event-driven architecture:
1. Upload file to S3
2. S3 triggers Lambda
3. Lambda processes file
4. Results saved

Fully automatic!

### Step-By-Step Instructions

**Step 1: Create S3 Bucket**
- Go to S3
- Click "Create Bucket"
- Name: `my-lambda-bucket-[random]`
- Click "Create"

**Step 2: Create Lambda Function**
- Go to Lambda
- Click "Create Function"
- Name: `ProcessS3Upload`
- Runtime: Python 3.11
- Click "Create"

**Step 3: Write Code**
```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get file info from S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    print(f"File uploaded: {key} to {bucket}")
    
    # Example: Read the file
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    
    print(f"File content: {content[:100]}")  # First 100 chars
    
    return {
        'statusCode': 200,
        'body': f'Processed {key}'
    }
```

**Step 4: Add S3 Trigger**
- Click "Add Trigger"
- Source: S3
- Bucket: Your bucket
- Event: "All object create events"
- Click "Add"

**Step 5: Test It**
- Go to S3
- Upload a file
- Check Lambda logs
- See: "File uploaded: [your-file]"
- Triggered automatically! 🎉

### Real-World Example

**Image Resize Pipeline:**
```
User uploads image → S3
    ↓
S3 triggers Lambda
    ↓
Lambda resizes image
    ↓
Saves to S3 again
    ↓
Website loads resized version
```

### Key Takeaway
✅ Event-driven automation  
✅ S3 integrates directly  
✅ Zero manual intervention  
✅ Scales automatically  

---

### Lab 12: Scheduled Lambda (Cron Jobs)

**Objective:** Run code on a schedule (like cron jobs).

**Why This Matters:**
Some tasks need to run regularly:
- Generate daily reports
- Cleanup old files
- Check system health
- Send reminder emails

### Step-By-Step Instructions

**Step 1: Create Lambda Function**
- Go to Lambda
- Click "Create Function"
- Name: `DailyReportGenerator`
- Runtime: Python 3.11
- Click "Create"

**Step 2: Write Code**
```python
from datetime import datetime

def lambda_handler(event, context):
    now = datetime.now()
    
    print(f"Daily report generated at {now}")
    
    # Your logic here
    # - Query database
    # - Generate report
    # - Send email
    # - etc.
    
    return {
        'statusCode': 200,
        'body': f'Report generated at {now}'
    }
```

**Step 3: Add EventBridge Rule (Scheduler)**
- In Lambda, click "Add Trigger"
- Choose "EventBridge (CloudWatch Events)"
- Create new rule
- Name: `DailyReportSchedule`
- Rule type: Schedule expression
- Enter: `cron(0 9 * * ? *)`  (Daily at 9 AM UTC)
- Click "Add"

**Step 4: Understand Cron Syntax**
```
cron(minute hour day-of-month month day-of-week year)

Examples:
cron(0 9 * * ? *)     = Every day at 9 AM UTC
cron(0 */6 * * ? *)   = Every 6 hours
cron(0 9 ? * MON *)   = Every Monday at 9 AM
cron(0 2 1 * ? *)     = First day of month at 2 AM
```

**Step 5: Test It**
- Change schedule to 2 minutes from now
- Wait and watch CloudWatch logs
- See function execute automatically

### Real-World Scenario

**Backup Service:**
```
Every night at 2 AM:
1. Lambda triggered automatically
2. Backs up database
3. Stores in S3
4. Sends confirmation email
5. No manual work!
```

### Cost
- Free tier: 1 million requests
- Way cheaper than always-running server

### Key Takeaway
✅ Run code on schedule  
✅ Replace cron jobs  
✅ Zero manual intervention  
✅ Cheap and reliable  

---

## SECTION 5: CONTAINERS & ORCHESTRATION (Labs 13-15)

### Lab 13: Docker Basics (Containers)

**Objective:** Understand containers before using ECS/EKS.

**Why This Matters:**
Containers package everything:
- Your code
- All dependencies
- Configuration
- OS libraries

"It works on my machine" → "It works everywhere"

### Understanding Containers

```
Without Containers:
Dev machine: Works! ✅
Prod server: Doesn't work ❌
(Different packages, versions, etc.)

With Containers:
Dev machine: Works! ✅
Prod server: Works! ✅
(Everything same)
```

### Step-By-Step Instructions

**Step 1: Install Docker**
- Mac: Download Docker Desktop
- Windows: Download Docker Desktop
- Linux: `sudo apt-get install docker.io`

**Step 2: Create Simple App**

Create folder `my-app/`:
```
my-app/
├─ app.py
└─ requirements.txt
```

app.py:
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Docker Container! 🐳'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

requirements.txt:
```
Flask==2.3.0
```

**Step 3: Create Dockerfile**

In same folder, create `Dockerfile`:
```dockerfile
# Start with Python image
FROM python:3.11

# Set work directory
WORKDIR /app

# Copy files
COPY . .

# Install dependencies
RUN pip install -r requirements.txt

# Run app
CMD ["python", "app.py"]
```

**Step 4: Build Container**
```bash
cd my-app
docker build -t my-first-app:1.0 .

# This creates a container image
```

**Step 5: Run Container**
```bash
docker run -p 5000:5000 my-first-app:1.0

# Open browser: http://localhost:5000
```

See your app running in container! 🎉

### What Just Happened
1. Docker created environment with everything
2. Installed Flask
3. Ran your app
4. Exposed port 5000
5. App is isolated and consistent

### Key Takeaway
✅ Containers ensure consistency  
✅ Package everything needed  
✅ Same everywhere (dev, prod, cloud)  
✅ Foundation for ECS/EKS  

---

### Lab 14: ECS - Running Containers

**Objective:** Run your container in AWS using ECS.

**Why This Matters:**
ECS (Elastic Container Service):
- Manages containers at scale
- Auto-scaling
- Load balancing
- No Kubernetes complexity

### Understanding ECS

```
You have 10 containers to run.
Without ECS: Manage each one (tedious)
With ECS: Just say "run 10 containers" → ECS does it ✅
```

### Step-By-Step Instructions

**Step 1: Push Container to Registry**
- Go to ECR (Elastic Container Registry)
- Click "Create Repository"
- Name: `my-first-app`
- Click "Create"
- Follow instructions to push your docker image

```bash
# Authenticate
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin ACCOUNT-ID.dkr.ecr.us-east-1.amazonaws.com

# Tag your image
docker tag my-first-app:1.0 ACCOUNT-ID.dkr.ecr.us-east-1.amazonaws.com/my-first-app:1.0

# Push to AWS
docker push ACCOUNT-ID.dkr.ecr.us-east-1.amazonaws.com/my-first-app:1.0
```

**Step 2: Create ECS Cluster**
- Go to ECS
- Click "Create Cluster"
- Name: `my-first-cluster`
- Infrastructure: Fargate (serverless containers)
- Click "Create"

**Step 3: Create Task Definition**
- Click "Create new task definition"
- Name: `my-app-task`
- Container name: `my-app`
- Image: Your ECR image URL
- Port: 5000
- Click "Create"

**Step 4: Create Service**
- In cluster, click "Create Service"
- Task Definition: Your task
- Number of tasks: 2 (run 2 containers)
- Load Balancer: Enabled
- Click "Create"

**Step 5: Access Your App**
- Get Load Balancer URL
- Open in browser
- See your containerized app running! 🌐

### Real-World Use
- Run multiple copies of your app
- Auto-scaling based on load
- Zero downtime updates
- Health checks and auto-recovery

### Cost
- Pay for EC2 instances or Fargate
- Much cheaper than Lambda for long-running

### Key Takeaway
✅ Containers at scale  
✅ Auto-scaling built-in  
✅ Simpler than Kubernetes  
✅ Production-ready  

---

### Lab 15: Load Balancing Traffic

**Objective:** Distribute traffic across multiple servers.

**Why This Matters:**
One server handling traffic → bottleneck → slow site
Multiple servers + load balancer → fast and reliable

### Understanding Load Balancers

```
Without Load Balancer:
All users → Single server → Slow/Crashes ❌

With Load Balancer:
User 1 → Server A ✅
User 2 → Server B ✅
User 3 → Server C ✅
Distributed, fast!
```

### Step-By-Step Instructions

**Step 1: Create Multiple EC2 Instances**
- Launch 2-3 EC2 instances (t2.micro)
- Install Apache on each
- Create different index.html on each (to see difference):

Server 1: `<h1>Response from Server 1</h1>`
Server 2: `<h1>Response from Server 2</h1>`
Server 3: `<h1>Response from Server 3</h1>`

**Step 2: Create Load Balancer**
- Go to EC2 → Load Balancers
- Click "Create Load Balancer"
- Type: Application Load Balancer (ALB)
- Name: `my-first-lb`
- Scheme: Internet-facing
- Click "Create"

**Step 3: Create Target Group**
- Go to Target Groups
- Click "Create Target Group"
- Type: Instances
- Name: `my-targets`
- Protocol: HTTP, Port: 80
- Click "Create"

**Step 4: Register Targets**
- In Target Group, click "Register Targets"
- Select your 3 EC2 instances
- Click "Register Targets"

**Step 5: Configure Load Balancer**
- In Load Balancer, select your LB
- Add listener: HTTP, port 80
- Forward to: Your target group
- Click "Save"

**Step 6: Test It**
- Get Load Balancer DNS name
- Refresh browser multiple times
- See different server responses
- Traffic being distributed! 🎯

### Health Checks

Load Balancer automatically:
- Checks if servers are healthy
- Removes unhealthy ones
- Adds them back when healthy

Set up health checks:
- Path: `/` (or `/health` if custom)
- Interval: 30 seconds
- Timeout: 5 seconds
- Healthy threshold: 2 consecutive successes
- Unhealthy threshold: 2 consecutive failures

### Real-World Scenario

**Busy Website:**
```
Peak traffic: 10,000 visitors/minute
1 server → Crash 💥
5 servers + Load Balancer:
├─ Server 1: 2,000 requests/min
├─ Server 2: 2,000 requests/min
├─ Server 3: 2,000 requests/min
├─ Server 4: 2,000 requests/min
└─ Server 5: 2,000 requests/min
All fast and responsive ✅
```

### Key Takeaway
✅ Distributes traffic evenly  
✅ Prevents overload  
✅ Auto health checks  
✅ High availability  

---

## SECTION 6: ADVANCED SCENARIOS (Labs 16-18)

### Lab 16: Auto Scaling Groups with Load Balancers

**Objective:** Combine auto-scaling with load balancing.

**Why This Matters:**
Load Balancer distributes traffic.
Auto Scaling adds servers when needed.
Together = Perfect scalability!

### Understanding the Combination

```
Normal traffic: 2 servers running
    ↓
Traffic spikes: 10x increase
    ↓
Auto Scaling detects high CPU
    ↓
Adds 8 more servers automatically
    ↓
Load Balancer distributes across all 10
    ↓
Traffic handled easily ✅
    ↓
Traffic drops back to normal
    ↓
Extra servers removed (cost saved!)
```

### Step-By-Step Instructions

**Step 1: Create Launch Template**
- Go to EC2 → Launch Templates
- Click "Create Launch Template"
- Name: `web-server-template`
- AMI: Amazon Linux 2
- Instance type: t2.micro
- User data (to install Apache):
```bash
#!/bin/bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
echo "<h1>Server $(hostname -f)</h1>" > /var/www/html/index.html
```
- Create template

**Step 2: Create Auto Scaling Group**
- Go to EC2 → Auto Scaling Groups
- Click "Create Auto Scaling Group"
- Name: `web-asg`
- Launch template: Your template
- VPC: Default VPC
- Subnets: Select multiple (for redundancy)
- Click "Next"

**Step 3: Configure Group Size**
- Desired capacity: 2 (normal load)
- Min size: 2 (never below this)
- Max size: 6 (never above this)
- Click "Next"

**Step 4: Add Load Balancer**
- Click "Attach to new load balancer"
- LB name: `web-lb`
- LB type: Application Load Balancer
- Click "Create Auto Scaling Group"

**Step 5: Create Scaling Policies**
- In your ASG, click "Edit"
- Add scaling policy:
  - Scale up when CPU > 70%: Add 1 instance
  - Scale down when CPU < 30%: Remove 1 instance

**Step 6: Test Scaling**
- Create load on instances
- Watch ASG add instances
- Traffic distributed by LB
- Stop load test
- Watch ASG remove instances

### Real-World Example

**Streaming Service During Event:**
```
8 PM: Typical evening traffic
└─ 5 servers running, cost $40/night

9 PM: Live event starts
└─ Requests spike 5x
└─ Auto Scaling adds 15 servers
└─ Cost now $120/night (but site stays fast)

11 PM: Event ends
└─ Traffic drops
└─ Extra servers removed
└─ Cost back to $40/night
```

### Key Takeaway
✅ Auto-scales with demand  
✅ Prevents outages  
✅ Optimizes costs  
✅ Zero manual work  

---

### Lab 17: Production-Ready Architecture

**Objective:** Build a complete, production-ready application.

**Why This Matters:**
This combines everything learned:
- EC2 for compute
- Auto Scaling for growth
- Load Balancing for distribution
- RDS for database
- CloudFront for caching
- CloudWatch for monitoring

### Architecture

```
┌─────────────────────────────────────────────────┐
│            Internet (Users)                      │
└───────────┬─────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────┐
│        CloudFront (CDN - Cache Layer)            │
│  (Serves static content from edge locations)    │
└───────────┬─────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────┐
│      Application Load Balancer                   │
│    (Distributes traffic)                        │
└─────────┬──────────────────────┬─────────────────┘
          │                      │
          ▼                      ▼
    ┌──────────────┐      ┌──────────────┐
    │  Web Server  │      │  Web Server  │
    │   (Auto      │      │   (Auto      │
    │  Scaling)    │      │  Scaling)    │
    └──────┬───────┘      └──────┬───────┘
           │                     │
           └──────────┬──────────┘
                      │
                      ▼
            ┌──────────────────┐
            │   RDS Database   │
            │  (Multi-AZ)      │
            └──────────────────┘
                      │
                      ▼
            ┌──────────────────┐
            │   S3 Storage     │
            │  (Static files)  │
            └──────────────────┘
```

### Step-By-Step Build

**Step 1: Create RDS Database**
- Go to RDS
- Click "Create Database"
- Engine: MySQL 8.0
- Instance: db.t2.micro
- DB name: `appdb`
- Username: `admin`
- Password: Create strong password
- Multi-AZ: Yes (for redundancy)
- Click "Create"

Wait 5-10 minutes for creation...

**Step 2: Create Auto Scaling Group with Web Servers**
- Use previous lab's ASG
- User data includes database connection:

```bash
#!/bin/bash
# Install and configure
sudo yum update -y
sudo yum install httpd php php-mysql -y

# Connect to database
cat > /var/www/html/index.php << 'EOF'
<?php
$db_host = 'your-rds-endpoint.us-east-1.rds.amazonaws.com';
$db_user = 'admin';
$db_pass = 'your-password';
$db_name = 'appdb';

$conn = new mysqli($db_host, $db_user, $db_pass, $db_name);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "<h1>Database Connected!</h1>";
echo "<p>Server: " . gethostname() . "</p>";
?>
EOF

sudo systemctl start httpd
sudo systemctl enable httpd
```

**Step 3: Create CloudFront Distribution**
- Go to CloudFront
- Click "Create Distribution"
- Origin: Your Load Balancer DNS
- Behavior:
  - Path pattern: `/static/*`
  - Cache policy: Caching optimized
  - TTL: 1 day
- Click "Create Distribution"

Wait for distribution to deploy (2-5 minutes)

**Step 4: Set Up CloudWatch Monitoring**
- Go to CloudWatch
- Create Dashboard: `MyAppDashboard`
- Add widgets:
  - Load Balancer request count
  - Target group health
  - ASG instance count
  - RDS CPU utilization
  - Network in/out

**Step 5: Create SNS Alerts**
- Go to SNS
- Create topic: `app-alerts`
- Subscribe with your email
- In CloudWatch, create alarms:
  - Alert if CPU > 80%
  - Alert if unhealthy targets
  - Alert if RDS disk space < 10GB

**Step 6: Test Everything**
- Load test your app
- Watch ASG scale
- Check CloudWatch dashboard
- Verify alerts work
- See CloudFront cache stats

### Real-World Metrics

Your production app now:
- ✅ Handles traffic spikes
- ✅ Multi-AZ redundant database
- ✅ Static content cached globally
- ✅ Automatic monitoring
- ✅ Email alerts for problems
- ✅ Cost ~$200-300/month (depending on traffic)

### Key Takeaway
✅ Production-ready system  
✅ Handles growth  
✅ Monitored and alerted  
✅ Redundant and reliable  

---

### Lab 18: Disaster Recovery & High Availability

**Objective:** Design system that survives failures.

**Why This Matters:**
What if:
- A data center fails?
- A database dies?
- A disk fills up?
- A server crashes?

Your system should survive all of this!

### Understanding HA & DR

```
Without HA:
Server fails → Website down → Customers angry ❌

With HA:
Server fails → Another server takes over → No downtime ✅
```

### Step-By-Step Instructions

**Step 1: Multi-AZ Architecture**
Deploy your system across multiple Availability Zones:

```
┌────────────────────────────────────────────┐
│         Region: us-east-1                  │
├─────────────────────┬──────────────────────┤
│  AZ-1a              │  AZ-1b               │
│ ┌───────────────┐   │ ┌───────────────┐   │
│ │  Web Server 1 │   │ │  Web Server 2 │   │
│ └───────┬───────┘   │ └───────┬───────┘   │
│         │           │         │           │
│         └─────┬─────┴────┬────┘           │
│               │          │                │
│               ▼          ▼                │
│        ┌──────────────────────┐           │
│        │  RDS (Primary)  AZ-1a│           │
│        │  RDS (Replica)  AZ-1b│           │
│        └──────────────────────┘           │
└────────────────────────────────────────────┘
```

**Step 2: RDS Multi-AZ Setup**
Already done in Lab 17 (Multi-AZ: Yes)
- Primary database in AZ-1a
- Automatic replica in AZ-1b
- Automatic failover if primary fails

**Step 3: ASG Across Multiple AZs**
In your ASG:
- Subnets: Select at least 2 AZs
- Desired capacity: 2
- Min: 2, Max: 6

This ensures:
- At least 1 server per AZ
- If AZ fails, other AZ handles load

**Step 4: Load Balancer Health Checks**
Already configured to:
- Check server health every 30 seconds
- Remove unhealthy servers
- Only send traffic to healthy servers

**Step 5: Database Backups**
In RDS:
- Backup retention: 30 days
- Automatic backups: Yes
- Copy to another region: Yes

This allows:
- Restore to any point in time
- Recovery if data corrupted
- Cross-region recovery

**Step 6: Create Snapshot Strategy**
- Daily automated snapshots
- Weekly manual snapshots
- Store snapshots in different region

**Step 7: Test Failover**
```
1. Simulate AZ failure:
   - Manual failover in RDS console
   - Watch traffic move to other AZ
   - No website downtime! ✅

2. Simulate server failure:
   - Terminate a web server
   - ASG automatically launches new one
   - Load balancer stops sending traffic immediately
   - No noticeable downtime! ✅

3. Simulate database failure:
   - Force RDS failover
   - Automatic replica becomes primary
   - Takes 1-2 minutes
   - Apps auto-reconnect! ✅
```

### Disaster Recovery Levels

```
RTO = Recovery Time Objective (how fast to recover)
RPO = Recovery Point Objective (how much data lost)

Level 1: Backup Only
├─ RTO: Hours or days
├─ RPO: Hours
└─ Cost: $50/month

Level 2: Multi-AZ (Your setup)
├─ RTO: Minutes
├─ RPO: Seconds
└─ Cost: $150-300/month

Level 3: Multi-Region
├─ RTO: Seconds
├─ RPO: Seconds
└─ Cost: $500+/month
```

Your system (Lab 17+18): **Level 2** - Production-grade!

### Real-World Scenario

**AWS Region Goes Down:**
```
6 PM: Major AWS region experiences power outage
Your system:
├─ Continues serving from other AZs
├─ No customer impact
├─ Competitors' sites might be down
└─ You gain market share! 📈
```

### Key Takeaway
✅ Multi-AZ redundancy  
✅ Automatic failover  
✅ Data protected  
✅ True high availability  

---

## SECTION 7: COMPARISON & DECISION MATRIX

### When to Use Each Service

| Scenario | Best Service | Why |
|----------|-------------|-----|
| Website running 24/7 | EC2 + ALB | Full control, cost-effective |
| Easy web app | Elastic Beanstalk | No infrastructure work |
| Event-driven task | Lambda | Run only when needed |
| Containerized microservices | ECS | Simpler than Kubernetes |
| Complex orchestration | EKS | Kubernetes power |
| Scheduled jobs | Lambda | Cheap and simple |
| GPU workload | EC2 with GPU | Needed for training |
| Database processing | RDS + Lambda | Serverless logic |

### Cost Comparison

```
Running a simple web app for 1 month:

EC2 (manual):
├─ 1 × t2.micro: $8.47
├─ 1 × ALB: $16
└─ Total: $24.47

Beanstalk (same app):
├─ 1 × t2.micro: $8.47
├─ Infrastructure included: Free
└─ Total: $8.47

Lambda (same app, lower traffic):
├─ 1M requests: Free
├─ Under 1GB memory: Free
└─ Total: $0
```

### Performance Comparison

| Metric | EC2 | Beanstalk | Lambda | ECS |
|--------|-----|-----------|--------|-----|
| **Startup time** | Minutes | Seconds | Milliseconds | Seconds |
| **Customization** | 100% | 80% | 30% | 90% |
| **Scalability** | Good | Excellent | Unlimited | Excellent |
| **Monitoring** | Manual | Built-in | CloudWatch | Built-in |
| **Complexity** | High | Medium | Low | High |

---

## Summary: What You Learned

### Labs 1-6: EC2 Fundamentals
- Create virtual computers
- Install software
- Connect remotely
- Manage costs
- Implement security
- Assign fixed IPs

### Labs 7-9: Elastic Beanstalk
- Deploy apps without servers
- Auto-scaling built-in
- Custom domains
- Zero infrastructure work

### Labs 10-12: Lambda
- Serverless functions
- Event-driven triggers
- Scheduled execution
- Pay only for use

### Labs 13-15: Containers & Load Balancing
- Package apps in containers
- Run at scale with ECS
- Distribute traffic
- High availability

### Labs 16-18: Production Systems
- Auto-scaling at scale
- Complete architectures
- Multi-AZ redundancy
- Disaster recovery

---

## Best Practices Summary

### Security
- ✅ Restrict security groups to minimum needed
- ✅ Never expose databases to internet
- ✅ Use IAM roles for service authentication
- ✅ Enable encryption at rest and transit
- ✅ Regular security group audits

### Cost Optimization
- ✅ Stop instances when not needed
- ✅ Use reserved instances for baseline traffic
- ✅ Auto-scale to match demand
- ✅ Use Lambda for sporadic tasks
- ✅ Monitor and set budget alerts

### High Availability
- ✅ Deploy across multiple AZs
- ✅ Use load balancers for distribution
- ✅ Multi-AZ databases
- ✅ Regular backups in different regions
- ✅ Test failover scenarios

### Performance
- ✅ Use CloudFront for static content
- ✅ Database read replicas for read-heavy
- ✅ Connection pooling for databases
- ✅ Monitor application metrics
- ✅ Use caching where appropriate

### Operations
- ✅ CloudWatch dashboards for visibility
- ✅ SNS alerts for critical metrics
- ✅ Log aggregation to CloudWatch
- ✅ Document runbooks for common issues
- ✅ Automated backups and testing

---

## Recommended Learning Path

**Week 1:** Labs 1-3 (Understand options, launch first instance)
**Week 2:** Labs 4-6 (Security, software, networking)
**Week 3:** Labs 7-9 (Beanstalk and easier deployments)
**Week 4:** Labs 10-12 (Lambda and serverless)
**Week 5:** Labs 13-15 (Containers and scaling)
**Week 6:** Labs 16-18 (Production-ready systems)

---

## Hands-On Projects

### Project 1: Personal Blog
- Use EC2 + WordPress
- CloudFront for speed
- Route53 for domain
- RDS for database
- Estimated cost: $20-30/month

### Project 2: Image Resizer
- S3 bucket for uploads
- Lambda to resize
- Store resized in S3
- CloudFront to serve
- Estimated cost: $5-10/month (mostly free tier)

### Project 3: Microservices Application
- 3 ECS services
- RDS database
- ALB routing
- Auto Scaling
- Estimated cost: $150-200/month

### Project 4: Data Pipeline
- Scheduled Lambda
- Process data from S3
- Store in RDS
- Generate reports
- Estimated cost: $20-30/month

---

## Getting Help

- **AWS Documentation:** docs.aws.amazon.com
- **AWS Forums:** forums.aws.amazon.com
- **Stack Overflow:** Tag `amazon-ec2`, `aws-lambda`, etc.
- **AWS Support:** Available in AWS Console (free tier limited)
- **YouTube:** Many free AWS tutorials

---

**Congratulations! You've completed 18 labs covering AWS Compute from basic to advanced!** 🚀

You now understand:
✅ How to choose right service  
✅ How to build and deploy apps  
✅ How to scale for growth  
✅ How to ensure reliability  
✅ How to optimize costs  

Next steps: Start building real projects! Practice is key to mastering AWS. 💪
