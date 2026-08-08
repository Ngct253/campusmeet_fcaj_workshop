---
title: "Proposal"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## CampusMeet — A serverless meeting-workflow platform

### 1. Executive summary

CampusMeet is a web application for study groups, capstone teams, and small projects to manage activities before, during, and after meetings in one place. It does not replace Google Meet or implement video calling. CampusMeet manages groups, schedules, meeting content, minutes, tasks, files, and access control, while integrating with Google Calendar/Meet when authorized.

The solution uses React/TypeScript for the web client, Amazon Cognito for authentication, Amazon API Gateway and AWS Lambda for APIs, Amazon DynamoDB for business data, Amazon S3 for files, EventBridge Scheduler and AWS Step Functions for asynchronous processing, and Amazon Bedrock for grounded AI features. AWS SAM and CloudFormation manage the infrastructure.

End-to-end goals:

- Register, confirm an email address, and sign in.
- Create groups, invite members, and enforce group-scoped roles.
- Schedule, update, and track meetings.
- Manage agendas, attachments, minutes, action items, and tasks.
- Synchronize Google Calendar/Meet with explicit state and retries.
- Manage transcripts and AI output through authorization, versioning, citations, and user confirmation.
- Provide testing, monitoring, cost controls, and cleanup procedures.

### 2. Problem and scope

Small teams commonly combine messaging, calendars, documents, and task boards. Meeting information becomes fragmented, action items are lost, file/transcript access is inconsistent, and Google or AI integrations can create duplicate or stale data.

CampusMeet's core scope includes:

- Cognito authentication and JWT-protected APIs.
- Profiles, groups, memberships, invitations, and notifications.
- Meetings, attendees, agendas, and meeting lifecycle management.
- Minutes, decisions, action items, tasks, and dashboards.
- A five-table DynamoDB data model.
- Direct S3 uploads through presigned URLs with object verification.
- Google OAuth, Calendar/Meet, reminders, and the Meet Add-on.
- Version-aware transcript reading, pagination, and editing.
- Citation-backed AI drafts with confirmation before official task creation.

Out of scope: a custom video/WebRTC service; recording without consent; AI mutations without confirmation; individual ranking from meeting data; or a production-readiness claim before smoke tests and live-environment verification are complete.

### 3. Status as of 8 August 2026

| Area | Verified status |
| --- | --- |
| Authentication and base API | Cognito, HTTP API, JWT Authorizer, and Lambda exist in source; the auth/API stack runs in AWS development |
| Data | Five DynamoDB tables are deployed and verified in `ap-southeast-1` |
| Groups and invitations | Frontend, API, repositories, and authorization checks are implemented |
| Core meeting workflow | The create, view, update, and cancel flow was deployed on 6 August 2026; service checks passed, while selected authorization scenarios still require suitable test data |
| Agendas, minutes, and tasks | UI, APIs, version controls, and related tests are present |
| Document upload | Presigned upload, metadata/checksum verification, and AIJob creation exist; audio completion still requires the Amazon Transcribe worker |
| Google Calendar/Meet | OAuth, side panel, and synchronization runtime are locally verified; full AWS and browser verification remains pending |
| Transcript | Read, pagination, and segment editing exist; approval/live transcription is not yet a complete cloud workflow |
| AI | Chat/draft/proposal and task confirmation exist as vertical slices; progress snapshots are locally verified but not fully deployed |
| Production readiness | Not achieved; smoke tests, browser tests, alarms, retention, security/cost review, and cleanup rehearsal remain |

This table distinguishes source implementation, local verification, and AWS deployment. A template or resource alone does not prove that an entire feature is complete.

### 4. Solution architecture

