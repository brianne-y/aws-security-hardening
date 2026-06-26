# AWS Security Audit Summary

**Environment:** AWS Account 106751264163  
**Audit Dates:** 06-11-26 - 06-17-26        
**Engineer:** Brianne Young  
**Tools Used:** AWS Config, CloudTrail, Security Hub, GuardDuty, CloudWatch, EventBridge

---

## Executive Summary

Three security misconfigurations were identified in the AWS environment through automated compliance monitoring using AWS Config. All three were confirmed as real exploitable vulnerabilities through direct demonstration before any remediation began. All three were remediated and verified Compliant. Continuous monitoring controls are now in place to detect and alert on future drift.

A pre-existing misconfiguration in the default VPC security group was also identified and remediated during the review. This finding was outside the original audit scope but was corrected as part of responsible security practice.

---

## Findings Before Remediation

| # | Finding | Resource | Severity | Config Rule | Impact Demonstrated |
|---|---------|----------|----------|-------------|-------------------|
| 1 | S3 bucket with public access enabled | audit-target bucket | HIGH | s3-bucket-public-read-prohibited | Credentials file accessed in incognito browser with no authentication |
| 2 | IAM user with AdministratorAccess attached directly | audit-test-user | CRITICAL | iam-user-no-policies-check | CLI commands confirmed full account access using the exposed credentials |
| 3 | Security group with SSH open to 0.0.0.0/0 | audit-open-ssh-sg | HIGH | restricted-ssh | GuardDuty SSH brute force finding confirmed within the audit window |

---

## Impact Demonstration

**Finding 1:** An internal configuration file containing API keys and database connection strings was accessible to anyone on the internet using only the S3 object URL. No credentials required. Confirmed by loading the file in an incognito browser window with no AWS session active.

**Finding 2:** Access keys were generated for the overpermissioned IAM user and used to demonstrate the blast radius via the AWS CLI. Commands confirmed the ability to list all S3 buckets, all IAM users, and the full scope of AdministratorAccess across all AWS actions and all resources. Access keys were deleted immediately after the demonstration.

**Finding 3:** Intentional failed SSH connection attempts were made against the EC2 instance from AWS CloudShell to simulate external brute force activity. GuardDuty SSH brute force findings confirmed that this type of activity is detected. The instance received connection attempts from internet scanners throughout the audit window.

---

## Remediation Actions Taken

**Finding 1 — S3 Public Access**  
Block all public access was re-enabled on the audit-target bucket. Automatic Config remediation was configured using a dedicated IAM role with a scoped inline policy containing only the two S3 actions required: s3:PutBucketPublicAccessBlock and s3:GetBucketPublicAccessBlock. If public access is re-enabled in the future, Config will detect and revert it automatically.  
Config rule result: COMPLIANT

**Finding 2 — IAM Over-Permission**  
AdministratorAccess was removed from audit-test-user. A group called audit-read-only-group was created with ReadOnlyAccess attached at the group level. The user was added to the group. This corrects both the permission scope and the structural problem of direct user-level policy attachment.  
Config rule result: COMPLIANT at the audit target resource level

**Finding 3 — Security Group Open SSH**  
The SSH inbound rule on audit-open-ssh-sg was updated from source 0.0.0.0/0 to a specific IP address. A pre-existing allow-all inbound rule on the default VPC security group was also identified and removed during this review.  
Config rule result: COMPLIANT at the audit target resource level

---

## Scope Notes

The iam-user-no-policies-check and restricted-ssh Config rules show additional noncompliant resources from prior portfolio projects. These resources are outside the scope of this audit engagement. The three resources that were in scope are confirmed Compliant at the individual resource level within each rule.

Automatic remediation was not configured for the IAM and SSH rules. The available managed automation actions for those findings carry risk of unintended impact on resources outside this audit scope. Manual remediation is the documented process for those findings.

---

## Continuous Monitoring Controls Now in Place

- AWS Config recording enabled for all resources in us-east-1 with three active managed compliance rules
- EventBridge rule triggers SNS email notification on any NON_COMPLIANT Config finding
- Automatic Config remediation configured for the S3 rule using a least-privilege IAM role
- CloudTrail logging all management API calls across all regions with log file validation enabled
- CloudWatch metric filter detecting AccessDenied and unauthorized API call patterns
- CloudWatch alarm fires and routes to SNS when the pattern threshold is exceeded
- GuardDuty enabled for continuous threat detection via VPC flow log, CloudTrail, and DNS analysis
- Security Hub aggregating findings across all active security services

---

## Evidence

All before-and-after screenshots are available in the screenshots/ folder of this repository.

Before state: s3-bucket-public-access-disabled, iam-admin-access-attached, security-group-ssh-open, config-rules-noncompliant, security-hub-before-state, guardduty-ssh-finding-detail

After state: s3-block-public-access-enabled, iam-remediated, security-group-remediated, config-all-compliant, cloudwatch-alarm-triggered, eventbridge-rule, sns-subscription-confirmed
