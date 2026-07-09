---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

Tại phần này, chúng tôi trình bày bản đề xuất tổng quan và tài liệu đặc tả kỹ thuật chi tiết của dự án **DocuFlow AI** - Nền tảng xử lý hóa đơn và biên nhận thông minh sử dụng kiến trúc serverless trên nền tảng AWS.

---

# DocuFlow AI
## Bảng phân công công việc chi tiết — Nhóm AeroOps (5 người)
**Phiên bản:** AeroOps Team v2.0 — Cập nhật theo kiến trúc admin đã duyệt  
**Cập nhật ngày:** 28/06/2026  
**Đối tượng sử dụng:** Toàn bộ thành viên team AeroOps, Mentor và Reviewer.

> **Nguyên tắc cập nhật:** Giữ đúng cấu trúc phân công cũ, nhưng cập nhật 100% theo kiến trúc mới đã được admin duyệt. Mỗi service trong diagram phải có owner, deliverable, evidence và cleanup responsibility. Không giữ các service không còn nằm trong kiến trúc hiện tại; tài liệu chỉ bám theo diagram đã được admin duyệt.

---

## 1. Tổng quan dự án

**DocuFlow AI** là nền tảng xử lý hóa đơn (invoice) và biên nhận (receipt) thông minh, được xây dựng theo kiến trúc hướng sự kiện (event-driven) và hoàn toàn serverless trên nền tảng AWS. Hệ thống kết hợp **Amazon Textract**, **AI Proxy Lambda & External AI API ngoài AWS**, **EventBridge + SQS + Step Functions**, và lớp **Observability/Security/Governance** để tự động hóa quá trình trích xuất, chuẩn hóa và kiểm thử dữ liệu tài chính.

### Luồng demo chính

```
User → CloudFront/Amplify → Cognito → API Gateway → Lambda Presigned URL → S3 Raw
     → EventBridge → SQS + DLQ → Job Starter Lambda → Step Functions
     → Validate Lambda → Textract → AI Proxy Lambda → External AI API ngoài AWS
     → Confidence/Status Lambda → DynamoDB + S3 Processed
     → CloudWatch/X-Ray → SNS/SES Alert
```

### Kiến trúc đã chốt

![Kiến trúc DocuFlow AI phiên bản đã được duyệt](/images/2-Proposal/Final_architecture.png)

---

## 2. Bảng phân công tổng quan

| Thành viên | Vai trò | Module chính | Services/Components chính | Độ khó |
| :--- | :--- | :--- | :--- | :--- |
| **Hoàng Trọng Trà** | Leader / Integration Owner | Ingestion, Queue & Workflow Orchestration | S3 raw/processed, EventBridge, SQS + DLQ, Job Starter Lambda, Step Functions | Khó |
| **Vũ Duy Tài** | AI Owner | Textract, AI Proxy Lambda & External AI Normalization | Textract AnalyzeExpense, AI Proxy Lambda, External AI API ngoài AWS, confidence/status logic | Khó nhất |
| **Nguyễn Hữu Tịnh** | Frontend/Auth Owner | Frontend, Auth & User Flow | CloudFront, Amplify, Cognito, API Gateway integration, upload/result/review UI | Trung bình |
| **Lâm Quang Lộc** | Data Owner | Data Persistence & Result Management | DynamoDB, S3 processed JSON, document metadata, job state, simple report | Trung bình |
| **Phạm Tùng Dương** | Ops/Security/IaC Owner | Observability, Security, IaC & Cost Control | IAM, KMS, Secrets Manager, CloudTrail, Budgets, SAM, CloudWatch, X-Ray, SNS/SES | Trung bình - Khó |

---

## 3. Service Ownership Map

Bảng này giúp tránh tình trạng service xuất hiện trong architecture nhưng không có người chịu trách nhiệm. Owner chịu trách nhiệm triển khai, test evidence, log/screenshot và cleanup cho service đó.

