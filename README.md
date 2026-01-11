# Cloud Threat Detection and Security Audit


![Alt text](/threatdetect.drawio.png)

- **CloudFormation** deploys intentionally insecure resources, EC2, open SSH, S3 public access, and IAM user admin access.

- **GuardDuty** continuously analyzes logs and detects suspicious activity.

- **AWS Config** evaluates resources against compliance rules like public S3 buckets and open ports.

- **Lambda audit function** performs custom checks for risky IAM users and public S3 buckets.

- **SNS** sends real-time alerts via email to the Security Engineering team.

---


# Scenario:

A mid-sized fintech company is expanding rapidly in the cloud. 

With dozens of AWS accounts and services being deployed, the security team struggles to keep track of risks such as public S3 buckets, open security groups, and unused IAM keys. 

Since they must comply with strict regulations, missing a misconfiguration or unusual login attempt could expose sensitive financial data and result in compliance violations.

---

# Solution:

**I've built a Cloud Threat Detection & Security Audit System that:**

- Deploys intentionally insecure resources automatically using AWS CloudFormation.

- Detects suspicious activity in near real-time using Amazon GuardDuty.

- Monitors for misconfigurations automatically with AWS Config Rules.

- Runs lightweight custom Lambda audits to check IAM and S3.

- Sends proactive alerts to the security team via Amazon SNS.

**This system enables the company to catch issues like public S3 buckets, risky IAM users, or open security groups before attackers or auditors find them.**

**These concepts are critical because cloud environments change rapidly, and relying only on manual checks is not enough. Automated detection ensures security gaps are caught early.**

---

## To test Lambda Audit Function, Config, and SNS alerts, we need resources that are deliberately insecure. 

**These resources simulate real-world security risks:**

- An EC2 instance with a wide-open security group (0.0.0.0/0).

- An S3 bucket with public read access.

- An IAM user with AdministratorAccess policy.

- EC2 Instance → Acts as a vulnerable server.

- S3 Bucket → Exposes sensitive data if left public.

- IAM User → Demonstrates the dangers of over-privileged accounts.

---

## Deploys intentionally insecure resources automatically using AWS CloudFormation.
<img width="1891" height="824" alt="Screenshot 2026-01-10 141723" src="https://github.com/user-attachments/assets/a20a073c-e71f-4e7d-ba75-fc5a410823b1" />
<img width="1613" height="437" alt="Screenshot 2026-01-10 141801" src="https://github.com/user-attachments/assets/ecb67e43-5d8b-4050-8537-aaa94428520c" />
<img width="1532" height="753" alt="Screenshot 2026-01-10 141833" src="https://github.com/user-attachments/assets/6ccdba14-c3e9-4843-8f00-f95d7e63b228" />
<img width="1540" height="401" alt="Screenshot 2026-01-10 141855" src="https://github.com/user-attachments/assets/41c3730e-e1c6-4387-9ab0-16b78c09064b" />

## Best Practice:
- Always enable S3 Block Public Access in real environments.

- IAM users with AdministratorAccess should be replaced with IAM roles and least-privilege access.

- Security groups should never allow 0.0.0.0/0 for all ports.

---

## Detects suspicious activity in near real-time using Amazon GuardDuty.

**Amazon GuardDuty provides intelligent threat detection across your AWS environment.**

**It continuously analyzes:**

- CloudTrail Logs → API activity

- VPC Flow Logs → Network traffic

- DNS Logs → Domain queries

**GuardDuty detects suspicious patterns such as unusual logins, privilege escalations, port scanning, and crypto-mining attempts.**

**By enabling it, we let AWS automatically monitor our intentionally insecure resources.**

**Generating sample findings.**
**This injects test alerts such as:**

- Recon:EC2/PortProbeUnprotectedPort

- UnauthorizedAccess:IAMUser/ConsoleLogin

- Trojan:EC2/BitcoinTool.B

<img width="1900" height="824" alt="Screenshot 2026-01-10 142753" src="https://github.com/user-attachments/assets/23e44e4f-1723-4a26-b05e-e288b36e9677" />

## Best Practice:

