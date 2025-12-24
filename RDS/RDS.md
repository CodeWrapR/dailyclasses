-- Query WITH index (fast)
SELECT * FROM large_table WHERE email = 'user100@example.com';
-- This is fast (index used)
```

**Step 5: View & Analyze Slow Query Log**

```bash
# Download slow query log
aws rds describe-db-log-files \
  --db-instance-identifier my-first-rds-db \
  --query 'DescribeDBLogFiles[0]'

# Get slowquery log
aws rds download-db-log-file-portion \
  --db-instance-identifier my-first-rds-db \
  --log-file-name error/mysql-error.log \
  --starting-token 0 > slowquery.log

cat slowquery.log | head -100
```

**Step 6: Analyze Query Performance**

```sql
-- EXPLAIN shows query execution plan
EXPLAIN SELECT * FROM large_table WHERE created_at < NOW() - INTERVAL 10 DAY;

-- Output:
-- id  select_type  table      type   key   rows     filtered  Extra
-- 1   SIMPLE       large_table ALL    NULL  10000    100      Using where

-- ALL = Full table scan (bad!)
-- Should show: range, ref, or eq_ref

-- EXPLAIN with FORMAT=JSON (more details)
EXPLAIN FORMAT=JSON SELECT * FROM large_table WHERE created_at < NOW() - INTERVAL 10 DAY\G

-- Check index usage
SELECT * FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE OBJECT_SCHEMA = 'myappdb';
```

**Step 7: Create Missing Indexes**

```sql
-- Add index on created_at
CREATE INDEX idx_created_at ON large_table(created_at);

-- Now query is fast!
EXPLAIN SELECT * FROM large_table WHERE created_at < NOW() - INTERVAL 10 DAY;

-- Output now shows:
-- key: idx_created_at
-- type: range
-- rows: Much fewer rows scanned
```

### Performance Tuning Workflow

```
1. Identify Slow Queries
   └─ Enable slow query log (long_query_time = 1)

2. Analyze Query Plans
   └─ Use EXPLAIN to find full table scans

3. Create Indexes
   └─ Add index on WHERE/JOIN columns

4. Monitor Performance
   └─ Check response time improvement

5. Tune Parameters
   └─ Increase buffer pool, connection limits

6. Monitor Again
   └─ Repeat until satisfied
```

### Real-World Example

```
Before Optimization:
├─ Query: SELECT * FROM users WHERE email = 'test@example.com'
├─ Execution: Full table scan (10M rows)
├─ Time: 45 seconds ❌
└─ Index: None

After Optimization:
├─ Added: CREATE INDEX idx_email ON users(email)
├─ Execution: Index lookup
├─ Time: 0.05 seconds ✅
└─ Improvement: 900x faster!
```

### Validation Checklist
- [ ] Parameter group modified
- [ ] Slow query log enabled
- [ ] Generated slow queries
- [ ] Downloaded and viewed slow query log
- [ ] Used EXPLAIN to analyze queries
- [ ] Created missing indexes
- [ ] Verified index improves performance
- [ ] Understand query optimization

### Key Takeaway
✅ Parameter tuning critical for performance  
✅ Slow query log finds problems  
✅ EXPLAIN shows execution plan  
✅ Proper indexes = 100x+ improvement  

---

## POST-LAB 4-6 CLEANUP

```bash
# Keep instance running (need for later labs)

# Costs so far:
# RDS t3.micro: ~$0.015/hour = ~$11/month
# Storage: 20 GB = ~$2/month
# Total: ~$13/month
```

---

## SECTION 3: BACKUP, RECOVERY & HIGH AVAILABILITY (Labs 7-10)

### Lab 7: Snapshots & Point-In-Time Recovery

**Objective:** Create backups and restore to previous state.

**Why This Matters:**
- Data loss happens
- Snapshots = Insurance
- Can restore any point in last 35 days
- RPO: Near zero (automated backups)
- RTO: Minutes (quick restore)

### Understanding RDS Backups

```
RDS Backup Types:

1. Automated Backups (Daily)
   ├─ Run daily during backup window
   ├─ Retained: 1-35 days (configurable)
   ├─ Cost: Included in RDS charge
   └─ Use: Point-in-time recovery

2. Manual Snapshots
   ├─ Created on-demand
   ├─ Retained: Forever (until deleted)
   ├─ Cost: Same as automated backup storage
   └─ Use: Before major changes, compliance

3. Backup Types:
   ├─ Full backup: Daily snapshot of everything
   ├─ Incremental: Only changes since last full
   └─ Transaction log: Enables PITR to any second
```

### Step-By-Step Instructions

**Step 1: View Existing Backups**

- Go to RDS Console
- Automated Backups
- Shows: Daily automated snapshots
- Shows: Retention period (default 7 days)

**Step 2: Create Manual Snapshot**

- Go to Databases
- Click your instance
- Actions → Create Snapshot
- Snapshot Identifier: `my-snapshot-backup-2024-01-15`
- Click "Create"

Status: Creating → Available (2-5 minutes)

**Step 3: View Snapshot**

- Go to Snapshots
- Shows: Size, creation time, retention

Snapshots detail shows:
- DB Instance ID: my-first-rds-db
- Snapshot ID: my-snapshot-backup-2024-01-15
- Status: Available
- Size: 100 MB (or actual size)
- Can be copied to another region
- Can be shared with other accounts

**Step 4: Create Data Change (Simulate Problem)**

```sql
-- Connect to RDS
mysql -h endpoint -u admin -p -D myappdb

-- Current employees
SELECT * FROM employees;
-- Shows: 4 employees

-- Oops! Delete all employees by mistake
DELETE FROM employees;
-- Deleted 4 rows ❌

SELECT * FROM employees;
-- Shows: 0 rows (empty!)

-- Realize mistake... need to restore!
```

**Step 5: Point-In-Time Recovery (PITR)**

Restore to before the DELETE:

- Go to RDS Databases
- Click your instance
- Actions → Restore to Point in Time

**Step 6: Configure Restore**

- New DB Instance Identifier: `my-first-rds-db-restored`
- Restore time: 
  - Latest: Yes (latest time before deletion)
  - Or: Specific timestamp (before DELETE)
- Instance class: db.t3.micro (same as original)
- Storage: 20 GB GP3
- VPC: Same VPC
- Subnet group: Same
- Security group: Same
- Backup retention: 7 days
- Multi-AZ: No
- Click "Restore"

**Step 7: Wait for Restore**

Status: Creating → Available (5-10 minutes)

New instance created with:
- All data from backup point
- Original employees table INTACT
- All schema and data recovered

**Step 8: Verify Restore**

```bash
# Connect to restored instance
mysql -h my-first-rds-db-restored.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       -u admin \
       -p \
       -D myappdb

-- Check employees (should be 4 now!)
SELECT * FROM employees;
-- Shows: 4 employees ✅
-- Data recovered!
```

**Step 9: Promote Restored Instance (Optional)**

If restore is good:
- Copy data back to original
- OR make restored instance = new primary
- OR delete original, rename restored

For now: Keep both for comparison

**Step 10: Delete Snapshots**

When no longer needed:
- Go to Snapshots
- Select snapshot
- Actions → Delete
- Confirm delete

This frees up snapshot storage (charged same as backup)

### Backup Strategy

```
Best Practice:

Automated Backups:
├─ Retention: 7-35 days
├─ Covers: Most scenarios
├─ Cost: Included

Manual Snapshots:
├─ Before major changes: Always
├─ Daily: Yes (or hourly for critical)
├─ Weekly: Full backup (7 day retention)
├─ Monthly: Archive (180 day retention)
└─ Cost: ~$0.023 per GB per month

Example (100 GB database):
├─ Automated: 7 snapshots × 100 GB = 700 GB = $16/month
├─ Manual daily: 7 snapshots × 100 GB = 700 GB = $16/month
├─ Total: ~$32/month backup cost
├─ Vs. data loss: PRICELESS ✅
```

### Real-World Disaster

```
Scenario: Ransomware deletes all database

Hour 0: Ransomware activates
├─ Deletes all data
├─ Corrupts tables
└─ Demands $1M ransom ❌

Hour 0.5: Restore from snapshot
├─ Latest manual snapshot: 1 hour old
├─ Restore time: 5 minutes
└─ Data recovered (only 1 hour data loss) ✅

Hour 1: Back in business
├─ System operational
├─ No ransom paid
├─ Insurance covers recovery time
└─ Total cost: $0 (insurance + manual snapshot storage)

Without backup: Data gone forever, business dies ❌
```

### Validation Checklist
- [ ] Manual snapshot created
- [ ] Can view snapshot details
- [ ] Simulated data loss (DELETE)
- [ ] Restored from PITR
- [ ] Verified data recovered
- [ ] Both instances exist
- [ ] Understand backup costs

### Key Takeaway
✅ Automated backups = 7-35 days retention  
✅ Manual snapshots = Long-term storage  
✅ PITR = Restore to any second  
✅ Snapshots = Data insurance  

---

### Lab 8: Multi-AZ & Automatic Failover

**Objective:** Enable high availability with Multi-AZ.

**Why This Matters:**
- AZ failure possible
- Multi-AZ = Automatic failover
- Zero data loss
- Minimal downtime (~1-2 minutes)
- Production requirement

### Understanding Multi-AZ

```
Single-AZ (Risky):
AZ-1a (us-east-1a)
└─ RDS Instance (Primary)
   └─ If AZ fails: DOWN! ❌
   └─ RTO: 1-2 hours (manual intervention)

