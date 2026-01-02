# CloudFormation Tutorial: Basic to Advanced

## Table of Contents
1. [CloudFormation Fundamentals](#cloudformation-fundamentals)
2. [Basic Project: VPC Setup](#basic-project-vpc-setup)
3. [Intermediate Project: Web Server Stack](#intermediate-project-web-server-stack)
4. [Advanced Project: Multi-Tier Application](#advanced-project-multi-tier-application)
5. [Expert Project: CI/CD Pipeline](#expert-project-cicd-pipeline)

---

## CloudFormation Fundamentals

### What is CloudFormation?

CloudFormation is an Infrastructure as Code (IaC) service that allows you to define and provision AWS infrastructure using templates written in JSON or YAML.

### Key Concepts

**Template**: A JSON or YAML file describing your AWS infrastructure
**Stack**: A collection of AWS resources created from a template
**Parameter**: Input variable that customizes template behavior
**Output**: Values returned from the stack
**Resource**: An AWS service component (EC2, RDS, S3, etc.)
**Mapping**: Key-value pairs for conditional values
**Condition**: Logical statements controlling resource creation

### Template Structure

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Template description

Parameters:
  # Input variables

Mappings:
  # Key-value lookup tables

Conditions:
  # Conditional logic

Resources:
  # AWS resources to create

Outputs:
  # Return values
```

### CloudFormation Benefits

- Infrastructure as code for version control
- Consistent, repeatable deployments
- Automatic resource management
- Easy stack updates and deletion
- Cross-stack references
- Drift detection
- Cost estimation before deployment

### Template Format

JSON and YAML are both supported. YAML is more readable and commonly used.

---

## Basic Project: VPC Setup

### Project Goal
Create a basic VPC with subnets, route tables, and internet gateway using CloudFormation.

### Template: basic-vpc.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Basic VPC with public and private subnets'

Parameters:
  VPCCidr:
    Type: String
    Default: 10.0.0.0/16
    Description: CIDR block for VPC

  PublicSubnetCidr:
    Type: String
    Default: 10.0.1.0/24
    Description: CIDR block for public subnet

  PrivateSubnetCidr:
    Type: String
    Default: 10.0.2.0/24
    Description: CIDR block for private subnet

Resources:
  # VPC
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VPCCidr
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: BasicVPC

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: BasicIGW

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MyVPC
      InternetGatewayId: !Ref InternetGateway

  # Public Subnet
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: !Ref PublicSubnetCidr
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet

  # Private Subnet
  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: !Ref PrivateSubnetCidr
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: PrivateSubnet

  # Route Table for Public Subnet
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: PublicRT

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnetAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  # Route Table for Private Subnet
  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: PrivateRT

  PrivateSubnetAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateSubnet
      RouteTableId: !Ref PrivateRouteTable

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref MyVPC
    Export:
      Name: !Sub '${AWS::StackName}-VPC-ID'

  PublicSubnetId:
    Description: Public Subnet ID
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnet-ID'

  PrivateSubnetId:
    Description: Private Subnet ID
    Value: !Ref PrivateSubnet
    Export:
      Name: !Sub '${AWS::StackName}-PrivateSubnet-ID'
```

### Deploy Stack

```bash
# Create stack
aws cloudformation create-stack \
  --stack-name basic-vpc-stack \
  --template-body file://basic-vpc.yaml \
  --parameters ParameterKey=VPCCidr,ParameterValue=10.0.0.0/16

# Monitor creation
aws cloudformation describe-stacks \
  --stack-name basic-vpc-stack \
  --query 'Stacks[0].StackStatus'

# Get outputs
aws cloudformation describe-stacks \
  --stack-name basic-vpc-stack \
  --query 'Stacks[0].Outputs'

# Delete stack
aws cloudformation delete-stack --stack-name basic-vpc-stack
```

---

## Intermediate Project: Web Server Stack

### Project Goal
Create an EC2 instance with Apache web server, security groups, and Elastic IP using parameters and conditions.

### Template: web-server.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Web server with Apache and custom configuration'

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    Description: EC2 instance type

  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 Key Pair for SSH access

  SSHLocation:
    Type: String
    Default: 0.0.0.0/0
    Description: IP address range for SSH access

  Environment:
    Type: String
    Default: Development
    AllowedValues:
      - Development
      - Production
    Description: Environment type

  VPCId:
    Type: AWS::EC2::VPC::Id
    Description: VPC ID for security group

  SubnetId:
    Type: AWS::EC2::Subnet::Id
    Description: Subnet ID for instance

Conditions:
  IsProduction: !Equals [!Ref Environment, Production]

Mappings:
  RegionAMI:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-2:
      AMI: ami-0d1cd67c26f5fca19
    eu-west-1:
      AMI: ami-0dad359ff462124ca

Resources:
  # Security Group
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for web server
      VpcId: !Ref VPCId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref SSHLocation
          Description: SSH access
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP access
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
          Description: HTTPS access
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 0.0.0.0/0
          Description: Allow all outbound traffic
      Tags:
        - Key: Name
          Value: WebServerSG

  # IAM Role for EC2
  EC2Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

  EC2InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Roles:
        - !Ref EC2Role

  # Elastic IP (only for production)
  ElasticIP:
    Type: AWS::EC2::EIP
    Condition: IsProduction
    Properties:
      Domain: vpc
      Tags:
        - Key: Name
          Value: WebServerEIP

  # EC2 Instance
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionAMI, !Ref 'AWS::Region', AMI]
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyPairName
      SubnetId: !Ref SubnetId
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      IamInstanceProfile: !Ref EC2InstanceProfile
      EbsOptimized: !If [IsProduction, true, false]
      Monitoring: !If [IsProduction, true, false]
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl enable httpd
          systemctl start httpd
          
          cat > /var/www/html/index.html <<EOF
          <html>
            <head><title>CloudFormation Web Server</title></head>
            <body>
              <h1>Hello from ${AWS::StackName}</h1>
              <p>Environment: ${Environment}</p>
              <p>Instance ID: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</p>
            </body>
          </html>
          EOF
      Tags:
        - Key: Name
          Value: WebServer
        - Key: Environment
          Value: !Ref Environment

  # Associate Elastic IP (only for production)
  AssociateElasticIP:
    Type: AWS::EC2::EIPAssociation
    Condition: IsProduction
    Properties:
      InstanceId: !Ref WebServerInstance
      AllocationId: !GetAtt ElasticIP.AllocationId

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref WebServerInstance

  PublicIP:
    Description: Public IP address
    Value: !If [IsProduction, !Ref ElasticIP, !GetAtt WebServerInstance.PublicIp]

  WebURL:
    Description: Web server URL
    Value: !Sub 'http://${WebServerInstance.PublicDnsName}'

  SecurityGroupId:
    Description: Security Group ID
    Value: !Ref WebServerSecurityGroup
```

### Deploy with Parameters

```bash
# Create stack with parameters
aws cloudformation create-stack \
  --stack-name web-server-stack \
  --template-body file://web-server.yaml \
  --parameters \
    ParameterKey=InstanceType,ParameterValue=t2.micro \
    ParameterKey=KeyPairName,ParameterValue=your-key-pair \
    ParameterKey=Environment,ParameterValue=Production \
    ParameterKey=VPCId,ParameterValue=vpc-xxxxx \
    ParameterKey=SubnetId,ParameterValue=subnet-xxxxx

# Update stack with new parameters
aws cloudformation update-stack \
  --stack-name web-server-stack \
  --template-body file://web-server.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.small \
    UsePreviousValue=true
```

---

## Advanced Project: Multi-Tier Application

### Project Goal
Create a complete three-tier architecture with nested stacks, cross-stack references, and modular components.

### Master Template: master-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Master stack for three-tier application'

Parameters:
  Environment:
    Type: String
    Default: development
    AllowedValues:
      - development
      - production
    Description: Environment name

  VPCCidr:
    Type: String
    Default: 10.0.0.0/16
    Description: VPC CIDR block

  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 Key Pair

Resources:
  # VPC Stack
  VPCStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      StackName: !Sub '${AWS::StackName}-vpc'
      TemplateURL: https://s3.amazonaws.com/your-bucket/vpc-stack.yaml
      Parameters:
        VPCCidr: !Ref VPCCidr
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # Security Groups Stack
  SecurityGroupStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      StackName: !Sub '${AWS::StackName}-sg'
      TemplateURL: https://s3.amazonaws.com/your-bucket/security-groups.yaml
      Parameters:
        VPCId: !GetAtt VPCStack.Outputs.VPCId
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # Application Stack
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: 
      - VPCStack
      - SecurityGroupStack
    Properties:
      StackName: !Sub '${AWS::StackName}-app'
      TemplateURL: https://s3.amazonaws.com/your-bucket/application-stack.yaml
      Parameters:
        VPCId: !GetAtt VPCStack.Outputs.VPCId
        PublicSubnetId: !GetAtt VPCStack.Outputs.PublicSubnetId
        PrivateSubnetId: !GetAtt VPCStack.Outputs.PrivateSubnetId
        WebServerSecurityGroupId: !GetAtt SecurityGroupStack.Outputs.WebServerSGId
        KeyPairName: !Ref KeyPairName
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # Database Stack
  DatabaseStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: ApplicationStack
    Properties:
      StackName: !Sub '${AWS::StackName}-db'
      TemplateURL: https://s3.amazonaws.com/your-bucket/database-stack.yaml
      Parameters:
        VPCId: !GetAtt VPCStack.Outputs.VPCId
        DatabaseSubnet1: !GetAtt VPCStack.Outputs.DatabaseSubnet1
        DatabaseSubnet2: !GetAtt VPCStack.Outputs.DatabaseSubnet2
        AppSecurityGroupId: !GetAtt SecurityGroupStack.Outputs.AppServerSGId
      Tags:
        - Key: Environment
          Value: !Ref Environment

Outputs:
  VPCId:
    Description: VPC ID
    Value: !GetAtt VPCStack.Outputs.VPCId
    Export:
      Name: !Sub '${AWS::StackName}-VPC-ID'

  ApplicationStackURL:
    Description: Application URL
    Value: !GetAtt ApplicationStack.Outputs.LoadBalancerURL

  DatabaseEndpoint:
    Description: Database endpoint
    Value: !GetAtt DatabaseStack.Outputs.DatabaseEndpoint
    Export:
      Name: !Sub '${AWS::StackName}-DB-Endpoint'
```

### VPC Stack: vpc-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'VPC and networking resources'

Parameters:
  VPCCidr:
    Type: String
    Default: 10.0.0.0/16

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VPCCidr
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: AppVPC

  InternetGateway:
    Type: AWS::EC2::InternetGateway

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [0, !GetAZs '']

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.3.0/24
      AvailabilityZone: !Select [1, !GetAZs '']

  DatabaseSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.4.0/24
      AvailabilityZone: !Select [0, !GetAZs '']

  DatabaseSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.5.0/24
      AvailabilityZone: !Select [1, !GetAZs '']

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC

  PublicRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnetAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

Outputs:
  VPCId:
    Value: !Ref VPC

  PublicSubnetId:
    Value: !Ref PublicSubnet

  PrivateSubnetId:
    Value: !Ref PrivateSubnet1

  DatabaseSubnet1:
    Value: !Ref DatabaseSubnet1

  DatabaseSubnet2:
    Value: !Ref DatabaseSubnet2
```

---

## Expert Project: CI/CD Pipeline

### Project Goal
Create a complete CI/CD pipeline with CodePipeline, CodeBuild, and CodeDeploy integrated with CloudFormation.

### Template: cicd-pipeline.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CI/CD Pipeline with CodePipeline, CodeBuild, and CloudFormation'

Parameters:
  GitHubRepo:
    Type: String
    Description: GitHub repository URL

  GitHubBranch:
    Type: String
    Default: main
    Description: GitHub branch to deploy

  GitHubToken:
    Type: String
    NoEcho: true
    Description: GitHub personal access token

Resources:
  # S3 Artifact Store
  ArtifactStore:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'pipeline-artifacts-${AWS::AccountId}-${AWS::Region}'
      VersioningConfiguration:
        Status: Enabled
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true

  # CodeBuild Project
  BuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      Name: !Sub '${AWS::StackName}-build'
      ServiceRole: !GetAtt CodeBuildRole.Arn
      Artifacts:
        Type: CODEPIPELINE
      Environment:
        ComputeType: BUILD_GENERAL1_SMALL
        Image: aws/codebuild/standard:5.0
        EnvironmentVariables:
          - Name: AWS_ACCOUNT_ID
            Value: !Ref AWS::AccountId
          - Name: AWS_DEFAULT_REGION
            Value: !Ref AWS::Region
      Source:
        Type: CODEPIPELINE
        BuildSpec: |
          version: 0.2
          phases:
            pre_build:
              commands:
                - echo "Running tests..."
                - echo "Building application..."
            build:
              commands:
                - echo "Packaging application..."
                - aws cloudformation package --template-file template.yaml --s3-bucket $ARTIFACT_BUCKET --output-template-file packaged.yaml
            post_build:
              commands:
                - echo "Build completed"
          artifacts:
            files:
              - packaged.yaml
              - '**/*'

  # CodePipeline
  Pipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      Name: !Sub '${AWS::StackName}-pipeline'
      RoleArn: !GetAtt CodePipelineRole.Arn
      ArtifactStore:
        Type: S3
        Location: !Ref ArtifactStore
      Stages:
        # Source Stage
        - Name: Source
          Actions:
            - Name: SourceAction
              ActionTypeId:
                Category: Source
                Owner: ThirdParty
                Provider: GitHub
                Version: '1'
              Configuration:
                Owner: !Select [3, !Split ['/', !Ref GitHubRepo]]
                Repo: !Select [4, !Split ['/', !Ref GitHubRepo]]
                Branch: !Ref GitHubBranch
                OAuthToken: !Ref GitHubToken
              OutputArtifacts:
                - Name: SourceOutput

        # Build Stage
        - Name: Build
          Actions:
            - Name: BuildAction
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: '1'
              Configuration:
                ProjectName: !Ref BuildProject
              InputArtifacts:
                - Name: SourceOutput
              OutputArtifacts:
                - Name: BuildOutput

        # Deploy Stage
        - Name: Deploy
          Actions:
            - Name: DeployAction
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: CloudFormation
                Version: '1'
              Configuration:
                ActionMode: CREATE_UPDATE
                StackName: !Sub '${AWS::StackName}-app'
                TemplatePath: BuildOutput::packaged.yaml
                Capabilities: CAPABILITY_IAM,CAPABILITY_NAMED_IAM
                RoleArn: !GetAtt CloudFormationRole.Arn
              InputArtifacts:
                - Name: BuildOutput

  # IAM Roles
  CodeBuildRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: codebuild.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser
      Policies:
        - PolicyName: BuildPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                  - s3:GetObject
                  - s3:PutObject
                  - cloudformation:ValidateTemplate
                Resource: '*'

  CodePipelineRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: codepipeline.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AWSCodePipelineFullAccess
      Policies:
        - PolicyName: PipelinePolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - s3:*
                  - codebuild:BatchGetBuilds
                  - codebuild:BatchGetProjects
                  - codebuild:StartBuild
                  - cloudformation:*
                  - iam:PassRole
                Resource: '*'

  CloudFormationRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: cloudformation.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AdministratorAccess

Outputs:
  PipelineUrl:
    Description: CodePipeline URL
    Value: !Sub 'https://console.aws.amazon.com/codesuite/codepipeline/pipelines/${Pipeline}/view'

  ArtifactBucket:
    Description: S3 bucket for artifacts
    Value: !Ref ArtifactStore
```

---

## Best Practices and Tips

### Template Design
- Keep templates modular and reusable
- Use nested stacks for complex infrastructure
- Implement parameters for flexibility
- Export outputs for cross-stack references
- Use descriptive resource logical IDs
- Add comprehensive tags for resource management

### Security
- Use IAM roles instead of hardcoded credentials
- Implement least privilege access
- Encrypt sensitive parameters
- Use CloudFormation drift detection
- Implement change sets before updates
- Enable CloudTrail logging

### Performance
- Use depends_on for complex dependencies
- Implement parallelization where possible
- Use CloudFormation change sets for updates
- Monitor stack creation with events
- Implement rollback on failure

### Common Functions

**Intrinsic Functions**
- `!Ref`: Reference parameters or resources
- `!GetAtt`: Get resource attributes
- `!Sub`: String substitution
- `!Select`: Select from lists
- `!Join`: Join list elements
- `!FindInMap`: Lookup mapping values
- `!If`: Conditional logic
- `!GetAZs`: Get availability zones

### Debugging

```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Create change set before updating
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template.yaml \
  --changes Summary

# Detect drift
aws cloudformation detect-stack-drift --stack-name my-stack

# Get stack events
aws cloudformation describe-stack-events --stack-name my-stack
```

---

## Resources

- AWS CloudFormation Documentation: https://docs.aws.amazon.com/cloudformation/
- CloudFormation User Guide: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/
- AWS CloudFormation Best Practices: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html
- CloudFormation Template Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html
