## 🔍 Enumeration Commands

### 🆔 Identity & Account Enumeration

```bash
# Who am I? (Always run first)
aws sts get-caller-identity #(sts mean Security Token Service)

# Get account summary
aws iam get-account-summary

# List account aliases
aws iam list-account-aliases # (Alias is a nickname for an account)

# Get password policy
aws iam get-account-password-policy # (Password policy is a set of rules that define the strength of passwords) 
```

### 👤 User Enumeration

```bash
# List all IAM users
aws iam list-users

# Get specific user details
aws iam get-user --user-name <username>

# List user's access keys
aws iam list-access-keys --user-name <username>

# List groups a user belongs to
aws iam list-groups-for-user --user-name <username>

# List policies attached to user
aws iam list-attached-user-policies --user-name <username>

# List inline policies on user
aws iam list-user-policies --user-name <username>

# Get inline policy document
aws iam get-user-policy --user-name <username> --policy-name <policyname>

# List MFA devices for user
aws iam list-mfa-devices --user-name <username>

# List signing certificates
aws iam list-signing-certificates --user-name <username>
```

### 👥 Group Enumeration

```bash
# List all groups
aws iam list-groups

# List users in a group
aws iam get-group --group-name <groupname>

# List policies attached to group
aws iam list-attached-group-policies --group-name <groupname>

# List inline policies on group
aws iam list-group-policies --group-name <groupname>
```

### 🎭 Role Enumeration

```bash
# List all roles
aws iam list-roles

# Get specific role details (includes trust policy)
aws iam get-role --role-name <rolename>

# List policies attached to role
aws iam list-attached-role-policies --role-name <rolename>

# List inline policies on role
aws iam list-role-policies --role-name <rolename>

# Get inline policy document
aws iam get-role-policy --role-name <rolename> --policy-name <policyname>

# List instance profiles (roles attached to EC2)
aws iam list-instance-profiles
aws iam get-instance-profile --instance-profile-name <name>
```

### 📜 Policy Enumeration

```bash
# List all customer-managed policies
aws iam list-policies --scope Local

# List all AWS-managed policies
aws iam list-policies --scope AWS

# List all policies (both)
aws iam list-policies

# Get policy details
aws iam get-policy --policy-arn <arn>

# Get policy version (actual document)
aws iam get-policy-version --policy-arn <arn> --version-id v1

# List all versions of a policy
aws iam list-policy-versions --policy-arn <arn>

# List all entities using a policy
aws iam list-entities-for-policy --policy-arn <arn>
```

### 🖥️ EC2 Enumeration

```bash
# List all EC2 instances
aws ec2 describe-instances

# List instances in a specific region
aws ec2 describe-instances --region us-east-1

# Get instance metadata (output filter)
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name,PublicIpAddress,PrivateIpAddress,IamInstanceProfile]' --output table

# List security groups
aws ec2 describe-security-groups

# List key pairs
aws ec2 describe-key-pairs

# List all regions
aws ec2 describe-regions

# List snapshots (yours)
aws ec2 describe-snapshots --owner-ids self

# List AMIs (yours)
aws ec2 describe-images --owners self

# List VPCs
aws ec2 describe-vpcs

# List subnets
aws ec2 describe-subnets
```

### 🪣 S3 Enumeration

```bash
# List all buckets
aws s3 ls
aws s3api list-buckets

# List bucket contents
aws s3 ls s3://<bucket-name>/
aws s3 ls s3://<bucket-name>/ --recursive

# Get bucket ACL
aws s3api get-bucket-acl --bucket <bucket-name>

# Get bucket policy
aws s3api get-bucket-policy --bucket <bucket-name>

# Check public access block settings
aws s3api get-public-access-block --bucket <bucket-name>

# Get bucket versioning
aws s3api get-bucket-versioning --bucket <bucket-name>

# Get bucket location (region)
aws s3api get-bucket-location --bucket <bucket-name>

# Download a file
aws s3 cp s3://<bucket>/<key> ./local-file

# Download entire bucket
aws s3 sync s3://<bucket-name> ./local-folder
```

### ⚡ Lambda Enumeration

```bash
# List all Lambda functions
aws lambda list-functions

# Get function details (includes env vars!)
aws lambda get-function --function-name <name>

# Get function configuration
aws lambda get-function-configuration --function-name <name>

# Get function policy (resource-based)
aws lambda get-policy --function-name <name>

# List function event source mappings
aws lambda list-event-source-mappings --function-name <name>

# List function aliases
aws lambda list-aliases --function-name <name>
```

### 🗄️ RDS Enumeration

```bash
# List all DB instances
aws rds describe-db-instances

# List DB clusters
aws rds describe-db-clusters

# List DB snapshots
aws rds describe-db-snapshots

# List DB snapshots (public ones)
aws rds describe-db-snapshots --snapshot-type public

# List DB subnet groups
aws rds describe-db-subnet-groups

# List DB security groups
aws rds describe-db-security-groups
```

### 🔐 Secrets Manager / SSM Enumeration

```bash
# List all secrets
aws secretsmanager list-secrets

# Get secret value (gold mine!)
aws secretsmanager get-secret-value --secret-id <name-or-arn>

# Describe a secret
aws secretsmanager describe-secret --secret-id <name-or-arn>

# SSM Parameter Store - list parameters
aws ssm describe-parameters

# Get SSM parameter value
aws ssm get-parameter --name <name> --with-decryption

# Get multiple parameters
aws ssm get-parameters-by-path --path / --recursive --with-decryption
```

### ☁️ CloudTrail Enumeration

```bash
# List trails
aws cloudtrail describe-trails

# Get trail status
aws cloudtrail get-trail-status --name <name>

# List event selectors (what's being logged)
aws cloudtrail get-event-selectors --trail-name <name>

# Look up recent events (last 90 days)
aws cloudtrail lookup-events --max-results 50

# Filter by event name
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin
```

---