| Layer | Service/Component | Owner chính | Trách nhiệm cụ thể |
| :--- | :--- | :--- | :--- |
| Edge / Frontend | CloudFront, Amplify | Nguyễn Hữu Tịnh | Frontend deploy, routing, UI build, screenshot workshop. |
| Authentication | Cognito | Nguyễn Hữu Tịnh | Login/logout, protected route, JWT handling. |
| API Layer | API Gateway | Hoàng + Tịnh | Contract backend/frontend, route mapping, auth integration. |
| Upload Backend | Lambda Presigned URL | Hoàng Trọng Trà | Generate presigned URL, initial metadata/status. |
| Raw Storage | S3 Raw Bucket | Hoàng Trọng Trà | S3 naming, upload target, event source. |
| Ingestion Event | EventBridge | Hoàng Trọng Trà | ObjectCreated routing vào processing queue. |
| Queue | SQS + DLQ | Hoàng Trọng Trà | Queue, retry visibility, DLQ behavior and evidence. |
| Workflow Starter | Job Starter Lambda | Hoàng Trọng Trà | Poll queue/start Step Functions execution. |
| Workflow | Step Functions Standard | Hoàng Trọng Trà | State machine, retry/catch, execution history. |
| Validation | Validate Lambda | Hoàng Trọng Trà | Validate input/file metadata and update status. |
| OCR/Extraction | Textract AnalyzeExpense | Vũ Duy Tài | Invoice/receipt extraction and raw output evidence. |
| AI Proxy | Lambda AI Proxy | Vũ Duy Tài | Gọi HTTPS ra External AI API, handle timeout/rate-limit, không lộ API key. |
| AI Normalization | External AI API ngoài AWS | Vũ Duy Tài | Normalize/classify result, valid JSON, review reason. |
| Status Logic | Confidence + Status Lambda | Vũ Duy Tài | Confidence score, EXTRACTED/REVIEW_REQUIRED/FAILED decision. |
| Metadata Store | DynamoDB | Lâm Quang Lộc | Document metadata, status, job state, idempotency. |
| Processed Result | S3 Processed JSON | Lâm Quang Lộc | result.json structure and sample output. |
| Logs/Metrics | CloudWatch | Phạm Tùng Dương | Log groups, metrics, alarms and dashboard evidence. |
| Tracing | X-Ray | Phạm Tùng Dương | Trace setup and screenshot if enabled in stack. |
| Alerting | SNS + SES | Phạm Tùng Dương | Alarm topic, email notification, alert test. |
| Security | IAM, KMS, Secrets Manager | Phạm Tùng Dương | Roles, encryption baseline, external API key secret. |
| Governance/Cost | CloudTrail, Budgets | Phạm Tùng Dương | Audit visibility, budget alerts, cost checklist. |
| Deployment | AWS SAM | Phạm Tùng Dương | sam build/deploy/delete, stack parameters, cleanup. |

---

## 4. Trách nhiệm chung bắt buộc

| Hạng mục chung | Owner chính | Người review | Kết quả cần chốt |
| :--- | :--- | :--- | :--- |
| Architecture MVP đã duyệt | Hoàng Trọng Trà | Tất cả | Không tự ý thêm/bớt service sau khi admin đã duyệt; mọi thay đổi phải thông báo team. |
| Data contract JSON | Hoàng + Tài + Lộc | Tịnh, Dương | Schema thống nhất cho document result, required fields, review fields, metadata và error fields. |
| Status model | Hoàng Trọng Trà | Tất cả | UPLOADED, QUEUED, PROCESSING, EXTRACTED, REVIEW_REQUIRED, FAILED, CORRECTED, APPROVED. |
| API contract | Hoàng + Tịnh + Lộc | Tài, Dương | upload-url, list documents, get document, review update; request/response mẫu phải chốt trước khi code. |
| S3 key naming | Hoàng + Lộc | Vũ Duy Tài | raw/{userId}/{documentId}/original và processed/{userId}/{documentId}/result.json. |
| External AI API & AI Proxy policy | Tài + Dương | Hoàng Trọng Trà | API key chỉ nằm trong Secrets Manager; frontend không gọi external AI trực tiếp; AI Proxy Lambda là lớp duy nhất gọi HTTPS ra external API; không log secret. |
| IAM boundary | Phạm Tùng Dương | Hoàng Trọng Trà | Role theo từng Lambda/Step Functions; privilege đủ dùng; review trước khi deploy. |
| Observability contract | Dương + Hoàng | Tất cả | Log có documentId, userId, status, errorType; có alarm cho workflow/lambda/DLQ/low confidence. |
| Demo script | Hoàng Trọng Trà | Tất cả | Kịch bản demo end-to-end, sample files, expected output, failure case và cleanup. |

