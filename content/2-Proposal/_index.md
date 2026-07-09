---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

In this section, we present the high-level proposal and detailed technical blueprint for **DocuFlow AI**, a serverless intelligent invoice and receipt processing platform on AWS.

---

# DocuFlow AI
## Detailed Work Assignment Table — AeroOps Team (5 Members)
**Version:** AeroOps Team v2.0 — Updated to reflect admin-approved architecture  
**Last Updated:** June 28, 2026  
**Target Audience:** All AeroOps team members, mentors, and reviewers.

> **Update Principles:** Maintain the original assignment structure but update 100% according to the newly admin-approved architecture. Every service in the diagram must have an owner, deliverable, evidence, and cleanup responsibility. Remove services no longer in the current architecture; this document strictly follows the admin-approved diagram.

---

## 1. Project Overview

**DocuFlow AI** is an intelligent invoice and receipt processing platform built on an event-driven, fully serverless architecture on AWS. The system combines **Amazon Textract**, **AI Proxy Lambda & External AI API (outside AWS)**, **EventBridge + SQS + Step Functions**, and an **Observability/Security/Governance** layer to automate the extraction, normalization, and validation of financial data.

### Main Demo Flow

```
User → CloudFront/Amplify → Cognito → API Gateway → Lambda Presigned URL → S3 Raw
     → EventBridge → SQS + DLQ → Job Starter Lambda → Step Functions
     → Validate Lambda → Textract → AI Proxy Lambda → External AI API (outside AWS)
     → Confidence/Status Lambda → DynamoDB + S3 Processed
     → CloudWatch/X-Ray → SNS/SES Alert
```

### Approved Architecture

![DocuFlow AI approved architecture](/images/2-Proposal/Final_architecture.png)

---

## 2. Team Assignment Overview

| Member | Role | Primary Module | Main Services/Components | Difficulty |
| :--- | :--- | :--- | :--- | :--- |
| **Hoang Trong Tra** | Leader / Integration Owner | Ingestion, Queue & Workflow Orchestration | S3 raw/processed, EventBridge, SQS + DLQ, Job Starter Lambda, Step Functions | Hard |
| **Vu Duy Tai** | AI Owner | Textract, AI Proxy Lambda & External AI Normalization | Textract AnalyzeExpense, AI Proxy Lambda, External AI API (outside AWS), confidence/status logic | Hardest |
| **Nguyen Huu Tinh** | Frontend/Auth Owner | Frontend, Auth & User Flow | CloudFront, Amplify, Cognito, API Gateway integration, upload/result/review UI | Medium |
| **Lam Quang Loc** | Data Owner | Data Persistence & Result Management | DynamoDB, S3 processed JSON, document metadata, job state, simple report | Medium |
| **Pham Tung Duong** | Ops/Security/IaC Owner | Observability, Security, IaC & Cost Control | IAM, KMS, Secrets Manager, CloudTrail, Budgets, SAM, CloudWatch, X-Ray, SNS/SES | Medium - Hard |

---

## 3. Service Ownership Map

This table prevents any service in the architecture from lacking an assigned owner. Each owner is responsible for deployment, test evidence, logs/screenshots, and cleanup.

