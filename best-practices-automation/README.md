# Best practices and automation
## Introduction
In this section we'll explore how to effectively use several AWS services to **automatically** detect, notify and remediate security incidents in your environment to resolve and respond to incidents without the need of manual intervention. We'll also explore security **best practices** that should be followed at all times. These best practices are built so that your solution has security built into every layer of its design.

## Automating security detection and remediation
Speed is **crucial** when responding to security incidents in your environment. To make the process as efficient as possible, we'll implement **as much automation as possible** to quickly **identify**, **respond**, and **remediate** detected security incidents. To help with this we'll look at `Amazon CloudWatch`, `Amazon GuardDuty` and `AWS Security Hub`.

### Using CloudWatch Events with AWS Lambda and SNS
In an earlier section we taked about how we can record and track infrastructure changes with AWS CloudTrail and AWS Config, and how these events can be written to an output channel to act on these events.

We can use these events on a data stream to then respond to potential security incidents. In this case, we can send certain events that appear on CloudTrail into **Amazon CloudWatch events**, and trigger an output task every time that event is triggered, in this case, triggering a **lambda function** that automatically remediates the issue. We can also send these alerts to an **SNS topic** simultaneously, which alerts us of the event.

These events can even be configured across accounts, which allows automated incident response across an entire organization.

### Using Amazon GuardDuty
As covered in earlier notes, Amazon GuardDuty is a fully managed threat detection service that uses **pattern anomaly detection** and machine learning to detect potential security incidents. This information is gathered through AWS CloudTrail logs, DNS logs and VPC flow logs. The service is **continuously learning** based in the environment's activity and gets better with usage.

As all data is processed intenrally by AWS, this has **no performance impact** on your infrastructure, and being **always-on** helps responding to potential incidents at any time, helping with critical timing-sensitive incidents. We can enable GuardDuty to immediately start gathering data for learning, and configure it to **automatically remediate certain events** if desired.

### Using AWS Security Hub
AWS Security Hub is a service focused around keeping **security findings** across your organization in a single place, so you can have **one centralised location** to easily and quickly check your security posture. It **aggregates information** from several AWS services, including:
- AWS Identity and Access Manager
- AWS Config
- Amazon Macie
- Amazon GuardDuty
- Amazon Inspector
- AWS Firewall Manager

In addition to first-party services from AWS it can also **integrate with external security tooling**, like **Splunk** or **CrowdStrike**. AWS Security Hub allows you to quickly assess your security posture against industry-leading standards and prioritize what should be fixed first.

AWS Security hub can also **remediate findings automatically** with its service integration capabilities

## Discovering security best practices
For each AWS service we've touched upon so far, there's a specific set of security best practices to get into, so there are numerous very specific security recommendations, but there are also **general best-practice guidelines** we should always follow. Here we shall review some of these common best practices that should be **implemented whenever possible**. We wll also dive into **AWS Trusted Advisor**, which alerts about deviations from common AWS security best practices.

### Common security best practices
There are many common AWS security best practices, we'll explore some of them, like:
- **Enabling Multi-factor** authentication for all access accounts possible in the environment so even with a credential leak, an attacker cannot act on the leaked access keys
- Enable AWS CloudTrail so you can track any actions that are executed on the account for potentially necessary auditing tasks
- **Remove your root account access keys** so programmatic access with absolute permissions into your account is not possible, as this account should be used as little as possible
- **Implement a strong password policy** so you're sure no easy to crack passwords are present on your environment
- **Implement the Principle of Least Privilege** so no account in your organization can act on any resources they should not be interacting with, wethere it's unintentionally or with malicious intent
- **Encrypt data at rest and in transit** so no attacker that has access to the resources can decrypt the contents without the properly secured keys
- **Implement automated remediation** so security incidents can be patched automatially as soon as they are detected and it doesn't depend on you being available

### Using AWS Trusted Advisor
AWS Trusted Advisor is an AWS service that scans your environment and gives you recommendations to patch and upgrade design flaws that need to be attended in five different domains:
- **Cost optimization** to identify resources that are not being optimally used and could be scaled down
- **Performance** checks that look for resources that could make use of provisioned throughput or are under-provisioned and need scaling up
- **Security** to identify weaknesses that could lead to incidents due to a vulnerability
- **Fault tolerance** to identify where our infrastructure could fail if a single availability zone has an incident
- **Service limits** usage detection for when you're approaching an AWS service soft limit

When you enable AWS Trusted Advisor, it will start scanning services on your platform and sending findings that you can correct on its service console. It will indicate *green checks*, or tests that have passed and are compliant, yellow checks which indicate **recommendations** and require some investigation, and red checks which indicate that **immediate action should be taken**.

The checks and scope of AWS Trusted Advisor is **tied to your Support plan with AWS**. Basic and Developer plans get access to 7 core Trusted Advisor checks, and Business or Enterprise support plan accounts get access to all Trusted Advisor checks, notifications and programmatic access to the reports. For each failing alert you'll get a set of **recommended actions**.

Te basic checks available with Basic or Developer support plans are:
- S3 bucket permissions
- Security groups - specific ports unrestricted
- IAM use
- MFA of root account
- EBS public snapshots
- RDS public snapshots
