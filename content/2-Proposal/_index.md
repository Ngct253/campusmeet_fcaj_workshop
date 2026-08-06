---
title: "Proposal"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CampusMeet

## A Serverless Meeting Workflow Platform for Study Groups and Small Project Teams

### 1. Executive Summary

CampusMeet is a web platform designed to support the activities that take place before, during, and after a meeting for study groups, student projects, and small collaborative teams.

In many teams, meeting-related information is scattered across chat applications, personal calendars, shared documents, and task boards. This makes it difficult to manage membership, enforce access rules, track scheduled meetings, and preserve outcomes in a consistent way.

CampusMeet is proposed as a centralized application built on a serverless AWS architecture. Its core scope includes:

- User registration, account confirmation, and sign-in.
- Group, membership, and invitation management.
- Creating, viewing, updating, and cancelling meetings.
- In-app notifications.
- Group-scoped, role-based authorization.
- A five-table Amazon DynamoDB data model.
- Logging and operational visibility through Amazon CloudWatch.

The core architecture uses Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon DynamoDB, AWS Identity and Access Management, and Amazon CloudWatch. Infrastructure is defined with AWS SAM and AWS CloudFormation so that validation, deployment, and resource management can follow a repeatable process.

The immediate goal is to deliver a testable end-to-end serverless workflow for authentication, authorization, and meeting data management. The same foundation can later support tasks, meeting minutes, transcripts, uploaded content, and AI-assisted features.

### 2. Problem Statement

#### Current Challenges

Small teams commonly rely on several disconnected tools to organize meetings:

- Messaging applications for discussions and invitations.
- Personal calendars for scheduling.
- Separate documents for notes and minutes.
- Task boards for post-meeting follow-up.

This fragmented workflow introduces several problems:

- Meeting information is difficult to find and maintain.
- Access control is inconsistent across tools.
- It is unclear who is allowed to create, edit, or cancel a meeting.
- Invitations and notifications can easily be missed.
- Group, meeting, and follow-up data are not managed in one place.
- Adding transcripts or AI features becomes difficult without a consistent authorization model.

#### Proposed Solution

CampusMeet provides a centralized web application in which users are authenticated through Amazon Cognito before they can access protected features.

The frontend sends a JWT to Amazon API Gateway. API Gateway validates the token before forwarding the request to AWS Lambda. The backend then performs resource-level authorization by checking whether the user is an active member of the relevant group and whether their role permits the requested action.

Application data is organized across five DynamoDB tables:

- `identity`: users, preferences, and notifications.
- `collaboration`: groups, memberships, invitations, and audit events.
- `meeting-data`: meetings and related records.
- `task-data`: tasks and task history.
- `ai-work`: AI jobs and future orchestration data.

Using five physical tables keeps infrastructure manageable while preserving clear boundaries between major business domains.

#### Project Scope

The core workshop scope covers:

- User authentication.
- Group and membership management.
- Invitations and notifications.
- Meeting management.
- Backend authorization.
- DynamoDB data design.
- Serverless infrastructure.
- Testing, logging, and monitoring.
- Deployment and resource cleanup procedures.

CampusMeet does not build its own video-conferencing service and is not intended to replace Google Meet. Google Calendar and Google Meet integration are treated as future extensions.

Live transcription, document processing, and AI assistance are also planned extensions rather than prerequisites for completing the core workshop scope.

#### Expected Benefits

- Centralized group and meeting data.
- Less manual coordination and information consolidation.
- Consistent access control across the application.
- Better visibility into invitations, memberships, and meeting status.
- On-demand scalability through serverless services.
- A reusable foundation for tasks, minutes, transcripts, and AI features.
- Low initial operating overhead through usage-based pricing.

### 3. Solution Architecture

CampusMeet follows a serverless web architecture with separate layers for the user interface, authentication, API access, business logic, data storage, and monitoring.

#### Main Request Flow

