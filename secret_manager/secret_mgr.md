# AWS Lambda with Secrets Manager

A comprehensive guide to retrieving secrets from AWS Secrets Manager using Lambda functions.

## Overview

This guide demonstrates how to create and manage secrets in AWS Secrets Manager and retrieve them using a Lambda function. This is a secure way to handle sensitive information like database credentials, API keys, and passwords.

## Prerequisites

- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Basic understanding of Lambda and IAM roles
- Python 3.9 or higher

## Architecture

```
Lambda Function → AWS Secrets Manager → Retrieve Secret Value
```

## Step-by-Step Setup

### Step 1: Create a Secret

Create a secret in AWS Secrets Manager using the AWS CLI:

```bash
aws secretsmanager create-secret \
  --name my-secret \
  --secret-string '{"username":"admin","password":"mypassword"}' \
  --region us-east-1
```

For a database credential secret:

```bash
aws secretsmanager create-secret \
  --name rds-credentials \
  --secret-string '{"host":"mydb.example.com","user":"dbadmin","password":"dbpassword"}' \
  --region us-east-1
```

### Step 2: Configure IAM Role

Create or update your Lambda execution role with the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:us-east-1:ACCOUNT-ID:secret:my-secret-*"
    }
  ]
}
```

Replace `ACCOUNT-ID` with your AWS Account ID.

### Step 3: Create Lambda Function

Create a new Lambda function with the Python code provided. The function:
- Initializes a Secrets Manager client
- Retrieves the secret by name
- Handles both string and binary secrets
- Returns appropriate error messages

### Step 4: Test the Function

Invoke the Lambda with a test event:

```json
{
  "secret_name": "my-secret"
}
```

Expected response:

```json
{
  "statusCode": 200,
  "body": {
    "message": "Secret retrieved successfully",
    "secret": {
      "username": "admin",
      "password": "mypassword"
    }
  }
}
```

## Usage Examples

### Basic Usage

```python
import boto3
import json

client = boto3.client('secretsmanager', region_name='us-east-1')
response = client.get_secret_value(SecretId='my-secret')
secret = json.loads(response['SecretString'])
```

### Using in Application Code

```python
def get_db_credentials():
    secret = get_secret('rds-credentials')
    return {
        'host': secret['host'],
        'user': secret['user'],
        'password': secret['password']
    }

def get_secret(secret_name):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])
```

## Best Practices

1. **Least Privilege**: Grant only the necessary permissions to access specific secrets
2. **Encryption**: Secrets are encrypted at rest using AWS KMS by default
3. **Rotation**: Enable automatic secret rotation for sensitive credentials
4. **Logging**: Enable CloudTrail to audit secret access
5. **Naming Convention**: Use consistent naming like `env/service/credential-type`
6. **Caching**: Cache secrets in memory when possible to reduce API calls
7. **Error Handling**: Always handle exceptions gracefully

## Error Handling

The Lambda function handles these errors:

- **ResourceNotFoundException**: Secret doesn't exist
- **AccessDeniedException**: IAM permissions insufficient
- **InvalidRequestException**: Invalid secret name or format
- **ThrottlingException**: Rate limited by API

## Cost Considerations

- **Secrets Manager**: $0.40 per secret per month + $0.05 per 10,000 API calls
- **Lambda**: First 1 million requests free, then $0.20 per million

## Troubleshooting

### Secret Not Found Error

```
Error: ResourceNotFoundException
```

**Solution**: Verify the secret name matches exactly and is in the correct region.

### Permission Denied Error

```
Error: An error occurred (AccessDeniedException)
```

**Solution**: Check that the Lambda execution role has `secretsmanager:GetSecretValue` permission.

### Timeout Error

**Solution**: Increase Lambda timeout in configuration or reduce secret size.

## Advanced Features

### Rotate Secrets Automatically

```bash
aws secretsmanager rotate-secret \
  --secret-id my-secret \
  --rotation-rules AutomaticallyAfterDays=30
```

### Version Secrets

Secrets Manager automatically versions secrets when updated:

```python
response = client.get_secret_value(
    SecretId='my-secret',
    VersionId='specific-version-id'
)
```

### Database Secret Templates

AWS provides templates for common databases:
- Amazon RDS MySQL
- Amazon RDS PostgreSQL
- Amazon Redshift
- MongoDB Atlas
- Other Databases

## Related Services

- **AWS KMS**: Encrypts secrets at rest
- **CloudTrail**: Audits secret access
- **Systems Manager Parameter Store**: Alternative for non-sensitive config
- **AWS Backup**: Backs up secret metadata

## References

- [AWS Secrets Manager Documentation](https://docs.aws.amazon.com/secretsmanager/)
- [Lambda Execution Role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
- [Boto3 Secrets Manager](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/secretsmanager.html)

## License

This guide is provided as-is for educational purposes.
