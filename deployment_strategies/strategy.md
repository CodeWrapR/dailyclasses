# EC2 Deployment Strategies Tutorial (AWS Console)

## Overview

Deployment strategies determine how you roll out new versions of applications with minimal downtime and risk. 

---

## 1. Blue-Green Deployment

### What it is
Run two identical production environments (Blue and Green). Deploy new code to Green, test it, then switch traffic instantly. Rollback is immediate if issues occur.

### Step-by-Step Setup

#### Step 1: Create Blue Environment (Current Production)
1. Go to **EC2 Dashboard** → **Instances** → **Launch Instances**
2. Create 2 instances:
   - **Name:** `blue-server-1`, `blue-server-2`
   - **AMI:** Amazon Linux 2
   - **Instance Type:** t2.micro
   - **Security Group:** Create "web-sg" (allow HTTP 80, HTTPS 443, SSH 22)
   - **User Data Script:**
   ```bash
   #!/bin/bash
   yum update -y
   yum install httpd -y
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Blue Environment - v1.0</h1>" > /var/www/html/index.html
   ```

#### Step 2: Create Green Environment (New Version)
1. Repeat Step 1 but name instances `green-server-1`, `green-server-2`
2. Use same security group
3. **User Data Script:**
   ```bash
   #!/bin/bash
   yum update -y
   yum install httpd -y
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Green Environment - v2.0</h1>" > /var/www/html/index.html
   ```
4. **Important:** Stop these instances after creation (keep them ready but inactive)

#### Step 3: Create Application Load Balancer
1. Go to **EC2** → **Load Balancers** → **Create Load Balancer**
2. Choose **Application Load Balancer**
3. **Configuration:**
   - **Name:** `app-lb`
   - **Scheme:** Internet-facing
   - **IP address type:** IPv4
   - **VPC:** Select your VPC
   - **Subnets:** Select at least 2 availability zones
4. **Security Group:** Select `web-sg`
5. Click **Next: Configure routing**

#### Step 4: Create Target Groups
1. **Blue Target Group:**
   - Go to **Target Groups** → **Create target group**
   - **Name:** `blue-tg`
   - **Protocol:** HTTP
   - **Port:** 80
   - **VPC:** Select your VPC
   - Click **Create**
   - Select Blue target group → **Targets tab** → **Edit**
   - Add: `blue-server-1` and `blue-server-2`
   - Click **Save**

2. **Green Target Group:**
   - Repeat above steps but name it `green-tg`
   - Add: `green-server-1` and `green-server-2`

#### Step 5: Configure Load Balancer Routing
1. Go to **Load Balancers** → Select `app-lb`
2. **Listeners tab** → Click **Edit**
3. Under HTTP:80 default action:
   - Click **Edit** → Select `blue-tg` as target group
   - Save
4. Now traffic goes to Blue environment

#### Step 6: Test Setup
```bash
# Get Load Balancer DNS name from AWS Console
# Test Blue environment is responding
curl http://<app-lb-dns-name>

# Output should show:
# <h1>Blue Environment - v1.0</h1>
```

### Deployment Process

#### Step 1: Start Green Environment
1. Go to **EC2** → **Instances**
2. Select `green-server-1` and `green-server-2`
3. Right-click → **Instance State** → **Start**
4. Wait for instances to be in "running" state (2-3 minutes)
5. Check **Status Checks** show "2/2 checks passed"

#### Step 2: Verify Green Environment
```bash
# Get one of the Green instance's public IP
# SSH into it and verify application is working
ssh -i your-key.pem ec2-user@<green-instance-public-ip>

# Check web server is running
curl http://localhost

# Expected output:
# <h1>Green Environment - v2.0</h1>
```

#### Step 3: Run Smoke Tests
```bash
# Test key application endpoints
curl http://<green-instance-public-ip>/api/health
curl http://<green-instance-public-ip>/api/status
curl http://<green-instance-public-ip>/login

# Check logs for errors
tail -f /var/log/httpd/error_log
tail -f /var/log/httpd/access_log
```