![CampusMeet AWS architecture](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

#### Synchronous request path

1. The browser loads the React client; the complete architecture distributes private S3 assets through CloudFront.
2. Cognito authenticates users and issues JWTs.
3. API Gateway validates JWTs before forwarding requests to Lambda.
4. Lambda derives identity from the token, checks membership/role, and executes the use case.
5. Repositories query DynamoDB through the required PK/SK/GSI pattern; normal requests do not use `Scan`.
6. Google Calendar/Meet and SES are called through stateful, idempotent adapters with retries.

#### Asynchronous file and AI path

1. The API validates authorization, type, size, and checksum before issuing an upload URL.
2. The client uploads directly to private S3 through a presigned URL.
3. The backend verifies the object through `HeadObject`.
4. Exactly one AIJob is created; Step Functions and workers handle long-running work.
5. Authorized sources are normalized and ingested into a Bedrock Knowledge Base/S3 Vectors.
6. Answers and drafts include citations and pass through preview/confirmation.

#### Five-table model

| Table | Primary data |
| --- | --- |
| `identity` | Profiles, preferences, integration state, and notifications |
| `collaboration` | Groups, memberships, invitations, and audit events |
| `meeting-data` | Meetings, agendas, attendees, reminders, attachments, minutes, and transcripts |
| `task-data` | Tasks, task history, and progress snapshots |
| `ai-work` | AIJobs, knowledge sources, conversations, citations, and proposals |

Binary files stay in S3, vectors stay in Bedrock Knowledge Bases/S3 Vectors, and DynamoDB stores metadata and orchestration state.

### 5. Delivery plan

| Phase | Outcome |
| --- | --- |
| 1. Foundation | Repository, shared contracts, CI, and environment configuration |
| 2. Identity | Cognito, JWT Authorizer, profiles, and protected routes |
| 3. Collaboration | Groups, memberships, invitations, notifications, and authorization |
| 4. Meetings | CRUD, attendees, agendas, lifecycle, and dashboards |
| 5. Post-meeting | Minutes, decisions, action items, and tasks |
| 6. Integrations | Google OAuth/Calendar/Meet, reminders, SES, and the Meet Add-on |
| 7. Files and transcripts | S3 presigned upload, AIJobs, and transcript version/edit/approval |
| 8. Grounded AI | Ingestion, RAG, citations, drafts, and proposal confirmation |
| 9. Hardening | Smoke tests, monitoring, security, cost, retention, and cleanup |

A feature is complete only when it has a shared contract, server-side authorization, UI loading/empty/error states, a happy-path test, and at least one negative or security test.

### 6. Security and quality

- JWT authentication and business authorization remain separate.
- The backend never trusts client-provided `userId`, `groupId`, roles, or approval metadata.
- Lambda uses least-privilege IAM execution roles.
- `.env` files, credentials, tokens, secrets, and user data are never committed.
- User-content S3 buckets remain private, with short-lived upload/download URLs.
- Concurrent updates use expected versions/conditional writes; required multi-item changes use transactions.
- Logs contain request IDs, resource IDs, state, and safe error codes—not sensitive meeting content.
- Quality gates include linting, type checking, tests, builds, formatting checks, and SAM validation.

### 7. Cost, risk, and expected outcome

Cost depends on API requests, Lambda/Step Functions duration, DynamoDB/S3 usage, CloudWatch Logs, SES, Transcribe, and Bedrock. Controls include AWS Budgets, resource tags, DynamoDB `PAY_PER_REQUEST`, bounded retention, change-set review, and cleanup rehearsals.

Primary risks are incomplete authorization, duplicate Google events, cross-group AI retrieval, use of unapproved transcripts, increased AI cost, and retained test resources. Mitigations are server-side authorization, idempotency, group/meeting/source-version filters, human confirmation, alarms, and documented cleanup.

The expected demonstrable flow is:

- Sign in → create a group → invite members.
- Schedule a meeting → manage the agenda → synchronize Google and reminders.
- Upload a document → verify S3 → process asynchronously.
- Transcript/minutes → action item → task → dashboard.
- Generate citation-backed AI answers or drafts → review and confirm.
- Manage infrastructure as code with logs, tests, cost alerts, and cleanup guidance.

The deliverable is not only an architecture diagram; it is a set of vertical slices with explicit evidence for source implementation, testing, and deployment status.