Multi-AZ (Safe):
AZ-1a (us-east-1a)          AZ-1b (us-east-1b)
└─ RDS Instance (Primary)  ←→  RDS Instance (Standby)
   └─ Synchronous replication
   └─ If AZ fails: Automatic failover! ✅
   └─ RTO: 1-2 minutes (automatic)
```

### Step-By-Step Instructions

**Step 1: Enable Multi-AZ on Instance**

- Go to RDS Databases
- Click your original instance (my-first-rds-db)
- Click "Modify"

**Step 2: Configure Multi-AZ**

- Find "Availability and durability"
- Multi-AZ deployment: Enable
- Shows: Creates standby instance in different AZ

**Step 3: Review Changes**

Summary shows:
- Instance: db.t3.micro
- Multi-AZ: Enabled
- Downtime: Yes (brief, during failover setup)

Scheduling:
- Apply immediately (if testing)
- OR schedule during maintenance window
- Select "Apply immediately" for testing

**Step 4: Apply Multi-AZ**

Click "Modify"

Status: Modifying → Available (5-10 minutes)

During modification:
- Primary instance pauses
- Standby instance created in different AZ
- Data synchronized
- Failover enabled
- Brief downtime (~1 minute)

**Step 5: Verify Multi-AZ**

- Go to instance details
- Shows: Multi-AZ: Enabled
- Shows: Primary AZ: us-east-1a
- Shows: Standby AZ: us-east-1b
- Shows: DB subnet group includes both AZs

**Step 6: Test Failover (Forced)**

CAUTION: This causes brief downtime!

```bash
# Via AWS CLI
aws rds reboot-db-instance \
  --db-instance-identifier my-first-rds-db \
  --force-failover

# Status: Rebooting (with failover)
# Duration: 1-2 minutes
# Failure: Primary AZ-1a → Standby AZ-1b

# After reboot:
# Primary AZ: us-east-1b (was standby)
# Standby AZ: us-east-1a (was primary)
```

Failover occurs:
1. Primary in AZ-1a stops accepting connections
2. Standby in AZ-1b promoted to primary
3. New standby created in AZ-1a
4. Connections resume (automatic reconnect)

**Step 7: Monitor Failover**

Via CloudWatch:
- Go to CloudWatch
- Metrics → RDS
- DBInstanceFailover: Count (increases by 1)

Via RDS Console:
- Database Activity: Shows connections dropped then resumed
- Logs: Shows failover events

**Step 8: Understand Automatic Failover**

With Multi-AZ enabled:

```
Scenario 1: Primary AZ fails
├─ AWS detects failure (15-30 seconds)
├─ Initiates failover
├─ Standby becomes primary
├─ New standby created
├─ Downtime: 1-2 minutes
└─ Data loss: ZERO ✅

Scenario 2: Primary database process fails
├─ AWS detects failure
├─ Restarts database process
├─ Standby remains standby
├─ Downtime: 1 minute
└─ Data loss: ZERO ✅

Scenario 3: Network connectivity lost
├─ Standby detects issue
├─ Failover initiated
├─ Downtime: 1-2 minutes
└─ Data loss: ZERO ✅

Scenario 4: Storage fails
├─ AWS moves instance to healthy storage
├─ Downtime: Few seconds
└─ Data loss: ZERO ✅
```

### Multi-AZ Cost

```
Single-AZ:
└─ db.t3.micro: ~$0.015/hour = $11/month

Multi-AZ:
└─ db.t3.micro × 2: ~$0.030/hour = $22/month
└─ Double cost but: Worth it for production!
```

### Real-World SLA

```
Single-AZ:
├─ Availability: ~99.0%
├─ Downtime/month: ~7 hours
└─ Recovery: Manual (hours)

Multi-AZ:
├─ Availability: ~99.95%
├─ Downtime/month: ~20 minutes
└─ Recovery: Automatic (<2 minutes)

Difference: Dramatic! Production MUST use Multi-AZ
```

### Validation Checklist
- [ ] Multi-AZ enabled
- [ ] Standby instance visible in different AZ
- [ ] Forced failover completed
- [ ] Failover metrics visible in CloudWatch
- [ ] Understand failover process
- [ ] Know Multi-AZ cost

### Key Takeaway
✅ Multi-AZ = High availability  
✅ Automatic failover in 1-2 minutes  
✅ Zero data loss  
✅ Production requirement  

---

### Lab 9: Read Replicas & Scaling

**Objective:** Create read replicas for scaling read operations.

**Why This Matters:**
- Write-heavy = One primary
- Read-heavy = Multiple replicas
- Offload reporting/analytics
- Global distribution
- Up to 15 read replicas per primary

### Understanding Read Replicas

```
Single Instance:
All Users → Write (1000 req/s) → SLOW ❌
         → Read (5000 req/s) → SLOW ❌

With Read Replicas:
Write Users → Primary → 1000 req/s ✅
Read Users  → Replica 1 → 2000 req/s ✅
           → Replica 2 → 2000 req/s ✅
           → Replica 3 → 2000 req/s ✅
Total: 1000 writes + 6000 reads = 7000 req/s ✅
```

### Step-By-Step Instructions

**Step 1: Create Read Replica**

- Go to RDS Databases
- Click your primary instance (my-first-rds-db)
- Actions → Create Read Replica

**Step 2: Configure Replica**

- Read Replica ID: `my-first-rds-read-replica-1`
- Instance class: db.t3.micro (can be different/larger)
- Availability Zone: Different AZ (or same region)
- Multi-AZ deployment: No (for testing)
- Click "Create"

Status: Creating → Available (2-5 minutes)

**Step 3: Create Another Replica (In Different Region)**

For geographic distribution:
- Actions → Create Read Replica
- Read Replica ID: `my-first-rds-read-replica-2`
- Destination region: eu-west-1
- Instance class: db.t3.micro
- Create

This creates replica in Europe (1-5 minute sync)

**Step 4: View Replica Relationships**

- Click primary instance
- "Replicas" section: Shows all read replicas
- Shows: Region, AZ, lag time

**Step 5: Connect to Replica**

```bash
# Get replica endpoint
# From RDS console: my-first-rds-read-replica-1.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com

# Connect
mysql -h my-first-rds-read-replica-1.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       -u admin \
       -p \
       -D myappdb

-- Check data (matches primary)
SELECT * FROM employees;
-- Shows: All employees ✅

-- Try INSERT (should fail or error)
INSERT INTO employees (name, email) VALUES ('Test', 'test@example.com');
-- Error: Read-only replica
-- Cannot write to read replica ✅
```

**Step 6: Monitor Replication Lag**

```bash
# Check replication lag (primary → replica)
aws rds describe-db-instances \
  --db-instance-identifier my-first-rds-read-replica-1 \
  --query 'DBInstances[0].ReplicationLag'

# Shows: Seconds behind primary
# Typically: < 1 second (same AZ)
# Can be: 1-10 seconds (different AZ)
# Can be: 10-100 seconds (different region)
```

**Step 7: Promote Replica to Standalone**

```bash
# If replica needs to become primary
aws rds promote-read-replica \
  --db-instance-identifier my-first-rds-read-replica-1

# Status: Modifying → Promoting → Available
# Duration: 1-5 minutes

# After promotion:
# - Replica becomes independent primary
# - Replication from original stops
# - Can now accept writes
# - Point for failover/migration
```

**Step 8: Use Cases for Read Replicas**

```
Use Case 1: Reporting
├─ Heavy analytics queries
├─ Replica dedicated to reports
├─ Primary unaffected ✅

Use Case 2: Disaster Recovery
├─ Replica in different region
├─ Promote to primary if original fails
├─ Quick failover

Use Case 3: Development
├─ Replica as dev database
├─ Real production data
├─ Safe testing

Use Case 4: Offload Backups
├─ Backup from replica (not primary)
├─ Primary unaffected
├─ No performance impact

Use Case 5: Global Scale
├─ Replicas in each region
├─ Users read from nearest replica
├─ Better latency
```

### Read Replica Cost

```
Primary Instance:
└─ db.t3.micro: $0.015/hour = $11/month

Each Read Replica:
└─ db.t3.micro: $0.015/hour = $11/month

Example (1 primary + 3 replicas):
└─ Total: $44/month

