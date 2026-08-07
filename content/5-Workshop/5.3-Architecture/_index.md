---
title: "System Architecture"
date: 2026-07-27
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# CampusMeet System Architecture

## Goal

This section explains how CampusMeet separates long-lived data, core API functionality, asynchronous processing, and the full application environment. Keeping these boundaries explicit makes later deployments easier to reason about and prevents a small application change from being mistaken for a full-system deployment.

## 1. Design principles

CampusMeet follows a few consistent rules:

- the frontend never reads DynamoDB directly;
- Cognito authenticates identity, while Lambda services enforce domain authorization;
- business data is stored in five shared DynamoDB tables;
- files and larger content are stored in S3;
- long-running and external-service work is asynchronous;
- Google Calendar/Meet is not the source of truth for meetings;
- AI retrieval is restricted to sources the caller is allowed to read;
- infrastructure changes are managed through SAM/CloudFormation.

## 2. Main request path

```text
User
 ↓
CampusMeet Web
 ↓
Amazon Cognito
 ↓ JWT
Amazon API Gateway
 ↓
AWS Lambda API
 ↓
Business service
 ↓
Repository / integration adapter
 ↓
DynamoDB / S3 / Google / Bedrock
```

A protected request therefore has two independent checks: token validation at the API boundary and resource authorization inside the application service.

## 3. Current infrastructure boundaries

The repository currently contains four important infrastructure templates.

| Template | Responsibility |
| --- | --- |
| `infra/data-foundation.yaml` | Five DynamoDB tables and the `meeting-data` stream |
| `infra/auth-integration.yaml` | Smaller dev/core stack for Cognito, HTTP API, and core M1/M2 functionality |
| `infra/user-content-orchestration.yaml` | User-content S3, Step Functions, reminders, Scheduler, and M4 orchestration resources |
| `infra/template.yaml` | Full application stack with frontend hosting, API, Cognito, Google sync, AI worker, Bedrock, and monitoring |

`auth-integration.yaml` and `template.yaml` are not interchangeable. The smaller auth/core stack is useful for early workshop exercises, while the full application stack is required when testing Minutes, Tasks, Upload, Google, and AI together.

## 4. Data foundation

The data stack owns:

```text
campusmeet-<env>-identity
campusmeet-<env>-collaboration
campusmeet-<env>-meeting-data
campusmeet-<env>-task-data
campusmeet-<env>-ai-work
```

Separating these tables from application resources reduces the chance that updating a Lambda or frontend replaces long-lived data. The `meeting-data` table also exposes a DynamoDB Stream used by asynchronous processing such as Google synchronization.

## 5. Auth/core stack

`infra/auth-integration.yaml` contains the smaller learning/dev deployment for:

- Cognito User Pool and client;
- HTTP API and JWT authorizer;
- core API Lambda;
- its execution role and log group.

It is suitable for Authentication, Groups, Invitations, Notifications, and core Meeting CRUD exercises.

It should not be used as evidence that every full-application route is already deployed.

## 6. User-content and orchestration stack

`infra/user-content-orchestration.yaml` owns resources used outside short synchronous API calls, including private user-content storage, AI orchestration, reminders, Scheduler-related roles, and SES-related configuration.

A document upload can therefore follow this path:

```text
API permission check
  ↓
presigned upload URL
  ↓
private S3
  ↓
Attachment completion
  ↓
AIJob / Step Functions
```

## 7. Full application stack

`infra/template.yaml` brings together the wider deployed application, including frontend S3/CloudFront, application Cognito, the full HTTP API, GoogleSyncWorker, AI Worker, Bedrock/vector resources, and monitoring.

This is the stack used for the later E2E chapters when a feature must be exercised as part of the complete product flow.

## 8. External integrations

Google synchronization is performed after the internal Meeting change is persisted. A Google failure produces synchronization state and retry behavior rather than deleting the CampusMeet meeting.

AI follows an asynchronous job path:

```text
API
 ↓
AIJob
 ↓
Step Functions
 ↓
AI Worker
 ↓
Bedrock / Knowledge Base
```

The backend filters group, meeting, source state, and user permissions before retrieved content is passed to the model.

## 9. Features outside the current core E2E

The wider design includes recording and Amazon Transcribe, but the workshop does not present live transcription, full recording lifecycle, or batch audio transcription as production-complete unless the corresponding runtime path has actually been implemented and verified.

## 10. Deployment order

A full environment is normally assembled in this order:

```text
Confirm AWS account and region
        ↓
Data foundation
        ↓
User-content/orchestration
        ↓
Full application stack
        ↓
Read CloudFormation outputs
        ↓
Configure frontend and Google
        ↓
Publish frontend
        ↓
Run E2E tests
```

Some values such as the real CloudFront origin or API callback URL only exist after the first deployment, so the final environment may require a second stack update with the real URLs.

## Result

Learners should be able to explain the request path, the difference between authentication and authorization, the responsibility of each infrastructure template, and the difference between an implemented design and a cloud integration that has actually passed E2E verification.
