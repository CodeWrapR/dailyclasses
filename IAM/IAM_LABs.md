# AWS IAM Labs: Complete Guide from Basic to Advanced

## Lab Overview
This guide contains 15 progressive labs covering IAM concepts from foundational to enterprise-level scenarios. Each lab includes objectives, prerequisites, step-by-step instructions, and real-world applications.

---

## SECTION 1: FOUNDATIONAL CONCEPTS (Labs 1-3)

### Lab 1: IAM Users and Programmatic Access

**Objective:** Create IAM users and generate access keys for programmatic access.

**Prerequisites:**
- AWS account with admin access
- AWS Management Console access

**Lab Steps:**

1. Navigate to IAM Dashboard → Users → Create User
2. Create user "developer-user-01"
3. Attach policy: "AmazonS3ReadOnlyAccess"
4. Generate access key (access key ID and secret access key)
5. Store credentials securely in AWS Secrets Manager
6. Test access using AWS CLI:
   ```bash
   aws configure --profile developer
   aws s3 ls --profile developer
   ```

**Real-World Application:**
- Creating credentials for CI/CD pipelines
- Onboarding developers with limited S3 permissions
- Automated system accounts for applications

**Validation Checklist:**
- [ ] User created successfully
- [ ] Access keys generated
- [ ] Can list S3 buckets via CLI
- [ ] Cannot create or delete buckets (read-only confirmed)

---

### Lab 2: IAM Groups and Bulk User Management

**Objective:** Organize users into groups and manage permissions at group level.

**Prerequisites:**
- Completion of Lab 1
- Basic understanding of IAM policies

**Lab Steps:**

1. Create groups:
   - "Developers" - Attach: AmazonEC2FullAccess, AmazonS3ReadOnlyAccess
   - "DevOps" - Attach: Administrator access with MFA requirement
   - "Finance" - Attach: CloudTrail read-only access

2. Create users: dev-user-1, dev-user-2, devops-admin-1
3. Add users to appropriate groups
4. Test permissions:
   ```bash
   # As dev-user, try EC2 operations
   aws ec2 describe-instances --profile dev-user-1
   
   # Try S3 write (should fail)
   aws s3 cp file.txt s3://bucket-name --profile dev-user-1
   ```

**Real-World Application:**
- Department-level access control
- Consistent permission management for team members
- Simplifying offboarding (remove from group)

**Validation Checklist:**
- [ ] 3 groups created with correct policies
- [ ] Users assigned to groups
- [ ] Developers can read S3, not write
- [ ] Developers have EC2 full access

---

### Lab 3: Role-Based Access Control (RBAC) Basics

**Objective:** Create IAM roles for service-to-service communication and cross-account access.

**Prerequisites:**
- Labs 1-2 completion
- Understanding of trust relationships

**Lab Steps:**

1. Create role: "EC2-S3-Access-Role"
   - Trust relationship: ec2.amazonaws.com
   - Attach policy: "AmazonS3FullAccess"

2. Create EC2 instance (optional: if testing hands-on)
   - Attach the role to the instance
   - Access EC2 via Systems Manager Session Manager

3. Test S3 access from instance:
   ```bash
   aws s3 ls
   aws s3 cp s3://source-bucket/file.txt . 
   ```

4. Create another role: "Lambda-DynamoDB-Role"
   - Trust: lambda.amazonaws.com
   - Attach: AmazonDynamoDBFullAccess

**Real-World Application:**
- EC2 instances accessing S3 without hardcoding credentials
- Lambda functions reading from DynamoDB
- Microservices with specific resource access

**Validation Checklist:**
- [ ] EC2 role created with S3 access
- [ ] EC2 instance can list S3 buckets
- [ ] No hardcoded credentials used
- [ ] Lambda role has DynamoDB access

---

## SECTION 2: POLICY MANAGEMENT (Labs 4-6)

### Lab 4: Custom IAM Policies and Policy Simulator

**Objective:** Write custom policies and validate them using the policy simulator.

**Prerequisites:**
- Labs 1-3 completion
- Understanding of JSON policy structure

**Lab Steps:**