But: Can handle 4x read load!
Cost per request: Much lower
Scaling pays for itself! ✅
```

### Validation Checklist
- [ ] Read replica created (same region)
- [ ] Read replica created (different region)
- [ ] Connected to replica successfully
- [ ] Verified read-only access
- [ ] Checked replication lag
- [ ] Understand replica use cases

### Key Takeaway
✅ Read replicas scale read operations  
✅ Up to 15 replicas per primary  
✅ Can be different regions  
✅ Can promote to standalone  

---

### Lab 10: RDS Proxy & Connection Pooling

**Objective:** Implement connection pooling with RDS Proxy.

**Why This Matters:**
- Connection overhead significant
- Applications create many connections
- Exhausts connection limit
- RDS Proxy pools connections
- 100x+ improvement possible

### Understanding Connection Pooling

```
Without Connection Pooling:
User 1 → New connection to RDS
User 2 → New connection to RDS
User 3 → New connection to RDS
...
User 100 → New connection to RDS
└─ 100 connections (overhead, slow)
└─ max_connections: 200 (limited)

With Connection Pooling (RDS Proxy):
User 1 → Proxy → Reuses connection to RDS
User 2 → Proxy → Reuses connection to RDS
User 3 → Proxy → Reuses connection to RDS
...
User 100 → Proxy → Reuses same 10 connections
└─ 10 connections (efficient, fast)
└─ max_connections: 200 (never reached)
└─ Latency: 1/10th ✅
```

### Step-By-Step Instructions

**Step 1: Create RDS Proxy**

- Go to RDS Console
- Proxies (left menu)
- Click "Create Proxy"

**Step 2: Configure Proxy**

Basic Details:
- Proxy Identifier: `my-rds-proxy`
- Engine Family: MySQL
- Engine Version: 8.0

Target RDS Database Instance:
- Database: my-first-rds-db
- Authentication:
  - Database username: admin
  - Secrets Manager secret: Create new
    - Secret name: rds-db-password
    - Select: Store a new secret
    - Username: admin
    - Password: Your password
  - Click "Create secret"

IAM Role:
- Create new role: Yes
- Auto-creates: ProxyRole with Secrets Manager access

**Step 3: Configure Connection Pooling**

Connection pool settings:
- Max connections: 100
  (Max connections proxy maintains to RDS)
- Init query: Leave empty
- Borrow timeout: 120 seconds
- Session pinning filters: Leave default

Client connections:
- Max idle connections: 1
  (Remove idle connections after 1 second)
- Connection borrow timeout: 120 seconds
- Execution timeout: 900 seconds

**Step 4: Create Proxy**

Click "Create"

Status: Creating → Available (1-3 minutes)

**Step 5: View Proxy Details**

- Proxy Endpoint: `my-rds-proxy.cxxxxxxxxxxx.proxy-us-east-1.rds.amazonaws.com`
- This is what applications connect to!
- Proxy routes to actual RDS instance

**Step 6: Connect Through Proxy**

Instead of connecting directly to RDS:

```bash
# Old way (direct to RDS)
mysql -h my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       -u admin \
       -p \
       -D myappdb

# New way (through proxy)
mysql -h my-rds-proxy.cxxxxxxxxxxx.proxy-us-east-1.rds.amazonaws.com \
       -u admin \
       -p \
       -D myappdb

# Connection established through proxy!
-- Query works exactly the same
SELECT * FROM employees;
-- Shows: 4 employees ✅
```

**Step 7: Monitor Proxy**

```bash
# Check proxy metrics
aws rds describe-db-proxies \
  --db-proxy-name my-rds-proxy \
  --query 'DBProxies[0]'

# Shows:
# - Status: available
# - Endpoints: proxy-us-east-1.rds.amazonaws.com
# - Target groups: my-first-rds-db
```

**Step 8: Performance Comparison**

Create test script:

```python
#!/usr/bin/env python3
import mysql.connector
import time

# Direct connection (no proxy)
def test_direct():
    times = []
    for i in range(100):
        start = time.time()
        conn = mysql.connector.connect(
            host="my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com",
            user="admin",
            password="MyPass123!",
            database="myappdb"
        )
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM employees")
        result = cursor.fetchall()
        conn.close()
        times.append(time.time() - start)
    
    print(f"Direct: Avg {sum(times)/len(times):.3f}s per connection")

# Through proxy
def test_proxy():
    times = []
    for i in range(100):
        start = time.time()
        conn = mysql.connector.connect(
            host="my-rds-proxy.cxxxxxxxxxxx.proxy-us-east-1.rds.amazonaws.com",
            user="admin",
            password="MyPass123!",
            database="myappdb"
        )
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM employees")
        result = cursor.fetchall()
        conn.close()
        times.append(time.time() - start)
    
    print(f"Proxy: Avg {sum(times)/len(times):.3f}s per connection")

test_direct()
test_proxy()

# Output:
# Direct: Avg 0.350s per connection
# Proxy: Avg 0.050s per connection
# Improvement: 7x faster! ✅
```

### RDS Proxy Use Cases

```
Use Case 1: Lambda Functions
├─ Each Lambda = New connection
├─ Proxy pools connections
├─ Without proxy: Lambda exhausts DB connections
├─ With proxy: 1000s of Lambdas, few DB connections ✅

Use Case 2: Web Applications
├─ App servers create new connections per request
├─ Proxy reuses connections
├─ Throughput 10x higher ✅

Use Case 3: Microservices
├─ Each service = New connections
├─ Proxy acts as single connection pool
├─ Centralized connection management ✅

Use Case 4: Serverless Applications
├─ Ephemeral functions
├─ Can't maintain persistent connections
├─ Proxy makes connections efficient ✅
```

### Proxy Cost

```
RDS Proxy:
├─ $0.015 per vCPU-hour
├─ Minimum 1 vCPU: $0.015/hour = $11/month
├─ 2 vCPU: $0.030/hour = $22/month
├─ Usually: 1-2 vCPU sufficient

Example:
├─ Without proxy: 500 DB connections, slow
├─ With proxy: 20 DB connections, fast
├─ Cost: +$11/month
├─ Throughput improvement: 10x
├─ Cost per transaction: 10x lower! ✅
```

### Validation Checklist
- [ ] RDS Proxy created
- [ ] Proxy endpoint accessible
- [ ] Connected through proxy successfully
- [ ] Queries work through proxy
- [ ] Understand connection pooling benefit
- [ ] Know proxy use cases

### Key Takeaway
✅ RDS Proxy enables connection pooling  
✅ Reduces connection overhead by 10x+  
✅ Essential for Lambda/serverless  
✅ Small cost for big benefit  

---

## POST-LAB 7-10 CLEANUP

```bash
# Keep instances (need for migration lab)

# Current costs:
# Primary instance: ~$11/month
# Multi-AZ: +$11/month = $22/month
# 2 Read replicas: +$22/month = $44/month
# RDS Proxy: +$11/month = $55/month
# Storage: ~$2-5/month
# Total: ~$57-60/month

# For production, this is necessary!
```

---

## SECTION 4: MIGRATION & ADVANCED (Labs 11-18)

### Lab 11: SQL Server Migration (Using Database Migration Service - DMS)

**Objective:** Migrate SQL Server database to AWS RDS.

**Why This Matters:**
- Moving on-premises databases to cloud
- Minimal downtime
- AWS DMS handles complexity
- Real-world scenario

### Understanding Database Migration

```
Migration Methods:

1. Native Tools (SQL Server)
   ├─ SQL Server Management Studio
   ├─ SQL Server backup/restore
   └─ Manual process (hours/days)

2. AWS Database Migration Service (DMS)
   ├─ Automated replication
   ├─ Minimal downtime
   ├─ Real-time sync
   └─ AWS manages complexity

3. AWS DataSync
   ├─ Large data volumes
   ├─ Network optimization
   └─ Parallel transfers

Migration Phases:
├─ Phase 1: Full load (all data)
├─ Phase 2: CDC (Continuous Data Capture)
├─ Phase 3: Cutover (switch to new)
```

### Step-By-Step Instructions

**Step 1: Prepare Source Database**

For this lab, we'll simulate migrating FROM a local SQL Server or another RDS instance.

If you have real SQL Server:
```sql
-- Enable CDC (Change Data Capture) for migration
USE [master]
GO

EXEC sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXEC sp_configure 'SQL Server agent XPs', 1
GO
RECONFIGURE
GO

-- Create test database
CREATE DATABASE SourceDB
GO

USE SourceDB
GO

-- Create test table
CREATE TABLE Customers (
  CustomerID INT PRIMARY KEY IDENTITY(1,1),
  FirstName NVARCHAR(50),
  LastName NVARCHAR(50),
  Email NVARCHAR(100),
  CreatedDate DATETIME DEFAULT GETDATE()
)
GO

-- Insert test data
INSERT INTO Customers (FirstName, LastName, Email) VALUES
('Alice', 'Johnson', 'alice@example.com'),
('Bob', 'Smith', 'bob@example.com'),
('Charlie', 'Brown', 'charlie@example.com')
GO
```

**Step 2: Create RDS Target Instance**

If not already created, create new RDS instance:
- Engine: MySQL 8.0 (or PostgreSQL)
- Instance: db.t3.micro
- Database: targetdb
- Note: In real migration, source and target might be different engines!

**Step 3: Create DMS Service Role**

```bash
# Create IAM role for DMS
aws iam create-role \
  --role-name dms-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "dms.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach# AWS RDS Labs: Complete Guide from Basic to Advanced
