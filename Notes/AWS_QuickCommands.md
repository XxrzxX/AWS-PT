## Quick Commands

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