1. A user accesses the CampusMeet web application.
2. Amazon Cognito handles registration, account confirmation, and sign-in.
3. The frontend receives a JWT and includes it in API requests.
4. Amazon API Gateway validates the token before invoking AWS Lambda.
5. Lambda processes the request and performs resource-level authorization.
6. The backend reads from or writes to Amazon DynamoDB.
7. Amazon CloudWatch captures logs, failures, and operational signals.
8. AWS SAM and CloudFormation define and manage the infrastructure.

#### AWS Services

- **Amazon Cognito**: User registration, sign-in, account recovery, and JWT issuance.
- **Amazon API Gateway**: HTTP API entry point and JWT validation.
- **AWS Lambda**: Business logic for groups, invitations, notifications, and meetings.
- **Amazon DynamoDB**: Application persistence using a five-table model.
- **AWS IAM**: Least-privilege access between Lambda and AWS resources.
- **Amazon CloudWatch**: Logs, diagnostics, and operational monitoring.
- **AWS CloudFormation**: Infrastructure as Code for AWS resources.
- **AWS SAM**: Serverless build, validation, and deployment workflows.
- **Amazon S3 and Amazon CloudFront**: Potential hosting and delivery layer for the production frontend.

#### Component Design

##### Web Application

The frontend is built with React, TypeScript, and Vite. Its main screens include:

- Registration, account confirmation, and sign-in.
- Password recovery.
- Group lists and membership management.
- Invitation workflows.
- Meeting lists, details, and forms.
- In-app notifications.

##### Authentication and Authorization

Amazon Cognito verifies user identity and issues JWTs. The frontend uses these tokens when calling protected APIs.

A valid JWT proves identity, but it does not grant unrestricted access to application data. The backend must still verify active group membership and the role required for each operation.

##### API and Business Logic

Amazon API Gateway provides the backend entry point. AWS Lambda implements application use cases and is responsible for:

- Input validation.
- Membership and role checks.
- Reading and writing data through repository interfaces.
- Consistent error responses.
- Diagnostic logging without exposing sensitive information.

##### Data Storage

| Table | Primary Data |
| --- | --- |
| `identity` | Users, preferences, and notifications |
| `collaboration` | Groups, memberships, invitations, and audit events |
| `meeting-data` | Meetings and related records |
| `task-data` | Tasks and task history |
| `ai-work` | Processing jobs, knowledge sources, and future AI data |

Composite keys, Global Secondary Indexes, and TTL are introduced only where required by defined access patterns.

##### Monitoring

Amazon CloudWatch is used to:

- Store Lambda logs.
- Track request-processing failures.
- Support authentication and authorization troubleshooting.
- Observe latency and service health.
- Assist with deployment and integration-test diagnostics.

### 4. Technical Implementation

#### Delivery Phases

##### Phase 1: AWS Fundamentals

- Become familiar with the AWS Management Console.
- Study IAM, VPC, EC2, Lambda, and RDS.
- Configure AWS Budgets.
- Practice resource cleanup.
- Study serverless architecture fundamentals.

##### Phase 2: Requirements and Architecture

- Define the problem CampusMeet will address.
- Establish the initial feature scope.
- Design the frontend, API, and database interaction flow.
- Select Amazon Cognito for authentication.
- Select API Gateway and Lambda for the backend.
- Select DynamoDB as the primary database.

##### Phase 3: Authentication Foundation

- Build registration and sign-in interfaces.
- Add email-based account confirmation.
- Implement password recovery.
- Protect authenticated routes.
- Add an API client that sends access tokens.
- Configure an API Gateway JWT Authorizer.
- Define the authentication stack with AWS SAM.

##### Phase 4: Data Foundation

- Identify the application's major data domains.
- Design partition and sort keys.
- Define required Global Secondary Indexes.
- Configure TTL for time-bound records.
- Define the five DynamoDB tables in CloudFormation.
- Apply least-privilege IAM permissions.
- Add scripts that validate the deployed table configuration.

##### Phase 5: Core Features

- Group and membership management.
- Invitation workflows.
- In-app notifications.
- Create, view, update, and cancel meeting operations.
- Group-scoped authorization checks.
- Frontend-to-API integration.