## Relational Database Service with SQL Database Management, Security, Monitoring & Disaster Recovery

---

## Overview

This guide contains 18 progressive labs covering AWS RDS (Relational Database Service) from foundational concepts to enterprise-level deployment. Each lab includes objectives, step-by-step instructions, practical SQL examples, connectivity testing, and complete cleanup procedures.

**Topics Covered:**
- RDS Fundamentals & Service Offerings
- Instance Creation & Management
- Database Engines (MySQL, PostgreSQL, SQL Server, MariaDB, Oracle)
- Connectivity & Security
- Parameter Groups & Option Groups
- Subnet Groups & Multi-AZ
- Backups & Snapshots
- Read Replicas & High Availability
- Performance Monitoring & Optimization
- Database Migration (SQL Server to RDS)
- Disaster Recovery & Data Restoration
- Cost Optimization

---

## SECTION 1: RDS FUNDAMENTALS (Labs 1-3)

### Lab 1: Understanding RDS Architecture & Service Offerings

**Objective:** Understand RDS concepts, service tiers, and offerings.

**Why This Matters:**
- RDS = Managed relational database
- AWS handles backups, patches, maintenance
- Different engines for different needs
- IaaS vs PaaS understanding

### Understanding RDS

```
Traditional Database:
You → Buy server → Install OS → Install DB → Manage → Maintain
├─ Cost: High
├─ Effort: Extreme
├─ Risk: High (you responsible for everything)
└─ Expertise: Need DBA

RDS (Managed Service):
You → AWS RDS → Database ready
├─ Cost: Lower
├─ Effort: Minimal
├─ Risk: Low (AWS handles ops)
└─ Expertise: No DBA needed!
```

### RDS Service Offerings

```
1. Traditional RDS (IaaS-like)
   └─ You manage: Database, users, backups, performance tuning
   └─ AWS manages: Hardware, OS, patches, availability
   └─ Cost: Pay per instance size + storage + data transfer

2. Aurora (Fully Managed, PaaS-like)
   └─ You manage: SQL queries, schema
   └─ AWS manages: Everything else (auto-scaling, replication)
   └─ Cost: Pay per storage + I/O operations

3. RDS Proxy
   └─ Connection pooling
   └─ Reduced overhead
   └─ Better scaling
```

### Supported Database Engines

```
Engine           Origin      Use Case            Cost
──────────────────────────────────────────────────────
MySQL 5.7/8.0    Open-source Web apps, startups Low
PostgreSQL       Open-source Complex queries      Low
MariaDB          Open-source MySQL alternative   Low
Oracle           Commercial Enterprise apps      HIGH
SQL Server       Microsoft   Windows ecosystem   HIGH
Aurora MySQL     AWS Native  High-performance    Medium
Aurora PostgreSQL AWS Native High-performance    Medium
```

### RDS Architecture

```
AWS Region (us-east-1)
│
├─ Availability Zone 1a
│  └─ RDS Instance (Primary)
│     ├─ EBS Storage (Multi-AZ enabled)
│     └─ Automated backups (daily)
│
├─ Availability Zone 1b
│  └─ RDS Instance (Standby/Multi-AZ)
│     └─ Synchronous replica (always in sync)
│
└─ S3 Backup Vault
   └─ 35-day backup history (automated)

If AZ-1a fails → Automatic failover to 1b
Downtime: 1-2 minutes (RTO) ✅
Zero data loss ✅
```

### RDS Service Tiers

```
Standard Tier:
├─ Single-AZ instance
├─ Manual backups possible
├─ RTO: 1+ hours (manual recovery)
├─ Cost: Low
└─ Use: Dev/Test/Non-critical

Multi-AZ Tier:
├─ Primary + Standby instance
├─ Automatic failover
├─ RTO: 1-2 minutes
├─ Cost: 2x single-AZ
└─ Use: Production/Critical

Enhanced High Availability:
├─ 3 instances (1 primary + 2 standby)
├─ RTO: < 1 minute
├─ Cost: 3x single-AZ
└─ Use: Mission-critical
```

### RDS vs Self-Managed vs Aurora

```
Feature              Self-Managed  RDS          Aurora
─────────────────────────────────────────────────────────
Availability         Manual        99.95%       99.99%
Backups              Your job      Automated    Automated
Patches              Your job      Automated    Automated
Scaling              Manual        Manual       Auto-scaling
Replication          Manual        Manual       Built-in
Cost                 Low/High      Medium       Medium/High
Effort               EXTREME       Low          Low
Performance          Variable      Good         Excellent
RTO (recovery time)  Hours         1-2 min      <1 min
```

### Real-World Decision Tree

```
Choosing database engine:

Need enterprise features (Oracle/SQL Server)?
├─ YES → Use commercial (expensive)
└─ NO → Continue

Need best performance & scalability?
├─ YES → Aurora (best choice)
└─ NO → Continue

Need free, open-source?
├─ YES → MySQL, PostgreSQL, MariaDB
└─ NO → Continue

Need compatibility with existing MySQL?
└─ YES → MySQL or MariaDB
```

### Validation Checklist
- [ ] Understand RDS is managed service
- [ ] Know difference between engines
- [ ] Understand Single-AZ vs Multi-AZ
- [ ] Know RTO/RPO concepts
- [ ] Understand cost differences

### Key Takeaway
✅ RDS = Managed relational database  
✅ AWS handles ops, you manage SQL  
✅ Multi-AZ = High availability  
✅ Aurora = Best performance  

---

### Lab 2: Creating Your First RDS Instance

**Objective:** Launch an RDS MySQL instance.

**Why This Matters:**
- Foundation for all labs
- Understand instance configuration
- Learn parameter selection
- Test connectivity

### Step-By-Step Instructions

**Step 1: Navigate to RDS Console**
- Open AWS Console
- Search for "RDS"
- Click "RDS" service
- Click "Databases"
- Click "Create Database"

**Step 2: Choose Engine**
- Engine type: MySQL
- Version: MySQL 8.0.35 (latest stable)
- Click "Next"

**Step 3: Choose Use Case**
- Templates: Production (or Dev/Test)
- For now: Dev/Test
- (Simpler configuration)

**Step 4: Configure DB Instance**

Basic Settings:
- DB Instance Identifier: `my-first-rds-db`
- Master Username: `admin`
- Master Password: Create strong password
  - Min 8 chars, include upper, lower, numbers, special
  - Example: `MyPass123!`
  - Click "Generate password" for auto-generation
- Confirm password

**Step 5: Choose Instance Class**
- Instance class: db.t3.micro (FREE tier eligible!)
  - 1 vCPU
  - 1 GB RAM
  - 5 Gbps network
- Storage: GP3 (General Purpose)
- Allocated storage: 20 GB (free tier: up to 20 GB)
- Storage autoscaling: Enable
  - Max: 100 GB
  - Grows automatically as needed

**Step 6: Configure Network & Security**
- VPC: Default VPC (or select specific)
- Public accessibility: Yes (for testing, restrict later)
  - WARNING: Only for testing!
  - Production: NO
- Security group: Create new
  - Name: `rds-mysql-sg`
  - Inbound rule added: MySQL port 3306

**Step 7: Database Authentication**
- Password authentication: YES
- IAM authentication: Enable (for app access)

**Step 8: Automated Backups**
- Backup retention period: 7 days (min for production)
- Backup window: Automatic (3am UTC)
- Copy backups to another region: Enable (optional)
  - For Multi-AZ

**Step 9: Multi-AZ Deployment**
- Multi-AZ: NO (for now, testing)
  - In production: YES
  - Creates standby instance
  - Cost: 2x
- Enable automated failover: YES (if Multi-AZ)

**Step 10: Monitoring**
- Enable Performance Insights: YES
- Log exports: Check all:
  - Error log
  - General log
  - Slow query log
  - Audit log (if enabled)
- Enable Enhanced Monitoring: NO (for now)

**Step 11: Maintenance**
- Minor version upgrade: Enable automatic
  - Keep DB patched
- Maintenance window: Automatic
  - During backup window

**Step 12: Database Options**
- Initial database name: `myappdb`
- Database port: 3306 (default)
- Parameter group: Default (or create custom)
- Option group: Default

**Step 13: Review & Create**

Summary shows:
- Instance: db.t3.micro
- Storage: 20 GB GP3
- Backup: 7 days
- Multi-AZ: NO
- Monitoring: Enhanced
- Cost: ~$10-15/month

Click "Create Database"

**Step 14: Wait for Creation**

Status: Creating → Available (5-10 minutes)

You can monitor progress:
- Go to Databases
- Click your instance
- See "Creating" status
- Wait for "Available" ✅

### What Was Created

```
RDS Instance Created:
├─ Instance ID: my-first-rds-db
├─ Engine: MySQL 8.0
├─ Size: db.t3.micro
├─ Storage: 20 GB
├─ Endpoint: my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com
├─ Port: 3306
├─ Username: admin
├─ Password: Your password
└─ Database: myappdb

Ready to connect! ✅
```

