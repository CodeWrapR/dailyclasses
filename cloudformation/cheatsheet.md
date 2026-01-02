# CloudFormation Keywords & Intrinsic Functions Cheatsheet

## Intrinsic Functions (Key Keywords)

### Reference Functions

#### !Ref (Reference)
Returns the value of a specified parameter or resource.

**Syntax:**
```yaml
!Ref ResourceLogicalId
!Ref ParameterName
```

**Usage Examples:**
```yaml
# Reference a parameter
SubnetId: !Ref MySubnet

# Reference a resource property
VpcId: !Ref MyVPC

# In output
Value: !Ref EC2Instance
```

**Returns:** Resource ID or parameter value

**Common Resources:**
- EC2 Instance → instance ID
- S3 Bucket → bucket name
- RDS Instance → database identifier

---

#### !GetAtt (Get Attribute)
Retrieves specific attributes of a resource. Used for values that aren't the primary identifier.

**Syntax:**
```yaml
!GetAtt ResourceLogicalId.AttributeName
```

**Usage Examples:**
```yaml
# Get EC2 public IP
PublicIP: !GetAtt EC2Instance.PublicIp

# Get S3 bucket ARN
BucketArn: !GetAtt MyBucket.Arn

# Get RDS endpoint
Endpoint: !GetAtt RDSInstance.Endpoint.Address

# Get database port
Port: !GetAtt RDSInstance.Endpoint.Port
```

**Common Attributes by Resource:**

| Resource | Common Attributes |
|----------|------------------|
| EC2::Instance | PublicIp, PrivateIp, PublicDnsName |
| S3::Bucket | Arn, DomainName |
| RDS::DBInstance | Endpoint.Address, Endpoint.Port |
| ALB | DNSName, HostedZoneId |
| IAM::Role | Arn |
| Lambda::Function | Arn |
| SQS::Queue | Arn, QueueUrl |

---

#### !GetAZs (Get Availability Zones)
Returns a list of availability zones for the region.

**Syntax:**
```yaml
!GetAZs RegionName
!GetAZs ''  # Empty string = current region
```

**Usage Examples:**
```yaml
# Get first AZ
AvailabilityZone: !Select [0, !GetAZs '']

# Get second AZ
AvailabilityZone: !Select [1, !GetAZs '']

# Get all AZs for specific region
AZs: !GetAZs us-east-1
```

**Returns:** List of AZ names

---

### Conditional Functions

#### !If (Conditional)
Returns one value if a condition is true, another if false.

**Syntax:**
```yaml
!If [ConditionName, ValueIfTrue, ValueIfFalse]
```

**Usage Examples:**
```yaml
# Conditional property value
DBInstanceClass: !If [IsProduction, db.t3.medium, db.t3.micro]

# Conditional string
Environment: !If [IsProd, Production, Development]

# Conditional number
VersioningStatus: !If [EnableVersioning, Enabled, Suspended]
```

**Must have condition defined:**
```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
```

---

#### !Equals (Equals)
Compares two values. Returns true if equal, false otherwise.

**Syntax:**
```yaml
!Equals [Value1, Value2]
```

**Usage Examples:**
```yaml
# Compare parameter to string
IsProduction: !Equals [!Ref Environment, prod]

# Compare two parameters
SameRegion: !Equals [!Ref Region1, !Ref Region2]

# Compare AWS::StackName to string
IsTestStack: !Equals [!Ref 'AWS::StackName', test-stack]
```

**Returns:** true or false

---

#### !Not (Not)
Negates a condition (reverses true/false).

**Syntax:**
```yaml
!Not [Condition]
```

**Usage Examples:**
```yaml
IsNotProduction: !Not [!Equals [!Ref Environment, prod]]

IsDisabled: !Not [!Equals [!Ref EnableFeature, 'true']]
```

---

#### !And (And)
Returns true only if ALL conditions are true.

**Syntax:**
```yaml
!And [Condition1, Condition2, ...]
```

**Usage Examples:**
```yaml
# All conditions must be true
ShouldCreateDatabase: !And
  - !Equals [!Ref Environment, prod]
  - !Equals [!Ref EnableDatabase, 'true']
  - !Equals [!Ref HasBackup, 'true']
```

---

#### !Or (Or)
Returns true if ANY condition is true.

**Syntax:**
```yaml
!Or [Condition1, Condition2, ...]
```

**Usage Examples:**
```yaml
# At least one condition must be true
IsHighEnv: !Or
  - !Equals [!Ref Environment, prod]
  - !Equals [!Ref Environment, staging]
```

---

### String Functions

#### !Sub (Substitute)
Performs string substitution. Replaces variables with their values.

**Syntax:**
```yaml
!Sub String
!Sub [String, {Var1: Value1, Var2: Value2}]
```

