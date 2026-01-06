# Store Secrets in AWS Secrets Manager - GUI Guide

A step-by-step guide to securely store secrets using the AWS Management Console.

## Prerequisites

- AWS Account with appropriate permissions
- Access to AWS Management Console

## Step 1: Navigate to Secrets Manager

1. Go to [AWS Management Console](https://console.aws.amazon.com)
2. Sign in with your credentials
3. Search for **"Secrets Manager"** in the search bar
4. Click on **AWS Secrets Manager** service

## Step 2: Create a New Secret

1. Click the **"Store a new secret"** button (orange button on top right)
2. You'll see the "Store a secret" wizard

## Step 3: Choose Secret Type

In the "Choose secret type" section, select one of the following:

### Option 1: Other Type of Secret (Generic)
- Use this for custom secrets (API keys, passwords, tokens)
- Click **"Other type of secret"**

### Option 2: Database Credentials
- Use this for database passwords
- Click **"Credentials for RDS database"**
- Select the database type (MySQL, PostgreSQL, Aurora, etc.)
- It will show your RDS instances to choose from

### Option 3: API Key or Other
- Click **"Other type of secret"** and manually enter your secret

## Step 4: Enter Secret Value

### For Generic Secrets (Key/Value Format)

1. In the "Key/value" tab, add your secret details:
   - **Key**: `username`
   - **Value**: `admin`

2. Click **"Add row"** to add more key-value pairs:
   - **Key**: `password`
   - **Value**: `MySecurePassword123!`

3. Click **"Add row"** for additional fields if needed:
   - **Key**: `host`
   - **Value**: `db.example.com`

### Alternatively, Use Plaintext Tab

1. Click the **"Plaintext"** tab
2. Paste your secret as plain text or JSON:

```json
{
  "username": "admin",
  "password": "MySecurePassword123!",
  "host": "db.example.com",
  "port": 3306
}
```

## Step 5: Set Encryption

1. Under **"Encryption key"**, choose:
   - **aws/secretsmanager** (AWS managed key - Free)
   - Or select a **customer managed key** if you have one

2. Click **"Next"**

## Step 6: Name and Description

1. Enter a **Secret name** (required):
   - Example: `my-database-credentials`
   - Use alphanumeric characters and underscores
   - Avoid hyphens at the end

2. Add a **Description** (optional):
   - Example: "Database credentials for production MySQL"

3. Add **Tags** (optional):
   - Key: `Environment`
   - Value: `Production`
   
   - Key: `Application`
   - Value: `MyApp`

4. Click **"Next"**

## Step 7: Configure Rotation (Optional)

1. Choose **"Disable automatic rotation"** for now (or enable if needed)
2. You can enable rotation later
3. Click **"Next"**

## Step 8: Review and Store

1. Review all your settings:
   - Secret type
   - Name
   - Description
   - Tags
   - Encryption

2. Click **"Store secret"** button

3. You'll see a success message with the secret ARN

## Step 9: Copy Your Secret ARN

After creation, copy the **Secret ARN** from the success page. It looks like:
```
arn:aws:secretsmanager:us-east-1:123456789012:secret:my-database-credentials-abcde
```

## Viewing Your Secret

1. Click on your secret name from the secrets list
2. You'll see the **"Secret details"** page with:
   - Secret name
   - ARN
   - Created date
   - Last accessed date
   - Encryption key

3. To view the secret value, click **"Retrieve secret value"** (if you have permissions)

## Using Your Secret in Lambda

Once created, reference it in your Lambda function:

```python
import boto3
import json

def lambda_handler(event, context):
    client = boto3.client('secretsmanager', region_name='us-east-1')
    
    try:
        response = client.get_secret_value(SecretId='my-database-credentials')
        secret = json.loads(response['SecretString'])
        
        username = secret['username']
        password = secret['password']
        host = secret['host']
        
        return {
            'statusCode': 200,
            'body': json.dumps({'message': 'Secret retrieved successfully'})
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

## Managing Your Secrets

### View All Secrets
1. Go to Secrets Manager
2. You'll see a list of all your secrets
3. Click on any secret to view details

### Update a Secret

1. Click on your secret
2. Click **"Edit secret"**
3. Modify the key-value pairs
4. Click **"Save changes"**
5. A new version is automatically created

### Delete a Secret

1. Click on your secret
2. Click **"Delete secret"**
3. Choose deletion window (7-30 days)
4. Confirm deletion

### View Secret Versions

1. Click on your secret
2. Scroll to **"Versions"** section
3. See all versions with staging labels (AWSCURRENT, AWSPENDING)

## Best Practices

1. **Use Descriptive Names**: `prod/db/credentials` instead of `secret1`
2. **Add Tags**: Organize by environment, application, team
3. **Enable Rotation**: Set up automatic rotation for critical secrets
4. **Use JSON Format**: Structure secrets with key-value pairs
5. **Limit Access**: Use IAM policies to restrict who can access
6. **Audit Access**: Enable CloudTrail logging to monitor who accessed secrets
7. **Never Share ARNs**: Keep secret ARNs confidential
8. **Use Customer Managed Keys**: For highly sensitive secrets, use custom KMS keys

## IAM Permissions Needed

For users to create and manage secrets, attach this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:CreateSecret",
        "secretsmanager:UpdateSecret",
        "secretsmanager:DeleteSecret",
        "secretsmanager:GetSecretValue",
        "secretsmanager:ListSecrets"
      ],
      "Resource": "*"
    }
  ]
}
```

## Cost

- **$0.40** per secret per month
- **$0.05** per 10,000 API calls to retrieve secrets

## Troubleshooting

### Can't See "Store a new secret" Button
- Ensure you have `secretsmanager:CreateSecret` permission
- Check your IAM policy

### Can't Retrieve Secret in Lambda
- Verify Lambda execution role has `secretsmanager:GetSecretValue` permission
- Confirm the secret name and region match

### Secret Not Found Error
- Check the exact secret name (case-sensitive)
- Verify the secret is in the correct region

## Next Steps

1. Create your first secret in the console
2. Test retrieval in Lambda
3. Update your Lambda code to use secrets
4. Set up rotation policies
5. Enable CloudTrail for audit logging
