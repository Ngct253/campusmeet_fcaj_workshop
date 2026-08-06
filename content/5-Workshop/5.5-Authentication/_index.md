---
title: "Authentication with Amazon Cognito"
date: 2026-07-27
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Authentication with Amazon Cognito

## Objectives

This section deploys the shared CampusMeet authentication stack, reads its CloudFormation outputs, and configures the React application so users can sign up, confirm an account, sign in, and access protected routes.

## Resources Created

`infra/auth-integration.yaml` defines:

| Resource | Responsibility |
| --- | --- |
| Amazon Cognito User Pool | User accounts and email verification |
| Cognito User Pool Client | Browser authentication through SRP and refresh-token flows without a client secret |
| Amazon API Gateway HTTP API | `$default` stage and JWT authorizer |
| AWS Lambda | `/health` and the current core API routes |
| Lambda IAM role | Logging and access to the required tables |
| CloudWatch Log Group | Seven-day log retention for the Lambda function |

Current User Pool settings include email usernames, automatic email verification, self-service registration, an eight-character password minimum with mixed character requirements, no browser client secret, and MFA set to `OFF` in the current template.

## 1. Verify the Account and Source Code

From the CampusMeet repository root:

```powershell
aws login
aws sts get-caller-identity
npm install
```

The account must match the approved development account. The data stack and its five DynamoDB tables must exist before the API stack uses them.

## 2. Validate and Build the SAM Template

```powershell
sam validate `
  --template-file infra/auth-integration.yaml `
  --lint `
  --region ap-southeast-1

npm run sam:build:auth
```

Do not continue when validation or the build fails.

## 3. Preview the CloudFormation Change Set

The deployment owner creates a change set without executing it:

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

Review the change set for:

- Cognito User Pool and app client.
- HTTP API.
- Lambda function and execution role.
- CloudWatch Log Group.
- No new DynamoDB table.
- No unexpected resource replacement.

## 4. Deploy the Stack

After review, execute the change set in CloudFormation or rerun the command without `--no-execute-changeset`:

```powershell
sam deploy `
  --template-file infra/auth-integration.yaml `
  --stack-name campusmeet-dev-auth `
  --resolve-s3 `
  --capabilities CAPABILITY_IAM `
  --parameter-overrides `
    AllowedOrigin=http://localhost:5173 `
    DataTablePrefix=campusmeet-dev `
  --region ap-southeast-1
```

Check the stack status:

```powershell
aws cloudformation describe-stacks `
  --stack-name campusmeet-dev-auth `
  --region ap-southeast-1 `
  --query "Stacks[0].StackStatus" `
  --output text
```

The expected status is `CREATE_COMPLETE` or `UPDATE_COMPLETE`.

## 5. Read the Outputs

```powershell
aws cloudformation describe-stacks `
  --stack-name campusmeet-dev-auth `
  --query "Stacks[0].Outputs" `
  --output table `
  --region ap-southeast-1
```

Frontend mapping:

| CloudFormation output | Frontend variable |
| --- | --- |
| `UserPoolId` | `VITE_COGNITO_USER_POOL_ID` |
| `UserPoolClientId` | `VITE_COGNITO_USER_POOL_CLIENT_ID` |
| `ApiUrl` | `VITE_API_BASE_URL` |

The stack uses the API Gateway `$default` stage, so do not append `/dev` to `ApiUrl`.

## 6. Configure the Web Application

Copy `apps/web/.env.example` to `apps/web/.env` and enter the deployment outputs:

```dotenv
VITE_COGNITO_USER_POOL_ID=<UserPoolId>
VITE_COGNITO_USER_POOL_CLIENT_ID=<UserPoolClientId>
VITE_API_BASE_URL=<ApiUrl>
```

Rules:

- Do not add a trailing slash to `VITE_API_BASE_URL`.
- Do not place passwords, tokens, or AWS credentials in `.env`.
- Do not commit `.env`.
- Restart Vite after changing the file.

## 7. Test the Authentication Flow

```powershell
npm run dev
```

Test in this order:

1. Open `http://localhost:5173/sign-up`.
2. Register with an email address that can receive the verification code.
3. Confirm the account using the code sent by Cognito.
4. Sign in at `/sign-in`.
5. Confirm that `/app` is accessible.
6. Sign out.
7. Confirm that a later request to `/app` returns to `/sign-in`.

Test the public and protected API paths:

```powershell
curl.exe -i "<ApiUrl>/health"
curl.exe -i "<ApiUrl>/me"
```

Expected behavior:

- `/health` is public and returns `200` when the API is available.
- `/me` returns `401` without a JWT.

The web application manages the Cognito session. Do not copy JWTs into documentation, screenshots, or issues.

## 8. API Gateway Authentication Configuration

The HTTP API uses:

- Stage `$default`.
- JWT identity from the `Authorization` header.
- An issuer derived from the deployed Cognito User Pool.
- The User Pool Client as the JWT audience.
- CORS origin from the `AllowedOrigin` parameter.
- Methods `GET`, `POST`, `PATCH`, `DELETE`, and `OPTIONS`.
- Headers `authorization`, `content-type`, `x-request-id`, and `idempotency-key`.

`GET /health` and `OPTIONS` do not use the authorizer. Other paths are handled through `/{proxy+}` and require a valid JWT.

## 9. Common Problems

| Symptom | Check |
| --- | --- |
| The web application reports missing AWS configuration | Confirm `apps/web/.env`, all three variables, and a restarted Vite process |
| The User Pool or app client cannot be found | The values must come from the same Region and stack |
| The API returns `404` when the URL includes `/dev` | Remove `/dev`; this stack uses `$default` |
| The browser reports a CORS error | `AllowedOrigin` must exactly match `http://localhost:5173` without a path |
| `/me` returns `401` from curl | This is expected when no JWT is sent |
| No confirmation email arrives | Check the email address, spam folder, and Cognito user state |

## Expected Result

- `campusmeet-dev-auth` reaches a successful stack state.
- The web application uses the correct `UserPoolId`, `UserPoolClientId`, and `ApiUrl`.
- A user can register, confirm, sign in, and sign out.
- `/health` is public while `/me` is protected by the JWT authorizer.
