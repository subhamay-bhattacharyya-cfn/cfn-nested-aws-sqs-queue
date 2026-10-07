# CloudFormation SQS Queue Template Repository

<!-- Row 1: Status - Most Important -->
[![Release](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)&nbsp;[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazon&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![SQS](https://img.shields.io/badge/SQS-Messaging-FF9900?logo=amazon&logoColor=white)](https://aws.amazon.com/sqs/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-sqs-queue/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/a83cfd2cc8a26aa4804eb3fbc0a9e6f7/raw/cfn-nested-aws-sqs-queue.json)](https://gist.github.com/subhamay-bhattacharyya/a83cfd2cc8a26aa4804eb3fbc0a9e6f7)

This repository contains a nested CloudFormation template for deploying SQS queues with flexible configuration, support for Standard and FIFO queue types, and cross-account access policies.

## Overview

This is a **nested stack template** designed to be invoked from a parent/root CloudFormation stack. The template is stored in this repository and should be uploaded to an S3 bucket for reference by parent stacks.

## Template Files

### CloudFormation Templates

- **`cloudformation/template.yaml`** — Nested template for SQS queue creation with support for:
  - Standard and FIFO queue types
  - Configurable visibility timeout
  - Message retention period
  - Long polling support
  - Content-based deduplication (FIFO only)
  - Cross-account access policies
  - Deterministic queue naming with optional CI suffix

### Parameter Files

- **`parameters/dev.json`** — Development environment parameters
- **`parameters/staging.json`** — Staging environment parameters
- **`parameters/prod.json`** — Production environment parameters

## Template Features

### SQS Queue Template (template.yaml)

- ✅ **Queue Types** — Support for both Standard and FIFO queues
- ✅ **Flexible Configuration** — Configurable visibility timeout, message retention, and long polling
- ✅ **FIFO Support** — Content-based deduplication for FIFO queues
- ✅ **Cross-Account Access** — Resource policy for granting access to other AWS accounts
- ✅ **Smart Queue Naming** — Project prefix, environment, region with optional CI suffix
- ✅ **Automatic Suffix** — FIFO queues automatically get `.fifo` suffix
- ✅ **Long Polling** — Optional long polling configuration for reduced API calls
- ✅ **Message Retention** — Configurable retention period (60 seconds to 14 days)
- ✅ **Visibility Timeout** — Configurable timeout for message processing
- ✅ **Automatic Tagging** — Environment and management tags for resource tracking

## Parameters

### Queue Naming & Environment

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | `proj-ztc` | Project name prefix (lowercase, alphanumeric, hyphens only) |
| `QueueBaseName` | String | `sqs-queue` | Base name for the SQS queue |
| `Environment` | String | `devl` | Deployment environment (devl, stag, prod) |
| `CiSuffix` | String | `""` | Optional CI suffix to append to queue name (e.g., pipeline ID) |

### Queue Type Configuration

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `QueueType` | String | `Standard` | Queue type (Standard or FIFO) |
| `EnableContentBasedDeduplication` | String | `false` | Enable content-based deduplication (FIFO queues only) |

### Message Configuration

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `VisibilityTimeout` | Number | `30` | Visibility timeout in seconds (0-43200) |
| `MessageRetentionPeriod` | Number | `345600` | Message retention period in seconds (60-1209600, default: 4 days) |
| `ReceiveMessageWaitTimeSeconds` | Number | `0` | Long polling wait time in seconds (0-20) |

### Cross-Account Access

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `CrossAccountArns` | CommaDelimitedList | `""` | Comma-separated list of cross-account principal ARNs (e.g., arn:aws:iam::123456789012:root) |

## Outputs

### SQS Queue Template Outputs

| Output | Type | Description |
| ----------- | ------ | ------------- |
| `QueueName` | String | Name of the created SQS queue (exported for cross-stack reference) |
| `QueueUrl` | String | URL of the created SQS queue (exported for cross-stack reference) |
| `QueueArn` | String | ARN of the created SQS queue (exported for cross-stack reference) |
| `QueueType` | String | Type of the queue (Standard or FIFO) |

## Usage

### 1. Upload Template to S3

```bash
aws s3 cp cloudformation/template.yaml s3://your-cfn-bucket/templates/sqs-queue.yaml
```

### 2. Reference from Parent Stack

In your parent/root CloudFormation template:

```yaml
SQSQueueNestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/sqs-queue.yaml
    Parameters:
      ProjectName: !Ref ProjectName
      QueueBaseName: task-queue
      Environment: !Ref Environment
      QueueType: Standard
      VisibilityTimeout: "30"
      MessageRetentionPeriod: "345600"
    Tags:
      - Key: Environment
        Value: !Ref Environment

Outputs:
  QueueName:
    Value: !GetAtt SQSQueueNestedStack.Outputs.QueueName
  QueueUrl:
    Value: !GetAtt SQSQueueNestedStack.Outputs.QueueUrl
  QueueArn:
    Value: !GetAtt SQSQueueNestedStack.Outputs.QueueArn
```

### 3. Deploy Using AWS CLI

#### Example 1: Basic Standard Queue

```bash
aws cloudformation create-stack \
  --stack-name myapp-sqs-dev \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=QueueBaseName,ParameterValue=task-queue \
    ParameterKey=Environment,ParameterValue=devl \
    ParameterKey=QueueType,ParameterValue=Standard
```

#### Example 2: FIFO Queue with Deduplication

```bash
aws cloudformation create-stack \
  --stack-name myapp-sqs-fifo \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=QueueBaseName,ParameterValue=order-queue \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=QueueType,ParameterValue=FIFO \
    ParameterKey=EnableContentBasedDeduplication,ParameterValue=true \
    ParameterKey=VisibilityTimeout,ParameterValue=60
```

#### Example 3: Queue with Long Polling

```bash
aws cloudformation create-stack \
  --stack-name myapp-sqs-longpoll \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=QueueBaseName,ParameterValue=events-queue \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=QueueType,ParameterValue=Standard \
    ParameterKey=ReceiveMessageWaitTimeSeconds,ParameterValue=20
```

#### Example 4: Queue with Cross-Account Access

```bash
aws cloudformation create-stack \
  --stack-name myapp-sqs-crossaccount \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=QueueBaseName,ParameterValue=shared-queue \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=QueueType,ParameterValue=Standard \
    ParameterKey=CrossAccountArns,ParameterValue="arn:aws:iam::123456789012:root,arn:aws:iam::210987654321:root"
```

#### Example 5: Queue with Custom Message Retention

```bash
aws cloudformation create-stack \
  --stack-name myapp-sqs-retention \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=QueueBaseName,ParameterValue=archive-queue \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=QueueType,ParameterValue=Standard \
    ParameterKey=MessageRetentionPeriod,ParameterValue=1209600
```

#### Example 6: Queue with CI Suffix

```bash
aws cloudformation create-stack \
  --stack-name myapp-sqs-ci \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=QueueBaseName,ParameterValue=test-queue \
    ParameterKey=Environment,ParameterValue=test \
    ParameterKey=CiSuffix,ParameterValue=$CI_PIPELINE_ID \
    ParameterKey=QueueType,ParameterValue=Standard
```

#### Monitoring Stack Creation

```bash
# Wait for stack creation to complete
aws cloudformation wait stack-create-complete --stack-name myapp-sqs-dev

# Get stack outputs
aws cloudformation describe-stacks \
  --stack-name myapp-sqs-dev \
  --query 'Stacks[0].Outputs' \
  --output table

# Get queue details
QUEUE_URL=$(aws cloudformation describe-stacks \
  --stack-name myapp-sqs-dev \
  --query 'Stacks[0].Outputs[?OutputKey==`QueueUrl`].OutputValue' \
  --output text)

aws sqs get-queue-attributes --queue-url $QUEUE_URL --attribute-names All
```

## Queue Naming Convention

The template generates queue names using the following pattern:

**Standard Queue (Without CI Suffix):**

```bash
{ProjectName}-{QueueBaseName}-{Environment}-{Region}
```

Example: `myapp-task-queue-devl-us-east-1`

**FIFO Queue (Without CI Suffix):**

```bash
{ProjectName}-{QueueBaseName}-{Environment}-{Region}.fifo
```

Example: `myapp-order-queue-prod-us-east-1.fifo`

**With CI Suffix:**

```bash
{ProjectName}-{QueueBaseName}-{Environment}-{Region}-{CiSuffix}[.fifo]
```

Example (Standard): `myapp-task-queue-devl-us-east-1-pipeline-123`
Example (FIFO): `myapp-order-queue-prod-us-east-1-pipeline-123.fifo`

## Best Practices Implemented

- ✅ Support for both Standard and FIFO queue types
- ✅ Configurable message retention period (60 seconds to 14 days)
- ✅ Flexible visibility timeout (0-43200 seconds)
- ✅ Long polling support to reduce API calls
- ✅ Content-based deduplication for FIFO queues
- ✅ Cross-account access policies for multi-account scenarios
- ✅ Smart queue naming with project prefix, environment, and region
- ✅ Automatic FIFO suffix for FIFO queues (.fifo)
- ✅ Optional CI suffix support for ephemeral test deployments
- ✅ Automatic tagging for resource management and tracking
- ✅ Export values for cross-stack references
- ✅ Same-account and cross-account IAM policies for flexible access control

## License

MIT