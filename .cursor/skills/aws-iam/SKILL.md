---
name: aws-iam
description: >-
  🔐 Senior Cloud Infrastructure Architect skill for AWS IAM. Creates and manages
  users, groups, roles, policies, permission boundaries, service control policies,
  trust relationships, access keys, and MFA using the AWS CLI with --query,
  --output json, and strict least-privilege design. Use when the user asks about
  IAM users, roles, policies, permissions, trust policies, cross-account access,
  service roles, access keys, MFA, policy simulation, or identity and access management.
requirements:
  - aws-cli >= 2.0
  - Configured AWS credentials with iam:* permissions (or scoped equivalents)
---

# AWS IAM — Senior Cloud Infrastructure Architect Skill

## Core Conventions

```
--output json          # machine-parseable
--query '…'            # reduce noise at API layer
```

> IAM is global — no `--region` flag is needed for most IAM API calls.

---

## Users

### Create user
```bash
aws iam create-user \
  --user-name jane-doe \
  --tags Key=Team,Value=platform Key=Env,Value=prod \
  --output json
```

### List users (name + creation date)
```bash
aws iam list-users \
  --query 'Users[*].{User:UserName,Created:CreateDate,ARN:Arn}' \
  --output json
```

### Attach managed policy to user
```bash
aws iam attach-user-policy \
  --user-name jane-doe \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

### Create programmatic access key
```bash
aws iam create-access-key \
  --user-name jane-doe \
  --query 'AccessKey.{ID:AccessKeyId,Secret:SecretAccessKey}' \
  --output json
# Store the SecretAccessKey immediately — it cannot be retrieved again.
```

### List & audit access keys
```bash
aws iam list-access-keys \
  --user-name jane-doe \
  --query 'AccessKeyMetadata[*].{ID:AccessKeyId,Status:Status,Created:CreateDate}' \
  --output json
```

### Deactivate and delete stale key
```bash
aws iam update-access-key \
  --user-name jane-doe \
  --access-key-id AKIAIOSFODNN7EXAMPLE \
  --status Inactive

aws iam delete-access-key \
  --user-name jane-doe \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

---

## Groups

### Create group and attach policy
```bash
aws iam create-group --group-name developers

aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

### Add user to group
```bash
aws iam add-user-to-group \
  --group-name developers \
  --user-name jane-doe
```

---

## Roles

### Create role with trust policy (EC2 service)
```bash
aws iam create-role \
  --role-name ec2-app-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }' \
  --description "Role for EC2 application instances" \
  --output json
```

### Trust policy — cross-account assume role
```bash
aws iam update-assume-role-policy \
  --role-name my-role \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::TRUSTED-ACCOUNT-ID:root"},
      "Action": "sts:AssumeRole",
      "Condition": {
        "Bool": {"aws:MultiFactorAuthPresent": "true"}
      }
    }]
  }'
```

### Attach managed policy to role
```bash
aws iam attach-role-policy \
  --role-name ec2-app-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### List roles and their ARNs
```bash
aws iam list-roles \
  --query 'Roles[*].{Name:RoleName,ARN:Arn,Created:CreateDate}' \
  --output json \
  --no-paginate
```

---

## Inline vs Managed Policies

| | Managed Policy | Inline Policy |
|---|---|---|
| **Reuse** | Across multiple principals | One principal only |
| **Audit** | Easier (versioned) | Harder |
| **Recommendation** | Prefer for shared permissions | Use for strict 1:1 binding |

### Create customer-managed policy
```bash
aws iam create-policy \
  --policy-name s3-read-specific-bucket \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }]
  }' \
  --output json
```

### Create new policy version
```bash
aws iam create-policy-version \
  --policy-arn arn:aws:iam::123456789012:policy/s3-read-specific-bucket \
  --policy-document file://policy-v2.json \
  --set-as-default
```

### Get effective policy (list attached policies for a role)
```bash
aws iam list-attached-role-policies \
  --role-name ec2-app-role \
  --query 'AttachedPolicies[*].{Name:PolicyName,ARN:PolicyArn}' \
  --output json
```

---

## Permission Boundaries

Set a permission boundary to cap maximum permissions for a role:

```bash
aws iam put-role-permissions-boundary \
  --role-name ec2-app-role \
  --permissions-boundary arn:aws:iam::123456789012:policy/max-permissions-boundary
```

---

## Policy Simulation

Test whether a principal can perform an action before deploying:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/ec2-app-role \
  --action-names s3:GetObject s3:PutObject \
  --resource-arns arn:aws:s3:::my-bucket/data/* \
  --query 'EvaluationResults[*].{Action:EvalActionName,Decision:EvalDecision}' \
  --output json
```

---

## Credential Report

Generate and download account-wide credential audit:

```bash
aws iam generate-credential-report
# Wait ~5 seconds, then:
aws iam get-credential-report \
  --query 'Content' \
  --output text | base64 --decode > credential-report.csv
```

---

## MFA Enforcement

Condition to require MFA in any policy statement:

```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  },
  "NumericLessThan": {
    "aws:MultiFactorAuthAge": "3600"
  }
}
```

---

## IAM Least-Privilege Design Principles

1. **Start with deny** — only add `Allow` statements for what is explicitly needed.
2. **Scope resources** — never use `"Resource": "*"` unless the API requires it (e.g., `iam:ListUsers`).
3. **Use conditions** — restrict by `aws:SourceVpc`, `aws:RequestedRegion`, `s3:prefix`, etc.
4. **Prefer roles over users** — avoid long-lived access keys.
5. **Rotate keys** — automate rotation; alert on keys older than 90 days via CloudWatch.
6. **Audit regularly** — use `get-credential-report` and AWS Access Analyzer.
