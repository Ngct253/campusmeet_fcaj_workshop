---
title: "Monitoring, Security, and Cost"
date: 2026-08-08
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# Monitoring, Security, and Cost

Once CampusMeet is running, the project still needs basic observability, access protection, and cost control.

## Monitoring

CloudWatch is used for API and worker logs. Logs should identify failed requests and resources without exposing JWTs, OAuth tokens, presigned URLs, or full user documents.

Important signals include API failures, repeated worker failures, AI job errors, and Google synchronization that does not recover after retries.

## Security

CampusMeet applies a few core rules:

- Cognito authenticates users while the backend enforces application permissions.
- Lambda and workers use scoped execution roles.
- Google secrets and tokens remain on the server side.
- User-content S3 buckets stay private.
- Production CORS points to the actual frontend origin.
- Production data can use PITR and deletion protection.

## Cost control

The main services to watch are DynamoDB, Lambda, API Gateway, S3, CloudFront, CloudWatch, Step Functions, and Bedrock.

Serverless pricing works well for a small project, but uncontrolled retries, excessive logs, and repeated AI calls can increase cost quickly. AWS Budgets should be configured for early alerts, and asynchronous workflows need clear retry limits and stopping conditions.

## Before release

Check that:

- no secrets are committed to Git;
- the frontend bundle contains no server-side credentials;
- the user-content bucket is private;
- production CORS uses the CloudFront origin;
- logs do not expose tokens;
- Budget alerts remain enabled.

There is no separate cleanup chapter because the production environment remains available for project demonstration and evaluation.