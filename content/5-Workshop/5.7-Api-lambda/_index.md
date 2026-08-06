---
title: "API Gateway and AWS Lambda"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# API Gateway and AWS Lambda

## Objectives

This section explains how CampusMeet receives HTTP requests, validates Cognito JWTs, routes requests to Lambda, and accesses data through application and repository layers. The implementation is based on `infra/auth-integration.yaml`, `services/api/src/auth-integration.ts`, and `docs/api-contract.md`.

## Current API Structure

The `campusmeet-dev-auth` stack creates one HTTP API and one core Lambda function:

```text
Amazon API Gateway HTTP API
        |
        +--- GET /health          No JWT required
        |
        +--- OPTIONS /{proxy+}    No JWT required
        |
        +--- ANY /{proxy+}        Cognito JWT required
                                  |
                                  v
                     AuthIntegrationFunction
```

The HTTP API uses the `$default` stage. Its base URL has this shape:

```text
https://<api-id>.execute-api.ap-southeast-1.amazonaws.com
```

Do not append `/dev`.

## 1. Request Processing Flow

A protected request follows these steps:

1. The web application reads the current Cognito session.
2. The JWT is sent in the `Authorization` header.
3. API Gateway validates the JWT against the deployed User Pool and app client.
4. Lambda reads the HTTP method, route, parameters, and request body.
5. Lambda obtains the user identity from JWT claims.
6. The application layer validates membership and role before calling a repository.
7. The repository performs the required DynamoDB read or write.
8. Lambda returns a response defined by shared contracts in `@campusmeet/shared`.

The backend never trusts a client-provided role. Authorization is checked against stored membership data.

## 2. HTTP API Configuration

`infra/auth-integration.yaml` configures:

| Property | Value |
| --- | --- |
| Stage | `$default` |
| Default CORS origin | `http://localhost:5173` |
| CORS methods | `GET`, `POST`, `PATCH`, `DELETE`, `OPTIONS` |
| Allowed headers | `authorization`, `content-type`, `x-request-id`, `idempotency-key` |
| JWT identity source | `$request.header.Authorization` |
| Default authorizer | `CognitoAuthorizer` |

`GET /health` is declared separately without the JWT authorizer. All business routes are handled through `/{proxy+}`.

## 3. Core Route Groups

| Feature area | Main routes | Source status |
| --- | --- | --- |
| Service health | `GET /health` | Implemented handler |
| Profile | `GET /me`, `PATCH /me` | Implemented |
| Groups | `GET /groups`, `POST /groups`, `GET/PATCH /groups/:groupId` | Implemented |
| Membership | `DELETE /groups/:groupId/members/:userId` | Implemented; Group Admin removal is blocked |
| Invitations | Routes under `/groups/:groupId/invitations` and `/invitations` | Implemented |
| Meetings | `GET /meetings`, routes under `/groups/:groupId/meetings`, and `/meetings/:meetingId` | Create, read, update, and cancel core exists in source |
| Notifications | `GET /notifications`, `POST /notifications/:notificationId/read` | Implemented |

Modules that do not yet have complete handlers may return `501 Not Implemented` according to the API contract. A documented route is not, by itself, evidence that the full feature is deployed.

## 4. Shared Contracts

The frontend and backend import request and response types from:

```text
@campusmeet/shared
```

Do not duplicate TypeScript interfaces for the same API contract. When changing a contract:

1. Update the shared type.
2. Update the backend.
3. Update the web application.
4. Update `docs/api-contract.md`.
5. Run type checking, tests, and builds.

Quality commands:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
```

## 5. Lambda Source Boundaries

```text
API Gateway event
      ↓
Request handler
      ↓
Application service
      ↓
Repository interface
      ↓
DynamoDB repository or in-memory repository
```

Rules:

- A handler should not query DynamoDB directly when a repository boundary exists.
- The application layer owns authorization and business rules.
- The repository maps logical operations to keys, indexes, and DynamoDB commands.
- Unit tests can use in-memory repositories.
- The shared AWS environment is used for integration and post-deployment smoke tests.

## 6. Lambda Data Permissions

The current execution role grants access to:

- `campusmeet-dev-identity` and its indexes.
- `campusmeet-dev-collaboration` and its indexes.
- `campusmeet-dev-meeting-data` and its indexes.

DynamoDB permissions are limited to `GetItem`, `BatchGetItem`, `Query`, `PutItem`, `UpdateItem`, `DeleteItem`, and `ConditionCheckItem`.

The function also has `cognito-idp:AdminGetUser` on the stack's User Pool so it can read a verified email when invitation workflows require it.

## 7. Deploy API Updates

When Lambda source or infrastructure changes:

```powershell
sam validate `
  --template-file infra/auth-integration.yaml `
  --lint `
  --region ap-southeast-1

npm run sam:build:auth
```

Preview the change set:

```powershell
sam deploy `
  --template-file infra/auth-integration.yaml `
  --stack-name campusmeet-dev-auth `
  --resolve-s3 `
  --capabilities CAPABILITY_IAM `
  --parameter-overrides `
    AllowedOrigin=http://localhost:5173 `
    DataTablePrefix=campusmeet-dev `
  --no-execute-changeset `
  --region ap-southeast-1
```

Review that changes remain within the stack. Do not manually patch routes or IAM permissions in the AWS Console and leave the template outdated.

## 8. Test the API

Public route:

```powershell
curl.exe -i "<ApiUrl>/health"
```

Expected result: `200` with service status information.

Protected route without a JWT:

```powershell
curl.exe -i "<ApiUrl>/me"
```

Expected result: `401`.

For authenticated flows, test through the CampusMeet web application so the application obtains and sends the JWT. Do not place real tokens in shared command history or documentation.

Authorization cases to verify:

- An authenticated non-member cannot read group details.
- A normal member cannot update a group or create invitations.
- A Group Admin can perform administrative actions in their own group.
- A user cannot access a meeting from another group.
- Create requests that may be retried use `Idempotency-Key` when required by the contract.

## 9. Logs and Errors

The Lambda log group is:

```text
/aws/lambda/campusmeet-dev-auth-api
```

The current template retains logs for seven days.

Logs may contain request IDs, methods, paths, status codes, error codes, and resource IDs required for diagnosis. Logs must not contain JWTs, OAuth tokens, passwords, confirmation codes, raw invitation tokens, signed URLs, or complete transcript and file content.

Common response meanings:

| Status | Meaning |
| --- | --- |
| `401` | Missing or invalid JWT |
| `403` | Authenticated but not authorized or not a group member |
| `404` | Resource or route not found |
| `409` | State conflict or existing data conflict |
| `501` | Contract exists but the module does not yet have a complete handler |

## Expected Result

- You can explain the path from the browser to DynamoDB.
- `/health` is public and business routes require a JWT.
- Frontend and backend use shared contracts from `@campusmeet/shared`.
- Every group-scoped operation validates membership and role in the backend.
- Logs support diagnosis without exposing sensitive data.