| Layer | Service/Component | Primary Owner | Specific Responsibility |
| :--- | :--- | :--- | :--- |
| Edge / Frontend | CloudFront, Amplify | Nguyen Huu Tinh | Frontend deploy, routing, UI build, workshop screenshots. |
| Authentication | Cognito | Nguyen Huu Tinh | Login/logout, protected routes, JWT handling. |
| API Layer | API Gateway | Tra + Tinh | Backend/frontend contract, route mapping, auth integration. |
| Upload Backend | Lambda Presigned URL | Hoang Trong Tra | Generate presigned URL, initial metadata/status. |
| Raw Storage | S3 Raw Bucket | Hoang Trong Tra | S3 naming, upload target, event source. |
| Ingestion Event | EventBridge | Hoang Trong Tra | ObjectCreated routing into the processing queue. |
| Queue | SQS + DLQ | Hoang Trong Tra | Queue, retry visibility, DLQ behavior and evidence. |
| Workflow Starter | Job Starter Lambda | Hoang Trong Tra | Poll queue/start Step Functions execution. |
| Workflow | Step Functions Standard | Hoang Trong Tra | State machine, retry/catch, execution history. |
| Validation | Validate Lambda | Hoang Trong Tra | Validate input/file metadata and update status. |
| OCR/Extraction | Textract AnalyzeExpense | Vu Duy Tai | Invoice/receipt extraction and raw output evidence. |
| AI Proxy | Lambda AI Proxy | Vu Duy Tai | Call HTTPS to External AI API, handle timeout/rate-limit, no API key exposure. |
| AI Normalization | External AI API (outside AWS) | Vu Duy Tai | Normalize/classify result, valid JSON, review reason. |
| Status Logic | Confidence + Status Lambda | Vu Duy Tai | Confidence score, EXTRACTED/REVIEW_REQUIRED/FAILED decision. |
| Metadata Store | DynamoDB | Lam Quang Loc | Document metadata, status, job state, idempotency. |
| Processed Result | S3 Processed JSON | Lam Quang Loc | result.json structure and sample output. |
| Logs/Metrics | CloudWatch | Pham Tung Duong | Log groups, metrics, alarms and dashboard evidence. |
| Tracing | X-Ray | Pham Tung Duong | Trace setup and screenshot if enabled in stack. |
| Alerting | SNS + SES | Pham Tung Duong | Alarm topic, email notification, alert test. |
| Security | IAM, KMS, Secrets Manager | Pham Tung Duong | Roles, encryption baseline, external API key secret. |
| Governance/Cost | CloudTrail, Budgets | Pham Tung Duong | Audit visibility, budget alerts, cost checklist. |
| Deployment | AWS SAM | Pham Tung Duong | sam build/deploy/delete, stack parameters, cleanup. |

---

## 4. Mandatory Shared Responsibilities

| Shared Item | Primary Owner | Reviewer | Required Outcome |
| :--- | :--- | :--- | :--- |
| Admin-approved Architecture MVP | Hoang Trong Tra | Everyone | No services added/removed after admin approval; all changes must be communicated to the team. |
| Data Contract JSON | Tra + Tai + Loc | Tinh, Duong | Unified schema for document result, required fields, review fields, metadata, and error fields. |
| Status Model | Hoang Trong Tra | Everyone | UPLOADED, QUEUED, PROCESSING, EXTRACTED, REVIEW_REQUIRED, FAILED, CORRECTED, APPROVED. |
| API Contract | Tra + Tinh + Loc | Tai, Duong | upload-url, list documents, get document, review update; sample request/response must be finalized before coding. |
| S3 Key Naming | Tra + Loc | Vu Duy Tai | raw/{userId}/{documentId}/original and processed/{userId}/{documentId}/result.json. |
| External AI API & AI Proxy Policy | Tai + Duong | Hoang Trong Tra | API key only in Secrets Manager; frontend must not call external AI directly; AI Proxy Lambda is the only layer calling HTTPS to external API; no secret logging. |
| IAM Boundary | Pham Tung Duong | Hoang Trong Tra | Per Lambda/Step Functions roles; least-privilege; reviewed before deploy. |
| Observability Contract | Duong + Tra | Everyone | Logs include documentId, userId, status, errorType; alarms for workflow/lambda/DLQ/low confidence. |
| Demo Script | Hoang Trong Tra | Everyone | End-to-end demo scenario, sample files, expected output, failure cases, and cleanup. |

---

## 5. Individual Member Task Details

### 5.1 Hoang Trong Tra — Ingestion, Queue & Workflow Orchestration + Integration Leadership

| Item | Details |
| :--- | :--- |
| **AWS services/components** | S3 raw/processed, EventBridge, SQS + DLQ, Lambda Presigned URL, Job Starter Lambda, Step Functions Standard, CloudWatch Logs, DynamoDB integration. |
| **Objective** | Maintain the system backbone. Ensure files are uploaded to the raw bucket, events flow through EventBridge/SQS, the workflow runs in correct order, statuses update properly, and other modules can integrate. |
| **Dependencies** | Upload/result API from Nguyen Huu Tinh; AI output from Vu Duy Tai; DynamoDB/S3 processed schema from Lam Quang Loc; IAM/deploy/logging from Pham Tung Duong. |

