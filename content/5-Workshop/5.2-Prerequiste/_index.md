---
title: "Environment Preparation and Deployment Architecture"
date: 2026-08-08
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Required environment

The CampusMeet repository uses Node.js 22 and npm. Git manages source history, while AWS CLI and AWS SAM CLI support account verification, template validation, and AWS deployment. PowerShell is used by selected project verification scripts.

The tools and source installation can be checked with:

```powershell
node --version
npm --version
git --version
aws --version
sam --version

git clone https://github.com/Ngct253/CampusMeet.git
cd CampusMeet
npm ci
```

Tool versions should remain compatible with the repository configuration. Successful dependency installation only confirms that the local environment is prepared; source, infrastructure, and AWS checks are still performed separately before handover.

## Repository structure

The repository is separated by responsibility:

| Area | Responsibility |
| --- | --- |
| `apps/web` | React interface, routes, and user-facing features |
| `services/api` | API, application logic, authorization, and repositories |
| `services/ai-worker` | Source normalization, AI jobs, and supporting service integrations |
| `packages/shared` | Shared types, enums, and contracts |
| `infra` | AWS SAM/CloudFormation templates |
| `scripts` | Configuration, infrastructure, and workflow verification |
| `docs` | Requirements, architecture, data model, and runbooks |

This structure allows data contracts and authorization rules to remain consistent across the frontend, API, and tests. AWS resources are managed through templates rather than being created manually and tied to one developer's machine. Before changes reach a shared environment, they still pass the checks described in the handover section.

## Connecting to the correct AWS environment

The CampusMeet development environment uses the `ap-southeast-1` Region. Before inspecting or updating a stack, the deployer should verify the account and Region:

```powershell
aws sts get-caller-identity
aws configure get region
```

Application, data, source storage, and knowledge retrieval resources remain in `ap-southeast-1`. The generation step is configured through the AI Worker to call Bedrock Mantle in `us-east-1`. This Region split requires review of authorization, permitted outbound context, latency, quotas, and cost; it does not move the primary business data Region.

After the auth/API stack is deployed, the frontend uses three public outputs:

```dotenv
VITE_COGNITO_USER_POOL_ID=...
VITE_COGNITO_USER_POOL_CLIENT_ID=...
VITE_API_BASE_URL=...
```

These values connect the frontend to the correct Cognito and API environment and are not secrets. Access keys, Google client secrets, OAuth tokens, and other server-side information must not be placed in `VITE_*` variables, source code, or Git.

## Deployment architecture

![CampusMeet high-level architecture diagram](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

The diagram moves from the user interface, identity, API, business processing, and storage to supporting services and operational visibility. The core meeting flow uses Cognito, API Gateway, Lambda, and DynamoDB; S3, calendar, transcription, and AI services are connected according to each capability's needs.

The target architecture does not mean that every branch is complete at the same level. The workshop distinguishes implemented or verified areas from work that still requires testing in the shared environment.

## Service responsibilities

| Component | Responsibility in CampusMeet |
| --- | --- |
| React, TypeScript, and Vite | Build the web interface and manage the user journey |
| Amazon Cognito | Registration, account confirmation, sign-in, and JWT issuance |
| API Gateway HTTP API | Validate the JWT before forwarding requests to the backend |
| AWS Lambda | Execute use cases, enforce access, and coordinate data access |
| Amazon DynamoDB | Store business information across five physical tables |
| Amazon S3 | Store private user content and serve web assets in the target architecture |
| EventBridge Scheduler | Invoke reminders at the appropriate time |
| Step Functions and AI Worker | Coordinate AI jobs, bounded retries, source normalization, and processing state |
| Bedrock Knowledge Base, Cohere embedding, and S3 Vectors | Retrieve approved sources in `ap-southeast-1` |
| Bedrock Mantle | Generate answers or drafts in `us-east-1` with `openai.gpt-oss-20b` |
| Amazon Transcribe | Support audio processing and transcription when the relevant flow is enabled |
| CloudWatch, SNS, and SES | Observe operations, deliver alerts, and support notifications |

The table only identifies the architectural responsibility of each service. Section 5.4 presents the Cognito–API–Lambda flow and the authorization checks performed before data is stored, avoiding repetition of the same process in several sections.

## Synchronous and asynchronous flows

In the synchronous path, the frontend calls the HTTP API with a JWT; API Gateway validates the token, and Lambda checks membership and role before querying DynamoDB. The response returns to the interface within the same request. The browser neither declares its own access rights nor connects directly to the data tables.

Work that depends on external services or takes longer is separated from the primary request. A DynamoDB Stream can invoke the Google synchronization worker, EventBridge Scheduler triggers reminders, and uploads use short-lived URLs so the browser sends files directly to private S3 storage.

The AI path follows this sequence:

```text
Reviewed and approved source
  → normalize under the kb/ storage area in Singapore
  → Bedrock Knowledge Base embeds and retrieves through S3 Vectors
  → backend filters by group, meeting, source, and version
  → AI Worker calls Bedrock Mantle in N. Virginia
  → return a citation-grounded answer or draft
  → user reviews and confirms before an official write
```

Step Functions manages AI job state and retries. The existence of a state machine, Knowledge Base, or model endpoint proves infrastructure configuration, not the quality of retrieval, citations, or end-to-end authorization.

## Environment setup sequence

1. Confirm the AWS account, Region, and deployment permissions.
2. Validate the data template, review the change, deploy it, and verify the five DynamoDB tables.
3. Deploy or update the authentication/API stack with the correct table prefix and allowed frontend origin.
4. Use the CloudFormation outputs for the User Pool ID, User Pool Client ID, and API URL to configure the interface.
5. Set up content and orchestration resources when the environment enables upload, reminders, transcription, or AI.
6. Check `/health`, sign-in, protected routes, logs, and the features enabled in that environment.

This sequence establishes dependencies in a clear order. It is a deployment and verification method, not a claim that the complete target architecture is production-ready.
