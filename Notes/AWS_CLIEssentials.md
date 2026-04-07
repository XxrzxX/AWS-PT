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