#### Step 4: Switch Traffic to Green
1. Go to **EC2** → **Load Balancers** → Select `app-lb`
2. **Listeners tab** → Click **Edit**
3. Change default action target group from `blue-tg` to `green-tg`
4. Click **Save**
5. **Traffic instantly switches to Green environment**

#### Step 5: Monitor for Issues
```bash
# Watch traffic hitting Green environment
curl http://<app-lb-dns-name>

# Should now show:
# <h1>Green Environment - v2.0</h1>

# Monitor error logs in CloudWatch:
# Go to CloudWatch → Logs → Select ALB logs
# Look for 5XX errors or high 4XX rates
```

#### Step 6: Rollback (if issues occur)
1. Go to **Load Balancers** → Select `app-lb`
2. **Listeners tab** → **Edit**
3. Change target group back to `blue-tg`
4. **Traffic instantly switches back to Blue**
5. This takes less than 1 second!

#### Step 7: Cleanup
- Once Green is stable for 24+ hours:
- Stop Blue instances to save costs
- Keep them running if you want quick rollback capability

### Pros and Cons
**Pros:**
- Zero downtime deployment
- Instant rollback (< 1 second)
- Easy to test before switching
- No traffic splitting complexity

**Cons:**
- Double infrastructure cost
- Database schema changes risky
- Requires two full environments

---

## 2. Canary Deployment

### What it is
Send small percentage of traffic (5-10%) to new version. Monitor metrics. If healthy, gradually increase traffic. If issues found, rollback immediately.

### Step-by-Step Setup

#### Step 1: Create Stable Environment (Current v1.0)
1. **Launch 9 instances:**
   - **Names:** `stable-server-1` through `stable-server-9`
   - **AMI:** Amazon Linux 2
   - **Type:** t2.micro
   - **User Data:**
   ```bash
   #!/bin/bash
   yum update -y
   yum install httpd -y
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Stable - v1.0</h1>" > /var/www/html/index.html
   echo "v1.0" > /var/www/html/version.txt
   ```

#### Step 2: Create Canary Instance (New v2.0)
1. **Launch 1 instance:**
   - **Name:** `canary-server-1`
   - **AMI:** Amazon Linux 2
   - **Type:** t2.micro
   - **User Data:**
   ```bash
   #!/bin/bash
   yum update -y
   yum install httpd -y
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Canary - v2.0</h1>" > /var/www/html/index.html
   echo "v2.0" > /var/www/html/version.txt
   ```

#### Step 3: Create Application Load Balancer
1. **Create Load Balancer** → **Application Load Balancer**
2. **Name:** `canary-lb`
3. Configure subnets and security group (HTTP 80)

#### Step 4: Create Target Groups
1. **Stable Target Group:**
   - **Name:** `stable-tg`
   - Add all 9 `stable-server` instances

2. **Canary Target Group:**
   - **Name:** `canary-tg`
   - Add `canary-server-1`

#### Step 5: Configure Weighted Routing
1. Go to **Listeners** → **HTTP:80**
2. Click **Edit**
3. Under **Default action:**
   - Click **Edit** → **Add action** → **Forward**
   - Configure weighted target groups:
     - `stable-tg` - Weight: 90
     - `canary-tg` - Weight: 10
   - Click **Save**

#### Step 6: Verify Setup
```bash
# Test canary routing (run multiple times, ~10% should hit canary)
for i in {1..100}; do
  curl http://<canary-lb-dns> | grep -o "v[0-9]\.[0-9]"
done

# Expected: ~90 lines show "v1.0", ~10 lines show "v2.0"
```

### Canary Deployment Process

#### Phase 1: Monitor Canary (5-10 minutes)
1. Check CloudWatch metrics:
   - Go to **CloudWatch** → **Dashboards**
   - Create dashboard with:
     - **Target Group** metrics (stable-tg and canary-tg)
     - **ALB** metrics (HTTP 5XX, HTTP 4XX, Target Response Time)