**Usage Examples:**
```yaml
# Simple substitution with references
StackName: !Sub '${AWS::StackName}-bucket'

# Multiple substitutions
Description: !Sub 'Bucket for ${ProjectName} in ${AWS::Region}'

# With explicit mappings
Identifier: !Sub
  - '${StackName}-${Env}'
  - StackName: !Ref 'AWS::StackName'
    Env: !Ref Environment

# Commonly used pseudo-parameters
UniqueId: !Sub '${AWS::StackId}-${AWS::AccountId}'
```

**Common Pseudo-Parameters:**
```yaml
${AWS::StackName}        # Stack name
${AWS::StackId}          # Stack ID
${AWS::Region}           # Region
${AWS::AccountId}        # AWS account ID
${AWS::NotificationARNs} # SNS topic ARNs
${AWS::Partition}        # aws or aws-cn
```

---

#### !Join (Join)
Concatenates values separated by a delimiter.

**Syntax:**
```yaml
!Join [Delimiter, [Value1, Value2, ...]]
```

**Usage Examples:**
```yaml
# Join with comma
Tags: !Join [',', [tag1, tag2, tag3]]

# Join with slash
Path: !Join ['/', [root, folder, file.txt]]

# Join with colons (for ARNs)
Arn: !Join [':', ['arn:aws:service', !Ref 'AWS::Region', !Ref 'AWS::AccountId']]
```

---

#### !Split (Split)
Splits a string by a delimiter, returns a list.

**Syntax:**
```yaml
!Split [Delimiter, String]
```

**Usage Examples:**
```yaml
# Split comma-separated string
Tags: !Split [',', 'tag1,tag2,tag3']

# Extract region from ARN
Region: !Select [3, !Split [':', !Ref ARN]]
```

---

### List Functions

#### !Select (Select)
Returns a single object from a list by index.

**Syntax:**
```yaml
!Select [Index, List]
```

**Usage Examples:**
```yaml
# Get first AZ
AvailabilityZone: !Select [0, !GetAZs '']

# Get second AZ
AvailabilityZone: !Select [1, !GetAZs '']

# Get first element from parameter list
FirstVpc: !Select [0, !Ref VpcList]
```

**Index:** 0-based (0 is first element)

---

#### !GetAZs (covered above)
Get list of availability zones.

---

### Parsing Functions

#### !ImportValue (Import Value)
Imports values exported from other stacks using cross-stack references.

**Syntax:**
```yaml
!ImportValue ExportedValueName
```

**Usage Examples:**
```yaml
# Import VPC ID from another stack
VpcId: !ImportValue MyProject-VpcId

# Import with Sub for dynamic names
SubnetId: !ImportValue !Sub '${ProjectName}-PublicSubnetId'
```

**Requires:** Stack must export the value using Outputs

---

### Base64 Functions

#### !Base64 (Base64)
Encodes a string in Base64 format. Commonly used for UserData scripts.

**Syntax:**
```yaml
!Base64 String
```

**Usage Examples:**
```yaml
# Encode script for EC2 UserData
UserData: !Base64
  Fn::Sub: |
    #!/bin/bash
    echo "Stack: ${AWS::StackName}" > /tmp/info.txt
    yum update -y
```

---

### Transformation Functions

#### Fn::Transform
Applies a macro to a template section.

**Syntax:**
```yaml
Transform: AWS::Include
Name: TransformLogicalId
```

**Usage Examples:**
```yaml
# Include external template
Transform: AWS::Include
Location: s3://my-bucket/snippets.yaml
```

---

## Conditions Block

### Define Conditions
Conditions are boolean expressions used to control resource creation.

**Syntax:**
```yaml
Conditions:
  ConditionName: !Expression
```

**Usage Example:**
```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
  IsHighEnv: !Or
    - !Equals [!Ref Environment, prod]
    - !Equals [!Ref Environment, staging]
  IsNotProduction: !Not [!Equals [!Ref Environment, prod]]
  HasDatabase: !And
    - !Equals [!Ref EnableDB, 'true']
    - !Equals [!Ref Environment, prod]
```

---

## Using Conditions on Resources

```yaml
Resources:
  MyResource:
    Type: AWS::S3::Bucket
    Condition: IsProduction  # Only create if IsProduction is true
    Properties:
      BucketName: my-bucket
```

---

## Using Conditions on Outputs

```yaml
Outputs:
  DatabaseEndpoint:
    Condition: HasDatabase  # Only output if HasDatabase is true
    Value: !GetAtt RDSInstance.Endpoint.Address
```

---

## Template Metadata Keywords

#### AWSTemplateFormatVersion
Specifies the template version.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
```

---

#### Description
Describes what the template does.

```yaml
Description: This template creates a VPC with EC2 instance
```

---

#### Metadata
Metadata about the template (AWS::CloudFormation::Init, AWS::CloudFormation::Designer).

```yaml
Metadata:
  AWS::CloudFormation::Init:
    config:
      packages:
        yum:
          httpd: []