### Automatic Backups

```
Daily automated backups:
Day 1: Snapshot 1
Day 2: Snapshot 2
...
Day 7: Snapshot 7
Day 8: Snapshot 1 deleted (retention = 7 days)

You can restore to ANY point in last 7 days!
Backup includes: All data, schema, users
```

### Validation Checklist
- [ ] Instance created and status "Available"
- [ ] Can see endpoint in instance details
- [ ] Note username: admin
- [ ] Note password (saved securely)
- [ ] Note database name: myappdb
- [ ] Backup retention: 7 days
- [ ] Understand instance cost

### Key Takeaway
✅ RDS instance created successfully  
✅ Multi-AZ for production (not enabled here)  
✅ Automated backups running  
✅ Ready for database operations  

---

### Lab 3: Database Subnet Groups & Parameter Groups

**Objective:** Understand VPC networking and database configuration.

**Why This Matters:**
- Subnet groups control network placement
- Parameter groups control DB behavior
- Essential for production setup
- Security and performance impact

### Understanding Subnet Groups

```
RDS Subnet Group = Network configuration

VPC contains:
├─ Public Subnets (internet accessible)
└─ Private Subnets (only internal access)

RDS Instance placement:
├─ Must be in private subnet (secure)
├─ Needs access to 2+ AZs (for Multi-AZ)
└─ Defined by Subnet Group
```

### Step-By-Step Instructions

**Step 1: Create Custom Subnet Group**

- Go to RDS Console
- Subnet Groups (left menu)
- Click "Create DB Subnet Group"

**Step 2: Configure Subnet Group**

Basic Details:
- Name: `rds-subnet-group`
- Description: "Subnet group for RDS instances"
- VPC: Default VPC

Add Subnets:
- Availability Zones: us-east-1a, us-east-1b
- Subnets: Select 2 subnets (different AZs)
  - If not visible, need to create subnets first
  - For default VPC: Should auto-appear
- Click "Create"

**Step 3: View Subnet Group**

Shows:
- Name: rds-subnet-group
- VPC: vpc-xxx
- Subnets: 2 subnets in 2 AZs
- Status: Complete

This controls where RDS instances launch!

**Step 4: Create Custom Parameter Group**

Parameter Group = Database configuration

- Go to RDS Console
- Parameter Groups (left menu)
- Click "Create Parameter Group"

**Step 5: Configure Parameter Group**

Basic Details:
- Family: mysql8.0
- Group Name: `custom-mysql-params`
- Description: "Custom parameters for optimization"
- Click "Create"

**Step 6: Edit Parameters**

Common parameters to modify:

```
Parameter              Default    Recommendation  Purpose
──────────────────────────────────────────────────────────
max_connections        151        Based on app    Connection limit
innodb_buffer_pool_pct 75         75-80           Cache percentage
slow_query_log         0          1               Log slow queries
long_query_time        10         1-2             What is "slow"
log_queries_not_using  0          1               Log inefficient
transaction_isolation  REPEATABLE READ           Consistency level
```

Modify parameters:
- Click on parameter group
- Click "Edit"
- Change parameters (example: max_connections)
  - max_connections: 200 (from 151)
- Click "Save"
- Status: "Modifying" → "Available" (immediate or with restart)

**Step 7: Apply Parameter Group to Instance**

Modify your RDS instance:
- Go to Databases
- Click your instance
- Modify
- DB parameter group: custom-mysql-params
- Scheduling:
  - Apply immediately (if non-production)
  - OR Schedule during maintenance window
- Click "Modify"

Instance restarts if parameter change requires it.

**Step 8: View Applied Parameters**

- Click instance
- Configuration tab
- Parameter group: Shows custom-mysql-params ✓
- Shows all active parameters

### Real-World Parameter Tuning

```
Web Application Defaults:
├─ max_connections: 200-500
├─ innodb_buffer_pool_size: 50% of RAM
├─ slow_query_log: 1 (for analysis)
├─ long_query_time: 0.5 seconds
└─ query_cache_type: 1 (if MySQL < 8.0)

Data Warehouse Defaults:
├─ max_connections: 50-100 (fewer, longer queries)
├─ innodb_buffer_pool_size: 75% of RAM
├─ sort_buffer_size: Increased
├─ read_rnd_buffer_size: Increased
└─ tmp_table_size: Larger (for joins)
```

### Option Groups

Similar to Parameter Groups but for optional features:
- Backtrack (MySQL only)
- Transparent Data Encryption (TDE)
- Performance Insights
- etc.

Most default to "Default" option group.

### Validation Checklist
- [ ] Subnet group created
- [ ] Contains 2+ subnets in different AZs
- [ ] Parameter group created
- [ ] Parameters modified (example: max_connections)
- [ ] Applied to RDS instance
- [ ] Instance shows custom parameter group
- [ ] Understand parameter impact

### Key Takeaway
✅ Subnet groups control network placement  
✅ Parameter groups control DB behavior  
✅ Changes may require restart  
✅ Critical for optimization  

---

## SECTION 2: CONNECTIVITY & SECURITY (Labs 4-6)

### Lab 4: Connecting to RDS Instance (MySQL Client)

**Objective:** Connect to RDS and execute SQL queries.

**Why This Matters:**
- Verify database is accessible
- Test connectivity from different sources
- Understand security implications
- First practical SQL interaction

### Step-By-Step Instructions

**Step 1: Get Connection Details**

- Go to RDS Dashboard
- Click your instance
- Connectivity & Security tab
- Note:
  - Endpoint: `my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com`
  - Port: `3306`
  - Username: `admin`
  - Database: `myappdb`

**Step 2: Install MySQL Client (Your Computer)**

```bash
# Mac
brew install mysql-client

# Ubuntu/Debian
sudo apt-get install mysql-client

# Windows
# Download from: https://dev.mysql.com/downloads/mysql/
# Or use: choco install mysql

# Verify installation
mysql --version
# Shows: mysql Ver 8.0.xx for Linux
```

**Step 3: Connect to RDS**

```bash
# Basic connection
mysql -h my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       -P 3306 \
       -u admin \
       -p

# After command, enter password when prompted
# You should see: mysql> prompt

# Alternative (password in command, not recommended for security)
mysql -h my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       -u admin \
       --password='MyPass123!' \
       -D myappdb
```

**Step 4: Test Connection**

Once connected (mysql> prompt):

```sql
-- Show databases
SHOW DATABASES;
-- Shows: information_schema, mysql, myappdb, performance_schema

-- Select current database
USE myappdb;
-- Shows: Database changed

-- Create test table
CREATE TABLE employees (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100),
  department VARCHAR(50),
  salary DECIMAL(10,2),
  hire_date DATE
);
-- Shows: Query OK, 0 rows affected

-- Insert test data
INSERT INTO employees (name, email, department, salary, hire_date) VALUES
('Alice Johnson', 'alice@company.com', 'Engineering', 95000.00, '2023-01-15'),
('Bob Smith', 'bob@company.com', 'Sales', 75000.00, '2023-02-20'),
('Charlie Brown', 'charlie@company.com', 'Marketing', 70000.00, '2023-03-10'),
('Diana Prince', 'diana@company.com', 'Engineering', 105000.00, '2022-06-05');
-- Shows: Query OK, 4 rows affected

-- Query data
SELECT * FROM employees;
-- Shows all employees

SELECT COUNT(*) as employee_count FROM employees;
-- Shows: 4

SELECT * FROM employees WHERE department = 'Engineering';
-- Shows: 2 engineers

SELECT department, AVG(salary) as avg_salary FROM employees GROUP BY department;
-- Shows avg salary by department

**Step 3: Create DMS Service Role**

```bash
# Create IAM role for DMS
aws iam create-role \
  --role-name dms-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "dms.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach necessary policies
aws iam attach-role-policy \
  --role-name dms-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonDMSVPCManagementRole

aws iam attach-role-policy \
  --role-name dms-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonDMSCloudWatchLogsRole

aws iam attach-role-policy \
  --role-name dms-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

**Step 4: Create DMS Replication Instance**

- Go to DMS Console
- Replication Instances
- Click "Create Replication Instance"

Configuration:
- Name: `dms-replication-instance`
- Instance class: dms.t3.micro
- Engine: DMS (default)
- VPC: Default VPC
- Multi-AZ: No (for testing)
- Storage: 20 GB
- Click "Create"

Status: Creating → Available (5-10 minutes)

**Step 5: Create Source Endpoint**

- Go to DMS Console
- Endpoints
- Click "Create Endpoint"
- Endpoint type: Source

For SQL Server (on-premises):
- Engine: SQL Server
- Server: Your SQL Server IP/hostname
- Port: 1433
- Username: sa (or user)
- Password: Your password
- Database: SourceDB
- Click "Test"

Shows: Connection successful ✅

For source RDS instance:
- Engine: MySQL
- Endpoint address: my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com
- Port: 3306
- Username: admin
- Password: Your password
- Database: myappdb
- Click "Test"

**Step 6: Create Target Endpoint**

- Click "Create Endpoint"
- Endpoint type: Target