1. Create custom policy: "S3-Specific-Bucket-Access"
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListAllBuckets",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    },
    {
      "Sid": "ReadSpecificBucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::prod-bucket",
        "arn:aws:s3:::prod-bucket/*"
      ]
    },
    {
      "Sid": "DenyDeleteOperations",
      "Effect": "Deny",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

2. Create user: "restricted-s3-user"
3. Attach custom policy to user
4. Use Policy Simulator to test:
   - Action: s3:ListBucket on prod-bucket (should Allow)
   - Action: s3:DeleteObject on prod-bucket (should Deny)
   - Action: s3:PutObject on prod-bucket (should Deny/Allow based on policy)

**Real-World Application:**
- Read-only access to production data
- Preventing accidental deletions via explicit Deny
- Audit trails for specific resources

**Validation Checklist:**
- [ ] Custom policy created
- [ ] Policy simulator shows correct Allow/Deny
- [ ] GetObject allowed
- [ ] DeleteObject denied

---

### Lab 5: Condition-Based Policies (IP Restriction & Time-Based)

**Objective:** Create policies that restrict access based on IP addresses and time conditions.

**Prerequisites:**
- Lab 4 completion
- Understanding of policy conditions

**Lab Steps:**

1. Create policy: "EC2-Office-Hours-Access"
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2FromOfficeIP",
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": [
            "203.0.113.0/24"
          ]
        }
      }
    },
    {
      "Sid": "AllowDuringBusinessHours",
      "Effect": "Allow",
      "Action": "iam:*",
      "Resource": "*",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "2025-01-01T09:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2025-01-01T17:00:00Z"
        }
      }
    }
  ]
}
```

2. Create user: "office-worker"
3. Attach policy and test from different IPs (use VPN if needed)
4. Verify time-based access restriction

**Real-World Application:**
- Preventing API access from non-corporate networks
- Restricting admin operations to business hours
- Compliance with access control policies
- Reducing risk of compromised credentials outside office

**Validation Checklist:**
- [ ] Policy created with IP conditions
- [ ] Access denied from non-office IPs
- [ ] Access allowed from office IP
- [ ] Time conditions evaluated correctly

---

### Lab 6: Resource-Based Policies and Cross-Account Access

**Objective:** Implement resource-based policies for S3 buckets and enable cross-account access.

**Prerequisites:**
- Labs 1-5 completion
- 2 AWS accounts (or simulate with single account using different principals)

**Lab Steps:**

1. In Account A (Production):
   - Create S3 bucket: "prod-data-bucket"
   - Create bucket policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCrossAccountRead",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-B-ID:root"
      },
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::prod-data-bucket",
        "arn:aws:s3:::prod-data-bucket/*"
      ]
    }
  ]
}
```

2. In Account B (Development):
   - Create role: "CrossAccountProdAccess"
   - Trust Account A
   - Create policy for S3 access

3. Assume role from Account B and access bucket from Account A

**Real-World Application:**
- Shared production data access for development teams
- Multi-account AWS organization setup
- Disaster recovery access between accounts
- CI/CD pipelines accessing shared artifacts

**Validation Checklist:**
- [ ] Cross-account bucket policy created
- [ ] Account B can assume cross-account role
- [ ] Account B can read from Account A bucket
- [ ] Write operations are denied

---

## SECTION 3: ADVANCED AUTHENTICATION (Labs 7-9)

### Lab 7: Multi-Factor Authentication (MFA)

**Objective:** Enforce MFA for sensitive operations and console access.

**Prerequisites:**
- Labs 1-3 completion
- Virtual MFA device (Google Authenticator, Authy)

**Lab Steps:**

1. Enable MFA for root account:
   - Security Credentials → MFA Devices → Activate MFA
   - Choose Virtual MFA device
   - Scan QR code with authenticator app
   - Enter 2 consecutive codes

