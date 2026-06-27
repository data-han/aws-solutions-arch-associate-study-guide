# Practice — Domain 1: Design Secure Architectures (30%)

> Original scenario questions in exam style. Answers + explanations at the bottom. Cover the answers and reason through each.


---
**Q1.** An application on EC2 needs to read objects from an S3 bucket. A developer stored IAM access keys in the application's config file. What is the most secure way to grant access?
- A. Rotate the access keys every 30 days
- B. Attach an IAM role to the EC2 instance via an instance profile
- C. Store the keys in environment variables
- D. Use a bucket ACL granting the developer's user access

**Q2.** A company must block traffic from a single malicious IP address to all instances in a subnet. Which control should they use?
- A. Security group inbound deny rule
- B. Network ACL deny rule on the subnet
- C. IAM policy with explicit deny
- D. Route table blackhole route

**Q3.** A regulated workload requires the company to fully control and audit encryption keys, with dedicated single-tenant hardware that meets FIPS 140-2 Level 3. Which service?
- A. AWS KMS with AWS managed keys
- B. AWS KMS with customer managed keys
- C. AWS CloudHSM
- D. AWS Secrets Manager

**Q4.** A team needs RDS database credentials to rotate automatically every 30 days without code changes. Which service?
- A. SSM Parameter Store SecureString
- B. AWS Secrets Manager
- C. AWS KMS
- D. IAM database authentication

**Q5.** A security team wants to be alerted to potential account compromise by analyzing VPC Flow Logs, DNS logs, and CloudTrail with no agents to deploy. Which service?
- A. Amazon Inspector
- B. Amazon Macie
- C. Amazon GuardDuty
- D. AWS Config

**Q6.** A company runs multiple AWS accounts under Organizations and wants to guarantee that no account — even an account admin — can disable CloudTrail. What should they use?
- A. An IAM permissions boundary
- B. A Service Control Policy (SCP)
- C. A resource-based policy on CloudTrail
- D. An IAM group policy applied to all users

**Q7.** Static assets in a private S3 bucket must be served only through CloudFront, never directly. What achieves this?
- A. Make the bucket public and add a bucket policy
- B. Use Origin Access Control (OAC) and a bucket policy restricting access to the distribution
- C. Enable S3 Transfer Acceleration
- D. Use a presigned URL for each object

**Q8.** Auditors ask "who deleted this DynamoDB table and when?" Which service provides the answer?
- A. CloudWatch Logs
- B. AWS Config
- C. CloudTrail
- D. VPC Flow Logs

**Q9.** A web application must be protected from SQL injection and cross-site scripting at the edge. Which service?
- A. AWS Shield Standard
- B. AWS WAF
- C. Security groups
- D. GuardDuty

**Q10.** A company must discover and classify PII stored across hundreds of S3 buckets. Which service?
- A. Amazon Macie
- B. Amazon Inspector
- C. AWS Config
- D. Amazon Detective

---

## Answers & explanations

1. **B** — Use IAM roles for EC2 (instance profile) so no long-lived keys exist. Rotating/storing keys (A, C) still exposes them. ACLs (D) are legacy and don't apply to an EC2 app.
2. **B** — Only **NACLs** support deny rules and operate at the subnet level. Security groups are allow-only (A wrong). IAM/route table don't filter by source IP at L3 for this purpose.
3. **C** — **CloudHSM** = dedicated, single-tenant, customer-controlled, FIPS 140-2 Level 3. KMS is multi-tenant managed.
4. **B** — **Secrets Manager** natively rotates RDS credentials on a schedule. Parameter Store has no built-in rotation.
5. **C** — **GuardDuty** is agentless threat detection analyzing those exact log sources. Inspector = vuln scanning, Macie = PII, Config = compliance.
6. **B** — **SCPs** set a permissions ceiling across accounts that even root/admins can't exceed. Permissions boundaries apply to individual identities, not whole accounts.
7. **B** — **OAC** (replaces OAI) plus a bucket policy locks the bucket to the CloudFront distribution only. Making it public defeats the purpose.
8. **C** — **CloudTrail** logs API calls (who/what/when). Config tracks configuration state, not the actor of every API call.
9. **B** — **WAF** filters L7 attacks (SQLi/XSS) and attaches to CloudFront/ALB/API Gateway. Shield = DDoS.
10. **A** — **Macie** discovers and classifies sensitive data/PII in S3.