**Specific Tasks:**
- Design the flow: upload → S3 raw → EventBridge → SQS → Job Starter Lambda → Step Functions.
- Design S3 key naming for raw and processed buckets together with Loc.
- Create Lambda to generate presigned URL and `POST /documents/upload-url` API.
- Configure EventBridge rule for S3 ObjectCreated events.
- Create SQS processing queue, DLQ, and retry/visibility timeout rules.
- Create Job Starter Lambda to poll queue and start Step Functions execution.
- Design Step Functions Standard Workflow: validate → Textract → External AI → confidence/status → save DynamoDB/S3 → notification.
- Finalize status transitions and retry/catch policies for happy path/failure path.
- Finalize integration contracts with frontend, AI, and data modules.
- Conduct final integration test: login → upload → workflow → result → alert → cleanup.

**Required Deliverables:**

| Deliverable | Description |
| :--- | :--- |
| Workflow diagram or ASL/state machine definition | Evidence/screenshot/log/documentation |
| S3 event + EventBridge rule evidence | Evidence/screenshot/log/documentation |
| Working SQS + DLQ configuration | Evidence/screenshot/log/documentation |
| Step Functions execution history for happy path/failure path | Evidence/screenshot/log/documentation |
| Status update in DynamoDB | Evidence/screenshot/log/documentation |
| Integration contract for all modules | Evidence/screenshot/log/documentation |
| End-to-end demo script | Evidence/screenshot/log/documentation |

---

### 5.2 Vu Duy Tai — Textract, AI Proxy Lambda & External AI Normalization

| Item | Details |
| :--- | :--- |
| **AWS services/components** | Amazon Textract AnalyzeExpense, Lambda AI parser, Lambda AI Proxy, External AI API (outside AWS), Secrets Manager integration, CloudWatch Logs. |
| **Objective** | Transform invoices/receipts into clean JSON containing document type, vendor, invoice date, currency, total amount, tax amount, line items if available, confidence score, and review reasons. |
| **Dependencies** | S3 raw path and workflow input from Tra; data schema from Loc; result UI fields from Tinh; secret/API key and IAM permissions from Duong. |

**Specific Tasks:**
- Use Textract AnalyzeExpense for invoice/receipt MVP.
- Write Lambda to process raw Textract output into intermediate JSON.
- Design a lean payload from AI Proxy Lambda to External AI API, preferring extracted text/fields over raw files.
- Write AI Proxy Lambda to call External AI API via HTTPS and normalize/classify results.
- Retrieve API key from Secrets Manager in AI Proxy Lambda; no hard-coding and no key exposure to frontend.
- Design prompt/request so external AI returns valid JSON without fabricating missing fields.
- Validate output using shared JSON schema.
- Design confidence scoring based on Textract confidence, completeness, schema validity, and business rules.
- Handle blurry files, missing vendor/date/total/currency, invalid JSON, timeout/rate limits, invalid API keys.
- Do not log API keys or sensitive payloads to CloudWatch.

**Required Deliverables:**

| Deliverable | Description |
| :--- | :--- |
| Sample Textract raw output | Evidence/screenshot/log/documentation |
| Intermediate JSON after Textract | Evidence/screenshot/log/documentation |
| AI Proxy request + External AI response sample | Evidence/screenshot/log/documentation |
| Normalized JSON matching schema | Evidence/screenshot/log/documentation |
| JSON schema validation | Evidence/screenshot/log/documentation |
| Confidence score calculation | Evidence/screenshot/log/documentation |
| Test cases: clear invoice, blurry receipt, missing field, invalid JSON, AI Proxy/external API failure | Evidence/screenshot/log/documentation |

---

### 5.3 Nguyen Huu Tinh — Frontend, Auth & User Flow

| Item | Details |
| :--- | :--- |
| **AWS services/components** | Amplify, CloudFront, Cognito, API Gateway integration, S3 presigned upload. |
| **Objective** | Create the user experience: login/logout, upload invoice/receipt, track status, view results, and edit extraction output when review is required. |
| **Dependencies** | API contract from Tra; Status/schema from Tra + Tai + Loc; Cognito/API Gateway config from Duong; result field display from Loc. |