---

## 5. Chi tiết công việc theo từng thành viên

### 5.1 Hoàng Trọng Trà — Ingestion, Queue & Workflow Orchestration + Integration Leadership

| Hạng mục | Nội dung |
| :--- | :--- |
| **AWS services/components** | S3 raw/processed, EventBridge, SQS + DLQ, Lambda Presigned URL, Job Starter Lambda, Step Functions Standard, CloudWatch Logs, DynamoDB integration. |
| **Mục tiêu** | Giữ xương sống hệ thống. Đảm bảo file upload được đưa vào raw bucket, event đi qua EventBridge/SQS, workflow chạy đúng thứ tự, status cập nhật đúng và các module khác tích hợp được. |
| **Phụ thuộc** | API upload/result từ Nguyễn Hữu Tịnh; AI output từ Vũ Duy Tài; DynamoDB/S3 processed schema từ Lâm Quang Lộc; IAM/deploy/logging từ Phạm Tùng Dương. |

**Công việc cụ thể:**
- Thiết kế luồng upload → S3 raw → EventBridge → SQS → Job Starter Lambda → Step Functions.
- Thiết kế S3 key naming cho raw và processed cùng Lộc.
- Tạo Lambda generate presigned URL và API `POST /documents/upload-url`.
- Cấu hình EventBridge rule cho S3 ObjectCreated event.
- Tạo SQS processing queue, DLQ và quy tắc retry/visibility timeout.
- Tạo Job Starter Lambda để poll queue và start Step Functions execution.
- Thiết kế Step Functions Standard Workflow: validate → Textract → External AI → confidence/status → save DynamoDB/S3 → notification.
- Chốt status transition và retry/catch policy cho happy path/failure path.
- Chốt integration contract với frontend, AI và data module.
- Làm integration test cuối: login → upload → workflow → result → alert → cleanup.

**Deliverables cần nộp:**

| Deliverable | Mô tả |
| :--- | :--- |
| Workflow diagram hoặc ASL/state machine definition | Evidence/screenshot/log/tài liệu tương ứng |
| S3 event + EventBridge rule evidence | Evidence/screenshot/log/tài liệu tương ứng |
| SQS + DLQ cấu hình chạy được | Evidence/screenshot/log/tài liệu tương ứng |
| Step Functions execution history cho happy path/failure path | Evidence/screenshot/log/tài liệu tương ứng |
| Status update trong DynamoDB | Evidence/screenshot/log/tài liệu tương ứng |
| Integration contract cho các module | Evidence/screenshot/log/tài liệu tương ứng |
| Demo script end-to-end | Evidence/screenshot/log/tài liệu tương ứng |

---

### 5.2 Vũ Duy Tài — Textract, AI Proxy Lambda & External AI Normalization

| Hạng mục | Nội dung |
| :--- | :--- |
| **AWS services/components** | Amazon Textract AnalyzeExpense, Lambda AI parser, Lambda AI Proxy, External AI API ngoài AWS, Secrets Manager integration, CloudWatch Logs. |
| **Mục tiêu** | Biến invoice/receipt thành JSON sạch, có document type, vendor, invoice date, currency, total amount, tax amount, line items nếu có, confidence score và review reasons. |
| **Phụ thuộc** | S3 raw path và workflow input từ Hoàng; data schema từ Lộc; result UI field từ Tịnh; secret/API key và IAM permission từ Dương. |

