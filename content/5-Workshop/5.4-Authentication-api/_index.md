---
title: "Authentication and API"
date: 2026-08-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Authentication and API

CampusMeet uses Amazon Cognito for user accounts and API Gateway + Lambda for backend requests.

## Cognito flow

```text
Sign up
  ↓
Confirm email
  ↓
Sign in
  ↓
Cognito issues JWT
  ↓
Frontend calls API
```

Passwords, server secrets, and AWS credentials are never placed in frontend source code.

## API Gateway and Lambda

API Gateway validates the user's JWT before forwarding protected requests to Lambda. Lambda then checks the user's actual group membership and permissions before reading or changing application data.

```text
React
  ↓
API Gateway
  ↓
Lambda
  ↓
Authorization check
  ↓
DynamoDB / integration service
```

## Frontend configuration

The production frontend needs public values such as:

```dotenv
VITE_COGNITO_USER_POOL_ID=...
VITE_COGNITO_USER_POOL_CLIENT_ID=...
VITE_API_BASE_URL=...
```

These values connect the web application to the correct AWS environment and are not secrets.

## Authentication vs. authorization

Authentication answers **who the user is**. Authorization answers **what that user is allowed to do**. CampusMeet always applies both layers.

For production readiness, the public health endpoint must work while protected application endpoints must require a valid session.