Configuration:
- Engine: MySQL (or PostgreSQL)
- Endpoint address: Target RDS endpoint
- Port: 3306
- Username: admin
- Password: Your password
- Database: targetdb
- Click "Test"

Shows: Connection successful ✅

**Step 7: Create Database Migration Task**

- Go to Tasks
- Click "Create Task"

Configuration:
- Task Name: `migrate-source-to-target`
- Replication Instance: dms-replication-instance
- Source: Your source endpoint
- Target: Your target endpoint
- Migration Type: Full load and CDC
  - Full load: Copy all data
  - CDC: Continuous replication
  - Optional: Migrate schema first

Task Settings:
- Enable CloudWatch logs: Yes
- Table mappings:
```json
{
  "rules": [
    {
      "rule-type": "selection",
      "rule-id": "1",
      "rule-name": "IncludeAllTables",
      "object-locator": {
        "schema-name": "%",
        "table-name": "%"
      },
      "rule-action": "include"
    }
  ]
}
```

- Click "Create Task"

**Step 8: Monitor Migration**

Status: Creating → Running (Full load) → CDC (Continuous replication)

Shows:
- Tables selected: N
- Rows transferred: X,XXX
- Errors: 0
- CDC Status: Enabled
- Replication lag: Seconds

**Step 9: Verify Migration**

Once full load complete:

```bash
# Connect to target database
mysql -h target-rds-endpoint.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       -u admin \
       -p \
       -D targetdb

-- Check tables
SHOW TABLES;
-- Shows: Migrated tables ✅

-- Check data
SELECT * FROM Customers;
-- Shows: All customers from source ✅

-- Count rows
SELECT COUNT(*) FROM Customers;
-- Shows: Same count as source ✅
```

**Step 10: Cutover (Switch to Target)**

When ready to switch:

1. Stop application writes to source
2. Wait for CDC to complete (replication lag = 0)
3. Verify all data matches
4. Update application connection string:
   - FROM: Source RDS endpoint
   - TO: Target RDS endpoint
5. Restart application
6. Verify application works
7. Decommission source (after waiting period)

### Migration Best Practices

```
Before Migration:
├─ Test on non-prod first
├─ Document schema differences
├─ Plan cutover window
├─ Notify stakeholders
└─ Have rollback plan

During Migration:
├─ Monitor replication lag
├─ Verify row counts
├─ Check data integrity
├─ Monitor application performance
└─ Have DBA on standby

After Migration:
├─ Run verification queries
├─ Performance testing
├─ Update documentation
├─ Archive old system
└─ Monitor for 30 days
```

### Real-World Migration Timeline

```
Monday 2:00 PM: Start DMS full load
├─ 100 GB database = 1-2 hours

Monday 4:00 PM: Full load complete
├─ 100M rows transferred
├─ CDC replication started

Monday-Friday: CDC runs
├─ Any changes at source replicated
├─ Replication lag: < 1 second

Friday 11:00 PM: Cutover window starts
├─ Stop application writes
├─ Verify lag = 0
├─ Data matches perfectly

Friday 11:15 PM: Switch application
├─ Update connection string
├─ Restart application
├─ Verify working

Friday 11:30 PM: Success!
├─ Application running on target
├─ Zero data loss
├─ Minimal downtime (30 min)

Cost:
├─ DMS instance: ~$11/month
├─ Source RDS: ~$22/month (Multi-AZ)
├─ Target RDS: ~$22/month (Multi-AZ)
├─ Data transfer: ~$1/GB (100GB = $100)
└─ Total: ~$155 migration cost
```

### Validation Checklist
- [ ] DMS replication instance created
- [ ] Source endpoint tested
- [ ] Target endpoint tested
- [ ] Migration task created
- [ ] Full load completed
- [ ] CDC running
- [ ] Data verified in target
- [ ] Row counts match

### Key Takeaway
✅ DMS simplifies database migration  
✅ Minimal downtime possible  
✅ Zero data loss with CDC  
✅ AWS handles complexity  

---

### Lab 12: Parameter Tuning for SQL Server Workloads

**Objective:** Optimize RDS for SQL Server specific workloads.

**Note:** If you created MySQL instance, this lab covers SQL Server parameters.
In real world, would apply same principles to your engine.

### SQL Server Specific Parameters

```
SQL Server RDS Parameters:

Memory:
├─ max server memory: 80% of instance RAM
├─ index create memory: 4096 MB
└─ tempdb size: 100 GB (for large workloads)

Parallel Processing:
├─ max degree of parallelism: 4-8
├─ query optimizer memory: Automatic
└─ max worker threads: CPU dependent

Transaction Log:
├─ recovery model: FULL (for PITR)
├─ transaction log retention: 24 hours
└─ checkpoint frequency: Automatic

Network:
├─ network packet size: 4096
├─ remote admin connections: 0 (disable)
└─ default network protocol: TCP/IP

Compatibility:
├─ database compatibility level: Latest
└─ default language: English
```

### Tuning Steps

**Step 1: Identify Performance Issues**

```sql
-- Check wait statistics (bottlenecks)
SELECT TOP 20
  wait_type,
  waiting_tasks_count,
  wait_time_ms,
  signal_wait_time_ms
FROM sys.dm_os_wait_stats
ORDER BY wait_time_ms DESC;

-- Check missing indexes
SELECT
  CONVERT(DECIMAL(18,2), user_seeks * avg_total_user_cost * (avg_user_impact * 0.01)) AS improvement_measure,
  mid.equality_columns,
  mid.inequality_columns
FROM sys.dm_db_missing_index_details mid
INNER JOIN sys.dm_db_missing_index_groups_stats migs
  ON mid.index_handle = migs.index_handle
ORDER BY improvement_measure DESC;

-- Check slow queries
SELECT TOP 10
  qs.total_elapsed_time,
  qs.execution_count,
  qs.last_execution_time,
  SUBSTRING(st.text, 1, 200) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY qs.total_elapsed_time DESC;
```

**Step 2: Adjust Memory Settings**

```sql
-- Check current settings
EXEC sp_configure 'max server memory';

-- Set max server memory (80% of instance)
-- For db.m5.large (8 GB RAM): 6400 MB
EXEC sp_configure 'max server memory', 6400;
RECONFIGURE;

-- Verify
EXEC sp_configure 'max server memory';
```

**Step 3: Tune Parallel Execution**

```sql
-- Set max degree of parallelism
EXEC sp_configure 'max degree of parallelism', 4;
RECONFIGURE;

-- Verify
EXEC sp_configure 'max degree of parallelism';
```

**Step 4: Create Missing Indexes**

```sql
-- Create recommended indexes
CREATE NONCLUSTERED INDEX idx_column_name
ON table_name (column_name)
WHERE condition;

-- Example
CREATE NONCLUSTERED INDEX idx_email_created
ON Customers (Email, CreatedDate);

-- Check index usage
SELECT
  OBJECT_NAME(ius.object_id) AS table_name,
  i.name AS index_name,
  ius.user_seeks,
  ius.user_scans,
  ius.user_lookups,
  ius.user_updates
FROM sys.dm_db_index_usage_stats ius
INNER JOIN sys.indexes i
  ON ius.object_id = i.object_id
  AND ius.index_id = i.index_id
ORDER BY ius.user_seeks + ius.user_scans DESC;
```

### Validation Checklist
- [ ] Identified performance issues with queries
- [ ] Reviewed wait statistics
- [ ] Found missing indexes
- [ ] Tuned memory parameters
- [ ] Created missing indexes
- [ ] Measured performance improvement

### Key Takeaway
✅ SQL Server tuning critical  
✅ Wait stats show bottlenecks  
✅ Missing indexes have big impact  
✅ Parameter tuning improves throughput  

---

## COMPLETE CLEANUP PROCEDURE

### Full RDS Cleanup

**Step 1: Delete RDS Proxy**

```bash
aws rds delete-db-proxy \
  --db-proxy-name my-rds-proxy \
  --force

# Wait for deletion
aws rds wait db-proxy-deleted --db-proxy-name my-rds-proxy
```

**Step 2: Delete Read Replicas**

```bash
# Delete each replica
aws rds delete-db-instance \
  --db-instance-identifier my-first-rds-read-replica-1 \
  --skip-final-snapshot

aws rds delete-db-instance \
  --db-instance-identifier my-first-rds-read-replica-2 \
  --skip-final-snapshot

# Wait for deletion
aws rds wait db-instance-deleted \
  --db-instance-identifier my-first-rds-read-replica-1
```

**Step 3: Delete Restored Instance**

```bash
aws rds delete-db-instance \
  --db-instance-identifier my-first-rds-db-restored \
  --skip-final-snapshot

# Wait
aws rds wait db-instance-deleted \
  --db-instance-identifier my-first-rds-db-restored
```

**Step 4: Delete Primary Instance**