2. Create user: "mfa-admin"
3. Enforce MFA requirement with policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListingUserMFA",
      "Effect": "Allow",
      "Action": "iam:ListUsers",
      "Resource": "*"
    },
    {
      "Sid": "AllowManagingOwnVirtualMFADevice",
      "Effect": "Allow",
      "Action": [
        "iam:CreateVirtualMFADevice",
        "iam:DeleteVirtualMFADevice"
      ],
      "Resource": "arn:aws:iam::*:mfa/${aws:username}"
    },
    {
      "Sid": "AllowDeactivateMFADevice",
      "Effect": "Allow",
      "Action": "iam:DeactivateMFADevice",
      "Resource": "arn:aws:iam::*:user/${aws:username}"
    },
    {
      "Sid": "DenyAllExceptListedIfNoMFA",
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:ListMFADevices",
        "iam:ListUsers",
        "iam:ListVirtualMFADevices",
        "iam:ResyncMFADevice",
        "sts:GetSessionToken"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

4. Test MFA-protected operations:
   - Try accessing console without MFA (should fail)
   - Enable MFA and retry
   - Use AWS CLI with MFA tokens

**Real-World Application:**
- Protecting privileged user accounts
- Compliance with security standards (SOC2, HIPAA)
- Preventing unauthorized access even if password is compromised
- Enforcing MFA for AWS root account (best practice)

**Validation Checklist:**
- [ ] MFA device registered
- [ ] Root account MFA enabled
- [ ] Policy prevents unMFA'd access
- [ ] Can authenticate with MFA token

---

### Lab 8: Temporary Security Credentials and STS AssumeRole

**Objective:** Generate temporary credentials and understand role assumption workflows.

**Prerequisites:**
- Labs 1-7 completion
- Understanding of STS service

**Lab Steps:**

1. Create role: "TemporaryEC2AdminRole"
   - Trust: IAM users in same account
   - Policy: EC2 full access

2. Create user: "temp-user"
3. Attach policy to user allowing AssumeRole:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::ACCOUNT-ID:role/TemporaryEC2AdminRole"
    }
  ]
}
```

4. Assume role using CLI:
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT-ID:role/TemporaryEC2AdminRole \
  --role-session-name temporary-session

# Use returned credentials (with expiration)
export AWS_ACCESS_KEY_ID=<AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
export AWS_SESSION_TOKEN=<SessionToken>

# Verify temporary credentials
aws sts get-caller-identity
```

5. Test credential expiration (typically 1 hour)

**Real-World Application:**
- Federated access (SAML, OIDC)
- Just-in-time (JIT) privilege escalation
- Cross-account access for third-party vendors
- Temporary elevated permissions for on-call engineers
- Container orchestration (ECS, EKS task roles)

**Validation Checklist:**
- [ ] Role created with proper trust relationship
- [ ] User can assume role
- [ ] Temporary credentials generated
- [ ] Credentials expire as expected
- [ ] Session token prevents permanent credential use

---

### Lab 9: Identity Federation and SAML Integration

**Objective:** Set up SAML-based federation for enterprise SSO.

**Prerequisites:**
- Labs 1-8 completion
- Understanding of SAML concepts
- Okta/Azure AD trial account (or use AWS SSO as alternative)

**Lab Steps:**

1. Create IAM SAML Identity Provider:
   - IAM → Identity Providers → Create Provider
   - Provider Type: SAML
   - Download AWS SAML metadata

2. In Okta/Azure AD:
   - Create SAML application for AWS
   - Upload AWS SAML metadata
   - Configure attribute mappings (email, groups)
   - Generate SAML metadata

3. Upload SAML provider metadata to AWS IAM

4. Create roles for SAML users:
   - Role: "SamlEC2Developers"
   - Trust policy for SAML provider
   - Condition: SAML subject matches specific pattern

5. Test SAML login:
   - Use IdP login portal
   - Redirect to AWS role selection
   - Access AWS console

**Alternative: Use AWS SSO (simpler setup)**
```bash
# Configure AWS SSO for organization
# Link to Okta or Azure AD
# Create permission sets (equivalent to roles)
# Assign users/groups to accounts
```

**Real-World Application:**
- Enterprise user management
- Single sign-on across AWS accounts
- Automatic role assignment based on group membership
- Audit trail through CloudTrail
- Compliance with IAM federation requirements

**Validation Checklist:**
- [ ] SAML IdP configured
- [ ] SAML roles created
- [ ] Users can login via SAML
- [ ] Correct roles assigned based on group membership
- [ ] CloudTrail logs SAML-based access