**Công việc cụ thể:**
- Dùng Textract AnalyzeExpense cho invoice/receipt MVP.
- Viết Lambda xử lý raw output của Textract thành intermediate JSON.
- Thiết kế payload gọn gửi từ AI Proxy Lambda sang External AI API, ưu tiên gửi text/fields đã extract thay vì raw file.
- Viết AI Proxy Lambda để gọi External AI API qua HTTPS và normalize/classify kết quả.
- Lấy API key từ Secrets Manager trong AI Proxy Lambda, không hard-code và không đưa key ra frontend.
- Thiết kế prompt/request để external AI trả JSON hợp lệ, không bịa field thiếu.
- Validate output bằng JSON schema chung.
- Thiết kế confidence scoring dựa trên Textract confidence, completeness, schema validity và business rules.
- Xử lý file mờ, thiếu vendor/date/total/currency, invalid JSON, timeout/rate limit, invalid API key.
- Không log API key hoặc payload nhạy cảm vào CloudWatch.

**Deliverables cần nộp:**

| Deliverable | Mô tả |
| :--- | :--- |
| Sample Textract raw output | Evidence/screenshot/log/tài liệu tương ứng |
| Intermediate JSON sau Textract | Evidence/screenshot/log/tài liệu tương ứng |
| AI Proxy request + External AI response mẫu | Evidence/screenshot/log/tài liệu tương ứng |
| Normalized JSON đúng schema | Evidence/screenshot/log/tài liệu tương ứng |
| JSON schema validation | Evidence/screenshot/log/tài liệu tương ứng |
| Confidence score calculation | Evidence/screenshot/log/tài liệu tương ứng |
| Test cases: invoice rõ, receipt mờ, missing field, invalid JSON, AI Proxy/external API fail | Evidence/screenshot/log/tài liệu tương ứng |

---

### 5.3 Nguyễn Hữu Tịnh — Frontend, Auth & User Flow

| Hạng mục | Nội dung |
| :--- | :--- |
| **AWS services/components** | Amplify, CloudFront, Cognito, API Gateway integration, S3 presigned upload. |
| **Mục tiêu** | Tạo trải nghiệm người dùng: login/logout, upload invoice/receipt, theo dõi status, xem result và chỉnh sửa kết quả khi cần review. |
| **Phụ thuộc** | API contract từ Hoàng; Status/schema từ Hoàng + Tài + Lộc; Cognito/API Gateway config từ Dương; result field display từ Lộc. |

**Công việc cụ thể:**
- Tạo frontend app và deploy qua Amplify, phân phối qua CloudFront theo kiến trúc.
- Tích hợp Cognito login/logout và protected routes.
- Tạo trang upload PDF/JPG/PNG.
- Gọi `POST /documents/upload-url` để lấy presigned URL.
- Upload file trực tiếp lên S3 bằng presigned URL.
- Hiển thị document list và document detail.
- Poll hoặc refresh status theo documentId.
- Hiển thị trạng thái: UPLOADED, QUEUED, PROCESSING, EXTRACTED, REVIEW_REQUIRED, FAILED, CORRECTED, APPROVED.
- Tạo reviewer form cho REVIEW_REQUIRED: sửa field và approve/correct.
- Xử lý UI states: uploading, queued, processing, extracted, review required, failed.

**Deliverables cần nộp:**

