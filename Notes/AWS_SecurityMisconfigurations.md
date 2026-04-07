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

---