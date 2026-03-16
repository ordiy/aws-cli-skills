---
name: aws-s3
description: >-
  ☁️ Senior Cloud Infrastructure Architect skill for Amazon S3. Manages buckets,
  objects, lifecycle policies, versioning, replication, access control, and
  pre-signed URLs using the AWS CLI with --query, --output json, and least-privilege
  IAM best practices. Use when the user asks about S3 buckets, objects, uploads,
  downloads, sync, ACLs, bucket policies, static hosting, encryption, or storage
  operations.
requirements:
  - aws-cli >= 2.0
  - Configured AWS credentials (aws configure or IAM role/instance profile)
---

# AWS S3 — Senior Cloud Infrastructure Architect Skill

## Core Conventions

Always apply these flags unless the user explicitly overrides:

```
--output json          # machine-parseable, predictable
--query '…'            # filter at the API layer, not in shell pipes
--region <region>      # explicit region, never rely on ambient default
--no-paginate          # add when expecting full result sets
```

---

## Bucket Operations

### List all buckets (with creation dates)
```bash
aws s3api list-buckets \
  --query 'Buckets[*].{Name:Name,Created:CreationDate}' \
  --output json
```

### Create bucket (us-east-1)
```bash
aws s3api create-bucket \
  --bucket my-bucket \
  --region us-east-1 \
  --output json
```

### Create bucket (any other region — requires LocationConstraint)
```bash
aws s3api create-bucket \
  --bucket my-bucket \
  --region eu-west-1 \
  --create-bucket-configuration LocationConstraint=eu-west-1 \
  --output json
```

### Block all public access (always apply to new buckets)
```bash
aws s3api put-public-access-block \
  --bucket my-bucket \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

### Delete bucket (must be empty)
```bash
aws s3api delete-bucket --bucket my-bucket --region us-east-1
```

---

## Object Operations

### Upload object
```bash
aws s3 cp local-file.zip s3://my-bucket/path/local-file.zip \
  --sse aws:kms \
  --storage-class STANDARD_IA
```

### Download object
```bash
aws s3 cp s3://my-bucket/path/file.txt ./file.txt
```

### Sync local directory → S3 (with delete)
```bash
aws s3 sync ./dist s3://my-bucket/dist \
  --delete \
  --sse aws:kms \
  --exclude "*.DS_Store"
```

### List objects (with size and last modified)
```bash
aws s3api list-objects-v2 \
  --bucket my-bucket \
  --prefix path/ \
  --query 'Contents[*].{Key:Key,Size:Size,Modified:LastModified}' \
  --output json
```

### Delete object
```bash
aws s3api delete-object --bucket my-bucket --key path/file.txt
```

### Generate pre-signed URL (expires in 1 hour)
```bash
aws s3 presign s3://my-bucket/private/report.pdf \
  --expires-in 3600
```

---

## Versioning

### Enable versioning
```bash
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled
```

### List object versions
```bash
aws s3api list-object-versions \
  --bucket my-bucket \
  --prefix path/file.txt \
  --query '{Versions:Versions[*].{Key:Key,VersionId:VersionId,IsLatest:IsLatest},DeleteMarkers:DeleteMarkers}' \
  --output json
```

---

## Encryption

### Enable SSE-KMS default encryption
```bash
aws s3api put-bucket-encryption \
  --bucket my-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "arn:aws:kms:us-east-1:123456789012:key/KEY-ID"
      },
      "BucketKeyEnabled": true
    }]
  }'
```

---

## Lifecycle Policies

### Transition objects to Glacier after 90 days, expire after 365
```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "archive-and-expire",
      "Status": "Enabled",
      "Filter": {"Prefix": "logs/"},
      "Transitions": [{"Days": 90, "StorageClass": "GLACIER"}],
      "Expiration": {"Days": 365}
    }]
  }'
```

---

## Bucket Policy (Least-Privilege Example)

Allow a specific IAM role read-only access to a prefix:

```bash
aws s3api put-bucket-policy \
  --bucket my-bucket \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Sid": "AllowRoleReadPrefix",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:role/my-role"},
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/data/*"
      ]
    }]
  }'
```

---

## CORS Configuration

```bash
aws s3api put-bucket-cors \
  --bucket my-bucket \
  --cors-configuration '{
    "CORSRules": [{
      "AllowedOrigins": ["https://app.example.com"],
      "AllowedMethods": ["GET", "PUT"],
      "AllowedHeaders": ["*"],
      "MaxAgeSeconds": 3000
    }]
  }'
```

---

## IAM Least-Privilege Reference

Minimum permissions for common S3 tasks:

| Task | Actions Required |
|------|-----------------|
| Read objects | `s3:GetObject` |
| Upload objects | `s3:PutObject` |
| List bucket | `s3:ListBucket` |
| Delete objects | `s3:DeleteObject` |
| Manage versioning | `s3:GetBucketVersioning`, `s3:PutBucketVersioning` |
| Manage lifecycle | `s3:GetLifecycleConfiguration`, `s3:PutLifecycleConfiguration` |

Always scope `s3:ListBucket` to the bucket ARN and object actions to `arn:aws:s3:::bucket/*`.

---

## Additional Resources

- For replication and Cross-Region setup, see [reference.md](reference.md)