##### Phase 6: Testing and Documentation

- Run linting and type checks.
- Run unit tests.
- Verify the frontend build.
- Validate AWS SAM templates.
- Verify the DynamoDB configuration.
- Test the authentication flow end to end.
- Document deployment and cleanup procedures.
- Prepare the workshop documentation.

#### Technical Requirements

- Node.js 22 LTS and npm 10 or later.
- React, TypeScript, and Vite.
- AWS CLI and AWS SAM CLI.
- An AWS account with the permissions required for the selected resources.
- Git and GitHub for source control.

#### Quality Gates

The primary validation commands include:

```bash
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
npm run sam:validate:data
```

AWS SAM templates should be validated before a CloudFormation stack is created or updated. Infrastructure changes should be reviewed through a change set before deployment to reduce the risk of unintended replacement or deletion.

#### Security Principles

- Do not use the root account for routine work.
- Enable MFA for administrative access.
- Never commit access keys, secret keys, tokens, or passwords.
- Use IAM execution roles for Lambda.
- Grant only the actions and resources required by each component.
- Enforce membership and role checks for every protected operation.
- Do not write JWTs, OAuth tokens, or sensitive meeting content to logs.
- Do not place real user data in source code or public documentation.

### 5. Roadmap and Milestones

| Stage | Main Deliverables |
| --- | --- |
| AWS fundamentals | AWS Console, Budgets, IAM, VPC, EC2, and serverless concepts |
| Project definition | CampusMeet problem statement, scope, and architecture |
| Authentication | Landing page, Cognito flows, protected routes, and JWT authorization |
| Authentication infrastructure | AWS SAM, API Gateway, Lambda, IAM, and CloudWatch |
| Data foundation | Five-table DynamoDB design and infrastructure |
| Core application | Groups, memberships, invitations, notifications, and meetings |
| Verification | Linting, type checks, tests, builds, and infrastructure validation |
| Workshop completion | Deployment, security, monitoring, and cleanup documentation |

After the core scope is complete, the platform may be extended with:

- Google Calendar synchronization.
- Google Meet link creation.
- Meeting minutes and action-item management.
- File and recording uploads.
- Live transcription.
- Amazon Bedrock-powered retrieval.
- An AI assistant that provides citations.

### 6. Budget Estimation

CampusMeet uses a serverless architecture and prioritizes services with usage-based pricing. Because the application is still under development, the current budget is preliminary and will be refined once expected user count, API traffic, storage volume, log retention, and deployment scope are known.