2. Watch for:
   - Error rates (should stay < 0.1%)
   - Response times (should be similar to stable)
   - CPU usage (should be normal)

```bash
# Manual health check script
#!/bin/bash
for i in {1..100}; do
  RESPONSE=$(curl -s -w "%{http_code}" http://<canary-lb-dns>/)
  STATUS_CODE=$(echo $RESPONSE | tail -c 4)
  
  if [ $STATUS_CODE -ne 200 ]; then
    echo "ERROR: Got status $STATUS_CODE"
    # Alert and rollback if needed
  fi
done
```

#### Phase 2: Increase to 25% Traffic
1. Go to **Load Balancers** → **Listeners**
2. **Edit** → Change weights:
   - `stable-tg` - Weight: 75
   - `canary-tg` - Weight: 25
3. **Save** → Monitor for 5 minutes

#### Phase 3: Increase to 50% Traffic
1. Change weights:
   - `stable-tg` - Weight: 50
   - `canary-tg` - Weight: 50
2. **Save** → Monitor for 10 minutes

#### Phase 4: Increase to 100% Traffic
1. Change weights:
   - `stable-tg` - Weight: 0 (remove)
   - `canary-tg` - Weight: 100
2. **Save** → Now all traffic on new version

#### Phase 5: Rollback (if issues detected)
1. Immediately change weights back:
   - `stable-tg` - Weight: 100
   - `canary-tg` - Weight: 0
2. **Save** → Traffic returns to stable version
3. Investigate issues in canary logs

#### Phase 6: Scale New Version
1. Once confident, launch more `canary-server` instances
2. Add them to `canary-tg`
3. Remove old `stable-server` instances

### Monitoring Script Example
```bash
#!/bin/bash
# Monitor canary deployment health

ERROR_THRESHOLD=1.0  # Alert if error rate > 1%
INTERVAL=30  # Check every 30 seconds

while true; do
  echo "[$(date)] Checking canary health..."
  
  # Check canary error rate from CloudWatch
  ERROR_RATE=$(aws cloudwatch get-metric-statistics \
    --namespace AWS/ApplicationELB \
    --metric-name HTTPCode_Target_5XX_Count \
    --statistic Sum \
    --dimensions Name=TargetGroup,Value=targetgroup/canary-tg/* \
    --start-time $(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300 \
    --output text | awk '{print $NF}')
  
  if (( $(echo "$ERROR_RATE > $ERROR_THRESHOLD" | bc -l) )); then
    echo "CRITICAL: Error rate $ERROR_RATE% exceeds threshold!"
    echo "Rolling back canary deployment..."
    # Trigger rollback here
    exit 1
  fi
  
  echo "✓ Canary healthy. Error rate: $ERROR_RATE%"
  sleep $INTERVAL
done
```

### Pros and Cons
**Pros:**
- Minimal risk (only 5-10% affected initially)
- Real user traffic validation
- Gradual rollout reduces blast radius
- Quick rollback with weight adjustment

**Cons:**
- Takes longer to fully deploy (20-30 minutes)
- Requires good monitoring and alerting
- More complex than blue-green
- Database migrations still challenging

---

## 3. Rolling Deployment

### What it is
Update servers one or few at a time. Remove from load balancer, update application, add back. Repeat with next batch.

### Step-by-Step Setup

#### Step 1: Create Initial Cluster
1. **Launch 6 instances (v1.0):**
   - **Names:** `app-server-1` through `app-server-6`
   - **AMI:** Amazon Linux 2
   - **Type:** t2.micro
   - **User Data:**
   ```bash
   #!/bin/bash
   yum update -y
   yum install httpd -y
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Running v1.0</h1>" > /var/www/html/index.html
   ```

#### Step 2: Create Load Balancer
1. **Create ALB** → **Name:** `rolling-lb`
2. **Target Group:**
   - **Name:** `app-tg`
   - Add all 6 instances
   - **Health Check settings:**
     - Path: `/`
     - Interval: 30 seconds
     - Healthy threshold: 2
     - Unhealthy threshold: 2

