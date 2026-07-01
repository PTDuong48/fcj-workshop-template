---
title: "Week 10 Worklog"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:

* Begin deploying and configuring the foundational AWS infrastructure for the final project.
* Ensure stringent security, comprehensive observability, and cost management across the system.
* Finalize Infrastructure as Code (IaC) to automate deployments.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Saturday | **Project Implementation:**<br>- Write/support SAM templates for primary resources.<br>- Design IAM roles for services (Lambda, Step Functions, API Gateway, Textract, DynamoDB, S3, SNS/SES).<br>- Setup KMS encryption baseline.<br>- Store External API keys securely in Secrets Manager.<br>- Configure S3 security (Block Public Access, encryption).<br>- Enable CloudTrail for auditing.<br>- Create CloudWatch log groups, metrics, and alarms (Step Functions failed, DLQ not empty...).<br>- Enable X-Ray tracing.<br>- Set up SNS topics and SES flows. | 20/06/2026 | 26/06/2026 | |


### Week 10 Achievements:

* **Infrastructure Automation (IaC):** Successfully wrote and deployed AWS SAM templates for key resources.
* **Robust Security Posture:** Designed and tightly scoped IAM Roles across various services (Lambda, Step Functions, API Gateway, Textract, S3, DynamoDB). Applied KMS for data encryption, utilized Secrets Manager to safeguard External AI API keys (granting read access solely to the AI Proxy Lambda), and enforced S3 Block Public Access. Enabled CloudTrail to audit all account activities.
* **Observability & Alerting:** Comprehensively configured CloudWatch (log groups, custom metrics) and integrated X-Ray tracing. Crucially, established a full suite of critical Alarms (e.g., Step Functions failures, non-empty DLQs, Lambda errors, low confidence spikes) and automated notification flows via SNS and SES (email).

