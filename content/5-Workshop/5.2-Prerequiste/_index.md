---
title: "Preparation and AWS Access"
date: 2026-07-27
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Preparation and AWS Access

Before deploying CampusMeet, prepare the local tools and verify the AWS account that will be used for the workshop.

## Required tools

- Node.js 22 and npm.
- Git.
- AWS CLI.
- AWS SAM CLI.
- PowerShell on Windows.

Quick check:

```powershell
node --version
npm --version
git --version
aws --version
sam --version
```

## CampusMeet source code

```powershell
git clone https://github.com/Ngct253/CampusMeet.git
cd CampusMeet
npm ci
```

The main project areas are:

```text
apps/web/       React frontend
services/api/   API and application logic
services/ai-worker/  AI processing
infra/          AWS infrastructure
scripts/        deployment utilities
```

Before deployment, run the normal project checks:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
```

## AWS account and Region

The workshop uses:

```text
ap-southeast-1 (Singapore)
```

Always verify the current AWS identity and Region before deploying:

```powershell
aws sts get-caller-identity
aws configure get region
```

Do not use the root account for daily work and do not share long-lived access keys between team members.

## Access model

Two types of permission should be kept separate:

- **Deployment permissions** for creating or updating CloudFormation resources.
- **Service execution roles** used by Lambda and workers to access DynamoDB, S3, Bedrock, and other AWS services.

Secrets such as Google client secrets, OAuth tokens, and AWS credentials must stay out of Git and out of frontend `VITE_*` variables.

Development resources can use the `campusmeet-dev-*` naming pattern. The final deployment should use a separate production environment such as `campusmeet-prod-*` to avoid mixing demo data with development data.