#### Step 3: Verify Load Balancer Working
```bash
# Test all 6 servers are responding
for i in {1..12}; do
  curl http://<rolling-lb-dns>/
done

# All should show "Running v1.0"
```

### Rolling Deployment Process

#### Batch 1: Update Servers 1-2 (33% capacity reduction)

**Step 1: Remove from Load Balancer**
1. Go to **Load Balancers** → Select `rolling-lb`
2. **Targets tab** → Select `app-server-1` and `app-server-2`
3. Click **Deregister** (Unhealthy - instance not returning traffic)
4. Wait for instances to show "Unused" status

**Step 2: Update Application**
```bash
# SSH into app-server-1
ssh -i your-key.pem ec2-user@<app-server-1-public-ip>

# Stop old application
sudo systemctl stop httpd

# Deploy new version
cd /var/www/html
sudo bash -c 'echo "<h1>Running v2.0</h1>" > index.html'

# Start application
sudo systemctl start httpd

# Test locally
curl http://localhost/
# Should show: <h1>Running v2.0</h1>

# Exit and repeat for app-server-2
```

**Step 3: Add Back to Load Balancer**
1. **Targets tab** → Click **Register targets**
2. Select `app-server-1` and `app-server-2`
3. Click **Register**
4. Wait for status to show "Healthy"

**Step 4: Monitor**
```bash
# Test traffic still works
for i in {1..10}; do
  curl http://<rolling-lb-dns>/
done

# Check for any 5XX errors
```

#### Batch 2: Update Servers 3-4
1. Deregister `app-server-3` and `app-server-4` from target group
2. SSH and update application to v2.0
3. Register back in target group
4. Monitor

#### Batch 3: Update Servers 5-6
1. Deregister `app-server-5` and `app-server-6`
2. SSH and update application to v2.0
3. Register back in target group
4. Monitor

### Automated Rolling Deployment Script
```bash
#!/bin/bash
# Automated rolling deployment script

LOAD_BALANCER="rolling-lb"
TARGET_GROUP="app-tg"
BATCH_SIZE=2
WAIT_TIME=60  # Wait 60 seconds for health check

# Get all instances in target group
INSTANCES=$(aws elbv2 describe-target-health \
  --target-group-arn $(aws elbv2 describe-target-groups \
    --names $TARGET_GROUP \
    --query 'TargetGroups[0].TargetGroupArn' \
    --output text) \
  --query 'TargetHealthDescriptions[].Target.Id' \
  --output text)

# Convert to array
IDS=($INSTANCES)
TOTAL=${#IDS[@]}

echo "Rolling deployment starting for $TOTAL instances..."

# Process in batches
for ((i=0; i<$TOTAL; i+=$BATCH_SIZE)); do
  BATCH=("${IDS[@]:$i:$BATCH_SIZE}")
  
  echo "[$(date)] Batch $((i/$BATCH_SIZE + 1)): Deploying to ${BATCH[@]}"
  
  # Deregister from load balancer
  for ID in "${BATCH[@]}"; do
    echo "  - Deregistering $ID..."
    aws elbv2 deregister-targets \
      --target-group-arn $(aws elbv2 describe-target-groups \
        --names $TARGET_GROUP \
        --query 'TargetGroups[0].TargetGroupArn' \
        --output text) \
      --targets Id=$ID
  done
  
  # Wait for deregistration
  sleep 10
  
  # Deploy new version
  for ID in "${BATCH[@]}"; do
    echo "  - Updating application on $ID..."
    # SSH and deploy (requires passwordless SSH setup)
    # ssh ec2-user@$ID 'deploy-script.sh'
  done
  
  # Register back to load balancer
  for ID in "${BATCH[@]}"; do
    echo "  - Registering $ID back..."
    aws elbv2 register-targets \
      --target-group-arn $(aws elbv2 describe-target-groups \
        --names $TARGET_GROUP \
        --query 'TargetGroups[0].TargetGroupArn' \
        --output text) \
      --targets Id=$ID
  done
  
  # Wait for health check to pass
  echo "  - Waiting for health checks to pass..."
  sleep $WAIT_TIME
  
  # Continue to next batch
  if [ $((i + $BATCH_SIZE)) -lt $TOTAL ]; then
    echo "Batch complete. Moving to next batch in 30 seconds..."
    sleep 30
  fi
done

echo "Rolling deployment complete!"
```

