---
title: "IAM and Environment Configuration"
date: 2026-07-27
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# IAM and Environment Configuration

## Objectives

This section prepares AWS access for the CampusMeet team, verifies the correct account and Region before deployment, and separates human permissions from the permissions used by Lambda at runtime.

## Responsibility Model

CampusMeet uses one shared AWS development account with distinct responsibilities:

| Role | Responsibility |
| --- | --- |
| Root account | Used only for account-level tasks that cannot be delegated; never used for daily development |
| Deployment owner | Reviews CloudFormation changes and deploys shared resources that require IAM permissions |
| Developer | Builds features, runs tests, reads logs, and uses approved development resources |
| Lambda execution role | Allows a Lambda function to write logs and access only the required AWS resources |

IAM users, passwords, sessions, and access keys must not be shared.

## 1. Confirm the Development Environment

The team must agree on:

- AWS account ID.
- Region `ap-southeast-1`.
- Deployment owner.
- Shared stack names.
- Resource prefix `campusmeet-dev`.

Workshop naming:

| Component | Name |
| --- | --- |
| Data stack | `campusmeet-dev-data` |
| Authentication and core API stack | `campusmeet-dev-auth` |
| Five-table prefix | `campusmeet-dev` |
| Local allowed origin | `http://localhost:5173` |

## 2. Establish the Deployment Owner

Only perform these steps when the account does not already have an appropriate administrator:

1. Sign in as root once.
2. Create the IAM user `campusmeet-admin` with Console access.
3. Attach `AdministratorAccess` to the deployment owner.
4. Enable MFA for root and the administrator account.
5. Sign out of root and continue with the administrator account.

Do not create another administrator when an equivalent deployment owner already exists.

## 3. Give Developers Individual Access

The deployment owner performs the following steps in the AWS Console:

1. Open **IAM → User groups**.
2. Create `CampusMeetDevelopers` when it does not exist.
3. Attach `SignInLocalDevelopmentAccess` so developers can use `aws login` with temporary credentials.
4. In a development account dedicated to CampusMeet, attach `PowerUserAccess` when developers need access to AWS services.
5. Create one IAM user for each developer.
6. Enable Console access, require a password change on first sign-in, and add the user to `CampusMeetDevelopers`.
7. Share the sign-in URL, IAM username, and temporary password through a private channel.

`PowerUserAccess` does not allow unrestricted IAM administration or `iam:PassRole`. Because `infra/auth-integration.yaml` creates a Lambda execution role, the deployment owner remains responsible for deploying that stack.

Do not create long-lived access keys for human users.

## 4. Sign In with the AWS CLI

Each developer uses their own Console credentials:

```powershell
aws --version
aws login
aws sts get-caller-identity
```

The result must show the expected session and the account ID approved by the team.

Store the account ID for verification commands:

```powershell
$AccountId = aws sts get-caller-identity --query Account --output text
$AccountId
```

Check the configured Region:

```powershell
aws configure get region
```

Workshop deployment commands use:

```text
ap-southeast-1
```

Do not continue when the account ID or Region is incorrect.

## 5. Work with Multiple AWS Profiles

When a workstation uses several AWS accounts, select one profile and use it consistently:

```powershell
aws login --profile campusmeet-dev
aws sts get-caller-identity --profile campusmeet-dev
```

Add `--profile campusmeet-dev` to related `sam deploy`, `aws cloudformation`, and verification commands. Do not mix profiles during one deployment.

## 6. Runtime Permissions for Lambda

Human users and Lambda functions do not share permissions.

In `infra/auth-integration.yaml`, the core Lambda execution role can:

- Write `CreateLogStream` and `PutLogEvents` to its own log group.
- Read and write the required data in the `identity`, `collaboration`, and `meeting-data` tables and their indexes.
- Call `cognito-idp:AdminGetUser` only for the stack's User Pool.

The DynamoDB actions are limited to the operations required by the current implementation:

```text
dynamodb:GetItem
dynamodb:BatchGetItem
dynamodb:Query
dynamodb:PutItem
dynamodb:UpdateItem
dynamodb:DeleteItem
dynamodb:ConditionCheckItem
```

Do not grant `dynamodb:*` or account-wide table access as a quick workaround.

## 7. Environment Variables and Sensitive Data

The web application uses three public deployment outputs:

```dotenv
VITE_COGNITO_USER_POOL_ID=<UserPoolId>
VITE_COGNITO_USER_POOL_CLIENT_ID=<UserPoolClientId>
VITE_API_BASE_URL=<ApiUrl>
```

Because `VITE_*` values are included in the browser build, they must never contain passwords, JWTs, AWS access keys, app client secrets, or OAuth tokens.

CloudFormation passes table names to Lambda through environment variables. Lambda uses its IAM role to access AWS and does not store AWS access keys in the function configuration.

## 8. Budget Alerts

Create an AWS Budget before deploying the shared environment and configure an email recipient. A budget alert does not block charges, but it helps identify retained resources or unexpected usage.

Review these common cost sources:

- CloudWatch log retention.
- DynamoDB tables preserved by `Retain` policies.
- Speech and AI services charged by usage.
- Resources created in a Region other than `ap-southeast-1`.

## 9. Pre-Deployment Checklist

- [ ] Root is not used for daily work.
- [ ] Every developer has an individual IAM user.
- [ ] No long-lived access key is shared.
- [ ] `aws sts get-caller-identity` returns the approved account.
- [ ] The deployment Region is `ap-southeast-1`.
- [ ] A deployment owner is assigned.
- [ ] Stack names and the resource prefix are agreed.
- [ ] A budget alert is configured.
- [ ] The repository contains no password, token, or AWS credential.

## Expected Result

Each developer can sign in with an individual AWS session, confirm the correct account and Region, and understand which shared infrastructure changes remain restricted to the deployment owner.
