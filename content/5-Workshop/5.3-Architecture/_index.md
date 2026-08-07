---
title: "System Architecture"
date: 2026-07-27
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# CampusMeet System Architecture

## Objectives

This section explains how the CampusMeet components work together and where responsibilities are separated across the web application, APIs, data stores, and external integrations. Understanding these boundaries is necessary before deploying resources, assigning IAM permissions, or implementing additional features.

## Design Principles

CampusMeet follows these principles:

- CampusMeet manages the workflow before, during, and after a meeting. It does not implement video calling or replicate Google Meet.
- The web application never accesses Amazon DynamoDB directly.
- Amazon Cognito authenticates users, while the backend still enforces membership and role authorization.
- Application data is stored in five DynamoDB tables organized around access requirements.
- Files, audio, and other large objects are stored in Amazon S3; DynamoDB stores metadata and application state.
- Long-running operations such as file processing, transcription, and AI generation are handled asynchronously.
- AI-generated results remain source-grounded drafts until an authorized user reviews and confirms them.

## Main Request Flow

![CampusMeet AWS Architecture Diagram](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

```text
User
  |
  v
CampusMeet Web
React + TypeScript + Vite
  |
  | 1. Sign up and sign in
  v
Amazon Cognito
  |
  | 2. Issue JWT
  v
Amazon API Gateway
  |
  | 3. Validate JWT
  v
AWS Lambda
  |
  | 4. Enforce membership and role
  v
Application and repository layers
  |
  +----------------------+----------------------+
  |                      |                      |
  v                      v                      v
DynamoDB               Amazon S3          External services
Application data       Files and audio    Google, SES, and AI
```

A typical read or write request follows these steps:

1. The user signs in through Amazon Cognito.
2. The web application sends the JWT in the `Authorization` header.
3. Amazon API Gateway validates the token signature, issuer, audience, and expiration.
4. AWS Lambda obtains the identity from the JWT instead of trusting a client-provided `userId` or role.
5. The backend verifies active membership and the required role for the requested group or meeting.
6. The repository performs `GetItem`, `Query`, conditional writes, or transactions against the appropriate table.
7. The response uses shared contracts from `@campusmeet/shared`.

## Component Responsibilities

| Component | Responsibility in CampusMeet |
| --- | --- |
| CampusMeet Web | User interface for profiles, groups, invitations, meetings, notifications, and later workflow features |
| Amazon Cognito | User registration, account confirmation, sign-in, and JWT issuance |
| Amazon API Gateway | HTTP API exposure, CORS handling, and JWT validation before Lambda invocation |
| AWS Lambda | Use-case execution, authorization, and repository coordination |
| Amazon DynamoDB | Application data stored in five physical tables |
| Amazon S3 | Private storage for files, audio, recordings, and large content |
| EventBridge Scheduler | One-time reminder schedules associated with meetings |
| AWS Step Functions | Coordination of long-running or retryable processing |
| Amazon Transcribe | Speech-to-text processing when transcription workflows run |
| Amazon Bedrock | Source-grounded questions, summaries, and content proposals |
| Amazon CloudWatch | Logs, metrics, and failure information |
| AWS IAM | Access control for users and AWS resources |
| AWS SAM and CloudFormation | Infrastructure definition, validation, and deployment |

## CloudFormation Stack Boundaries

CampusMeet separates data resources from application resources to reduce the risk of affecting persistent data during application updates.

| Template | Responsibility |
| --- | --- |
| `infra/data-foundation.yaml` | Owns the five shared DynamoDB tables |
| `infra/auth-integration.yaml` | Owns the Cognito User Pool, app client, HTTP API, Lambda function, IAM role, and log group for authentication and the current core APIs |
| `infra/template.yaml` | Describes the extended application architecture and references the five tables through a stable prefix; it does not recreate the data tables |

The platform deployment order is:

```text
Verify AWS account and IAM access
        ↓
Deploy the data stack
        ↓
Verify the five DynamoDB tables
        ↓
Deploy the Cognito and API stack
        ↓
Read CloudFormation outputs
        ↓
Configure and test the web application
```

## Authentication and Authorization

Two independent checks are required:

| Layer | Performed by | Check |
| --- | --- | --- |
| Authentication | Amazon Cognito and API Gateway | Valid JWT, correct User Pool and app client, valid signature, and unexpired token |
| Authorization | Lambda and application services | Active membership, sufficient role, and access to the requested group, meeting, or resource |

A valid JWT does not grant access to every group. Before returning group details, the backend still reads the corresponding membership. Administrative operations such as updating a group, creating invitations, or cancelling a meeting require the `GROUP_ADMIN` role.

## Data Boundaries

CampusMeet uses five DynamoDB tables:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

The tables use `PK` and `SK` keys with secondary indexes for known access requirements. Normal application requests must not rely on `Scan`.

Binary objects do not pass through Lambda for storage in DynamoDB. The large-file flow is:

1. The client requests upload permission.
2. The backend verifies membership, file type, size, and related metadata.
3. The backend returns a short-lived upload URL.
4. The browser uploads directly to a private S3 bucket.
5. The backend verifies the object before persisting metadata and starting later processing.

## External Integration Boundaries

- Google Calendar is the primary integration for creating or updating calendar events and requesting Google Meet links.
- The Google Meet REST API is used only when recordings or transcripts actually exist and the connected account has the required authorization.
- In-application notifications are the primary record; an email failure must not remove a notification that was already created.
- Calls to Google, Amazon Transcribe, Amazon Bedrock, and other services require explicit state, idempotency, and controlled retries.
- AI retrieval must filter by group, meeting scope, approval status, and user authorization before sending content to a model.

## Security and Operations

- Lambda uses an IAM execution role and does not read long-lived AWS access keys from environment variables.
- Logs must not contain JWTs, OAuth tokens, passwords, signed URLs, or complete sensitive content.
- S3 buckets used for the web application and user content remain private.
- Competing updates use conditional writes or optimistic versions.
- Mutations that must update several items atomically use DynamoDB transactions.
- CloudWatch is used to inspect API, Lambda, and asynchronous workflow failures.
- AWS Budgets provides cost alerts for the shared environment.

## Files to Review

| File | Purpose |
| --- | --- |
| `docs/architecture.md` | Overall architecture and component boundaries |
| `docs/CampusMeet-SRS.md` | Business requirements and product scope |
| `docs/dynamodb-data-model.md` | Physical five-table DynamoDB model |
| `docs/api-contract.md` | API paths, shared contracts, and implementation status |
| `infra/data-foundation.yaml` | Data stack |
| `infra/auth-integration.yaml` | Cognito, HTTP API, and core Lambda stack |

## Expected Result

After completing this section, you should be able to:

- Explain the path from the web application to API Gateway, Lambda, and DynamoDB.
- Separate JWT authentication from group-scoped authorization.
- Identify resources owned by the data stack and application stacks.
- Explain why large files belong in S3 rather than DynamoDB.
- Identify external integrations that require explicit failure and retry handling.