### Pros and Cons
**Pros:**
- Minimal extra infrastructure needed
- Gradual deployment reduces risk
- If issue found, only newly deployed servers affected
- Easy to pause and investigate

**Cons:**
- Takes longer than blue-green (30+ minutes for large clusters)
- Mixed versions running during deployment
- Database migrations require extra care
- Gradual user exposure to bugs

---

## 4. Immutable Deployment (Bonus)

### What it is
Create new instances with new application version, add to load balancer, remove old instances. Each instance is treated as immutable (never updated, replaced entirely).

### Step-by-Step Setup

#### Step 1: Create v1.0 Instances
```bash
# Launch 3 instances with tag: version=1.0
```

#### Step 2: Create v2.0 Instances
```bash
# Launch 3 new instances with v2.0 application
# Tag them: version=2.0
```

#### Step 3: Add v2.0 to Load Balancer
1. **Targets tab** → **Register targets**
2. Add all 3 v2.0 instances

#### Step 4: Remove v1.0 from Load Balancer
1. **Targets tab** → Deregister v1.0 instances
2. All traffic now on v2.0

#### Step 5: Terminate v1.0 Instances
1. Go to **Instances** → Select v1.0 instances
2. **Terminate**

### Pros and Cons
**Pros:**
- Immutable infrastructure (better for debugging)
- Clean separation between versions
- Easy rollback (keep old instances running temporarily)

**Cons:**
- Higher infrastructure cost during transition
- Slower than blue-green
- Wasteful if doing frequent deployments

---

## Comparison Table

| Strategy | Downtime | Cost | Speed | Risk | Rollback Time |
|----------|----------|------|-------|------|---------------|
| Blue-Green | Zero | 2x | Very Fast | Low | < 1 sec |
| Canary | Zero | 1.5x | Slow (20-30 min) | Very Low | < 1 min |
| Rolling | Zero | 1x | Medium (30+ min) | Medium | 10-15 min |
| Immutable | Zero | 1.5x | Medium | Low | 5-10 min |

---

## Best Practices for All Strategies

### 1. Health Checks
```bash
# Always configure health checks before deploying
# In Load Balancer target group:
# - Path: /health or /status
# - Interval: 30 seconds
# - Healthy threshold: 2 checks
# - Unhealthy threshold: 2 checks
```

### 2. Monitoring Before Deploy
```bash
# Create CloudWatch dashboard with:
- Application Error Rate (ALB 5XX count)
- Target Response Time
- Target CPU Utilization
- Memory Usage
- Disk I/O
- Database Query Times
```

### 3. Automated Rollback
```bash
#!/bin/bash
# Trigger rollback if error rate spikes

ERROR_RATE=$(aws cloudwatch get-metric-statistics \
  --metric-name HTTPCode_Target_5XX_Count \
  --statistics Sum | grep -o '[0-9]*')

if [ $ERROR_RATE -gt 10 ]; then
  echo "ERROR: Rolling back deployment"
  # Execute rollback commands
fi
```

### 4. Database Migrations
- Always run migrations BEFORE deploying new code
- Ensure migrations are backward compatible
- Test rollback scenarios

### 5. Testing Strategy
```bash
# Pre-deployment testing
1. Unit tests (local)
2. Integration tests (staging)
3. Load tests (staging)
4. Smoke tests (post-deploy)
5. Canary monitoring (production)
```