```

---

#### Transform
Serverless Application Model (SAM) or macro transformations.

```yaml
Transform: AWS::Serverless-2016-10-31
```

---

#### Parameters
Input values for the template.

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    Description: EC2 instance type
```

---

#### Mappings
Static mapping tables (like lookup tables).

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-12345678
    us-west-2:
      AMI: ami-87654321
```

**Usage:**
```yaml
ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
```

---

#### Resources (Required)
The AWS resources to create.

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-bucket
```

---

#### Outputs
Return values from the stack.

```yaml
Outputs:
  BucketName:
    Description: Name of the bucket
    Value: !Ref MyBucket
    Export:
      Name: MyBucketName  # For cross-stack reference
```

---

## Resource Properties

#### Type
Specifies the AWS resource type.

```yaml
Type: AWS::EC2::Instance
Type: AWS::S3::Bucket
Type: AWS::RDS::DBInstance
```

---

#### Properties
Configure the resource.

```yaml
Properties:
  BucketName: my-bucket
  VersioningConfiguration:
    Status: Enabled
```

---

#### CreationPolicy
Wait for resource to reach desired state.

```yaml
CreationPolicy:
  ResourceSignal:
    Count: 1
    Timeout: PT15M
```

---

#### DeletionPolicy
Define what happens when stack is deleted.

```yaml
DeletionPolicy: Delete      # Default: delete resource
DeletionPolicy: Retain      # Keep resource
DeletionPolicy: Snapshot    # Create snapshot before delete
DeletionPolicy: RetainExceptOnCreate
```

---

#### UpdateReplacePolicy
What happens when resource is replaced during update.

```yaml
UpdateReplacePolicy: Delete
UpdateReplacePolicy: Retain
```

---

#### DependsOn
Explicit resource dependency.

```yaml
DependsOn: MySecurityGroup  # Create security group first
```

---

#### Tags
Key-value pairs for resource tagging.

```yaml
Tags:
  - Key: Name
    Value: my-resource
  - Key: Environment
    Value: production
```

---

## Quick Reference Table

| Function | Use Case | Returns |
|----------|----------|---------|
| !Ref | Get resource ID or parameter value | ID/String |
| !GetAtt | Get specific resource property | Attribute value |
| !GetAZs | List availability zones | List |
| !Select | Get item from list by index | Single item |
| !Join | Concatenate values | String |
| !Split | Split string into list | List |
| !Sub | String substitution | String |
| !If | Conditional value | Value |
| !Equals | Compare values | Boolean |
| !Not | Negate condition | Boolean |
| !And | All conditions true | Boolean |
| !Or | Any condition true | Boolean |
| !Base64 | Encode to Base64 | Encoded string |
| !ImportValue | Cross-stack reference | Imported value |
| !FindInMap | Lookup from mappings | Mapped value |

---

## Common Patterns

### Creating Unique Names
```yaml
BucketName: !Sub '${AWS::StackName}-${AWS::AccountId}'
```

### Conditional Resource Creation
```yaml
Conditions:
  CreateProdResources: !Equals [!Ref Environment, prod]

Resources:
  ProducetionDB:
    Type: AWS::RDS::DBInstance
    Condition: CreateProdResources
```

### Cross-Stack Reference
**Stack A:**
```yaml
Outputs:
  VpcId:
    Export:
      Name: MyProject-VpcId
    Value: !Ref MyVPC
```

**Stack B:**
```yaml
VpcId: !ImportValue MyProject-VpcId
```

### Environment-Based Configuration
```yaml
Mappings:
  EnvironmentConfig:
    dev:
      InstanceType: t3.micro
      DBSize: 20
    prod:
      InstanceType: t3.medium
      DBSize: 100

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvironmentConfig, !Ref Environment, InstanceType]
```

### Conditional Output
```yaml
Outputs:
  DatabaseEndpoint:
    Condition: CreateDatabase
    Description: RDS endpoint
    Value: !GetAtt RDSInstance.Endpoint.Address
```

---

## Tips & Tricks

1. **Always use !Ref for simple resource references** - it's cleaner than !GetAtt
2. **Use !Sub for complex string building** - more readable than !Join
3. **Conditions are evaluated at stack creation time** - they're not dynamic
4. **Use Mappings for lookup tables** - cleaner than many conditional statements
5. **Export important values** - enables cross-stack references
6. **Pseudo parameters start with AWS::** - always available
7. **Index in !Select is 0-based** - first item is index 0
8. **GetAZs with empty string** - gets AZs for the current region
9. **Use DependsOn for explicit ordering** - when implicit dependencies aren't detected
10. **Tag everything** - makes resource organization and cost tracking easier
