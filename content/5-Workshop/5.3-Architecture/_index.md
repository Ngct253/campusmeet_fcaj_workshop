---
title: "High-level Architecture"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## CampusMeet architecture

![CampusMeet high-level architecture diagram](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

The diagram moves from user experience, identity, business processing, and storage to supporting services and operational visibility. Each layer has a distinct responsibility while contributing to the meeting and follow-up journey.

## Services and responsibilities

| Component | Responsibility in CampusMeet |
| --- | --- |
| React, TypeScript, and Vite | Build the web interface and manage the user journey |
| Amazon Cognito | Registration, account confirmation, sign-in, and JWT issuance |
| API Gateway HTTP API | Validate tokens before forwarding requests to the backend |
| AWS Lambda | Execute use cases, enforce access, and coordinate repositories or integrations |
| Amazon DynamoDB | Store business information across five physical tables |
| Amazon S3 | Store the target frontend assets and private user content |
| EventBridge Scheduler | Invoke one-time reminders at the appropriate time |
| Step Functions and AI Worker | Coordinate longer jobs, retries, and processing states |
| Amazon Transcribe and Bedrock | Support transcription, source processing, retrieval, and grounded generation |
| CloudWatch, SNS, and SES | Observe operations, deliver alerts, and attempt email notifications |

## A simple information flow

When a member opens a meeting, CampusMeet first confirms the account and access to the group. It then retrieves the appropriate information and displays it. When a user updates minutes or a task, the change is stored so other authorized members can follow it.

Documents are uploaded to private file storage rather than placed directly inside meeting records. CampusMeet links each file to the correct group and meeting.

A business request follows this simplified path:

```text
React
  → Cognito provides a JWT
  → API Gateway validates the token
  → Lambda resolves the user and checks membership/role
  → The application service applies the rule
  → A repository reads or writes DynamoDB/S3
  → The API returns the appropriate result to the interface
```

API Gateway confirms that the token is valid, but it does not decide which group the person may read. The backend still authorizes `groupId`, `meetingId`, and related resources for each action.

## Extended content processing

Reviewed or approved documents and transcripts can become knowledge sources while retaining their group, meeting, and source-version context. AI answers and drafts remain limited by user access, include citations, and require confirmation before becoming official minutes or tasks.

## Infrastructure boundaries

CampusMeet separates infrastructure templates by responsibility:

- `infra/data-foundation.yaml` manages exactly five DynamoDB tables and stays outside the application lifecycle to reduce data risk.
- `infra/auth-integration.yaml` is the Cognito, HTTP API, and core integration stack used in the development environment; it does not represent the entire target architecture.
- `infra/user-content-orchestration.yaml` owns the user-content bucket, orchestration, reminders, Scheduler role, and related email configuration.
- `infra/template.yaml` is the application stack and receives table names and required outputs through parameters instead of recreating resources owned elsewhere.

Separating the stacks makes changes easier to review and prevents an interface or API update from unintentionally changing the data foundation. Table names, bucket references, and endpoints are passed through environment configuration; credentials are not hard-coded in templates or source.

## AWS environment setup sequence

1. Confirm the account, Region, and deployment permissions.
2. Validate the data template, review the planned change, deploy it, and verify the five tables.
3. Set up user-content/orchestration when the environment enables upload, reminders, transcripts, or AI.
4. Deploy or update the auth/API/application stack with the correct table names and related outputs.
5. Configure the frontend with the User Pool ID, User Pool Client ID, and API URL.
6. Check `/health`, sign-in, protected routes, logs, and the workflows enabled in that environment.

This is a deployment and verification order, not a claim that the complete target architecture is production-ready. The auth-integration stack supports the current development/core scope, while the full application stack still requires evidence from outputs, smoke tests, and actual logs.

## Why the architecture is organized this way

The architecture separates responsibilities so that supporting capabilities do not become mixed into the core meeting workflow. The interface focuses on the user experience; identity and central processing enforce access rules; business information and files are stored according to their different characteristics; and external integrations connect through clear boundaries. A change to calendar synchronization, transcription, or AI therefore does not change the meaning of groups, meetings, minutes, and tasks.

Large files and audio remain in private file storage, while CampusMeet manages only the references and context needed to connect them to the correct meeting. Longer work such as audio processing, document normalization, or content generation is tracked through visible states instead of forcing a user to wait on one screen. This approach requires careful state and failure handling, but it keeps the main journey responsive and allows a supporting operation to be retried after a temporary failure.

## Main architecture principles

- Sign-in verifies identity, while authorization is still checked for each group and meeting resource.
- Google Meet remains an external meeting service; CampusMeet manages the surrounding workflow and outcomes rather than building video conferencing.
- Meeting information, files, and AI-assisted content retain links to the group and original source for traceability.
- Only documents or transcripts approved through the appropriate flow become official knowledge sources.
- AI output remains a cited draft; an authorized user confirms changes to minutes or tasks.
- Errors, processing states, and cost require monitoring so advanced capabilities do not hide operational problems.
