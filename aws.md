# AWS Services: Comprehensive Guide 🚀

> Complete reference for AWS offerings, capabilities, limitations, complementary services, and real-world use cases

[![AWS](https://img.shields.io/badge/AWS-Cloud%20Services-FF9900?logo=amazon-aws)](https://aws.amazon.com/)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-Feb%202026-green.svg)](/)

## 📑 Quick Navigation

- [Compute](#-compute-services) | [Storage](#-storage-services) | [Database](#-database-services)
- [Networking](#-networking--content-delivery) | [Security](#-security-identity--compliance)
- [Analytics](#-analytics-services) | [ML/AI](#-machine-learning--ai)
- [Containers](#-containers--serverless) | [Developer Tools](#-developer-tools)

---

## 🖥️ Compute Services

### Amazon EC2 (Elastic Compute Cloud)

**What It Does:** Virtual servers in the cloud with complete control over the computing environment

**Key Capabilities:**
- 500+ instance types optimized for different workloads (compute, memory, storage, GPU)
- Multiple pricing models: On-Demand, Reserved (up to 72% discount), Spot (up to 90% discount), Savings Plans
- Auto Scaling for automatic capacity management
- Placement groups for low-latency networking or high availability
- Nitro System for enhanced security and performance

**Limitations:**
- ❌ Manual OS and application patching required
- ❌ Cold start time (minutes to launch instances)
- ❌ Pay for idle capacity
- ❌ Requires infrastructure management (networking, security groups)
- ❌ No built-in high availability (must configure yourself)

**Services That Fill the Gaps:**
- ✅ **AWS Systems Manager** → Automated patching and configuration
- ✅ **AWS Lambda** → Eliminates cold start and idle costs
- ✅ **Elastic Beanstalk** → Automated infrastructure provisioning
- ✅ **Auto Scaling + ALB** → Automatic high availability
- ✅ **EC2 Image Builder** → Automated AMI creation

**Real-World Use Case:**
**E-commerce Platform**
- Setup: t3.large for web tier (Auto Scaling 2-20 instances), c5.xlarge for app servers, r5.2xlarge for caching, Spot instances for batch processing
- Result: 60% cost reduction using Reserved Instances for baseline + Spot for traffic spikes
- Scale: Handles Black Friday traffic of 100K concurrent users

---

### AWS Lambda

**What It Does:** Run code without provisioning servers - serverless compute

**Key Capabilities:**
- Event-driven execution (200+ AWS service integrations)
- Automatic scaling from zero to thousands of concurrent executions
- Pay only for compute time (charged per 100ms)
- Supports 10+ languages (Python, Node.js, Java, Go, .NET, Ruby)
- 15-minute max execution time
- 10GB memory, 10GB ephemeral storage

**Limitations:**
- ❌ 15-minute timeout (not for long-running jobs)
- ❌ Cold start latency (50ms-5s depending on runtime)
- ❌ 10GB ephemeral storage limit
- ❌ Stateless (no persistent state between invocations)
- ❌ Vendor lock-in to AWS

**Services That Fill the Gaps:**
- ✅ **ECS Fargate** → Long-running tasks beyond 15 minutes
- ✅ **Lambda SnapStart** (Java) → 90% reduction in cold starts
- ✅ **Provisioned Concurrency** → Eliminates cold starts
- ✅ **EFS** → Persistent shared storage
- ✅ **Step Functions** → Orchestrate long workflows by chaining Lambdas

**Real-World Use Case:**
**Real-Time Image Processing**
- Architecture: S3 upload → Lambda → Generate 3 thumbnail sizes → DynamoDB metadata
- Volume: 10 million images/day, 99.9% processing within 2 seconds
- Cost: $450/month vs $2,500/month for equivalent EC2 setup

---

### Amazon ECS/EKS

**ECS - What It Does:** Fully managed container orchestration service

**Key Capabilities:**
- Launch types: EC2 (self-managed) or Fargate (serverless)
- Service auto-scaling based on metrics
- Integration with ALB/NLB for load balancing
- IAM roles at task level
- ECS Exec for debugging running containers
- Capacity Providers for mixed EC2/Spot/Fargate

**EKS - What It Does:** Managed Kubernetes control plane

**Key Capabilities:**
- 100% upstream Kubernetes compatibility
- Automatic HA control plane across 3 AZs
- Fargate support for serverless pods
- Integration with AWS services (IAM, VPC, ELB, EBS, EFS)
- GPU support for ML workloads
- IRSA (IAM Roles for Service Accounts)

**Limitations:**
- ❌ **ECS:** Less feature-rich than Kubernetes, AWS-specific
- ❌ **EKS:** $0.10/hour control plane cost ($73/month), steeper learning curve
- ❌ Both: Complex networking with awsvpc mode

**Services That Fill the Gaps:**
- ✅ **AWS App Mesh** → Service mesh for microservices
- ✅ **ECR** → Private container registry with vulnerability scanning
- ✅ **AWS Copilot** → Simplified ECS deployment CLI
- ✅ **Karpenter** (EKS) → Advanced node autoscaling

**Real-World Use Case:**
**Microservices E-commerce Platform**
- Architecture: 15 microservices on ECS Fargate, App Mesh for service communication, ALB with path-based routing
- Services: User, Product, Cart, Order, Payment, Inventory, Notification
- Scale: 50,000 requests/minute, auto-scales 5-100 tasks per service
- Advantage: Single load balancer for all services, isolated deployments

---

## 💾 Storage Services

### Amazon S3 (Simple Storage Service)

**What It Does:** Unlimited object storage with 99.999999999% durability

**Key Capabilities:**
- 8 storage classes for cost optimization (Standard, IA, Glacier, Deep Archive)
- Versioning for accidental deletion protection
- Lifecycle policies for automatic tier transitions
- Server-side encryption (SSE-S3, SSE-KMS, SSE-C)
- S3 Event Notifications (Lambda, SQS, SNS)
- Transfer Acceleration for fast global uploads
- Cross-Region Replication (CRR)

**Limitations:**
- ❌ Not a file system (object storage, not block/file)
- ❌ 5TB max object size
- ❌ No POSIX permissions
- ❌ Glacier retrieval: hours for Deep Archive
- ❌ High latency for small files (>10ms per request)

**Services That Fill the Gaps:**
- ✅ **Amazon EFS** → POSIX-compliant shared file system
- ✅ **Amazon FSx** → Windows file server or Lustre HPC
- ✅ **S3 Intelligent-Tiering** → Automatic cost optimization
- ✅ **CloudFront** → Low-latency content delivery
- ✅ **S3 Glacier Instant Retrieval** → Millisecond access for archive

**Real-World Use Cases:**

**1. Data Lake for Analytics**
- Volume: 500TB, 10TB added monthly
- Architecture: Raw data → S3 Standard → IA (30 days) → Glacier (90 days)
- Querying: Athena for ad-hoc, Glue for ETL, QuickSight for dashboards
- Cost: $5,000/month vs $50,000 on-premise SAN

**2. Static Website Hosting**
- Traffic: 1 million visitors/month
- Setup: S3 + CloudFront CDN + Route 53 + Lambda@Edge for A/B testing
- Cost: $20/month vs $200/month for EC2 web servers

---

### Amazon EBS (Elastic Block Store)

**What It Does:** Block-level storage volumes for EC2 instances

**Key Capabilities:**
- 6 volume types: gp3/gp2 (SSD), io2/io1 (Provisioned IOPS SSD), st1/sc1 (HDD)
- Up to 64,000 IOPS and 1,000 MB/s throughput
- Point-in-time snapshots to S3
- Encryption with KMS
- Multi-Attach (io2 volumes only)
- Elastic Volumes (resize without downtime)

**Limitations:**
- ❌ Single-AZ only (not HA across AZs)
- ❌ Attached to one instance at a time (except Multi-Attach)
- ❌ Max 64TB per volume
- ❌ More expensive than S3 for long-term storage

**Services That Fill the Gaps:**
- ✅ **EFS** → Multi-AZ shared file storage
- ✅ **S3** → Backup destination for snapshots
- ✅ **AWS Backup** → Centralized backup management
- ✅ **EBS Data Lifecycle Manager** → Automated snapshot creation

**Real-World Use Case:**
**Production MySQL Database**
- Config: io2 Block Express, 10,000 IOPS, 500 MB/s, encrypted
- Backups: Automated daily snapshots, 30-day retention
- Performance: Sub-millisecond latency, consistent IOPS
- Availability: Snapshots allow rapid recovery in case of failure

---

### Amazon EFS (Elastic File System)

**What It Does:** Fully managed NFS file system with multi-AZ durability

**Key Capabilities:**
- Shared access from thousands of EC2 instances simultaneously
- Petabyte-scale with automatic scaling
- Two storage classes: Standard and Infrequent Access (IA)
- Lifecycle management for automatic tiering
- Two performance modes: General Purpose and Max I/O
- Integration with Lambda for serverless file processing

**Limitations:**
- ❌ 3x more expensive than EBS gp3
- ❌ Only available in AWS regions
- ❌ No Windows support (Linux/macOS via NFS)
- ❌ Higher latency than EBS (~1-3ms vs <1ms)

**Services That Fill the Gaps:**
- ✅ **FSx for Windows** → Windows-native SMB file system
- ✅ **FSx for Lustre** → High-performance HPC file system
- ✅ **EFS Intelligent-Tiering** → Automatic cost optimization
- ✅ **EBS** → Lower cost for single-instance block storage

**Real-World Use Case:**
**Shared Web Content for Auto Scaling**
- Scenario: WordPress site with user-uploaded images
- Setup: EFS mounted on all web servers at /var/www/html/uploads, Auto Scaling 5-50 instances
- Benefit: All instances see same files instantly
- Cost: $150/month vs $500/month for EBS + rsync scripts

---

## 🗄️ Database Services

### Amazon RDS (Relational Database Service)

**What It Does:** Managed relational databases (MySQL, PostgreSQL, Oracle, SQL Server, MariaDB)

**Key Capabilities:**
- Automated backups with point-in-time recovery (35 days)
- Multi-AZ for HA (automatic failover in 60-120 seconds)
- Read replicas for read scaling (up to 5)
- Automated patching and version upgrades
- Performance Insights for query-level monitoring
- Blue/Green deployments for zero-downtime updates
- Storage auto-scaling

**Limitations:**
- ❌ No SSH access to underlying OS
- ❌ Limited custom database extensions
- ❌ Vertical scaling requires downtime (except storage)
- ❌ Max 64TB storage (except Aurora)
- ❌ Backup retention max 35 days

**Services That Fill the Gaps:**
- ✅ **Aurora Serverless** → Auto-scaling capacity without downtime
- ✅ **DynamoDB** → Unlimited horizontal scaling (NoSQL)
- ✅ **ElastiCache** → Caching layer to reduce load
- ✅ **RDS Proxy** → Connection pooling for Lambda
- ✅ **Aurora Global Database** → Sub-second cross-region replication

**Real-World Use Case:**
**E-commerce Transactional Database**
- Config: Aurora PostgreSQL Multi-AZ, db.r6g.4xlarge, 5 read replicas
- Performance: 20,000 transactions/second, <10ms latency
- HA: 99.99% uptime, automatic failover in 60 seconds
- Cost: $800/month vs $5,000 for self-managed

---

### Amazon DynamoDB

**What It Does:** Fully managed NoSQL database with single-digit millisecond latency

**Key Capabilities:**
- Unlimited throughput and storage
- Auto-scaling for read/write capacity
- Global Tables for multi-region replication
- Point-in-time recovery (35 days)
- DynamoDB Streams for change data capture
- Transactions for ACID operations
- On-demand or provisioned capacity pricing

**Limitations:**
- ❌ No complex joins or aggregations
- ❌ 400KB item size limit
- ❌ Must design access patterns upfront
- ❌ No built-in full-text search
- ❌ Hot partition issues if poorly designed key

**Services That Fill the Gaps:**
- ✅ **OpenSearch** → Full-text search and aggregations
- ✅ **Athena + S3** → Ad-hoc analytics on exported data
- ✅ **DAX** → Microsecond read latency
- ✅ **RDS/Aurora** → Complex relational queries

**Real-World Use Case:**
**IoT Device Telemetry**
- Volume: 1 million devices, 1 reading/minute = 1.44 billion writes/day
- Design: Partition key: device_id, Sort key: timestamp
- Processing: DynamoDB Streams → Lambda → Aggregate to S3
- Cost: $1,500/month with on-demand pricing
- Scaling: No capacity planning, automatic

---

### Amazon ElastiCache

**What It Does:** Fully managed in-memory caching (Redis and Memcached)

**Key Capabilities:**
- Microsecond latency for reads/writes
- Redis: Persistence, replication, clustering, pub/sub
- Automatic failover for Redis (Multi-AZ)
- Redis Cluster Mode for horizontal scaling (500 nodes)
- Encryption at rest and in transit

**Limitations:**
- ❌ Expensive for large datasets
- ❌ Memcached has no persistence
- ❌ No cross-region replication (use Global Datastore)
- ❌ Vertical scaling requires manual intervention

**Services That Fill the Gaps:**
- ✅ **DynamoDB DAX** → Managed cache for DynamoDB
- ✅ **Redis Global Datastore** → Cross-region replication
- ✅ **MemoryDB for Redis** → Durable Redis primary

**Real-World Use Case:**
**Database Query Cache**
- Setup: ElastiCache Redis (cache.r6g.large, 3 replicas), 5-minute TTL
- Pattern: Cache-aside in application code
- Performance: 95% cache hit rate, <1ms latency
- Impact: Database CPU reduced from 80% to 20%

---

## 🌐 Networking & Content Delivery

### Amazon VPC (Virtual Private Cloud)

**What It Does:** Logically isolated virtual network within AWS

**Key Capabilities:**
- Complete control over IP ranges (CIDR blocks)
- Subnets across multiple AZs
- Internet Gateway for public access
- NAT Gateway for private subnet internet access
- VPC Peering for connecting VPCs
- Transit Gateway for hub-and-spoke topology
- Security Groups (stateful) and Network ACLs (stateless)
- Flow Logs for traffic monitoring

**Limitations:**
- ❌ VPC Peering doesn't support transitive routing
- ❌ NAT Gateway has hourly cost + data transfer charges
- ❌ Cross-region VPC Peering has data transfer costs
- ❌ Complex to manage at scale

**Services That Fill the Gaps:**
- ✅ **Transit Gateway** → Simplifies multi-VPC connectivity
- ✅ **Direct Connect** → Dedicated network connection
- ✅ **Network Firewall** → Advanced threat protection
- ✅ **VPC Reachability Analyzer** → Troubleshoot connectivity

**Real-World Use Case:**
**Multi-Tier Web Application**
- VPC: 10.0.0.0/16
- Public subnets (10.0.1.0/24, 10.0.2.0/24) → Web + ALB
- Private subnets (10.0.10.0/24, 10.0.11.0/24) → App tier
- Private subnets (10.0.20.0/24, 10.0.21.0/24) → Database
- NAT Gateway in public subnets for private internet access
- Security groups restrict traffic between tiers

---

### Amazon CloudFront

**What It Does:** Global Content Delivery Network (CDN) with 450+ edge locations

**Key Capabilities:**
- Caches content close to users (sub-10ms latency)
- Origins: S3, EC2, ALB, custom HTTP servers
- Lambda@Edge and CloudFront Functions for edge compute
- SSL/TLS with custom certificates
- Signed URLs/cookies for private content
- HTTP/2 and HTTP/3 (QUIC) support

**Limitations:**
- ❌ Not suitable for dynamic content
- ❌ Cache invalidation costs ($0.005 per path)
- ❌ Origin must be publicly accessible
- ❌ Additional data transfer costs

**Services That Fill the Gaps:**
- ✅ **API Gateway** → Managed API with caching for dynamic content
- ✅ **Global Accelerator** → Anycast IPs for non-HTTP traffic
- ✅ **WAF** → DDoS protection and web application firewall

**Real-World Use Case:**
**Global Media Streaming**
- Volume: 10 million users worldwide
- Setup: S3 stores video (HLS segments) → CloudFront → Lambda@Edge for geo-personalization
- Performance: <50ms latency globally, 99.9% cache hit rate
- Cost: $10,000/month vs $50,000 direct from origin

---

### Elastic Load Balancing (ELB)

**What It Does:** Distributes incoming traffic across multiple targets

**Types:**
- **ALB (Application):** HTTP/HTTPS, path/host-based routing, WebSocket
- **NLB (Network):** TCP/UDP, ultra-low latency, static IPs, millions req/sec
- **GWLB (Gateway):** Third-party appliances (firewalls, IDS/IPS)

**Key Capabilities:**
- Health checks for target availability
- Cross-zone load balancing
- Sticky sessions (session affinity)
- SSL/TLS offloading
- Integration with Auto Scaling, ECS, Lambda

**Limitations:**
- ❌ ALB has slight latency vs NLB
- ❌ NLB doesn't support path-based routing
- ❌ Cross-zone load balancing has data transfer costs (NLB)

**Services That Fill the Gaps:**
- ✅ **Route 53** → DNS-based load balancing across regions
- ✅ **Global Accelerator** → Global static IPs with health checks
- ✅ **ACM** → Free SSL/TLS certificates

**Real-World Use Case:**
**Microservices Architecture (ALB)**
- Path-based routing: `/api/users/*` → User service, `/api/products/*` → Product service
- 15 microservices behind single ALB
- Health checks on `/health` endpoint
- Benefit: Simplified DNS, cost-effective

---

## 🔐 Security, Identity & Compliance

### AWS IAM (Identity and Access Management)

**What It Does:** Control access to AWS resources

**Key Capabilities:**
- Users, groups, and roles for access control
- Fine-grained permissions using JSON policies
- Multi-Factor Authentication (MFA)
- Temporary credentials via STS
- Cross-account access via roles
- SAML 2.0 and OIDC federation for SSO

**Limitations:**
- ❌ Complex policy syntax
- ❌ Max 10 managed policies per user/group/role
- ❌ Policy size limit: 6,144 characters (inline)
- ❌ Eventually consistent

**Services That Fill the Gaps:**
- ✅ **AWS SSO (IAM Identity Center)** → Centralized SSO
- ✅ **Secrets Manager** → Automatic secret rotation
- ✅ **Access Analyzer** → Identify overly permissive policies

**Real-World Use Case:**
**Least Privilege Access**
- Role: ReadOnlyAccess to production, MFA required
- Permission boundary prevents privilege escalation
- CloudTrail logs all actions
- Developers can troubleshoot but not modify

---

### AWS KMS (Key Management Service)

**What It Does:** Managed encryption key creation and management

**Key Capabilities:**
- Customer Master Keys (CMKs): AWS-managed, customer-managed
- Automatic key rotation (yearly)
- Integration with 100+ AWS services
- Asymmetric keys for signing
- Multi-region keys

**Limitations:**
- ❌ 4 KB data size limit per Encrypt API call
- ❌ Request rate limits: 5,500-100,000 req/sec
- ❌ Cannot export key material

**Services That Fill the Gaps:**
- ✅ **Envelope Encryption** → Encrypt large data by encrypting data keys
- ✅ **CloudHSM** → FIPS 140-2 Level 3 dedicated HSM
- ✅ **Secrets Manager** → Encrypt secrets using KMS

**Real-World Use Case:**
**Encrypted S3 Data Lake**
- Healthcare patient records with SSE-KMS
- Key policy restricts decryption to specific roles
- Automatic key rotation, CloudTrail logs key usage
- HIPAA compliance for data at rest

---

### AWS WAF (Web Application Firewall)

**What It Does:** Protect web applications from common exploits

**Key Capabilities:**
- OWASP Top 10 protection
- Rules: IP addresses, HTTP headers, body, URI strings
- Managed rule groups from AWS and third-party
- Rate-based rules to block DDoS
- Geo-blocking by country
- Integration with CloudFront, ALB, API Gateway

**Limitations:**
- ❌ $0.60 per million requests + $1/rule/month
- ❌ Complex rule syntax
- ❌ 1,500 WCU limit per WebACL
- ❌ False positives require tuning

**Services That Fill the Gaps:**
- ✅ **Shield Advanced** → Enhanced DDoS protection
- ✅ **GuardDuty** → Threat detection using ML
- ✅ **Network Firewall** → Layer 3/4 protection

**Real-World Use Case:**
**SQL Injection Prevention**
- Managed Rule: SQLi_QUERYARGUMENTS
- Blocks 99.9% of SQL injection attempts
- CloudWatch alarms for blocked requests
- Protection without code changes

---

## 📊 Analytics Services

### Amazon Athena

**What It Does:** Serverless SQL queries on S3 data

**Key Capabilities:**
- Pay-per-query: $5 per TB scanned
- Standard SQL (ANSI SQL-compliant)
- Supports CSV, JSON, Parquet, ORC, Avro
- Partitioning for query optimization
- Federated queries to RDS, Aurora, DynamoDB
- ACID transactions with Apache Iceberg

**Limitations:**
- ❌ Slow for small, frequent queries
- ❌ Max query execution: 30 minutes
- ❌ Max result set: 10 GB
- ❌ Charged per data scanned (partitioning critical)

**Services That Fill the Gaps:**
- ✅ **Glue** → ETL to optimize data formats (Parquet reduces costs by 90%)
- ✅ **QuickSight** → BI dashboards with SPICE caching
- ✅ **Redshift Spectrum** → Join S3 data with Redshift

**Real-World Use Case:**
**Log Analysis**
- ALB logs in S3 (Parquet), partitioned by date
- Query: "Find IPs with >1000 requests/hour"
- Volume: 100 GB scanned/day
- Cost: $5/day vs $500/month Splunk license

---

### Amazon EMR (Elastic MapReduce)

**What It Does:** Managed Hadoop, Spark, Presto, Hive ecosystem

**Key Capabilities:**
- Serverless (EMR Serverless) or cluster-based
- Auto-scaling clusters based on YARN metrics
- Spot instance support (70% cost savings)
- S3 integration via EMRFS
- EMR Notebooks for interactive analysis

**Limitations:**
- ❌ Complex to set up and tune
- ❌ Idle cluster costs
- ❌ EMR Serverless cold start: 1-2 minutes
- ❌ Requires Hadoop/Spark expertise

**Services That Fill the Gaps:**
- ✅ **Glue** → Serverless ETL for simple transformations
- ✅ **Athena** → Ad-hoc queries without cluster management
- ✅ **Kinesis** → Real-time streaming data

**Real-World Use Case:**
**Large-Scale ETL Pipeline**
- Volume: 10 TB daily web logs
- Setup: EMR Serverless with Spark, Spot instances
- Processing: 10 TB in 2 hours
- Cost: $100/day vs $1,000/day always-on cluster

---

### AWS Glue

**What It Does:** Serverless ETL service

**Key Capabilities:**
- Glue Data Catalog for centralized metadata
- Glue Crawlers for automatic schema discovery
- Python and Scala Spark jobs
- Glue DataBrew for visual data preparation (no code)
- Job bookmarking to process only new data
- Integration with S3, RDS, Redshift, DynamoDB

**Limitations:**
- ❌ Limited to Spark-based transformations
- ❌ Cold start: 2-10 minutes
- ❌ More expensive than EMR for long-running jobs
- ❌ Max job timeout: 48 hours

**Services That Fill the Gaps:**
- ✅ **EMR** → Complex Spark jobs, long-running workloads
- ✅ **Lambda** → Simple transformations without Spark overhead
- ✅ **Step Functions** → Orchestrate complex ETL workflows

**Real-World Use Case:**
**Data Lake Cataloging**
- Volume: 500 TB S3 data lake
- Crawlers scan S3 daily, auto-detect schema
- Glue Data Catalog provides metadata to Athena/Redshift/EMR
- 100 analysts querying via Athena

---

### Amazon Kinesis

**What It Does:** Real-time data streaming

**Types:**
- **Data Streams:** Custom applications, 24h-365d retention
- **Data Firehose:** Managed delivery to S3, Redshift, OpenSearch
- **Data Analytics:** SQL queries on streaming data
- **Video Streams:** Video ingestion for ML

**Key Capabilities:**
- Shard-based scaling (1 MB/s or 1,000 records/s per shard)
- Integration with Lambda for event-driven processing
- Server-side encryption with KMS

**Limitations:**
- ❌ Data Streams requires manual shard management
- ❌ Max record size: 1 MB
- ❌ Firehose minimum batch interval: 60 seconds
- ❌ No message ordering across shards

**Services That Fill the Gaps:**
- ✅ **Lambda** → Process streams without managing consumers
- ✅ **SQS** → Simpler queuing for non-real-time
- ✅ **MSK (Kafka)** → Open-source alternative

**Real-World Use Case:**
**Clickstream Analytics**
- Volume: 10,000 events/sec, <1 second latency
- Architecture: Website → Kinesis Data Streams → Lambda → DynamoDB
- Firehose archives to S3 for historical analysis
- Use Case: Real-time product recommendations

---

## 🤖 Machine Learning & AI

### Amazon SageMaker

**What It Does:** Fully managed ML platform (data prep → training → deployment)

**Key Capabilities:**
- SageMaker Studio for collaborative notebooks
- Built-in algorithms (XGBoost, Image Classification)
- Automatic model tuning (hyperparameter optimization)
- SageMaker Autopilot for AutoML (no code)
- Model monitoring for drift detection
- Multi-model endpoints for cost efficiency
- Feature Store for reusable features

**Limitations:**
- ❌ Expensive for long-running notebooks
- ❌ Learning curve for MLOps pipelines
- ❌ Inference costs can be high
- ❌ Cold start for serverless inference (5-10s)

**Services That Fill the Gaps:**
- ✅ **Lambda** → Lightweight model inference
- ✅ **Comprehend/Rekognition** → Pre-trained AI services
- ✅ **Glue** → Data preparation before training

**Real-World Use Case:**
**Image Classification for E-commerce**
- Dataset: 1 million labeled product images in S3
- Training: SageMaker built-in algorithm on ml.p3.2xlarge (GPU)
- Deployment: Multi-model endpoint
- Performance: 98% accuracy, 100ms inference latency
- Cost: $500/month training + inference vs $5,000 manual tagging

---

### Amazon Rekognition

**What It Does:** Pre-trained computer vision API (no ML expertise needed)

**Key Capabilities:**
- Image analysis: object detection, facial analysis, text (OCR)
- Video analysis: activity detection, face tracking, content moderation
- Custom labels for domain-specific detection (10-100 images)
- Face comparison and search
- PPE detection for safety compliance

**Limitations:**
- ❌ Pay-per-use (expensive at scale)
- ❌ Limited customization
- ❌ Requires internet access
- ❌ Privacy concerns for facial recognition

**Services That Fill the Gaps:**
- ✅ **SageMaker** → Custom vision models
- ✅ **Textract** → Advanced document analysis
- ✅ **Comprehend** → Text sentiment analysis

**Real-World Use Case:**
**Content Moderation for Social Media**
- Volume: 1 million images/day
- Architecture: S3 upload → Lambda → Rekognition DetectModerationLabels
- Flags: "Explicit Nudity" >80% confidence → Quarantine for review
- Accuracy: 95% reduction in manual review workload

---

### Amazon Comprehend

**What It Does:** Natural Language Processing (NLP) API

**Key Capabilities:**
- Sentiment analysis (positive, negative, neutral, mixed)
- Entity recognition (people, places, dates, organizations)
- Key phrase extraction
- Language detection (100+ languages)
- Topic modeling for document clustering
- Custom classification and entity recognition
- PII detection

**Limitations:**
- ❌ Max document size: 100 KB
- ❌ Pay-per-use (expensive at scale)
- ❌ Real-time API latency: 100-500ms

**Services That Fill the Gaps:**
- ✅ **SageMaker** → Custom NLP models (BERT, GPT)
- ✅ **Textract** → Extract text from PDFs/images
- ✅ **Bedrock** → Generative AI for text generation

**Real-World Use Case:**
**Customer Support Ticket Routing**
- Volume: 10,000 tickets/day
- Architecture: Zendesk → Lambda → Comprehend sentiment + entities
- Routing: Negative sentiment → Priority queue, topic-based team assignment
- Result: 30% faster resolution times

---

### Amazon Bedrock

**What It Does:** Fully managed generative AI service (access to foundation models)

**Key Capabilities:**
- Models: Claude (Anthropic), Llama (Meta), Titan (Amazon), Stable Diffusion
- No infrastructure management (serverless)
- Customization via fine-tuning and RAG
- Knowledge Bases for RAG with automatic embeddings
- Guardrails for responsible AI
- Agents for multi-step task automation

**Limitations:**
- ❌ Expensive at high volume
- ❌ Model selection limited to supported providers
- ❌ Cold start latency (1-5 seconds)
- ❌ Requires prompt engineering expertise

**Services That Fill the Gaps:**
- ✅ **OpenSearch** → Vector database for RAG
- ✅ **Kendra** → Enterprise search for knowledge retrieval
- ✅ **SageMaker** → Fine-tune open-source models on custom data

**Real-World Use Case:**
**Customer Service Chatbot**
- Setup: Bedrock Knowledge Base with company policy documents, Claude 3 model
- Guardrails: Block harmful content and PII sharing
- Performance: 80% of questions answered without human agent
- Integration: Lambda connects chatbot to claims system

---

## 🐳 Containers & Serverless

### AWS Fargate

**What It Does:** Serverless compute for containers (no server management)

**Key Capabilities:**
- Works with ECS and EKS
- Pay only for vCPU and memory used
- Automatic scaling
- Isolated compute environment per task/pod
- Support for Spot pricing (70% discount)

**Limitations:**
- ❌ Higher cost than EC2 for sustained workloads
- ❌ Limited instance type choices
- ❌ Cold start time (30-60 seconds)

**Services That Fill the Gaps:**
- ✅ **EC2** → Lower cost for sustained workloads
- ✅ **Lambda** → Sub-second cold starts for event-driven

**Real-World Use Case:**
**CI/CD Build Agents**
- Trigger: CodePipeline → ECS task on Fargate Spot
- 500 builds/day, avg 8 minutes per build
- Cost: $120/month vs $800/month for dedicated EC2

---

### AWS App Runner

**What It Does:** Fully managed container-based web applications (PaaS)

**Key Capabilities:**
- Deploy from source code or container image
- Automatic scaling (0-100 instances)
- Built-in load balancing and SSL
- Auto-deploy on git push
- Pay for compute + requests

**Limitations:**
- ❌ Limited to web applications
- ❌ Less control than ECS/EKS
- ❌ Regional service (not multi-region)

**Services That Fill the Gaps:**
- ✅ **Elastic Beanstalk** → Multi-region, more config options
- ✅ **ECS Fargate** → More control over infrastructure

**Real-World Use Case:**
**Startup API Deployment**
- Setup: GitHub repo → App Runner auto-deploys on push
- Scaling: 1-25 instances based on requests
- Cost: $100/month for low-traffic startup
- Benefit: Zero infrastructure management

---

## 🛠️ Developer Tools

### AWS CodePipeline, CodeBuild, CodeDeploy

**What They Do:** Managed CI/CD services

**CodePipeline:** Orchestrates build, test, deploy stages
**CodeBuild:** Serverless build service (compile, test, package)
**CodeDeploy:** Automated deployment to EC2, ECS, Lambda, on-premise

**Key Capabilities:**
- CodePipeline: Multi-stage workflows, manual approvals, integrations with GitHub/Bitbucket
- CodeBuild: Docker-based build environments, parallel builds, caching
- CodeDeploy: Blue/green deployments, canary releases, automatic rollback

**Limitations:**
- ❌ CodeBuild: Max 8-hour build timeout
- ❌ CodeDeploy: Limited rollback strategies vs advanced tools
- ❌ Steeper learning curve than GitHub Actions

**Services That Fill the Gaps:**
- ✅ **GitHub Actions / Jenkins** → More mature ecosystem
- ✅ **AWS Proton** → Standardized infrastructure templates
- ✅ **Step Functions** → Complex orchestration

**Real-World Use Case:**
**Automated Deployment Pipeline**
- Flow: GitHub push → CodePipeline → CodeBuild (test) → CodeDeploy (blue/green to ECS)
- Safety: Manual approval before production
- Rollback: Automatic rollback if health checks fail
- Result: 10 deployments/day, 99% success rate

---

### AWS CloudFormation

**What It Does:** Infrastructure as Code (IaC) for AWS resources

**Key Capabilities:**
- JSON/YAML templates define infrastructure
- StackSets for multi-account/region deployments
- Drift detection for manual changes
- Change sets for previewing updates
- Nested stacks for modular templates
- Integration with Service Catalog

**Limitations:**
- ❌ Verbose YAML/JSON syntax
- ❌ Limited to AWS (not multi-cloud)
- ❌ Slower than Terraform for complex stacks
- ❌ Difficult to debug failed deployments

**Services That Fill the Gaps:**
- ✅ **Terraform** → Multi-cloud support
- ✅ **CDK (Cloud Development Kit)** → Define infrastructure in code (Python, TypeScript)
- ✅ **AWS SAM** → Simplified templates for serverless

**Real-World Use Case:**
**Multi-Account Organization**
- Setup: CloudFormation StackSets deploy VPC, security groups across 20 accounts
- Templates: Standardized networking, IAM roles
- Governance: Centralized control, prevent drift
- Benefit: Consistent infrastructure, reduced manual work

---

## 🔄 Application Integration

### Amazon SQS (Simple Queue Service)

**What It Does:** Fully managed message queuing service

**Key Capabilities:**
- Standard queues: unlimited throughput, at-least-once delivery
- FIFO queues: exactly-once processing, preserved order
- Dead-letter queues for failed messages
- Long polling to reduce costs
- Delay queues for deferred processing
- Message retention: 1 minute to 14 days
- Max message size: 256 KB (use S3 for larger)

**Limitations:**
- ❌ Standard queues: no ordering guarantee
- ❌ FIFO queues: 300 messages/sec (3,000 with batching)
- ❌ No pub/sub (use SNS)
- ❌ No message priority

**Services That Fill the Gaps:**
- ✅ **SNS** → Pub/sub for multiple subscribers
- ✅ **Kinesis** → Real-time streaming with ordering
- ✅ **EventBridge** → Event-driven architecture

**Real-World Use Case:**
**Order Processing Pipeline**
- Architecture: API → SQS → Lambda (process order) → DynamoDB
- Dead-letter queue: Failed orders for manual review
- Scaling: Lambda auto-scales with queue depth
- Reliability: At-least-once delivery, retry logic

---

### Amazon SNS (Simple Notification Service)

**What It Does:** Fully managed pub/sub messaging service

**Key Capabilities:**
- Push notifications to mobile devices (iOS, Android)
- Email, SMS, HTTP endpoints
- Fan-out to multiple SQS queues
- Message filtering by attributes
- FIFO topics for ordered delivery
- Message archiving to S3

**Limitations:**
- ❌ Max message size: 256 KB
- ❌ No guaranteed delivery to HTTP endpoints (use SQS)
- ❌ SMS costs can add up

**Services That Fill the Gaps:**
- ✅ **SQS** → Guaranteed delivery with retries
- ✅ **EventBridge** → Event routing with rules
- ✅ **Pinpoint** → Advanced mobile campaigns

**Real-World Use Case:**
**Multi-Channel Notifications**
- Trigger: Critical system alert
- Fan-out: SNS → SQS (email service), Lambda (Slack webhook), SMS (on-call engineer)
- Filtering: Only production alerts trigger SMS
- Benefit: Single publish to multiple subscribers

---

### Amazon EventBridge

**What It Does:** Serverless event bus for event-driven architectures

**Key Capabilities:**
- Route events between AWS services, SaaS apps, custom apps
- Event patterns for filtering (JSON)
- Scheduled events (cron/rate expressions)
- Schema registry for event discovery
- Archive and replay events
- Cross-account event delivery

**Limitations:**
- ❌ Max event size: 256 KB
- ❌ Slight latency vs direct service invocation
- ❌ Debugging complex event flows can be challenging

**Services That Fill the Gaps:**
- ✅ **SNS** → High-throughput pub/sub
- ✅ **Step Functions** → Orchestrate multi-step workflows
- ✅ **CloudWatch Events** (legacy)

**Real-World Use Case:**
**Event-Driven Microservices**
- Events: OrderCreated, PaymentProcessed, ShipmentDispatched
- Routing: EventBridge routes to appropriate Lambda functions
- Integration: Shopify webhooks → EventBridge → processing
- Benefit: Decoupled services, easy to add new consumers

---

### AWS Step Functions

**What It Does:** Serverless workflow orchestration

**Key Capabilities:**
- Visual workflow designer
- State machines (Standard and Express)
- Parallel execution and error handling
- Integration with 200+ AWS services
- Human approval steps via callbacks
- Maximum execution time: 1 year (Standard), 5 minutes (Express)

**Limitations:**
- ❌ Standard workflows: $0.025 per 1,000 state transitions
- ❌ Complex workflows can be difficult to debug
- ❌ Learning curve for ASL (Amazon States Language)

**Services That Fill the Gaps:**
- ✅ **Lambda** → Simple orchestration in code
- ✅ **Glue Workflows** → ETL-specific orchestration
- ✅ **Airflow (MWAA)** → Complex data pipelines

**Real-World Use Case:**
**Order Fulfillment Workflow**
- Steps: Validate order → Check inventory → Process payment → Ship → Send notification
- Error handling: Payment failure → rollback inventory → notify customer
- Human approval: Orders >$10K require manager approval
- Benefit: Visual workflow, automatic retries, audit trail

---

## 🚀 Migration & Transfer

### AWS Database Migration Service (DMS)

**What It Does:** Migrate databases to AWS with minimal downtime

**Key Capabilities:**
- Homogeneous migrations (Oracle → RDS Oracle)
- Heterogeneous migrations (Oracle → Aurora PostgreSQL)
- Continuous data replication (CDC)
- Schema conversion with AWS SCT
- Support for 20+ database engines

**Limitations:**
- ❌ Initial full load can be slow for large databases
- ❌ Complex transformations require custom logic
- ❌ Ongoing replication has data transfer costs

**Services That Fill the Gaps:**
- ✅ **AWS SCT** → Schema conversion for heterogeneous migrations
- ✅ **DataSync** → File transfer (not databases)
- ✅ **Snowball** → Offline data transfer for TB/PB-scale

**Real-World Use Case:**
**Oracle to Aurora Migration**
- Source: 5TB Oracle database on-premise
- Process: Full load (7 days) → CDC for ongoing replication
- Cutover: Zero-downtime switch during low-traffic window
- Result: 40% cost reduction, improved performance

---

### AWS DataSync

**What It Does:** Fast data transfer between on-premise and AWS

**Key Capabilities:**
- 10x faster than open-source tools
- Automatic encryption and compression
- Bandwidth throttling
- Scheduled transfers
- Transfer to S3, EFS, FSx
- Verification of data integrity

**Limitations:**
- ❌ Requires DataSync agent on-premise (VM or hardware)
- ❌ Data transfer costs over internet
- ❌ Not real-time (scheduled or one-time)

**Services That Fill the Gaps:**
- ✅ **Storage Gateway** → Continuous hybrid storage
- ✅ **Direct Connect** → Dedicated network connection
- ✅ **Snowball** → Offline transfer for >10TB

**Real-World Use Case:**
**Data Center Decommission**
- Volume: 100TB NFS file shares
- Setup: DataSync agent on-premise → EFS in AWS
- Transfer: Incremental transfers over Direct Connect
- Time: 100TB in 2 weeks vs 6 months over internet

---

### AWS Snowball / Snowmobile

**What They Do:** Physical data transfer devices for petabyte-scale migrations

**Snowball Edge:** 80TB or 210TB device, compute capabilities
**Snowmobile:** 100PB trailer truck

**Key Capabilities:**
- Encrypted transfer (256-bit encryption)
- Cluster mode for high availability
- Edge computing with Lambda and EC2
- Import to S3, export from S3

**Limitations:**
- ❌ Lead time for device delivery (days to weeks)
- ❌ Manual process (ship device)
- ❌ Not suitable for <10TB (use DataSync)

**Services That Fill the Gaps:**
- ✅ **DataSync** → Faster for <10TB over network
- ✅ **Direct Connect** → Continuous high-bandwidth transfers

**Real-World Use Case:**
**Genomics Data Migration**
- Volume: 500TB genomic sequencing data
- Process: Copy to 7x Snowball Edge devices → Ship to AWS → Import to S3
- Time: 2 weeks vs 6 months over internet
- Cost: $2,000 vs $50,000 in data transfer fees

---

## 🎥 Media Services

### Amazon Elastic Transcoder / MediaConvert

**What They Do:** Transcode video and audio files to different formats

**Elastic Transcoder:** Simple, preset-based transcoding
**MediaConvert:** Professional-grade, advanced features

**Key Capabilities:**
- Input: 100+ formats (MP4, MOV, AVI, MKV)
- Output: Adaptive bitrate streaming (HLS, DASH)
- Thumbnails, watermarks, captions
- DRM support (Widevine, PlayReady, FairPlay)
- MediaConvert: HDR, Dolby Atmos, IMF

**Limitations:**
- ❌ Pay-per-minute pricing (can be expensive for large volumes)
- ❌ Processing time varies by complexity
- ❌ No real-time transcoding (use MediaLive)

**Services That Fill the Gaps:**
- ✅ **MediaLive** → Real-time live streaming
- ✅ **CloudFront** → CDN for video delivery
- ✅ **S3** → Video storage

**Real-World Use Case:**
**Video Platform (Netflix-like)**
- Upload: User uploads 4K video to S3
- Process: MediaConvert creates 5 qualities (4K, 1080p, 720p, 480p, 360p)
- Output: HLS manifest for adaptive streaming
- Delivery: CloudFront CDN serves to users
- Cost: $0.015 per minute vs $0.50 per minute in-house

---

### Amazon Kinesis Video Streams

**What It Does:** Ingest, process, and store video streams

**Key Capabilities:**
- Ingest from cameras, mobile devices, drones
- Playback via HLS
- Integration with Rekognition for video analysis
- Time-encoded indexing
- Encryption at rest and in transit
- Retention: 1 hour to 10 years

**Limitations:**
- ❌ Pay for ingestion + storage (costs add up)
- ❌ Playback latency (3-5 seconds)
- ❌ Limited analytics (use Rekognition)

**Services That Fill the Gaps:**
- ✅ **Rekognition Video** → Automated video analysis
- ✅ **SageMaker** → Custom ML models on video
- ✅ **MediaLive** → Live streaming with low latency

**Real-World Use Case:**
**Smart Home Security Cameras**
- Setup: 10,000 cameras stream to Kinesis Video Streams
- Analysis: Rekognition detects people, cars, packages
- Alerts: SNS notification when person detected at night
- Storage: 7-day retention for footage review

---

## 💻 End User Computing

### Amazon WorkSpaces

**What It Does:** Fully managed Virtual Desktop Infrastructure (VDI)

**Key Capabilities:**
- Windows and Linux desktops
- Pay-as-you-go or monthly pricing
- Integration with Active Directory
- MFA support
- Persistent or non-persistent desktops
- GPU-enabled for graphics workloads

**Limitations:**
- ❌ More expensive than physical desktops long-term
- ❌ Requires good internet connection
- ❌ Limited to AWS regions

**Services That Fill the Gaps:**
- ✅ **AppStream 2.0** → Application streaming (no full desktop)
- ✅ **WorkSpaces Web** → Browser-based access

**Real-World Use Case:**
**Remote Workforce**
- Users: 500 employees working from home
- Setup: WorkSpaces with company applications pre-installed
- Security: Data stays in AWS, no local storage
- Cost: $35/user/month vs $1,000 per physical desktop

---

### Amazon AppStream 2.0

**What It Does:** Stream desktop applications to browsers (no desktop needed)

**Key Capabilities:**
- Stream Windows applications to any browser
- On-demand or always-on fleets
- Auto-scaling based on demand
- User data persistence in S3
- Home folders and application settings sync

**Limitations:**
- ❌ Windows applications only
- ❌ Latency-sensitive apps may have issues
- ❌ Per-user-hour pricing can be expensive

**Services That Fill the Gaps:**
- ✅ **WorkSpaces** → Full desktop experience
- ✅ **WorkSpaces Web** → SaaS application access

**Real-World Use Case:**
**CAD Software Access**
- Application: AutoCAD streamed to contractors
- Setup: g4dn instances (GPU) for 3D rendering
- Scaling: Auto-scale 10-50 instances based on usage
- Benefit: No need to ship high-end workstations

---

## 🌐 IoT Services

### AWS IoT Core

**What It Does:** Managed cloud service for IoT devices

**Key Capabilities:**
- MQTT, HTTPS, WebSocket protocols
- Device shadows for offline state
- Rules engine for data routing
- Integration with 20+ AWS services
- Device authentication with X.509 certificates
- Billion+ messages per day

**Limitations:**
- ❌ Pay-per-message pricing
- ❌ Message size limit: 128 KB
- ❌ Rules engine limited to simple transformations

**Services That Fill the Gaps:**
- ✅ **IoT Greengrass** → Edge computing on devices
- ✅ **Kinesis** → High-throughput data ingestion
- ✅ **Lambda** → Complex data processing

**Real-World Use Case:**
**Smart Factory**
- Devices: 10,000 sensors on production line
- Data: Temperature, pressure, vibration every 10 seconds
- Processing: IoT Core rules → Lambda → DynamoDB + S3
- Alerts: Anomalies trigger SNS to maintenance team
- Cost: $500/month for 250M messages

---

## 📖 Summary: When to Use What

### Compute Decision Tree
- **Serverless event-driven?** → Lambda
- **Containers with orchestration?** → ECS/EKS
- **Full OS control needed?** → EC2
- **Batch processing?** → AWS Batch
- **Web app PaaS?** → App Runner or Elastic Beanstalk

### Storage Decision Tree
- **Object storage (files, backups)?** → S3
- **Block storage (databases, VMs)?** → EBS
- **Shared file system (NFS)?** → EFS
- **Windows file server?** → FSx for Windows
- **HPC high-performance?** → FSx for Lustre

### Database Decision Tree
- **Relational (OLTP)?** → RDS or Aurora
- **NoSQL key-value (scale)?** → DynamoDB
- **In-memory cache?** → ElastiCache
- **Data warehouse (analytics)?** → Redshift
- **Graph relationships?** → Neptune
- **Document database?** → DocumentDB

### Analytics Decision Tree
- **Ad-hoc SQL on S3?** → Athena
- **Large-scale ETL?** → EMR or Glue
- **Real-time streaming?** → Kinesis
- **BI dashboards?** → QuickSight
- **Search and analytics?** → OpenSearch

### ML/AI Decision Tree
- **Custom ML models?** → SageMaker
- **Pre-trained vision?** → Rekognition
- **Pre-trained NLP?** → Comprehend
- **Generative AI?** → Bedrock
- **Document extraction?** → Textract

---

## 🎯 Cost Optimization Tips

1. **Use Reserved Instances / Savings Plans** for predictable workloads (up to 72% savings)
2. **Leverage Spot Instances** for fault-tolerant workloads (up to 90% savings)
3. **Right-size resources** using AWS Compute Optimizer recommendations
4. **Use S3 Intelligent-Tiering** for automatic storage optimization
5. **Enable S3 Lifecycle policies** to transition to cheaper storage classes
6. **Use CloudFront** to reduce origin server costs
7. **Delete unused EBS volumes and snapshots**
8. **Use Lambda for infrequent workloads** vs always-on EC2
9. **Schedule non-production resources** to shut down after hours
10. **Monitor with Cost Explorer** and set billing alarms

---

## 🔧 Best Practices

### Security
- Enable MFA on root account and IAM users
- Use IAM roles instead of long-term credentials
- Enable CloudTrail for audit logging
- Encrypt data at rest (S3, EBS, RDS)
- Use VPCs with private subnets for resources
- Implement least privilege access

### Reliability
- Design for failure (multi-AZ, multi-region)
- Use Auto Scaling for elasticity
- Implement health checks and monitoring
- Enable automated backups
- Test disaster recovery procedures
- Use multiple Availability Zones

### Performance
- Use CloudFront for global content delivery
- Implement caching layers (ElastiCache, CloudFront)
- Choose the right instance types for workloads
- Use Provisioned IOPS for latency-sensitive databases
- Monitor with CloudWatch and set alarms

### Cost Optimization
- Review AWS Cost Explorer monthly
- Use AWS Budgets for cost control
- Tag all resources for cost allocation
- Delete unused resources
- Use Reserved Instances for steady-state workloads

---

## 📚 Additional Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Training and Certification](https://aws.amazon.com/training/)
- [AWS CLI Reference](https://docs.aws.amazon.com/cli/)
- [AWS Pricing Calculator](https://calculator.aws/)

---

## 🤝 Contributing

Found an error or want to add a use case? Feel free to submit a PR!

---

## 📄 License

MIT License - Feel free to use this guide for your projects

---

**Last Updated:** February 2026
**Maintained by:** Cloud Engineering Community
