---
title: "Objectives, Preparation, and Access"
date: 2026-08-08
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## CampusMeet objectives

- Reduce fragmented information.
- Give meetings a clear purpose and preparation process.
- Preserve outcomes instead of ending with a conversation only.
- Convert follow-up actions into trackable tasks.
- Help users retrieve information while respecting access boundaries.

## Technical environment preparation

The current repository uses Node.js 22 and npm. AWS CLI and AWS SAM CLI are required only when validating or deploying AWS resources, while PowerShell supports selected verification scripts. The following commands check the tools and install the source:

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

Before connecting changes to a shared environment, the repository should pass the source, type, test, build, and infrastructure checks described in the handover section. These checks do not replace AWS verification, but they remove common failures before deployment.

## Connecting the AWS environment

The CampusMeet development environment uses the `ap-southeast-1` Region. Before updating a stack, the deployer should verify the AWS account and Region to avoid creating resources in the wrong environment:

```powershell
aws sts get-caller-identity
aws configure get region
```

After the auth/API stack is deployed, the frontend uses three public outputs:

```dotenv
VITE_COGNITO_USER_POOL_ID=...
VITE_COGNITO_USER_POOL_CLIENT_ID=...
VITE_API_BASE_URL=...
```

These values connect the frontend to the correct Cognito and API environment and are not secrets. Access keys, Google client secrets, OAuth tokens, and other server-side information must not be placed in `VITE_*` variables, source code, or Git.

## Access

CampusMeet organizes access around groups. Having an account does not allow a person to view everything. Administrators coordinate their group and members, while members use capabilities appropriate to their role. People outside a group cannot view its meetings, documents, or internal work.

Infrastructure access also separates deployer permissions from service execution roles. A deployer needs only the permissions required to update the stacks in scope. Lambda functions and workers use dedicated roles for the exact tables, buckets, and services they need. Using several AWS services does not justify giving every team member or every Lambda function `AdministratorAccess`.

Access is determined from stored membership information rather than a role claimed by the user. AI suggestions also remain drafts until an authorized user reviews them.

## Access-decision approach

Each action is considered through four questions:

1. Is the account signed in and valid?
2. Is the person an active member of the group?
3. Does the current role permit the specific action?
4. Does the action target the correct group, meeting, and version?

This approach separates identity from resource access: signing in does not permit reading another group's data, and viewing a meeting does not automatically permit editing or approval. Members receive access only through valid membership, while organizers or administrators perform actions that need additional authority. Documents and transcripts used by AI retain their group, meeting, and source context so answers cannot use information outside the requester's access.

## Protecting content

Access is checked for each group and meeting action, not only at sign-in. Documents, transcripts, minutes, tasks, and AI sources inherit the access boundary of their original content.

## Usage criterion

CampusMeet provides value when users can complete the core journey: sign in, join a group, create or view a meeting, access documents, record outcomes, and track assigned work. Advanced features should support this journey without making it harder to understand.