| Deliverable | Mô tả |
| :--- | :--- |
| Login/logout page và protected route | Evidence/screenshot/log/tài liệu tương ứng |
| Upload page hoạt động với presigned URL | Evidence/screenshot/log/tài liệu tương ứng |
| Document list page | Evidence/screenshot/log/tài liệu tương ứng |
| Document detail/result page | Evidence/screenshot/log/tài liệu tương ứng |
| Reviewer page cơ bản | Evidence/screenshot/log/tài liệu tương ứng |
| UI error states | Evidence/screenshot/log/tài liệu tương ứng |
| Screenshot cho workshop guide | Evidence/screenshot/log/tài liệu tương ứng |

---

### 5.4 Lâm Quang Lộc — Data Persistence & Result Management

| Hạng mục | Nội dung |
| :--- | :--- |
| **AWS services/components** | DynamoDB, S3 processed JSON, Lambda data access functions/API integration. |
| **Mục tiêu** | Thiết kế nơi lưu metadata/status/result để frontend query được, workflow update được và demo có báo cáo đơn giản từ dữ liệu đã extract. |
| **Phụ thuộc** | Output JSON từ Tài; status update từ Hoàng; frontend display requirement từ Tịnh; IAM policy cho DynamoDB/S3 từ Dương. |

**Công việc cụ thể:**
- Thiết kế DynamoDB table document metadata/status/job state.
- Chốt PK/SK/GSI phù hợp cho query theo user, documentId, status, createdAt.
- Thiết kế item schema gồm document metadata, extraction fields, confidence, review info và error info.
- Thiết kế S3 processed result JSON tại `processed/{userId}/{documentId}/result.json`.
- Tạo sample DynamoDB item và sample result.json.
- Thiết kế query get document by ID và list documents by user/status.
- Thiết kế review update: corrected fields, approved status, reviewedAt, reviewedBy.
- Tạo báo cáo đơn giản: total documents, extracted count, review required count, failed count, total amount by vendor/month nếu dữ liệu đủ.
- Đảm bảo schema data khớp output AI và UI.
- Chuẩn bị sample data cho demo.

**Deliverables cần nộp:**

| Deliverable | Mô tả |
| :--- | :--- |
| DynamoDB table design | Evidence/screenshot/log/tài liệu tương ứng |
| PK/SK/GSI proposal | Evidence/screenshot/log/tài liệu tương ứng |
| Sample DynamoDB item | Evidence/screenshot/log/tài liệu tương ứng |
| S3 processed JSON mẫu | Evidence/screenshot/log/tài liệu tương ứng |
| Data API response mẫu | Evidence/screenshot/log/tài liệu tương ứng |
| Lightweight report/query | Evidence/screenshot/log/tài liệu tương ứng |
| Data model explanation cho báo cáo | Evidence/screenshot/log/tài liệu tương ứng |

---

### 5.5 Phạm Tùng Dương — Observability, Security, IaC & Cost Control

| Hạng mục | Nội dung |
| :--- | :--- |
| **AWS services/components** | IAM, KMS, Secrets Manager, CloudTrail, AWS Budgets, AWS SAM, CloudWatch, X-Ray, SNS, SES, S3 security baseline. |
| **Mục tiêu** | Đảm bảo hệ thống deploy được, debug được, có cảnh báo lỗi, có security baseline, không lộ external API key và có cleanup để kiểm soát credits. |
| **Phụ thuộc** | Danh sách resource từ tất cả module; workflow/log fields từ Hoàng; AI secret requirement từ Tài; API/auth từ Tịnh; DynamoDB/S3 permission từ Lộc. |

**Công việc cụ thể:**
- Viết hoặc hỗ trợ SAM template cho resource chính.
- Thiết kế IAM roles cho Lambda, Step Functions, API Gateway integration, Textract, DynamoDB, S3, SNS/SES.
- Thiết lập KMS encryption baseline theo kiến trúc.
- Lưu External AI API key trong Secrets Manager; chỉ AI Proxy Lambda được đọc secret cần thiết.
- Bật S3 Block Public Access và encryption.
- Bật CloudTrail để có audit visibility cho account/project.
- Tạo CloudWatch log groups, metric filters/custom metrics nếu cần.
- Tạo alarms: Step Functions failed, DLQ not empty, Lambda error, low confidence spike.
- Bật X-Ray tracing cho Lambda/API/Step Functions nếu được cấu hình trong stack.
- Tạo SNS topic và SES/email notification flow theo kiến trúc.
- Tạo AWS Budget alerts và checklist kiểm tra chi phí.
- Viết deploy/cleanup commands và hướng dẫn xóa resource sau demo.