The [AWS Pricing Calculator](https://calculator.aws/) can be used to prepare a detailed estimate when those operating assumptions have been finalized.

#### Services Included in the Estimate

| Service | Purpose | Main Cost Drivers |
| --- | --- | --- |
| Amazon Cognito | Registration, sign-in, and identity management | Monthly active users |
| Amazon API Gateway | HTTP API for the frontend | Number of API requests |
| AWS Lambda | Business-logic execution | Invocations, duration, and memory |
| Amazon DynamoDB | Application data storage | Reads, writes, storage, and indexes |
| Amazon CloudWatch | Logging and monitoring | Log ingestion and retention |
| Amazon S3 | Frontend assets or uploaded files | Storage and request volume |
| Amazon CloudFront | Frontend and static-content delivery | Data transfer |
| Amazon SES | Optional email notifications | Number of messages sent |

Amazon Transcribe, Amazon Bedrock, AWS Step Functions, and Bedrock Knowledge Bases will only be added to the estimate if transcript and AI features are implemented.

#### Development-Environment Assumptions

- A small number of test users.
- Low API traffic.
- DynamoDB in `PAY_PER_REQUEST` mode.
- Lambda invoked only when requests are received.
- Appropriate CloudWatch log-retention settings.
- Test files removed when no longer needed.
- One shared development environment.
- No continuously running AI or transcription workloads.

For a student development and testing environment, several services may remain within AWS Free Tier allowances or incur only limited charges. Free Tier eligibility, however, should not be treated as a guarantee that the environment will always be free of cost.

#### Cost-Control Measures

- Configure AWS Budgets and billing alerts.
- Verify the AWS account and Region before deployment.
- Tag CampusMeet resources consistently.
- Set appropriate CloudWatch log-retention periods.
- Use key-based DynamoDB queries instead of `Scan` for normal application requests.
- Review CloudFormation change sets before deployment.
- Remove unused test stacks and resources.
- Avoid retaining test recordings or uploaded files longer than necessary.
- Keep the data stack separate from the application stack to reduce accidental data deletion risk.

A formal cost estimate will be added after the deployment architecture, expected usage, and optional CampusMeet modules have been finalized.

### 7. Risk Assessment

#### Risk Matrix

| Risk | Impact | Likelihood |
| --- | --- | --- |
| IAM permissions are broader than required | High | Medium |
| Membership or role checks are incomplete | High | Medium |
| DynamoDB design does not match access patterns | High | Medium |
| A CloudFormation update affects existing data | High | Low |
| Frontend and backend API contracts diverge | Medium | Medium |
| Logs or meeting data expose sensitive information | High | Medium |
| Logs or unused resources create unexpected cost | Medium | Medium |
| Google Workspace, transcript, or AI integrations are unstable | Medium | Medium |
| Project scope expands faster than implementation progress | Medium | High |

#### Mitigation Strategy

**Authentication and authorization**

- Use Amazon Cognito and an API Gateway JWT Authorizer for identity verification.
- Perform membership and role checks in the backend for every protected operation.
- Never treat a valid JWT as unrestricted access to application data.
- Apply least-privilege IAM policies.

**DynamoDB and data integrity**

- Define access patterns before modifying tables or indexes.
- Access DynamoDB through repository interfaces rather than directly from handlers.
- Use conditional writes or transactions when consistency requirements justify them.
- Verify table schema, TTL, and indexes before and after deployment.

**Infrastructure deployment**

- Validate AWS SAM and CloudFormation templates before deployment.
- Review change sets before updating a stack.
- Keep the data stack separate from the application stack.
- Avoid unmanaged resource changes through the AWS Console.
- Maintain deployment, verification, rollback, and cleanup runbooks.

**Data protection**

- Do not log JWTs, OAuth tokens, secrets, or sensitive meeting content.
- Do not commit `.env` files, credentials, or user data.
- Apply consent and retention rules to recordings, transcripts, and uploads.
- Keep S3 buckets private and issue only time-limited access when required.

**Quality and scope control**

- Run linting, type checks, unit tests, and builds before integration.
- Maintain shared types and a clear API contract between frontend and backend.
- Prioritize authentication, groups, memberships, and meeting management.
- Treat Google Workspace, transcription, and AI as independent extension modules.
- Ensure core meeting functionality does not depend entirely on external services.

#### Contingency Plan

- Roll back the application stack to a verified version after a failed deployment.
- Keep the data stack independent to reduce the risk of data loss.
- Restore source code from a tested commit.
- Use in-memory repositories or DynamoDB Local during development.
- Temporarily disable Google, transcription, or AI integrations if an external service fails.
- Keep core meeting management available when optional modules are unavailable.
- Recreate test stacks from the documented runbook when an environment is misconfigured.

### 8. Expected Outcomes

By the end of the workshop scope, CampusMeet is expected to provide:

- A serverless application with a clear frontend-to-backend request flow.
- User registration, account confirmation, and sign-in through Amazon Cognito.
- API protection through JWT authorization.
- Resource access checks based on group membership and role.
- Group, membership, invitation, and notification workflows.
- Create, view, update, and cancel meeting operations.
- A five-table DynamoDB data model.
- Infrastructure managed through AWS SAM and CloudFormation.
- Repeatable testing, deployment, and cleanup procedures.
- CloudWatch-based logging and troubleshooting.
- Workshop documentation that explains the architecture and implementation process end to end.

Longer term, the same authentication, data, and authorization foundation can support tasks, meeting minutes, document uploads, live transcription, and an AI assistant that returns source-backed answers.
