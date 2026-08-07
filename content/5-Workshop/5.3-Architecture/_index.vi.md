---
title: "Kiến trúc hệ thống"
date: 2026-07-27
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Kiến trúc hệ thống CampusMeet

CampusMeet sử dụng kiến trúc serverless để giảm việc quản lý máy chủ và tách rõ frontend, API, dữ liệu và các tác vụ bất đồng bộ.

## Luồng chính

```text
Người dùng
   ↓
CloudFront + React
   ↓
Amazon Cognito
   ↓
API Gateway
   ↓
AWS Lambda
   ↓
DynamoDB / S3
   ↓
Google / Bedrock khi cần
```

Cognito xác thực người dùng, nhưng quyền truy cập dữ liệu vẫn được kiểm tra ở backend dựa trên group, role và tài nguyên mà người dùng đang thao tác.

## Các thành phần chính

| Dịch vụ | Vai trò trong CampusMeet |
| --- | --- |
| Amazon Cognito | Xác thực người dùng |
| API Gateway | Nhận request từ frontend |
| AWS Lambda | Xử lý nghiệp vụ |
| DynamoDB | Lưu Group, Meeting, Minutes, Task và dữ liệu liên quan |
| Amazon S3 | Lưu frontend và tệp người dùng |
| CloudFront | Phục vụ frontend production qua HTTPS |
| EventBridge Scheduler | Reminder và retry theo thời gian |
| Step Functions | Điều phối các công việc dài |
| Amazon Bedrock | AI và retrieval |
| CloudWatch | Log và giám sát |

## Hạ tầng của dự án

CampusMeet tách hạ tầng thành các template để dễ triển khai và giảm rủi ro ảnh hưởng đến dữ liệu:

- `data-foundation.yaml`: các bảng DynamoDB.
- `auth-integration.yaml`: stack nhỏ dùng cho auth/core ở môi trường dev.
- `user-content-orchestration.yaml`: S3 user content, reminder và orchestration.
- `template.yaml`: application stack đầy đủ với API, frontend hosting, Google sync, AI và monitoring.

Trong bản production E2E, `template.yaml` là stack chính cho ứng dụng đầy đủ; `auth-integration.yaml` không đại diện cho toàn bộ CampusMeet.

## Dữ liệu và tích hợp ngoài

DynamoDB là nguồn dữ liệu nghiệp vụ chính. Google Calendar/Meet chỉ đồng bộ thông tin lịch và không thay thế Meeting bên trong CampusMeet.

Các tệp lớn được lưu trên S3 thay vì DynamoDB. Các tác vụ như Google sync, reminder hoặc AI được xử lý bất đồng bộ để một lỗi từ dịch vụ ngoài không làm mất dữ liệu cuộc họp chính.

## Thứ tự triển khai production

```text
Data stack
   ↓
User-content / orchestration
   ↓
Full application stack
   ↓
Lấy API URL + CloudFront domain
   ↓
Build và publish frontend
   ↓
E2E trên production URL
```

CloudFront URL sau cùng là đường dẫn production dùng để demo và nộp bài.