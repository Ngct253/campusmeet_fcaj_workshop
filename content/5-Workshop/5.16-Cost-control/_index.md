---
title: "Cost Control"
date: 2026-08-08
weight: 16
chapter: false
pre: " <b> 5.16. </b> "
---

# Cost Control

## Goal

CampusMeet uses mostly managed and serverless AWS services, so cost generally grows with usage. That model is convenient for a student project, but it does not mean every service is automatically free. This section identifies the main cost drivers and the controls that keep test environments predictable.

## 1. AWS Budgets

Create a budget before a large deployment and send alerts to an email address that is actually monitored. Common thresholds are 50%, 80%, and 100% of the monthly budget.

A budget is an alerting mechanism, not an automatic shutdown system. When it fires, use Billing or Cost Explorer to identify the service that changed.

## 2. DynamoDB

CampusMeet tables use `PAY_PER_REQUEST`, which is suitable for irregular development traffic. Cost is driven by reads, writes, storage, and optional protection such as point-in-time recovery.

Normal application requests should use designed keys and indexes rather than full-table scans.

## 3. Lambda and API Gateway

Lambda cost depends on invocations and execution time; API Gateway depends on requests. Avoid unbounded retries and avoid keeping synchronous functions waiting on external services when an asynchronous worker is more appropriate.

## 4. S3 and CloudFront

S3 cost comes from storage, requests, and some transfer patterns. CloudFront adds request and transfer cost for the deployed frontend. Keep user content private and apply lifecycle policies that match the project rather than storing unnecessary large test files indefinitely.

## 5. CloudWatch Logs

CloudWatch charges for log ingestion and retention. Use a sensible retention period, avoid logging large payloads, and prevent error loops that generate thousands of duplicate messages.

## 6. EventBridge Scheduler

CampusMeet uses schedules for reminders and delayed retries. Duplicate schedules or retry loops are both a correctness problem and a cost problem, so each workflow needs clear replacement/cancellation behavior and a retry limit.

## 7. Step Functions

Standard Workflows charge by state transition. Polling loops such as `Wait → Check → Wait` need a sensible delay and an explicit stopping condition. Never leave a dependency polling forever after a permanent failure.

## 8. SES

Reminder email volume is normally small, but failures should not be retried forever. Sender verification and sandbox restrictions should be configured correctly before debugging application code.

## 9. Amazon Bedrock and RAG

AI usage can become one of the faster-growing cost areas because generation depends on model usage and input/output size. Limit unnecessary context, avoid repeatedly ingesting unchanged documents, keep retrieval scope small, and watch retries.

Knowledge Base/vector storage and retrieval also contribute to the total AI cost; generation is not the only billed component.

## 10. Amazon Transcribe

Transcribe is a future cost driver if full live or batch transcription is enabled. Because transcription is not part of the current core E2E release, the workshop does not require enabling it merely to complete the demo.

## 11. Development and production

Development environments favor lower cost and rapid iteration. Production may justify stronger data protection such as PITR and deletion protection, even when those protections add some cost.

## 12. After a large test run

Use Billing/Cost Explorer to inspect DynamoDB, S3, Lambda, CloudWatch, Step Functions, and Bedrock usage. Also check for unexpectedly active schedules or executions and confirm budget notifications still work.

## Result

The workshop finishes with cost visibility and operational limits. It does not include a separate resource-cleanup chapter because the CampusMeet environment remains available for subsequent deployment and E2E work.
