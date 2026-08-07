---
title: "Monitoring and Security"
date: 2026-08-08
weight: 14
chapter: false
pre: " <b> 5.14. </b> "
---

# Monitoring and Security

## Goal

A deployable application also needs enough visibility to diagnose failures and enough boundaries to protect user data. This section covers the operational controls already represented in the CampusMeet architecture and infrastructure.

## 1. CloudWatch Logs

API, AI, Google sync, and reminder functions should have separate log streams or log groups so failures can be traced to the component that handled them.

Useful log fields include request IDs, resource IDs, operation names, status, error codes, revision/attempt values, and latency.

## 2. Do not log secrets

Avoid writing the following values into logs:

- JWTs;
- passwords or confirmation codes;
- Google authorization codes and tokens;
- presigned S3 URLs;
- raw invitation tokens;
- full private documents or transcripts.

## 3. Metrics and alarms

The application infrastructure includes monitoring for important API and worker failures. Useful signals include elevated Lambda errors, long AI worker duration, and Google synchronization that still fails after retries. SNS can be used as the notification target for alarms.

The goal is not to create an alarm for every event, but to surface failures that users or maintainers would otherwise miss.

## 4. Least-privilege IAM

Each function should use a role that grants only the AWS actions and resources it needs. An AI worker needs different permissions from a reminder function; sharing one administrator role between them would remove that boundary.

## 5. Secrets Manager

Google OAuth client secrets remain server-side. They must not be placed in `VITE_*`, frontend `.env` files committed to Git, logs, issues, or pull requests.

## 6. Private S3

User-content buckets stay private. Browsers receive short-lived signed URLs only after the backend checks access. Production CORS should be restricted to the real frontend origin instead of remaining a wildcard simply to avoid browser errors.

## 7. Authentication and authorization

Cognito/API Gateway answers "who is calling?". The application service still answers "may this user access this group, meeting, task, or source?".

A valid JWT is therefore not permission to read every resource.

## 8. Idempotency and concurrency

Idempotency protects create/convert operations from duplicate retries. Versioned conditional writes protect resources such as meetings and minutes from stale updates. These controls solve different problems and both are needed in a distributed application.

## 9. Retry and stale work

Background work must not overwrite newer state. For example, a retry for Google sync revision 7 should become a no-op when the meeting is already on revision 8.

## 10. Production data protection

The data stack supports controls such as point-in-time recovery, deletion protection, `DeletionPolicy: Retain`, and `UpdateReplacePolicy: Retain`. Production environments should enable the protections appropriate to the value of their stored data.

## 11. Release checks

Before release run:

```powershell
npm run infra:validate
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
npm audit --omit=dev
```

Do not blindly run `npm audit fix --force` immediately before a release; first determine whether a reported vulnerability is in a reachable runtime dependency or development tooling.

## 12. Post-deployment checks

Verify that `/health` works, protected APIs reject missing JWTs, cross-group access is denied, user-content S3 is not public, sensitive Google secrets are not present in the frontend bundle, and worker failures are visible in CloudWatch.

## Result

CampusMeet combines identity, domain authorization, scoped IAM, private storage, safe logging, concurrency controls, and operational monitoring rather than relying on a single security layer.