- GuardDuty should be enabled in every AWS region (attackers often target unused ones).

- Findings can be sent to CloudWatch Events/EventBridge for automated remediation.

- In production, forward findings to AWS Security Hub or a SIEM tool for centralized monitoring.

- GuardDuty is low maintenance, once enabled, it continuously runs in the background.

---

## While GuardDuty detects active threats, AWS Config helps ensure that your AWS resources remain compliant with security best practices.

## Config continuously records configuration changes and evaluates them against predefined rules or your own custom rules.

**By enabling Config:**

- Detects misconfigurations, like open security groups or public S3 buckets.

- Views compliance reports for the insecure resources.


**In the insecure setup:**

- EC2 Security Group (0.0.0.0/0) → Should fail the rule restricted-ssh.

- Public S3 Bucket → Should fail the rule s3-bucket-public-read-prohibited.

- IAM User with Admin → Can be flagged with iam-user-no-policies-check.

<img width="1877" height="686" alt="Screenshot 2026-01-10 142835" src="https://github.com/user-attachments/assets/9081747b-69dc-427f-857a-8d7973a71da9" />
<img width="1914" height="515" alt="Screenshot 2026-01-10 143114" src="https://github.com/user-attachments/assets/bbad19bb-0296-4a8b-b2e2-b18738e7590b" />


## Best Practice:

- Always enable AWS Config in all regions for full coverage.

- Pair Config with Auto Remediation, automatically close open SG ports.

- Forward Config compliance data to Security Hub for a centralized view.

- Treat Config as your compliance baseline, and GuardDuty as your threat detection layer.

---

## While GuardDuty and Config Rules provide strong built-in detection, you need custom checks for your own environment.

**In this step, I created a Lambda function that runs a lightweight security audit:**

- Checks if any S3 buckets are public.

- Checks if any IAM users have AdministratorAccess.

- Lambda Function → Runs on demand or scheduled to scan your account.

- AWS SDK (boto3 via Lambda runtime) → Calls S3 + IAM APIs to check misconfigurations.

- CloudWatch Logs → Stores audit results.

<img width="1894" height="566" alt="Screenshot 2026-01-10 143422" src="https://github.com/user-attachments/assets/9d495c43-d9e4-4957-b031-22a089d17b79" />
<img width="1911" height="818" alt="Screenshot 2026-01-10 134954" src="https://github.com/user-attachments/assets/73015ce3-9736-4499-922c-c1ac2adad0f7" />
<img width="1874" height="385" alt="Screenshot 2026-01-10 135045" src="https://github.com/user-attachments/assets/7b1a0a9c-5afb-464a-866e-962982a760cc" />

## Best Practice:

- Custom Lambda checks are great for quick wins but don’t replace Config Rules or Security Hub.

- Keep findings actionable, avoid alert fatigue.

- In production, findings should be sent to SNS → Email/Slack or a ticketing system.


---

## Security monitoring is incomplete without notifications. 

**By integrating Amazon SNS directly into our Lambda function, we can immediately send email alerts whenever a GuardDuty finding, Config violation, or Lambda audit result is detected.**

<img width="1880" height="649" alt="Screenshot 2026-01-10 140928" src="https://github.com/user-attachments/assets/b28d1764-a61b-4bd8-a5ea-c9bbb3540fb1" />
<img width="1545" height="534" alt="Screenshot 2026-01-10 141127" src="https://github.com/user-attachments/assets/87a0f2b2-d76e-4154-a840-73e33bbe1b4f" />

## Best Practice:

- In production, always use least privilege instead of full AmazonSNSFullAccess.

- SNS can also integrate with Slack, Lambda, or HTTP endpoints for broader workflows.

- Keep emails concise, too many alerts = alert fatigue.

---

# Conclusion:

## This project reflects real-world security practices by combining threat detection, compliance checks, custom auditing, and automated alerts in a cloud environment.

- For real-world setups, integrate GuardDuty + Config with AWS Security Hub for centralized compliance.

- Always review and scope IAM permissions to least privilege.

- Regularly schedule security audits and route alerts to a dedicated incident response team.