---

## SECTION 4: ADVANCED SCENARIOS (Labs 10-15)

### Lab 10: Service Control Policies (SCPs) and Organization Boundaries

**Objective:** Implement organization-wide guardrails using SCPs.

**Prerequisites:**
- Labs 1-9 completion
- AWS Organization with multiple accounts
- Or: Single account with organizational structure

**Lab Steps:**

1. Navigate to AWS Organizations (must have organization enabled)
2. Create SCP: "PreventS3Deletion"
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "s3:DeleteBucket",
        "s3:DeleteObject",
        "s3:DeleteObjectVersion"
      ],
      "Resource": "*"
    }
  ]
}
```

3. Create another SCP: "EnforceCloudTrailLogging"
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "cloudtrail:DeleteTrail",
        "cloudtrail:StopLogging",
        "cloudtrail:UpdateTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

4. Attach SCPs to OU (organizational unit) or specific accounts
5. Test: Try deleting S3 bucket (should be denied even with admin policy)

**Real-World Application:**
- Organization-wide security guardrails
- Preventing accidental data deletions
- Enforcing compliance requirements
- Blocking certain regions
- Preventing EC2 instance type changes

**Validation Checklist:**
- [ ] SCPs created
- [ ] Attached to correct OUs
- [ ] S3 deletion prevented
- [ ] CloudTrail modifications blocked
- [ ] Regular policies still allow other operations

---

### Lab 11: Permission Boundaries

**Objective:** Implement maximum permission limits without explicitly denying actions.

**Prerequisites:**
- Labs 1-10 completion
- Understanding of policy evaluation logic

**Lab Steps:**

1. Create permission boundary policy: "DeveloperBoundary"
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*",
        "rds:DescribeDBInstances"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "iam:*",
        "organizations:*"
      ],
      "Resource": "*"
    }
  ]
}
```

2. Create user: "bounded-developer"
3. Set permission boundary on user
4. Attach full admin policy: AdministratorAccess
5. Test: User can access EC2/S3 but cannot access IAM despite admin policy

6. Create role with permission boundary for delegation

**Real-World Application:**
- Developers with broad permissions within specific services
- Preventing privilege escalation through policy attachment
- Controlled delegation of admin privileges
- Compliance frameworks requiring privilege limits

**Validation Checklist:**
- [ ] Permission boundary set on user
- [ ] User has AdministratorAccess policy
- [ ] User can create EC2 instances
- [ ] User cannot modify IAM users
- [ ] Boundary evaluation logs show intersection of policies

---

### Lab 12: Resource Tags and Attribute-Based Access Control (ABAC)

**Objective:** Implement fine-grained access control based on resource and principal tags.

**Prerequisites:**
- Labs 1-11 completion
- Understanding of tagging strategies

**Lab Steps:**