**Specific Tasks:**
- Create frontend app and deploy via Amplify, distributed via CloudFront per the approved architecture.
- Integrate Cognito login/logout and protected routes.
- Create upload page for PDF/JPG/PNG files.
- Call `POST /documents/upload-url` to retrieve the presigned URL.
- Upload files directly to S3 using the presigned URL.
- Display document list and document detail.
- Poll or refresh status by documentId.
- Display statuses: UPLOADED, QUEUED, PROCESSING, EXTRACTED, REVIEW_REQUIRED, FAILED, CORRECTED, APPROVED.
- Create reviewer form for REVIEW_REQUIRED: edit fields and approve/correct.
- Handle UI states: uploading, queued, processing, extracted, review required, failed.

**Required Deliverables:**

| Deliverable | Description |
| :--- | :--- |
| Login/logout page and protected route | Evidence/screenshot/log/documentation |
| Upload page working with presigned URL | Evidence/screenshot/log/documentation |
| Document list page | Evidence/screenshot/log/documentation |
| Document detail/result page | Evidence/screenshot/log/documentation |
| Basic reviewer page | Evidence/screenshot/log/documentation |
| UI error states | Evidence/screenshot/log/documentation |
| Screenshots for workshop guide | Evidence/screenshot/log/documentation |

---

### 5.4 Lam Quang Loc — Data Persistence & Result Management

| Item | Details |
| :--- | :--- |
| **AWS services/components** | DynamoDB, S3 processed JSON, Lambda data access functions/API integration. |
| **Objective** | Design the storage layer for metadata/status/results so that the frontend can query it, the workflow can update it, and the demo has a simple report from extracted data. |
| **Dependencies** | Output JSON from Tai; status updates from Tra; frontend display requirements from Tinh; IAM policy for DynamoDB/S3 from Duong. |

**Specific Tasks:**
- Design DynamoDB table for document metadata/status/job state.
- Finalize PK/SK/GSI for querying by user, documentId, status, createdAt.
- Design item schema including document metadata, extraction fields, confidence, review info, and error info.
- Design S3 processed result JSON at `processed/{userId}/{documentId}/result.json`.
- Create sample DynamoDB item and sample result.json.
- Design queries: get document by ID and list documents by user/status.
- Design review update: corrected fields, approved status, reviewedAt, reviewedBy.
- Create simple report: total documents, extracted count, review required count, failed count, total amount by vendor/month if data is sufficient.
- Ensure data schema matches AI output and UI requirements.
- Prepare sample data for the demo.

**Required Deliverables:**

| Deliverable | Description |
| :--- | :--- |
| DynamoDB table design | Evidence/screenshot/log/documentation |
| PK/SK/GSI proposal | Evidence/screenshot/log/documentation |
| Sample DynamoDB item | Evidence/screenshot/log/documentation |
| Sample S3 processed JSON | Evidence/screenshot/log/documentation |
| Data API response sample | Evidence/screenshot/log/documentation |
| Lightweight report/query | Evidence/screenshot/log/documentation |
| Data model explanation for the report | Evidence/screenshot/log/documentation |

---

### 5.5 Pham Tung Duong — Observability, Security, IaC & Cost Control

| Item | Details |
| :--- | :--- |
| **AWS services/components** | IAM, KMS, Secrets Manager, CloudTrail, AWS Budgets, AWS SAM, CloudWatch, X-Ray, SNS, SES, S3 security baseline. |
| **Objective** | Ensure the system is deployable, debuggable, has error alerts, maintains a security baseline, does not expose the external API key, and has cleanup procedures to control credits. |
| **Dependencies** | Resource list from all modules; workflow/log fields from Tra; AI secret requirements from Tai; API/auth from Tinh; DynamoDB/S3 permissions from Loc. |