**Deliverables cần nộp:**

| Deliverable | Mô tả |
| :--- | :--- |
| SAM deploy template/commands | Evidence/screenshot/log/tài liệu tương ứng |
| IAM role/policy summary | Evidence/screenshot/log/tài liệu tương ứng |
| Secrets Manager setup evidence | Evidence/screenshot/log/tài liệu tương ứng |
| CloudWatch logs/alarms screenshot | Evidence/screenshot/log/tài liệu tương ứng |
| X-Ray trace screenshot | Evidence/screenshot/log/tài liệu tương ứng |
| SNS/SES alert test | Evidence/screenshot/log/tài liệu tương ứng |
| Budget alert screenshot | Evidence/screenshot/log/tài liệu tương ứng |
| Deploy script và cleanup script | Evidence/screenshot/log/tài liệu tương ứng |
| Security + cost control checklist | Evidence/screenshot/log/tài liệu tương ứng |

---

## 6. RACI Matrix

> **Ký hiệu:** R = Responsible (Thực hiện), A = Accountable (Chịu trách nhiệm), C = Consulted (Tham vấn), I = Informed (Nhận thông báo).

| Hạng mục | Trà | Tài | Tịnh | Lộc | Dương |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Chốt architecture MVP | **A/R** | C | C | C | C |
| Data contract JSON | **A/R** | **R** | C | **R** | I |
| Status model | **A/R** | C | C | C | I |
| Upload/auth flow | C | I | **A/R** | C | C |
| API contract | **A/R** | C | **R** | **R** | C |
| S3 ingestion | **A/R** | C | C | C | C |
| EventBridge/SQS/DLQ | **A/R** | I | I | C | C |
| Step Functions workflow | **A/R** | C | I | C | C |
| Textract extraction | C | **A/R** | I | C | C |
| AI Proxy / External AI normalization | C | **A/R** | I | C | C |
| Secrets Manager/API key | I | C | I | I | **A/R** |
| DynamoDB/S3 processed | C | C | I | **A/R** | C |
| CloudWatch/X-Ray | C | I | I | C | **A/R** |
| SNS/SES alert | C | I | I | I | **A/R** |
| IAM/KMS/CloudTrail/Budgets | C | I | I | I | **A/R** |
| SAM deploy/cleanup | C | I | I | I | **A/R** |
| Final demo integration | **A/R** | **R** | **R** | **R** | **R** |

---

## 7. Timeline đề xuất theo tuần

| Tuần | Mục tiêu chung | Trà | Tài | Tịnh | Lộc | Dương |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Chốt thiết kế | Architecture/status/API draft | Textract + External AI plan | UI/API requirements | DynamoDB schema | SAM/security/cost plan |
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

## 8. Checklist bàn giao theo từng module

| Module | Checklist phải có |
| :--- | :--- |
| **Frontend & Auth** | Login/logout; protected route; upload UI; status/result page; reviewer page; screenshots. |
| **Ingestion & Orchestration** | S3 event; EventBridge; SQS + DLQ; Job Starter Lambda; Step Functions; retry/catch; execution screenshot. |
| **AI Extraction** | Textract output; AI Proxy + External AI normalized JSON; confidence score; schema validation; edge-case tests. |
| **Data Persistence** | DynamoDB table; sample item; S3 processed result; get/list/update/approve query; lightweight report. |
| **Observability/Security/IaC** | IAM roles; KMS/Secrets Manager; CloudWatch/X-Ray; SNS/SES; CloudTrail; Budgets; SAM deploy/cleanup; security checklist. |
| **Integration** | Upload → queue → workflow → Textract → External AI → DynamoDB/S3 → UI → alert path chạy được với ít nhất 3 sample files. |