1. Create policy: "ABACProjectAccess"
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2OperationsOnTaggedResources",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:DescribeInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Project": "${aws:PrincipalTag/Project}"
        }
      }
    },
    {
      "Sid": "AllowS3ObjectsMatchingUserTeam",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::company-data/*",
      "Condition": {
        "StringEquals": {
          "s3:ExistingObjectTag/Team": "${aws:PrincipalTag/Team}"
        }
      }
    }
  ]
}
```

2. Create users with tags:
   - User: "alice", Tags: Team=DataEng, Project=Alpha
   - User: "bob", Tags: Team=DevOps, Project=Beta

3. Tag EC2 instances:
   - Instance1: Project=Alpha
   - Instance2: Project=Beta

4. Test access:
   - Alice can manage Instance1 (Project=Alpha)
   - Alice cannot manage Instance2 (Project=Beta)
   - Bob can manage Instance2

**Real-World Application:**
- Multi-tenant environments
- Cost center-based access control
- Environment-based permissions (prod/dev/staging)
- Scaling access control without policy proliferation
- Dynamic access based on organizational structure

**Validation Checklist:**
- [ ] ABAC policy created
- [ ] Users tagged correctly
- [ ] Resources tagged appropriately
- [ ] Access matches tag combinations
- [ ] Cross-project access denied

---

### Lab 13: Audit, Logging, and Access Analysis

**Objective:** Monitor and audit IAM activity using CloudTrail, CloudWatch, and Access Analyzer.

**Prerequisites:**
- Labs 1-12 completion
- S3 bucket for CloudTrail logs

**Lab Steps:**

1. Enable CloudTrail for entire account:
   - Create S3 bucket with encryption
   - CloudTrail → Create Trail
   - Enable for all regions
   - Log API calls and management events

2. Create CloudWatch log group and metric filters:
```bash
# Log unauthorized API calls
{
  ($.errorCode = "*UnauthorizedOperation") || 
  ($.errorCode = "AccessDenied*")
}
```

3. Use IAM Access Analyzer:
   - Analytics → Access Analyzer
   - Select resource type: IAM roles
   - Analyze findings:
     * External access (cross-account/org)
     * Overly permissive policies
     * Unused access

4. Create custom reports:
   - Query CloudTrail logs in Athena
   - Find privilege escalation attempts
   - Track permission changes

5. Set up alerts:
```bash
# Create SNS topic for high-risk actions
# Link to CloudWatch alarms
# Monitor: root account usage, policy changes, role assumption
```

**Real-World Application:**
- Compliance audits (SOC2, ISO 27001)
- Incident response and forensics
- Identifying unused permissions
- Detecting unauthorized access attempts
- Real-time alerts for suspicious activity

**Validation Checklist:**
- [ ] CloudTrail enabled and logging
- [ ] S3 logs encrypted and protected
- [ ] CloudWatch alarms configured
- [ ] Access Analyzer shows findings
- [ ] Can query historical API calls

---

### Lab 14: Privilege Escalation Prevention and Least Privilege Enforcement

**Objective:** Prevent privilege escalation and implement least privilege at scale.

**Prerequisites:**
- Labs 1-13 completion
- Understanding of privilege escalation techniques

**Lab Steps:**

1. Create restrictive policy preventing privilege escalation:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PreventPolicyModification",
      "Effect": "Deny",
      "Action": [
        "iam:CreatePolicyVersion",
        "iam:DeletePolicy",
        "iam:DeletePolicyVersion",
        "iam:DeleteRolePolicy",
        "iam:DeleteUserPolicy",
        "iam:DeleteGroupPolicy",
        "iam:PutUserPolicy",
        "iam:PutRolePolicy",
        "iam:PutGroupPolicy",
        "iam:SetDefaultPolicyVersion"
      ],
      "Resource": "*"
    },
    {
      "Sid": "PreventRoleAssumptionModification",
      "Effect": "Deny",
      "Action": [
        "iam:UpdateAssumeRolePolicy",
        "iam:CreateRole"
      ],
      "Resource": "*"
    },
    {
      "Sid": "PreventUserCreation",
      "Effect": "Deny",
      "Action": [
        "iam:CreateUser",
        "iam:CreateGroup",
        "iam:AddUserToGroup",
        "iam:AttachUserPolicy",
        "iam:AttachGroupPolicy"
      ],
      "Resource": "*"
    },
    {
      "Sid": "PreventAccessKeyGeneration",
      "Effect": "Deny",
      "Action": [
        "iam:CreateAccessKey"
      ],
      "Resource": "arn:aws:iam::*:user/*",
      "Condition": {
        "StringNotEquals": {
          "aws:username": "${aws:PrincipalTag/CanCreateKeys}"
        }
      }
    }
  ]
}
```

2. Test privilege escalation scenarios:
   - Attach policy to self (should fail)
   - Create new admin user (should fail)
   - Modify role trust policy (should fail)
   - Generate access keys (should fail)

3. Implement least privilege workflow:
   - Every policy starts with minimal permissions
   - Requests for additional access go through approval
   - Permissions reviewed quarterly
   - Unused permissions removed automatically

**Real-World Application:**
- Preventing insider threats
- Meeting security audit requirements
- Protecting against compromised credentials
- Maintaining security posture with rapid scaling
- Compliance with principle of least privilege

**Validation Checklist:**
- [ ] Privilege escalation paths blocked
- [ ] Deny policies prevent policy modification
- [ ] Users cannot create new roles with higher privileges
- [ ] All escalation vectors covered
- [ ] Regular users cannot elevate themselves

---

### Lab 15: Multi-Account Strategy and Cross-Account Workflows

**Objective:** Design and implement enterprise multi-account IAM architecture.

**Prerequisites:**
- Labs 1-14 completion
- Multiple AWS accounts (or simulate scenario)
- Understanding of account structures

**Lab Steps:**

1. Design account structure:
```
Organization (Root)
├── Management Account (Billing, Org Management)
├── Security Account (CloudTrail, GuardDuty, Logs)
├── Production Account
│   ├── Production VPC
│   └── Prod-Admin role
├── Staging Account
│   └── Staging-Admin role
└── Development Account
    └── Dev-Admin role
```

2. Create roles in each account:
   - Each account has: Environment-Admin, ReadOnly, Developer roles
   - All trust: Management account

3. In Management account, create roles:
   - SecurityAudit: Can assume read-only roles in all accounts
   - ProdAdmin: Can assume prod-admin in Production account only
   - DeveloperRole: Can assume dev-admin in Dev account only

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": [
        "arn:aws:iam::PROD-ACCOUNT-ID:role/ProdAdminRole",
        "arn:aws:iam::DEV-ACCOUNT-ID:role/DevAdminRole"
      ],
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id"
        },
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

4. Implement cross-account logging:
   - CloudTrail in each account delivers to Security account
   - Security account centralizes all logs
   - Create log aggregation rules

5. Set up cross-account permissions:
   - Lambda in Prod can assume role in Security to write logs
   - RDS backup can be copied across accounts
   - AMIs shared between accounts with permission boundaries

6. Test workflows:
```bash
# Assume production role from management account
aws sts assume-role \
  --role-arn arn:aws:iam::PROD-ACCOUNT-ID:role/ProdAdminRole \
  --role-session-name prod-session \
  --external-id unique-external-id

# Now operate in production account
aws ec2 describe-instances --region us-east-1
```

**Real-World Application:**
- Enterprise-grade AWS governance
- Workload isolation and security boundaries
- Regulatory compliance (separate prod/dev accounts)
- Cost allocation and billing per department
- Automated resource provisioning across accounts
- Disaster recovery with cross-account replication

**Validation Checklist:**
- [ ] Multiple accounts created and organized
- [ ] Cross-account roles configured
- [ ] External IDs used for extra security
- [ ] Can assume roles from management account
- [ ] CloudTrail aggregates across accounts
- [ ] Least privilege enforced per account
- [ ] Resource sharing works correctly

---

## Summary: Key Concepts Covered

| Lab | Core Concept | Real-World Use |
|-----|--------------|----------------|
| 1-3 | Users, Groups, Roles | Basic access management |
| 4-6 | Policies & Resources | Fine-grained access control |
| 7-9 | Authentication | Enterprise security |
| 10-11 | SCPs & Boundaries | Organization governance |
| 12 | ABAC | Multi-tenant/scaled access |
| 13 | Audit & Logging | Compliance & forensics |
| 14 | Privilege Escalation | Threat prevention |
| 15 | Multi-Account | Enterprise architecture |

---

## Recommended Learning Path

**Week 1:** Labs 1-3 (Users, Groups, Roles)
**Week 2:** Labs 4-6 (Policies & Resource-based access)
**Week 3:** Labs 7-9 (Authentication & Federation)
**Week 4:** Labs 10-12 (Advanced organization controls)
**Week 5:** Labs 13-14 (Audit & Security hardening)
**Week 6:** Lab 15 (Multi-account enterprise setup)

---

## Best Practices Summary

1. **Always use roles instead of access keys** for applications
2. **Enable MFA on root account** immediately
3. **Use permission boundaries** for privilege escalation prevention
4. **Implement least privilege** from day one
5. **Tag everything** for ABAC and cost tracking
6. **Enable CloudTrail** in all accounts
7. **Use SCPs** to enforce organization-wide policies
8. **Regularly audit** using Access Analyzer
9. **Implement external IDs** for cross-account trust
10. **Automate permission removal** for unused access