```bash
# Final snapshot (optional, for safety)
aws rds delete-db-instance \
  --db-instance-identifier my-first-rds-db \
  --final-db-snapshot-identifier my-rds-final-backup

# Or without snapshot (if testing only)
aws rds delete-db-instance \
  --db-instance-identifier my-first-rds-db \
  --skip-final-snapshot

# Wait
aws rds wait db-instance-deleted \
  --db-instance-identifier my-first-rds-db
```

**Step 5: Delete Snapshots**

```bash
# List all snapshots
aws rds describe-db-snapshots \
  --query 'DBSnapshots[].DBSnapshotIdentifier'

# Delete manual snapshots
aws rds delete-db-snapshot \
  --db-snapshot-identifier my-snapshot-backup-2024-01-15

# Automated snapshots auto-delete based on retention
```

**Step 6: Delete Parameter Groups**

```bash
aws rds delete-db-parameter-group \
  --db-parameter-group-name custom-mysql-params
```

**Step 7: Delete Subnet Groups**

```bash
aws rds delete-db-subnet-group \
  --db-subnet-group-name rds-subnet-group
```

**Step 8: Verify Cleanup**

```bash
# Check no databases remain
aws rds describe-db-instances --query 'DBInstances[*].DBInstanceIdentifier'
# Should be empty

# Check no proxies
aws rds describe-db-proxies
# Should be empty

# Verify billing
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity DAILY \
  --metrics BlendedCost \
  --filter file://filter.json
```

### Cost Summary

```
RDS Costs During Labs:

Single-AZ Instance:
├─ db.t3.micro: $0.015/hour = $11/month
├─ Storage 20GB: $0.46/month
└─ Total: ~$11.50/month

Multi-AZ Deployment:
├─ Instances: $22/month
├─ Storage: $0.46/month
└─ Total: ~$22.50/month

With Read Replicas (3 total):
├─ Instances: $33/month
├─ Storage: $1.40/month
└─ Total: ~$34.50/month

With RDS Proxy:
├─ Proxy: $11/month
├─ Total: ~$45.50/month

Snapshots:
├─ Automated: Included
├─ Manual snapshots: ~$0.023/GB/month
└─ 1GB snapshot: $0.02/month

Total for all labs: ~$50-60/month

BUT: Within free tier for first year!
```

### Cleanup Checklist

- [ ] RDS Proxy deleted
- [ ] All read replicas deleted
- [ ] Restored instance deleted
- [ ] Primary instance deleted
- [ ] Manual snapshots deleted
- [ ] Parameter groups deleted
- [ ] Subnet groups deleted
- [ ] No RDS resources remaining
- [ ] AWS Billing shows $0 RDS charges

---

## FINAL SUMMARY

### What You've Learned (18 Labs)

**Labs 1-3: Fundamentals**
- ✅ RDS architecture and service offerings
- ✅ Creating RDS instances
- ✅ Subnet groups and parameter groups

**Labs 4-6: Connectivity & Security**
- ✅ Connecting to RDS instances
- ✅ Security groups and network access
- ✅ Parameter tuning and query optimization

**Labs 7-10: Backup & High Availability**
- ✅ Snapshots and point-in-time recovery
- ✅ Multi-AZ deployments
- ✅ Read replicas for scaling
- ✅ RDS Proxy for connection pooling

**Labs 11-12: Migration & Advanced**
- ✅ Database migration with DMS
- ✅ SQL Server specific tuning
- ✅ Performance optimization strategies

---

## AWS RDS Best Practices

### Security
✅ Use Multi-AZ for production
✅ Enable encryption at rest
✅ Use SSL/TLS for connections
✅ Implement IAM authentication
✅ Restrict security group access
✅ Create database-level users
✅ Regular security patches
✅ Enable automated backups

### Performance
✅ Tune parameters for workload
✅ Create indexes for slow queries
✅ Monitor slow query log
✅ Use read replicas for scaling
✅ Implement RDS Proxy
✅ Monitor CloudWatch metrics
✅ Right-size instance type
✅ Use Performance Insights

### Cost Optimization
✅ Right-size instances
✅ Use Auto Scaling when available
✅ Delete unused read replicas
✅ Schedule backups properly
✅ Clean up old snapshots
✅ Use spot instances (where possible)
✅ Monitor reserved instances
✅ Consolidate databases

### Disaster Recovery
✅ Enable Multi-AZ
✅ Automated backups (7+ days)
✅ Manual snapshots before changes
✅ Cross-region replicas
✅ Test restore procedures
✅ Document RPO/RTO
✅ Regular backup verification
✅ Disaster recovery drills

### Monitoring
✅ CloudWatch dashboards
✅ CPU/Memory alerts
✅ Connection count monitoring
✅ Storage space alerts
✅ Replication lag monitoring
✅ Query performance analysis
✅ Database activity logs
✅ Automatic failover testing

---

## Real-World RDS Architecture

```
High Availability Production Setup:

Primary Region (us-east-1):
├─ RDS Instance (Primary, Multi-AZ)
│  ├─ Availability Zone 1a
│  │  └─ Primary instance (write)
│  ├─ Availability Zone 1b
│  │  └─ Standby instance (automatic failover)
│  └─ Read Replicas:
│     ├─ Replica 1 (same region, same AZ)
│     ├─ Replica 2 (same region, different AZ)
│     └─ Replica 3 (different region for DR)
├─ RDS Proxy
│  └─ Connection pooling for applications
└─ Automated Backups
   └─ 35 day retention (daily snapshots)

Secondary Region (eu-west-1):
├─ Cross-region Read Replica
│  └─ For disaster recovery
└─ Promoted to Primary (if region fails)

Monitoring:
├─ CloudWatch dashboards
├─ SNS alerts
├─ Database logs
└─ Slow query analysis

Cost:
├─ Primary instance: ~$22/month (Multi-AZ)
├─ 3 Read replicas: ~$33/month
├─ RDS Proxy: ~$11/month
├─ Backups: ~$5/month
├─ Data transfer: ~$20/month
└─ Total: ~$91/month for production-grade setup

For mission-critical databases:
└─ Cost is insurance against data loss
└─ Worth every penny! ✅
```

---

## Congratulations! 🎉

You've completed 18 comprehensive RDS labs covering:
- Database fundamentals
- Instance creation & management
- Security & access control
- Backup & disaster recovery
- High availability & failover
- Read replicas & scaling
- Connection pooling
- Database migration
- Performance optimization
- Real-world architectures

You now have **enterprise-level RDS knowledge**!

### Next Steps

1. **Implement RDS in production:**
   - Use Multi-AZ
   - Enable automated backups
   - Implement read replicas
   - Set up monitoring

2. **Migrate on-premises databases:**
   - Use AWS DMS
   - Minimal downtime
   - Zero data loss

3. **Optimize costs:**
   - Right-size instances
   - Use read replicas efficiently
   - Clean up old snapshots

4. **Stay secure:**
   - Keep patches current
   - Monitor security groups
   - Use IAM authentication
   - Enable encryption

5. **Ensure reliability:**
   - Test failover regularly
   - Verify backup restoration
   - Monitor metrics continuously
   - Plan for disasters

---

**Your RDS journey starts now! Happy database management! 🚀**
-- Shows table sizes
```

**Step 5: Test Connection from EC2 (If Available)**

Create EC2 instance in same VPC:
```bash
# On EC2 instance
sudo yum install mysql -y

# Connect to RDS
mysql -h my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       -u admin \
       --password='MyPass123!' \
       -D myappdb

# Should work (internal network) ✅
```

**Step 6: Connection Pooling (Optional)**

For applications:
```bash
# Use ProxySQL for connection pooling
# Or: Use RDS Proxy (Lab 10)

# Without pooling:
# Each request = New connection
# = Overhead, slow

# With pooling:
# Reuse connections
# = Fast, efficient
```

**Step 7: Disconnect**

```sql
-- Exit MySQL
EXIT;
-- Or: QUIT;
-- Returns to bash prompt
```

### Connection Troubleshooting

```
Problem: Connection refused

Check:
1. RDS instance is "Available"
   aws rds describe-db-instances --query 'DBInstances[0].DBInstanceStatus'

2. Security group allows port 3306
   aws ec2 describe-security-groups --group-ids sg-xxx --query 'SecurityGroups[0].IpPermissions'
   Must show: Port 3306, Protocol: tcp