---

## 9. Những điểm dễ gây vỡ tích hợp

| Rủi ro | Người chịu trách nhiệm chính | Cách giảm rủi ro |
| :--- | :--- | :--- |
| External AI output không khớp schema | Hoàng + Tài + Lộc | Chốt JSON schema trước; validate bằng Lambda; dùng sample output chung. |
| Frontend upload xong nhưng status không chạy | Tịnh + Hoàng | Chốt flow upload → S3 → event → queue → workflow; log documentId ở từng bước. |
| External AI API key bị lộ | Dương + Tài | Lưu trong Secrets Manager; frontend không gọi external AI; chỉ AI Proxy Lambda gọi HTTPS ra ngoài; không log secret; không commit key. |
| Workflow fail nhưng khó debug | Hoàng + Dương | Step Functions history + CloudWatch structured logs + X-Ray trace. |
| DynamoDB query không khớp UI | Lộc + Tịnh | Chốt response mẫu cho list/detail/review; mock trước khi backend xong. |
| IAM quá rộng hoặc thiếu quyền | Dương + từng owner | Mỗi module liệt kê action/resource cần dùng; Dương review role. |
| DLQ có message nhưng không ai xử lý | Hoàng + Dương | Alarm DLQ not empty; quy định owner kiểm tra và replay/delete message. |
| Chi phí vượt credits | Dương + Hoàng | Budget alerts; giới hạn sample files; cleanup cuối ngày; không tạo resource ngoài diagram. |

---

## 10. Quy tắc làm việc nhóm

- Không thêm service mới ngoài kiến trúc đã duyệt nếu chưa họp team.
- Không đổi tên field trong JSON output nếu chưa cập nhật data contract và báo cho frontend/data module.
- Không hard-code external API key trong frontend, backend hoặc GitHub.
- Mỗi người làm module riêng nhưng phải có integration checkpoint hằng tuần.
- Mỗi module phải có ít nhất một screenshot/log chứng minh đã chạy được.
- Tất cả resource phải dùng prefix `docuflow-dev-*` và tag `Project=DocuFlowAI`.
- Final deploy do Dương/Trà điều phối để tránh trùng resource hoặc xóa nhầm stack.
- Sau mỗi buổi test phải kiểm tra Billing, CloudFormation stack, S3 bucket, Lambda, Step Functions, DynamoDB, CloudWatch, SNS/SES.
- Trước demo phải freeze schema, endpoint, status model và resource name.
- Sau workshop phải chạy cleanup script và xác nhận không còn resource tốn phí.

---

## 11. Data Contract & Status Model

Data contract cần được chốt trước khi các module code song song. Không đổi tên field nếu chưa review chung.

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

### Status flow

```
UPLOADED → QUEUED → PROCESSING → EXTRACTED | REVIEW_REQUIRED | FAILED → CORRECTED → APPROVED
```

| Trạng thái | Mô tả |
| :--- | :--- |
| **UPLOADED** | File đã upload lên S3 Raw thành công, bản ghi khởi tạo đã lưu vào DB. |
| **QUEUED** | Message đã vào SQS, chờ Job Starter Lambda kích hoạt. |
| **PROCESSING** | Step Functions execution đang chạy. |
| **EXTRACTED** | Trích xuất thành công, confidenceScore >= 0.80 và đủ các trường bắt buộc. |
| **REVIEW_REQUIRED** | Hoàn tất nhưng thiếu field quan trọng hoặc confidenceScore < 0.80. |
| **FAILED** | Lỗi validate đầu vào hoặc exception không thể khôi phục trong Textract/External AI. |
| **CORRECTED** | Người dùng đã sửa field trong reviewer form. |
| **APPROVED** | Người dùng đã phê duyệt kết quả sau review. |

