---
title: "Prerequisites"
date: 2026-07-27
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Prerequisites

Before deploying CampusMeet, prepare the local development environment, AWS account, and access model used by the project. The requirements in this section follow the current CampusMeet repository and deployment runbook.

## 1. AWS Account and Environment

Prepare the following:

- An AWS account used for the CampusMeet development environment.
- The AWS account ID agreed by the team.
- AWS Region `ap-southeast-1`.
- One deployment owner who can create the IAM roles required by the infrastructure templates.
- A separate IAM user for each team member.
- An AWS Budget alert configured before resources are deployed.

Do not use the root account for daily work. Do not share IAM users, and do not create or distribute long-lived access keys for human users.

The shared AWS development environment should be used for integration testing and deployment verification. Daily feature development should be performed locally with an in-memory repository or DynamoDB Local when appropriate.

## 2. Required Tools

| Tool | Requirement | Purpose |
| --- | --- | --- |
| Node.js | Version 22 LTS | Runs the frontend, backend, and JavaScript/TypeScript tooling |
| npm | Version 10 or later | Installs dependencies and runs project scripts |
| Git | A supported version | Retrieves and manages the source code |
| AWS CLI | Version 2.32.0 or later | Signs in, verifies identity, and interacts with AWS |
| AWS SAM CLI | Installed | Validates, builds, and deploys SAM templates |
| PowerShell | Installed | Runs the current AWS commands and verification scripts |

Verify the tools after installation:

```powershell
node --version
npm --version
git --version
aws --version
sam --version
```

Do not continue until Node.js, npm, AWS CLI, and AWS SAM CLI are available from the command line.

## 3. Retrieve the CampusMeet Source Code

Clone the repository and enter the project directory:

```powershell
git clone https://github.com/Ngct253/CampusMeet.git
cd CampusMeet
```

Install dependencies for all workspaces:

```powershell
npm install
```

CampusMeet is organized as a monorepo with the following main directories:

```text
apps/web/          React and Vite frontend
services/api/      API Lambda and application layers
packages/shared/   Shared types, enums, and DTOs
infra/             AWS SAM and CloudFormation templates
scripts/           AWS resource verification scripts
docs/              Requirements, architecture, and deployment documents
```

## 4. Validate the Source Code Before Deployment

Run the baseline checks from the CampusMeet repository root:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
```

These commands check coding rules, TypeScript types, automated tests, project builds, and file formatting.

Validate the data foundation template:

```powershell
npm run sam:validate:data -- --region ap-southeast-1
```

A successful template validation confirms that the template structure is valid. It does not prove that the resources have been deployed to AWS.

## 5. Run the Frontend Locally

Start the web application:

```powershell
npm run dev
```

The current Vite configuration serves the application at:

```text
http://localhost:5173
```

At this stage, the interface can start locally, but features that depend on Cognito and the API will work only after the AWS stacks are deployed and `apps/web/.env` contains the correct CloudFormation outputs.

Do not invent placeholder values for the User Pool, User Pool Client, or API URL. Do not commit `.env` files, tokens, passwords, or AWS credentials.

## 6. Sign In with the AWS CLI

Each member signs in with their assigned IAM user:

```powershell
aws login
aws sts get-caller-identity
```

The `get-caller-identity` result must show the intended AWS account and identity. Record the account ID for later verification steps:

```powershell
$AccountId = aws sts get-caller-identity --query Account --output text
$AccountId
```

Check the configured Region:

```powershell
aws configure get region
```

The workshop deployment Region is:

```text
ap-southeast-1
```

When a machine is configured for multiple AWS accounts, use the correct AWS CLI profile consistently. Do not deploy until both the AWS account ID and Region have been verified.

## 7. CampusMeet Resource Conventions

The following values are used throughout the workshop:

| Component | Value |
| --- | --- |
| Environment | `dev` |
| AWS Region | `ap-southeast-1` |
| Table prefix | `campusmeet-dev` |
| Data stack | `campusmeet-dev-data` |
| Authentication stack | `campusmeet-dev-auth` |
| Local frontend origin | `http://localhost:5173` |

The data stack manages five DynamoDB tables:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

Do not create tables, indexes, or permission changes manually in the AWS Console when those resources are managed by CloudFormation.

## 8. Readiness Checklist

Continue to the architecture and deployment sections only after confirming that:

- The CampusMeet repository is accessible.
- Node.js 22 LTS and npm 10 or later are available.
- Git, AWS CLI, AWS SAM CLI, and PowerShell are available.
- Project dependencies have been installed.
- The baseline source checks complete successfully.
- The correct IAM user is signed in through the AWS CLI.
- The AWS account ID matches the team development environment.
- The deployment Region is `ap-southeast-1`.
- Root credentials and shared access keys are not being used.
- An AWS Budget alert is configured.

After these prerequisites are complete, proceed to the CampusMeet system architecture and stack boundaries.
