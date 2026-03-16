---
name: aws-lambda
description: >-
  ⚡ Senior Cloud Infrastructure Architect skill for AWS Lambda. Creates, deploys,
  invokes, configures, and manages Lambda functions, layers, aliases, versions,
  event source mappings, and environment variables using the AWS CLI with
  --query, --output json, and IAM least-privilege best practices. Use when the
  user asks about Lambda functions, serverless deployments, function invocation,
  triggers, layers, aliases, function URLs, concurrency, or event-driven architecture.
requirements:
  - aws-cli >= 2.0
  - Configured AWS credentials with appropriate Lambda + IAM permissions
---

# AWS Lambda — Senior Cloud Infrastructure Architect Skill

## Core Conventions

```
--output json          # structured output
--query '…'            # filter at the API layer
--region <region>      # explicit, never ambient
```

---

## Function Deployment

### Create function (zip deployment)
```bash
# Package first
zip -r function.zip . -x "*.git*"

aws lambda create-function \
  --function-name my-function \
  --runtime python3.12 \
  --role arn:aws:iam::123456789012:role/lambda-exec-role \
  --handler app.handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 256 \
  --environment Variables='{LOG_LEVEL=INFO,ENV=production}' \
  --region us-east-1 \
  --output json
```

### Update function code only
```bash
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip \
  --region us-east-1 \
  --output json
```

### Deploy from S3 artifact
```bash
aws lambda update-function-code \
  --function-name my-function \
  --s3-bucket my-deploy-bucket \
  --s3-key builds/function-v1.2.3.zip \
  --region us-east-1 \
  --output json
```

### Update configuration
```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 60 \
  --memory-size 512 \
  --environment Variables='{LOG_LEVEL=DEBUG,ENV=staging}' \
  --output json
```

---

## Invocation

### Synchronous invocation
```bash
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  --cli-binary-format raw-in-base64-out \
  --log-type Tail \
  response.json \
  --query 'LogResult' \
  --output text | base64 --decode
```

### Asynchronous invocation (fire-and-forget)
```bash
aws lambda invoke \
  --function-name my-function \
  --invocation-type Event \
  --payload '{"key": "value"}' \
  --cli-binary-format raw-in-base64-out \
  /dev/null
```

---

## Listing & Inspection

### List all functions (name + runtime + last modified)
```bash
aws lambda list-functions \
  --query 'Functions[*].{Name:FunctionName,Runtime:Runtime,Modified:LastModified,Memory:MemorySize}' \
  --output json \
  --no-paginate
```

### Get function configuration
```bash
aws lambda get-function-configuration \
  --function-name my-function \
  --query '{ARN:FunctionArn,Role:Role,Timeout:Timeout,Memory:MemorySize,Env:Environment}' \
  --output json
```

### Get function URL (if configured)
```bash
aws lambda get-function-url-config \
  --function-name my-function \
  --output json
```

---

## Versions & Aliases

### Publish a new version
```bash
aws lambda publish-version \
  --function-name my-function \
  --description "Release v1.2.3" \
  --output json
```

### Create alias pointing to version
```bash
aws lambda create-alias \
  --function-name my-function \
  --name production \
  --function-version 5 \
  --output json
```

### Weighted alias for canary deployment (10% → new version)
```bash
aws lambda update-alias \
  --function-name my-function \
  --name production \
  --function-version 5 \
  --routing-config AdditionalVersionWeights={"6"=0.1} \
  --output json
```

---

## Concurrency

### Set reserved concurrency (prevent noisy-neighbour impact)
```bash
aws lambda put-function-concurrency \
  --function-name my-function \
  --reserved-concurrent-executions 50
```

### Configure provisioned concurrency on alias
```bash
aws lambda put-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier production \
  --provisioned-concurrent-executions 10
```

---

## Layers

### Publish a layer
```bash
zip -j layer.zip requirements.txt  # or the actual packages directory
aws lambda publish-layer-version \
  --layer-name common-deps \
  --description "Shared Python dependencies" \
  --zip-file fileb://layer.zip \
  --compatible-runtimes python3.11 python3.12 \
  --output json
```

### Attach layer to function
```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:123456789012:layer:common-deps:3
```

---

## Event Source Mappings

### SQS trigger
```bash
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-queue \
  --batch-size 10 \
  --maximum-batching-window-in-seconds 5 \
  --output json
```

### List event source mappings
```bash
aws lambda list-event-source-mappings \
  --function-name my-function \
  --query 'EventSourceMappings[*].{UUID:UUID,Source:EventSourceArn,State:State}' \
  --output json
```

---

## Delete Function

```bash
aws lambda delete-function --function-name my-function
```

---

## IAM Execution Role — Minimum Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:log-group:/aws/lambda/my-function:*"
    }
  ]
}
```

Add additional statements only for specific resources the function must access (S3 bucket, DynamoDB table, SQS queue).

---

## IAM Least-Privilege Reference

| Task | Action Required |
|------|----------------|
| Deploy code | `lambda:UpdateFunctionCode` |
| Update config | `lambda:UpdateFunctionConfiguration` |
| Invoke | `lambda:InvokeFunction` |
| Manage aliases | `lambda:CreateAlias`, `lambda:UpdateAlias` |
| Manage concurrency | `lambda:PutFunctionConcurrency` |
| Add trigger | `lambda:CreateEventSourceMapping` |
| Attach layer | `lambda:UpdateFunctionConfiguration` |
