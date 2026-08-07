---
title: "System Architecture"
date: 2026-07-27
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# CampusMeet System Architecture

CampusMeet uses a serverless AWS architecture to separate the frontend, API, data, and asynchronous integrations.

## Main request flow

```text
User
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
Google / Bedrock when needed
```

Cognito authenticates the user, while the backend still checks group membership, role, and resource scope before returning application data.

## Main AWS services

| Service | Role |
| --- | --- |
| Amazon Cognito | Authentication |
| API Gateway | HTTP API |
| AWS Lambda | Business logic |
| DynamoDB | Main application data |
| Amazon S3 | Frontend assets and user files |
| CloudFront | Production frontend over HTTPS |
| EventBridge Scheduler | Reminders and delayed retries |
| Step Functions | Long-running workflows |
| Amazon Bedrock | AI and retrieval |
| CloudWatch | Logs and monitoring |

## Infrastructure templates

CampusMeet separates infrastructure into several templates:

- `data-foundation.yaml` for DynamoDB tables;
- `auth-integration.yaml` for the smaller development auth/core stack;
- `user-content-orchestration.yaml` for user content and asynchronous workflows;
- `template.yaml` for the full application stack, including production frontend hosting, API, Google sync, AI, and monitoring.

The full production E2E uses the complete application stack rather than assuming the smaller auth stack contains every feature.

## Data and external services

DynamoDB remains the source of truth for CampusMeet business data. Google Calendar/Meet is an external synchronization target, not the primary Meeting store.

Large files are stored in S3. Google sync, reminders, and AI processing run asynchronously so a temporary external-service failure does not remove the main CampusMeet data.

## Production deployment order

```text
Data stack
  ↓
User-content / orchestration
  ↓
Full application stack
  ↓
Read API URL + CloudFront domain
  ↓
Build and publish frontend
  ↓
Run E2E on production URL
```

The final CloudFront URL is the production link used for the project demo and submission.