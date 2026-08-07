---
title: "API Gateway and AWS Lambda"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# API Gateway and AWS Lambda

## Goal

This section explains how CampusMeet accepts HTTP requests, validates JWTs, routes requests to Lambda handlers, and applies business authorization. The early workshop exercises use `infra/auth-integration.yaml`, while the route overview also reflects the current full application handler in `services/api/src/index.ts`.

## 1. Request flow

```text
Frontend
  ↓ Authorization: Bearer <JWT>
API Gateway HTTP API
  ↓ JWT authorizer
Lambda handler/router
  ↓
Application/domain service
  ↓
Repository or integration adapter
```

The frontend does not decide the caller's role. The backend reads identity from the validated token and checks current membership and resource scope.

## 2. Public and protected endpoints

`GET /health` is public. Business endpoints require a valid JWT.

For example:

```text
GET /me without JWT
→ 401
```

CORS controls which browser origins can call the API; it does not replace authentication or authorization.

## 3. Core routes in the auth stack

The smaller `auth-integration.yaml` deployment covers core areas such as:

```text
/health
/me
/groups
/groups/:groupId
/groups/:groupId/invitations
/groups/:groupId/members/:userId
/groups/:groupId/meetings
/meetings
/meetings/:meetingId
/invitations
/notifications
```

This stack is useful for the Authentication, Collaboration, and core Meeting sections.

## 4. Full application routes

The full application handler extends the API with additional groups of routes.

### Meeting and Google sync

```text
/groups/:groupId/meetings
/meetings
/meetings/:meetingId
/meetings/:meetingId/cancel
/meetings/:meetingId/google-sync/retry
```

### Attachments

```text
/meetings/:meetingId/attachments/*
/attachments/:attachmentId/download-url
```

### Minutes, tasks, and dashboard

```text
/meetings/:meetingId/minutes
/meetings/:meetingId/minutes/action-items/:actionItemId/task
/tasks
/tasks/:taskId/status
/dashboard
```

### Google integration

```text
/integrations/google/connect
/integrations/google/callback
/integrations/google/meet-context
```

### AI

```text
/meetings/:meetingId/ai/chat
/groups/:groupId/ai/search
/meetings/:meetingId/ai/minutes-draft
/meetings/:meetingId/ai/task-proposals
/groups/:groupId/ai/progress-analysis
/ai/jobs/:aiJobId
```

A route existing in source is not proof that it is available in a smaller deployed stack. Always verify which template and Lambda handler are running in the environment under test.

## 5. Handler boundaries

The router matches method and path and passes the request to the appropriate handler. Handlers should delegate business rules to services and data access to repositories/adapters instead of embedding every DynamoDB expression in HTTP code.

```text
API event
  ↓
Handler
  ↓
Business service
  ↓
Repository / port
  ↓
AWS adapter
```

## 6. Shared contracts

Frontend and backend share DTOs and types from `@campusmeet/shared`. Contract changes should update the shared types, backend, frontend, documentation, and then pass typecheck/test/build together.

## 7. Domain authorization

API Gateway confirms token validity, but Lambda still verifies permissions. Reading a meeting, for example, requires resolving its group and checking active membership before returning data.

## 8. Idempotency and conflicts

Idempotency prevents duplicate processing of the same create/convert intent. Version-aware conditional writes prevent stale clients from overwriting newer data. These are separate controls and both matter in the CampusMeet workflow.

## 9. Error handling

Common response categories include:

| Status | Meaning |
| --- | --- |
| `400` | Invalid input |
| `401` | Missing/invalid authentication |
| `403` | Authenticated but not authorized |
| `404` | Resource or route not found |
| `409` | State/version conflict |
| `500` | Internal or unclassified dependency error |

Do not return stack traces or secrets to the browser.

## 10. Safe logging

Useful logs contain request IDs, method/path, status, resource identifiers, error codes, and latency. They should not contain JWTs, passwords, raw invitation tokens, Google OAuth tokens, presigned URLs, or full private transcripts/documents.

## 11. Pre-deployment checks

```powershell
npm run infra:validate
npm run lint
npm run typecheck
npm run test
npm run build
```

For the smaller auth/core stack:

```powershell
sam validate --template-file infra/auth-integration.yaml --lint --region ap-southeast-1
npm run sam:build:auth
```

For the full application:

```powershell
npm run sam:validate:app -- --region ap-southeast-1
npm run sam:build:app
```

## 12. Smoke test

After deployment, verify `/health` returns 200 and a protected endpoint without JWT returns 401. Then continue through the browser with Group → Meeting → Minutes → Task → Dashboard for the full application.

## Result

The API layer validates identity at the gateway, applies business authorization in Lambda services, uses shared contracts with the frontend, and clearly distinguishes the smaller core stack from the full application deployment.
