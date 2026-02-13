# Lab Management System - Foundational AWS Stack

This project implements the foundational AWS infrastructure for the Lab Management System using Terraform.

## Components

### 1. Foundational AWS Stack
- **VPC**: Virtual Private Cloud with public and private subnets across multiple availability zones
- **SQS Queues**: Main queue with Dead Letter Queue (DLQ) for reliable message processing
- **IAM Role**: Lambda execution role with appropriate permissions for all required services
- **S3 Buckets**: 
  - `tf-modules`: Storage for Terraform modules
  - `tf-artifacts`: Storage for Terraform artifacts
  - `logs`: Storage for application logs
- **DynamoDB**: `LabRuns` table for tracking lab execution runs

### 2. Terraform State Management
- **S3 Backend**: Secure storage for Terraform state files
- **DynamoDB Lock Table**: State locking to prevent concurrent modifications
- **Workspace Strategy**: Isolation per tenant using Terraform workspaces

### 3. Containerized Lambda Runtime
- **Docker Image**: Contains Terraform, AWS CLI, and jq
- **Lambda Function**: Executes Terraform commands in a serverless environment

## Directory Structure

```
├── foundational-aws-stack/           # Main Terraform configuration
│   ├── main.tf                     # Main configuration
│   ├── variables.tf                # Input variables
│   ├── outputs.tf                  # Output values
│   ├── backend.tf                  # Backend configuration
│   └── modules/                    # Terraform modules
│       ├── vpc/
│       ├── sqs/
│       ├── iam/
│       ├── s3/
│       └── dynamodb/
├── terraform-backend/              # Backend infrastructure
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── containerized-lambda-runtime/   # Containerized Lambda
│   ├── Dockerfile
│   ├── lambda_function.py
│   ├── build.sh
│   ├── publish.sh
│   └── test.sh
└── state-isolation-strategy.md     # Documentation
```

## Deployment Steps

### 1. Deploy Backend Infrastructure
First, deploy the S3 bucket and DynamoDB table for Terraform state management:

```bash
cd terraform-backend
terraform init
terraform plan
terraform apply
```

### 2. Update Backend Configuration
Update `/foundational-aws-stack/backend.hcl` with the outputs from the previous step, then:

```bash
cd ../foundational-aws-stack
terraform init
```

When prompted to migrate state, select "yes".

### 3. Deploy Main Infrastructure
```bash
terraform plan
terraform apply
```

### 4. Deploy Containerized Lambda
```bash
cd ../containerized-lambda-runtime
./build.sh <AWS_ACCOUNT_ID> [AWS_REGION]
./publish.sh <AWS_ACCOUNT_ID> [AWS_REGION]
```

### 5. Set Up Workspaces
```bash
cd ../foundational-aws-stack
terraform workspace new tenant1
terraform workspace new tenant2
# Deploy resources for each tenant as needed
```

## Outputs

The main Terraform configuration outputs ARNs and endpoints for all created resources:

- VPC ID and subnet IDs
- SQS queue URLs and ARNs
- Lambda execution role ARN
- S3 bucket names
- DynamoDB table name and ARN

## State Isolation Strategy

The solution implements state isolation using Terraform workspaces:
1. Each tenant gets its own workspace
2. Resources are tagged with tenant, course, and lab information
3. State is stored in S3 with DynamoDB for locking
4. Workspaces provide complete isolation between tenants

## Testing

- Unit tests for individual modules
- Integration tests for the full infrastructure
- State isolation verification script included
- Lambda runtime verification script included

## Security Considerations

- S3 buckets have versioning and server-side encryption enabled
- Public access is blocked on all S3 buckets
- IAM roles follow principle of least privilege
- VPC provides network isolation
- VPC flow logs can be enabled for additional monitoring# java-3tier-app