**Specific Tasks:**
- Write or assist with SAM templates for main resources.
- Design IAM roles for Lambda, Step Functions, API Gateway, Textract, DynamoDB, S3, SNS/SES.
- Establish KMS encryption baseline per the architecture.
- Store External AI API key in Secrets Manager; only AI Proxy Lambda reads the required secret.
- Enable S3 Block Public Access and encryption.
- Enable CloudTrail for audit visibility across the account/project.
- Create CloudWatch log groups, metric filters/custom metrics as needed.
- Create alarms: Step Functions failed, DLQ not empty, Lambda error, low confidence spike.
- Enable X-Ray tracing for Lambda/API/Step Functions if configured in the stack.
- Create SNS topic and SES/email notification flow per the architecture.
- Create AWS Budget alerts and cost verification checklist.
- Write deploy/cleanup commands and instructions for resource deletion after the demo.

**Required Deliverables:**

| Deliverable | Description |
| :--- | :--- |
| SAM deploy template/commands | Evidence/screenshot/log/documentation |
| IAM role/policy summary | Evidence/screenshot/log/documentation |
| Secrets Manager setup evidence | Evidence/screenshot/log/documentation |
| CloudWatch logs/alarms screenshot | Evidence/screenshot/log/documentation |
| X-Ray trace screenshot | Evidence/screenshot/log/documentation |
| SNS/SES alert test | Evidence/screenshot/log/documentation |
| Budget alert screenshot | Evidence/screenshot/log/documentation |
| Deploy script and cleanup script | Evidence/screenshot/log/documentation |
| Security + cost control checklist | Evidence/screenshot/log/documentation |

---

## 6. RACI Matrix

> **Legend:** R = Responsible, A = Accountable, C = Consulted, I = Informed.

| Item / Task | Tra | Tai | Tinh | Loc | Duong |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Finalize Architecture MVP | **A/R** | C | C | C | C |
| Data Contract JSON | **A/R** | **R** | C | **R** | I |
| Status Model | **A/R** | C | C | C | I |
| Upload/Auth Flow | C | I | **A/R** | C | C |
| API Contract | **A/R** | C | **R** | **R** | C |
| S3 Ingestion | **A/R** | C | C | C | C |
| EventBridge/SQS/DLQ | **A/R** | I | I | C | C |
| Step Functions Workflow | **A/R** | C | I | C | C |
| Textract Extraction | C | **A/R** | I | C | C |
| AI Proxy / External AI Normalization | C | **A/R** | I | C | C |
| Secrets Manager/API Key | I | C | I | I | **A/R** |
| DynamoDB/S3 Processed | C | C | I | **A/R** | C |
| CloudWatch/X-Ray | C | I | I | C | **A/R** |
| SNS/SES Alert | C | I | I | I | **A/R** |
| IAM/KMS/CloudTrail/Budgets | C | I | I | I | **A/R** |
| SAM Deploy/Cleanup | C | I | I | I | **A/R** |
| Final Demo Integration | **A/R** | **R** | **R** | **R** | **R** |

---

## 7. Proposed Weekly Timeline

| Week | Overall Goal | Tra | Tai | Tinh | Loc | Duong |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Finalize design | Architecture/status/API draft | Textract + External AI plan | UI/API requirements | DynamoDB schema | SAM/security/cost plan |
| 2 | Foundation | S3/EventBridge/SQS base | Textract PoC | Cognito + Amplify skeleton | DynamoDB table | IAM + budget + secret plan |
| 3 | Upload & queue | Presigned URL + event/queue | Sample Textract output | Upload UI to S3 | Sample item/result JSON | S3/IAM baseline |
| 4 | Workflow skeleton | Step Functions happy path | Textract parser Lambda | Start process UX | Status query design | CloudWatch logs |
| 5 | AI integration | Connect workflow to AI Lambda | AI Proxy + External AI normalize + validation | Processing/result UI | Save result format | Secrets Manager setup |
| 6 | Storage integration | Save DynamoDB/S3 result | Confidence score | Detail result page | Get/list document API | IAM refinement |
| 7 | Review flow | Status transition | Review reasons | Reviewer UI | Corrected/approved storage | SNS/SES alert |
| 8 | Error handling | Retry/catch/failure path | Timeout/rate-limit/invalid JSON tests | UI error states | Data validation | Alarms + DLQ |
| 9 | Observability & report | Integration fixes | AI edge cases | UI polish | Lightweight report | X-Ray + CloudTrail + budgets |
| 10 | E2E testing | End-to-end test owner | AI test cases | UI screenshots | Data sample validation | Security/cost checklist |
| 11 | Documentation | Demo script | AI explanation | Frontend guide | Data model guide | Deploy/cleanup guide |
| 12 | Final demo | Demo integration | AI demo | Frontend demo | Data/report demo | Ops/security demo |

