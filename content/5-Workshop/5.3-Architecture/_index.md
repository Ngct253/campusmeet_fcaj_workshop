---
title: "IAM, CloudFormation, and AWS Configuration"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Three access layers

CampusMeet uses three control layers with different purposes. Separating them prevents identity, application authorization, and AWS service permissions from being treated as the same decision.

| Layer | Responsibility |
| --- | --- |
| Amazon Cognito | Verify user identity and issue a JWT for the signed-in session |
| Application authorization | The backend checks membership, role, group, meeting, and permitted action |
| AWS IAM | Define which Lambda or AWS service may call an API and access a resource |

A signed-in user does not receive IAM permission to access DynamoDB. The interface sends a JWT to the API, while Lambda uses its own execution role to read or write data after the backend completes the business authorization check.

## IAM for the authentication and API stack

The `infra/auth-integration.yaml` template creates a dedicated execution role for the authentication/API Lambda. Its trust policy allows only the Lambda service to assume it. Attached policies are limited to the current needs:

- write events to the Lambda log group;
- perform required DynamoDB reads and writes against `identity`, `collaboration`, `meeting-data`, and their indexes;
- read the verified email from the specific Cognito User Pool through `AdminGetUser`.

The role does not use `AdministratorAccess` and does not grant direct permissions to the browser. Workers and supporting orchestration use separate roles so S3, Scheduler, and AI permissions are not mixed into the core authentication Lambda.

## Application authorization

After API Gateway validates the JWT, the backend still considers four questions for each action:

1. Is the account signed in and valid?
2. Is the person an active member of the group?
3. Does the current role permit the specific action?
4. Does the action target the correct group, meeting, and version?

Successful sign-in does not permit reading another group's data, and permission to view a meeting does not automatically permit editing or approval. Documents, transcripts, minutes, tasks, and AI sources inherit the access boundary of their original group and meeting.

## Infrastructure boundaries

CampusMeet separates infrastructure templates by responsibility:

- `infra/data-foundation.yaml` manages five DynamoDB tables outside the application lifecycle to reduce data risk.
- `infra/auth-integration.yaml` creates Cognito, the HTTP API, IAM role, Lambda, and log group for the authentication/API scope used in development.
- `infra/user-content-orchestration.yaml` owns the user-content bucket, orchestration, reminders, Scheduler role, and related email configuration.
- `infra/template.yaml` is the application stack and receives table names and required outputs through parameters instead of recreating resources owned by another stack.

Separating the stacks makes changes easier to review and prevents an interface or API update from unintentionally changing the data foundation. Table names, bucket references, and endpoints are passed through CloudFormation parameters and outputs; credentials are not hard-coded in templates or source.

At the 08 August 2026 verification point, the data, authentication, and user-content stacks were `UPDATE_COMPLETE`. The `campusmeet-dev-app` application stack still manages the frontend, application API, Google worker, AI Worker, Knowledge Base, and monitoring through SAM/CloudFormation, but its current state is `UPDATE_ROLLBACK_FAILED` because of `ApiLambdaRole`. Existing AI resources remain active in the control plane; however, the team must recover the stack and review the next change set before another deployment. The workshop therefore does not report all four stacks as complete.

![CampusMeet CloudFormation stacks in completed state](images/5-Workshop/campusmeet-evidence/cloudformation-stacks.png)

*The three stable CampusMeet stacks have been updated successfully in the development environment. The application stack is assessed separately because its rollback state still requires recovery.*

## Authentication/API stack structure

The authentication/API stack accepts two main parameters: `AllowedOrigin` limits the frontend origin that may call the API, and `DataTablePrefix` selects the correct environment tables. CloudFormation then manages the related resources together:

| Resource group | Content |
| --- | --- |
| Identity | Cognito User Pool and User Pool Client for the web application |
| API entry point | API Gateway HTTP API with JWT authorizer and CORS |
| Processing | Node.js Lambda for the health endpoint and protected routes |
| Permissions | IAM role limited to logs, required tables, and the User Pool |
| Observability | CloudWatch Log Group with a defined retention period |

After deployment, the stack exports `UserPoolId`, `UserPoolClientId`, `ApiUrl`, and the three table names used by this scope. The frontend receives the required public values, while Lambda receives table names and the User Pool ID through server-side environment variables.

![Resources in the authentication and API stack](images/5-Workshop/campusmeet-evidence/cloudformation-auth-resources.png)

*The authentication/API stack manages Lambda, its IAM role, API Gateway, the Cognito User Pool and client, and the CloudWatch Log Group together. The resource list also provides a reliable way to identify the active resources instead of inferring them from display names.*

## CloudFormation principles and change verification

- Resources are declared in templates so environments can be recreated and reviewed through Git history.
- Parameters describe environment differences, while outputs provide identifiers required by another stack or the frontend.
- The data lifecycle remains separate from the application lifecycle and uses protection appropriate to each environment.
- Template changes are validated, reviewed for scope, and then deployed to the intended account and Region.
- After deployment, stack status, outputs, real resources, and CloudWatch Logs are checked rather than relying only on a completed deploy command.

This organization shows that IAM and CloudFormation are not secondary configuration. They define security boundaries, connect components, and make the authentication/API and data-foundation deployments repeatable and verifiable.