---

## 12. API Contract sơ bộ

| Endpoint | Mục đích | Owner | Response chính |
| :--- | :--- | :--- | :--- |
| `POST /documents/upload-url` | Frontend lấy presigned URL | Tịnh + Trà | documentId, uploadUrl, s3RawPath, expiresIn |
| `GET /documents` | List documents theo user/status | Lộc + Tịnh | items[], nextToken |
| `GET /documents/{documentId}` | Xem detail/result/status | Lộc + Tịnh | document metadata + extracted fields |
| `POST /documents/{documentId}/review` | Sửa/approve kết quả | Lộc + Tịnh | status, correctedFields, reviewedAt |

---

## 13. Definition of Done

- Upload được ít nhất 3 sample invoice/receipt từ frontend sau khi login.
- EventBridge/SQS/Step Functions chạy đúng với happy path và failure path.
- Textract extract được field chính và External AI API trả JSON hợp lệ.
- DynamoDB lưu status/metadata/result đúng schema; S3 processed có result.json.
- Frontend hiển thị list/detail/result/review flow.
- CloudWatch logs có documentId; alarm hoạt động; SNS/SES gửi được notification test.
- Secrets Manager không lộ API key; IAM role có phạm vi quyền rõ ràng.
- Có Budget alert và cleanup script; sau demo xóa được stack/resource chính.
- Mỗi thành viên có evidence riêng cho phần báo cáo cá nhân.

---

## 14. Ước tính ngân sách & Kiểm soát chi phí

### Ước tính chi phí vận hành (Cho 300 hóa đơn/tháng)

| Dịch vụ | Chi phí ước tính |
| :--- | :--- |
| AWS Lambda | $0.00/tháng (trong Free Tier) |
| Amazon S3 (lưu trữ & request) | ~$0.15/tháng |
| AWS Amplify (hosting frontend) | ~$0.35/tháng |
| Amazon API Gateway | ~$0.01/tháng |
| Amazon Textract (AnalyzeExpense) | ~$0.08/tháng |
| External AI API (qua AI Proxy) | ~$0.02/tháng |
| Truyền tải dữ liệu & SNS Alerts | ~$0.02/tháng |
| **Tổng ước tính** | **~$0.70 USD/tháng** |

### Quy định kiểm soát chi phí

- **AWS Budgets:** Cảnh báo tự động khi chi phí vượt **$5.00** và **$10.00**.
- **Giới hạn file:** Lambda validate từ chối file lớn hơn **5 MB** hoặc quá **3 trang**.
- **Lifecycle Rules:** Tự động xóa file trong raw/processed bucket sau **14 ngày**.
- **Cleanup sau demo:** Chạy cleanup script và xác nhận không còn resource tốn phí.

---

## 15. Đánh giá rủi ro & Biện pháp khắc phục

| Rủi ro xác định | Ảnh hưởng | Xác suất | Biện pháp giảm thiểu |
| :--- | :--- | :--- | :--- |
| **External AI phản hồi JSON sai cấu trúc** | Trung bình | Trung bình | Validate cấu trúc JSON ngay sau bước External AI; chuyển sang `FAILED` hoặc `REVIEW_REQUIRED` nếu sai. |
| **Chất lượng ảnh mờ / OCR lỗi** | Cao | Trung bình | Áp dụng confidenceScore threshold; gắn cờ `REVIEW_REQUIRED` để người dùng xác nhận. |
| **External AI API hết hạn ngạch hoặc lỗi kết nối** | Cao | Thấp | Xử lý timeout/rate-limit trong AI Proxy Lambda; có retry hợp lý. |
| **Lộ API key / Credentials leak** | Nghiêm trọng | Thấp | Lưu trong Secrets Manager; không hard-code; không commit key lên Git. |
| **Chi phí vượt credits** | Trung bình | Thấp | Budget alerts; cleanup script; giới hạn file size và sample data. |