---

## 8. Module Handover Checklist

| Module | Required Checklist Items |
| :--- | :--- |
| **Frontend & Auth** | Login/logout; protected route; upload UI; status/result page; reviewer page; screenshots. |
| **Ingestion & Orchestration** | S3 event; EventBridge; SQS + DLQ; Job Starter Lambda; Step Functions; retry/catch; execution screenshot. |
| **AI Extraction** | Textract output; AI Proxy + External AI normalized JSON; confidence score; schema validation; edge-case tests. |
| **Data Persistence** | DynamoDB table; sample item; S3 processed result; get/list/update/approve queries; lightweight report. |
| **Observability/Security/IaC** | IAM roles; KMS/Secrets Manager; CloudWatch/X-Ray; SNS/SES; CloudTrail; Budgets; SAM deploy/cleanup; security checklist. |
| **Integration** | Upload → queue → workflow → Textract → External AI → DynamoDB/S3 → UI → alert path working with at least 3 sample files. |

---

## 9. Common Integration Failure Points

| Risk | Primary Owner | Mitigation Strategy |
| :--- | :--- | :--- |
| External AI output does not match schema | Tra + Tai + Loc | Finalize JSON schema first; validate via Lambda; use shared sample output. |
| Frontend upload succeeds but status does not progress | Tinh + Tra | Finalize flow: upload → S3 → event → queue → workflow; log documentId at each step. |
| External AI API key exposed | Duong + Tai | Store in Secrets Manager; frontend must not call external AI; only AI Proxy Lambda calls HTTPS externally; no secret logging; no key commits. |
| Workflow fails but is difficult to debug | Tra + Duong | Step Functions history + CloudWatch structured logs + X-Ray trace. |
| DynamoDB query does not match UI | Loc + Tinh | Finalize response samples for list/detail/review; mock before backend is complete. |
| IAM too broad or insufficient permissions | Duong + each owner | Each module lists required actions/resources; Duong reviews roles. |
| DLQ has messages but no one handles them | Tra + Duong | Alarm on DLQ not empty; assign owner to check and replay/delete messages. |
| Costs exceed credits | Duong + Tra | Budget alerts; limit sample files; cleanup daily; no resources outside diagram. |

---

## 10. Team Working Rules

- Do not add new services outside the approved architecture without a team meeting.
- Do not rename fields in JSON output without updating the data contract and notifying the frontend/data modules.
- Do not hard-code external API keys in the frontend, backend, or GitHub.
- Each member works on their own module but must attend weekly integration checkpoints.
- Each module must have at least one screenshot/log proving it works.
- All resources must use the prefix `docuflow-dev-*` and tag `Project=DocuFlowAI`.
- Final deploy coordinated by Duong/Tra to avoid duplicate resources or accidental stack deletion.
- After each test session, check Billing, CloudFormation stack, S3 bucket, Lambda, Step Functions, DynamoDB, CloudWatch, SNS/SES.
- Before the demo, freeze schema, endpoints, status model, and resource names.
- After the workshop, run the cleanup script and confirm no costly resources remain.

---

## 11. Data Contract & Status Model

The data contract must be finalized before modules develop in parallel. Do not rename fields without a joint review.

```json
{
  "documentId": "doc-001",
  "userId": "user-123",
  "fileName": "invoice-001.pdf",
  "documentType": "INVOICE",
  "status": "EXTRACTED",
  "vendorName": "ABC Company",
  "invoiceDate": "2026-06-01",
  "currency": "VND",
  "totalAmount": 2500000,
  "taxAmount": 250000,
  "confidenceScore": 0.91,
  "reviewReasons": [],
  "aiProvider": "external-ai-api",
  "normalizationMethod": "TEXTRACT_PLUS_AI_PROXY_EXTERNAL_API",
  "s3RawPath": "s3://docuflow-dev-raw-bucket/raw/user-123/doc-001/original.pdf",
  "s3ProcessedPath": "s3://docuflow-dev-processed-bucket/processed/user-123/doc-001/result.json",
  "createdAt": "2026-06-08T10:00:00Z",
  "updatedAt": "2026-06-08T10:01:00Z"
}
```

