# Cloud Security Hardening and Compliance Audit

## Scenario — What Problem Are We Solving?

This project simulates a scenario that plays out in real cloud environments more often than anyone likes to admit. A company that has been moving fast discovers its AWS account has drifted away from secure defaults, and nobody noticed because there was nothing watching.

A B2B SaaS company has been on AWS for 18 months. Engineers built things quickly, made changes when they needed to, and moved on. No formal security governance. No automated compliance monitoring. No audit trail beyond what the team could remember. That worked fine, until the company signed a letter of intent with its first enterprise customer, a financial services firm that requires vendors to pass a cloud security review before any contract can execute.

The procurement team submits three questions. Are your S3 buckets protected against unintended public access? Do your IAM users operate under the principle of least privilege? Is SSH access to your compute resources restricted to authorized sources only?

The engineering team answers yes to all three. A security audit reveals the actual state of the account, which tells a different story. 

A bucket created for a temporary marketing file share seven months ago still has Block Public Access disabled. A developer uploaded a configuration file containing an internal API key in March. That file has been publicly accessible for seven months with no authentication required. An IAM user created for an external contractor in January still has AdministratorAccess attached directly. The contractor's engagement ended in April and the credentials were never deactivated. A security group created during a production debugging session in May has SSH open to 0.0.0.0/0. An EC2 instance is running with that security group attached. GuardDuty confirms active brute force attempts against it within 24 hours.

The answer to all three questions on the questionnaire is no. Submitting those answers as-is would be a misrepresentation to an enterprise customer and a compliance liability for the business.

The goal of this project is to audit the environment, demonstrate the real-world impact of each finding, remediate each misconfiguration, implement continuous compliance monitoring so this cannot happen again, and produce documented evidence that the account meets the requirements the questionnaire asks about.

This is my fourth AWS project. The first three were about building infrastructure. This one is about auditing it: finding what is wrong, proving the consequences are real, fixing it, and making sure it stays fixed.

---

## Architecture

This project is not a traditional infrastructure deployment. There is no VPC or subnet structure to provision from scratch. The architecture is a security monitoring and enforcement layer applied on top of an existing AWS account, built around a before-and-after audit structure.

Three resources are created deliberately as misconfigured audit targets. Monitoring tools are enabled to detect them. The before state is documented with evidence of real-world impact. Each misconfiguration is remediated. The after state is verified and documented in a written audit summary.

```
BEFORE STATE:
  S3 bucket with public access enabled    →  credentials file accessible to anyone on the internet
  IAM user with AdministratorAccess       →  full account access exposed through forgotten credentials
  EC2 instance with SSH open to world     →  brute force attempts confirmed by GuardDuty within 24 hours

MONITORING:
  AWS Config        →  detects all three violations against managed compliance rules
  AWS CloudTrail    →  logs every API call and configuration change with timestamps
  Security Hub      →  aggregates Config and GuardDuty findings in one dashboard
  GuardDuty         →  detects real SSH brute force activity from VPC flow log analysis

AFTER STATE:
  All three Config rules: COMPLIANT
  EventBridge and SNS fire on any future compliance drift
  CloudWatch alarm triggers on unauthorized API call patterns
  Audit summary report documents all findings and remediation evidence
```

*Architecture diagram coming soon.*

---

## AWS Services Used

- **AWS Config** is the core detection tool. It continuously monitors resources against compliance rules and flags violations within minutes of them occurring.
- **AWS CloudTrail** logs every API call in the account with timestamps and identity attached. Log file validation creates a cryptographic hash chain proving logs have not been tampered with after delivery.
- **AWS Security Hub** aggregates findings from Config and GuardDuty into one centralized dashboard. A security score reflects the overall compliance posture of the account.
- **Amazon GuardDuty** analyzes VPC flow logs, CloudTrail logs, and DNS logs for malicious activity. In this project it surfaces real SSH brute force attempts against the deliberately vulnerable EC2 instance.
- **AWS IAM** is the target of one of the three audit remediations. AdministratorAccess attached directly to a user violates least privilege and is flagged by the iam-user-no-policies-check Config rule.
- **Amazon S3** is the target of one of the three audit remediations. A bucket with public access enabled violates the s3-bucket-public-read-prohibited Config rule and exposes files without authentication.
- **Amazon EC2** provides the t2.micro instance launched with the vulnerable security group to generate real VPC flow log activity for GuardDuty to analyze. It is terminated after findings are captured.
- **Amazon EventBridge** triggers an SNS email whenever Config reports any resource moving to NON_COMPLIANT. This is the continuous alerting layer for any future drift.
- **Amazon SNS** sends email alerts when EventBridge detects a compliance change or when the CloudWatch unauthorized API alarm fires.
- **Amazon CloudWatch** hosts the metric filter that watches CloudTrail logs for unauthorized API call patterns and fires an alarm when the pattern is detected.

---

## Obstacles — Constraints and Security Requirements

**The misconfigurations must be documented before anything is fixed.**
In a real engagement a client needs proof that the vulnerabilities existed before they will authorize the remediation work. Screenshots of Config violations, GuardDuty findings, and direct demonstrations of exploitability are the evidence package. The before state must be documented first.

**Real-world impact must be demonstrated, not just described.**
Saying a public S3 bucket is a security risk is not the same as showing a credentials file downloading in an incognito browser with no authentication. Saying an overpermissioned IAM user is dangerous is not the same as running CLI commands that prove full account access using those credentials. Saying an open SSH port attracts attacks is not the same as showing GuardDuty findings confirming active brute force attempts within 24 hours of launch. This project demonstrates the consequences rather than describing them.

**Continuous monitoring must replace one-time checking.**
Fixing three misconfigurations and declaring the environment secure is not the same as making it stay secure. EventBridge must be configured to fire an SNS alert the moment any resource drifts out of compliance again. The difference between a security audit and a security posture is what happens after the audit is done.

**Unauthorized API activity must be detectable in real time.**
CloudTrail logs every API call but does not alert on its own. A CloudWatch metric filter on the CloudTrail log group must detect the pattern of AccessDenied and unauthorized access errors that signal either compromised credentials being used or permission boundaries being probed. The alarm fires before a human would notice manually.

**The audit must produce documented evidence.**
The technical fixes are half the job. The other half is the audit summary, a written document that translates the findings, the demonstrated impact, and the remediation actions into something a non-technical stakeholder can read and act on. This project produces that document as a deliverable committed to the repository alongside the screenshots.

---

## Actions — What Was Built and Why

Coming soon, project is currently being built! 
---

## Let's Connect!

Brianne Young | Cloud Engineer | [LinkedIn](https://www.linkedin.com/in/brianne-young0/) | [GitHub](https://github.com/brianne-y)