3. Network connectivity
   ping endpoint (won't work for RDS)
   telnet endpoint 3306 (check port reachable)
   nc -zv endpoint 3306

4. Credentials correct
   Username: admin (what you set)
   Password: What you set
   Database: myappdb (what you specified)

Fix:
├─ If instance down: Start it
├─ If SG wrong: Modify inbound rule
├─ If password wrong: Modify in RDS console
└─ If network issue: Check VPC/subnet
```

### Validation Checklist
- [ ] MySQL client installed
- [ ] Can connect to RDS
- [ ] Can run SELECT queries
- [ ] Created test table
- [ ] Inserted test data
- [ ] Can query data
- [ ] Understand connection string

### Key Takeaway
✅ RDS connectivity works  
✅ Can execute SQL queries  
✅ Database operational  
✅ Ready for next labs  

---

### Lab 5: Security Groups & Network Access Control

**Objective:** Control who can access RDS instance.

**Why This Matters:**
- RDS must be protected
- Only specific sources should access
- Network segmentation
- Prevent unauthorized access

### Understanding RDS Security

```
RDS Security Layers:

1. Network Level (Security Groups)
   ├─ Inbound: Only port 3306 from specific IPs
   ├─ Outbound: Default allow
   └─ First checkpoint: No access = Can't connect

2. Database Level (Users & Passwords)
   ├─ Username: admin
   ├─ Password: Strong password
   └─ Second checkpoint: Wrong password = Can't authenticate

3. Application Level (Query permissions)
   ├─ Per-table permissions
   ├─ Per-schema permissions
   └─ Third checkpoint: No permissions = Can't modify
```

### Step-By-Step Instructions

**Step 1: View Current Security Group**

- Go to RDS instance
- Connectivity & security tab
- VPC security groups: Shows current (probably sg-default)
- Click security group

Shows:
- Inbound rules: Current settings
- Outbound rules: Usually allow all

**Step 2: Modify Security Group (Restrict Access)**

Current setup (insecure for production):
- Inbound: 3306 from 0.0.0.0/0 (anyone!)

Change to (production):
- Go to Security Groups
- Select RDS security group
- Inbound rules → Edit
- Remove: 3306 from 0.0.0.0/0
- Add: 3306 from specific sources:
  - From: Your office IP: 203.0.113.45/32
  - From: EC2 security group: sg-web-servers
  - From: Application server IP: 10.0.1.50/32

Save

**Step 3: Test Restricted Access**

From allowed IP:
```bash
mysql -h endpoint -u admin -p
# Works! ✅
```

From blocked IP:
```bash
mysql -h endpoint -u admin -p
# Error: Can't connect (timeout/refused)
# This is correct! ✅
```

**Step 4: Create Database-Level Users**

Instead of using `admin` for everything:

```sql
-- Connect as admin
mysql -h endpoint -u admin -p

-- Create application user
CREATE USER 'appuser'@'10.0.1.%' IDENTIFIED BY 'AppPass123!';

-- Create read-only user
CREATE USER 'readonly'@'10.0.1.%' IDENTIFIED BY 'ReadPass123!';

-- Create backup user
CREATE USER 'backup'@'10.0.0.0/255.255.0.0' IDENTIFIED BY 'BackupPass123!';

-- Grant permissions to appuser
GRANT SELECT, INSERT, UPDATE, DELETE ON myappdb.* TO 'appuser'@'10.0.1.%';

-- Grant read-only permissions
GRANT SELECT ON myappdb.* TO 'readonly'@'10.0.1.%';

-- Grant backup permissions
GRANT SELECT, LOCK TABLES ON myappdb.* TO 'backup'@'10.0.0.0/255.255.0.0';

-- Apply permissions
FLUSH PRIVILEGES;

-- Show users
SELECT user, host FROM mysql.user;

-- Verify appuser permissions
SHOW GRANTS FOR 'appuser'@'10.0.1.%';
```

**Step 5: Test User Access**

Connect as appuser:
```bash
mysql -h endpoint -u appuser -p -D myappdb

# Can run SELECT, INSERT, UPDATE, DELETE
INSERT INTO employees (name, email, department) VALUES ('Eve Davis', 'eve@company.com', 'HR');
-- Query OK ✅

SELECT * FROM employees;
-- Works ✅

DROP TABLE employees;
-- Error: Access denied (not allowed) ✅
```

Connect as readonly:
```bash
mysql -h endpoint -u readonly -p -D myappdb

SELECT * FROM employees;
-- Works ✅

INSERT INTO employees (name, email) VALUES ('Frank', 'frank@company.com');
-- Error: Access denied (read-only) ✅

UPDATE employees SET salary = 100000;
-- Error: Access denied ✅
```

**Step 6: Enable Encryption in Transit**

SSL/TLS connection:

```bash
# Download RDS certificate
wget https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# Connect with SSL
mysql -h endpoint \
       -u admin \
       -p \
       --ssl-ca=global-bundle.pem \
       --ssl-mode=REQUIRED

# Connection now encrypted! ✅
```

**Step 7: Enable Encryption at Rest**

- Go to RDS instance
- Modify
- Storage encryption: Enable (if not already)
  - Note: Must restart
- Apply immediately (if testing) or schedule
- Instance restarts with encryption enabled

### IAM Authentication (Advanced)

Instead of passwords:

```bash
# Generate temporary auth token
TOKEN=$(aws rds generate-db-auth-token \
  --hostname my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
  --port 3306 \
  --region us-east-1 \
  --username iamuser)

# Connect using token (valid 15 minutes)
mysql -h my-first-rds-db.cxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
       --port=3306 \
       -u iamuser \
       --password="$TOKEN" \
       --enable-cleartext-plugin \
       --ssl-ca=rds-ca-bundle.pem \
       --ssl-mode=REQUIRED

# Benefits:
# ├─ No password stored
# ├─ Token expires (15 min)
# ├─ Audit via IAM
# └─ Better for apps
```

### Validation Checklist
- [ ] Security group modified (restricted access)
- [ ] Created application users (appuser, readonly)
- [ ] Tested user access permissions
- [ ] Users can't exceed their permissions
- [ ] SSL/TLS enabled
- [ ] Encryption at rest enabled
- [ ] Understand IAM authentication

### Key Takeaway
✅ Security groups restrict network access  
✅ Database users control SQL permissions  
✅ Encryption protects data  
✅ Layered security approach  

---

### Lab 6: Parameter Groups Deep Dive & Query Optimization

**Objective:** Optimize database performance through parameter tuning.

**Why This Matters:**
- Database performance critical
- Parameters affect throughput
- Wrong settings = slow queries
- Proper tuning = 10x+ improvement

### Understanding Key Parameters

```
Connection Parameters:
├─ max_connections: Max simultaneous connections
├─ interactive_timeout: Idle connection timeout
└─ wait_timeout: How long to wait

Memory Parameters:
├─ innodb_buffer_pool_size: Most important!
├─ tmp_table_size: Temp table size
└─ sort_buffer_size: Sort operations

Query Parameters:
├─ long_query_time: Log queries slower than this
├─ slow_query_log: Enable slow query logging
└─ max_allowed_packet: Max query size

Performance Parameters:
├─ query_cache_type: Enable query cache
├─ innodb_flush_log_at_trx_commit: Durability vs speed
└─ innodb_log_file_size: Transaction log size
```

### Step-By-Step Instructions

**Step 1: Calculate Optimal Buffer Pool Size**

For db.t3.micro (1 GB RAM):
```
Total RAM: 1 GB
Reserved for OS/Connections: 256 MB
Available for cache: 750 MB

innodb_buffer_pool_size = 750 MB = 786432000 bytes
```

For db.r6i.xlarge (32 GB RAM):
```
Total RAM: 32 GB
innodb_buffer_pool_size = 24 GB (75% of total)
```

**Step 2: Modify Parameters for Performance**

- Go to Parameter Groups
- Click custom-mysql-params
- Click "Edit"

Change these parameters:

```
Parameter                          New Value        Reason
────────────────────────────────────────────────────────
max_connections                    500              More concurrent users
innodb_buffer_pool_size            786432000        Cache frequently used data
innodb_buffer_pool_instances       4                Better concurrency
innodb_log_file_size               512M             Faster transactions
slow_query_log                     1                Enable logging
long_query_time                    1                Log queries > 1 second
log_queries_not_using_indexes      1                Find missing indexes
innodb_flush_log_at_trx_commit     1                Durability (default)
query_cache_type                   0                Disabled (MySQL 8.0)
tmp_table_size                     16M              Larger temp tables
sort_buffer_size                   2M               Better sort performance
```

Click "Modify" and apply (may require restart).

**Step 3: Enable Slow Query Log**

```bash
# Connect to RDS
mysql -h endpoint -u admin -p

-- View current slow query log setting
SHOW VARIABLES LIKE 'slow%';

-- The slow query log is now being written to:
-- /var/log/mysql/slowquery.log (on RDS)
```

**Step 4: Generate Slow Queries**

```sql
-- Create test data
CREATE TABLE large_table (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_email (email)
);

-- Insert 10,000 rows
INSERT INTO large_table (name, email) 
SELECT 
  CONCAT('User ', seq),
  CONCAT('user', seq, '@example.com')
FROM (
  SELECT @row := @row + 1 as seq
  FROM (SELECT 1 UNION SELECT 2 UNION SELECT 3) t1,
       (SELECT 1 UNION SELECT 2 UNION SELECT 3) t2,
       (SELECT 1 UNION SELECT 2 UNION SELECT 3) t3,
       (SELECT 1 UNION SELECT 2 UNION SELECT 3) t4,
       (SELECT @row := 0) r
) seq
LIMIT 10000;

-- Query WITHOUT index (slow)
SELECT * FROM large_table WHERE created_at < NOW() - INTERVAL 10 DAY;
-- This goes to slow query log!

-- Query WITH index (fast)
SELECT * FROM large_table WHERE email = 'user100@example.com';
-- This is fast (index used)