### Status Flow

```
UPLOADED → QUEUED → PROCESSING → EXTRACTED | REVIEW_REQUIRED | FAILED → CORRECTED → APPROVED
```

| Status | Description |
| :--- | :--- |
| **UPLOADED** | File successfully written to S3 Raw; initial record saved to DB. |
| **QUEUED** | Message entered SQS, awaiting Job Starter Lambda trigger. |
| **PROCESSING** | Step Functions execution is running. |
| **EXTRACTED** | Successful extraction; confidenceScore >= 0.80 and all required fields present. |
| **REVIEW_REQUIRED** | Extraction complete but missing key fields or confidenceScore < 0.80. |
| **FAILED** | Input validation failed or an unrecoverable exception occurred in Textract/External AI. |
| **CORRECTED** | User has edited fields in the reviewer form. |
| **APPROVED** | User has approved the result after review. |

---

## 12. Preliminary API Contract

| Endpoint | Purpose | Owner | Main Response |
| :--- | :--- | :--- | :--- |
| `POST /documents/upload-url` | Frontend retrieves presigned URL | Tinh + Tra | documentId, uploadUrl, s3RawPath, expiresIn |
| `GET /documents` | List documents by user/status | Loc + Tinh | items[], nextToken |
| `GET /documents/{documentId}` | View detail/result/status | Loc + Tinh | document metadata + extracted fields |
| `POST /documents/{documentId}/review` | Edit/approve results | Loc + Tinh | status, correctedFields, reviewedAt |

---

## 13. Definition of Done

- At least 3 sample invoices/receipts uploaded from the frontend after login.
- EventBridge/SQS/Step Functions run correctly for both happy path and failure path.
- Textract extracts key fields and the External AI API returns valid JSON.
- DynamoDB stores status/metadata/result per schema; S3 processed contains result.json.
- Frontend displays list/detail/result/review flow.
- CloudWatch logs include documentId; alarms active; SNS/SES sends notification test.
- Secrets Manager does not expose API key; IAM roles have clearly scoped permissions.
- Budget alert and cleanup script in place; stack/resources can be deleted after the demo.
- Each team member has individual evidence for their personal report section.

---

## 14. Budget Estimation & Cost Control

### Infrastructure Cost Breakdown (For 300 documents/month)

| Service | Estimated Cost |
| :--- | :--- |
| AWS Lambda | $0.00/month (within Free Tier) |
| Amazon S3 (storage & requests) | ~$0.15/month |
| AWS Amplify (frontend hosting) | ~$0.35/month |
| Amazon API Gateway | ~$0.01/month |
| Amazon Textract (AnalyzeExpense) | ~$0.08/month |
| External AI API (via AI Proxy) | ~$0.02/month |
| Data transfer & SNS Alerts | ~$0.02/month |
| **Total Estimate** | **~$0.70 USD/month** |

### Cost Control Policies

- **AWS Budgets:** Automatic alerts when costs exceed **$5.00** and **$10.00**.
- **File Limits:** Lambda validation rejects files larger than **5 MB** or over **3 pages**.
- **Lifecycle Rules:** Automatically expire files in raw/processed buckets after **14 days**.
- **Post-Demo Cleanup:** Run cleanup script and confirm no costly resources remain.

---

## 15. Risk Assessment & Mitigation

| Identified Risk | Impact | Probability | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **External AI returns malformed JSON** | Medium | Medium | Validate JSON structure immediately after External AI step; transition to `FAILED` or `REVIEW_REQUIRED` if invalid. |
| **Poor image quality / OCR errors** | High | Medium | Apply confidenceScore threshold; flag as `REVIEW_REQUIRED` for user confirmation. |
| **External AI API quota exhausted or connection failure** | High | Low | Handle timeout/rate-limit in AI Proxy Lambda; implement reasonable retries. |
| **API key / Credentials leak** | Critical | Low | Store in Secrets Manager; no hard-coding; no key commits to Git. |
| **Costs exceed credits** | Medium | Low | Budget alerts; cleanup script; limit file size and sample data. |
