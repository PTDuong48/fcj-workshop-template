---
title: "Worklog Tuần 10"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

* Bắt đầu triển khai và cấu hình hạ tầng AWS cơ bản cho dự án cuối khóa.
* Đảm bảo tính bảo mật, giám sát (Observability), và quản lý chi phí cho toàn bộ hệ thống.
* Hoàn thiện Infrastructure as Code (IaC) để tự động hóa việc triển khai.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 7 | **Triển khai dự án:**<br>- Viết/hỗ trợ SAM template cho resource chính.<br>- Thiết kế IAM roles cho các dịch vụ (Lambda, Step Functions, API Gateway, Textract, DynamoDB, S3, SNS/SES).<br>- Thiết lập KMS encryption baseline.<br>- Lưu trữ External API key trong Secrets Manager an toàn.<br>- Cấu hình bảo mật S3 (Block Public Access, encryption).<br>- Kích hoạt CloudTrail để audit.<br>- Tạo CloudWatch log groups, metrics và alarms (Step Functions failed, DLQ not empty...).<br>- Bật X-Ray tracing.<br>- Tạo SNS topic và SES flow. | 20/06/2026 | 26/06/2026 | |


### Kết quả đạt được tuần 10:

* **Tự động hóa hạ tầng (IaC):** Viết và triển khai thành công AWS SAM template cho các tài nguyên chính.
* **Tăng cường Bảo mật (Security):** Thiết kế và phân quyền chặt chẽ các IAM Roles cho Lambda, Step Functions, API Gateway, Textract, S3, DynamoDB. Ứng dụng KMS để mã hóa dữ liệu, sử dụng Secrets Manager để bảo vệ External AI API key (chỉ cấp quyền đọc cho AI Proxy Lambda), và kích hoạt S3 Block Public Access. Bật CloudTrail nhằm theo dõi (audit) mọi hành động trên account.
* **Giám sát và Cảnh báo (Observability & Alerting):** Cấu hình toàn diện CloudWatch (log groups, custom metrics) cùng với X-Ray tracing. Đặc biệt, hệ thống đã thiết lập đầy đủ các Alarm quan trọng (như lỗi Step Functions, DLQ có message, Lambda lỗi, low confidence) và tích hợp luồng thông báo tự động qua SNS/SES (email).

