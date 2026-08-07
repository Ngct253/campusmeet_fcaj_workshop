---
title: "Meeting Content and Asynchronous Processing"
date: 2026-08-08
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

# Meeting Content and Asynchronous Processing

## Goal

Some CampusMeet operations should not keep an HTTP request open while waiting for a file upload, Google API, reminder schedule, or AI ingestion job. This section explains the asynchronous paths that already exist in the codebase and clearly separates them from transcription features that are not part of the current core E2E release.

## 1. Direct S3 uploads

Large files are uploaded directly from the browser to a private S3 bucket.

```text
Browser
  ↓ request upload permission
API validates access and metadata
  ↓
presigned S3 URL
  ↓
Browser uploads directly to S3
  ↓
complete request
  ↓
Attachment metadata is stored
```

The Lambda does not proxy the full file. Presigned URLs are temporary credentials and should not be written into long-lived logs.

## 2. Attachment metadata

DynamoDB stores metadata and workflow state, not the binary object itself. Typical fields identify the attachment, related meeting/group, S3 key, content type, and completion status.

## 3. Document ingestion

Documents used for AI retrieval follow an asynchronous path:

```text
Attachment complete
   ↓
AIJob: INGEST_SOURCE
   ↓
Step Functions
   ↓
AI Worker
   ↓
Bedrock ingestion
   ↓
Knowledge source becomes READY
```

Starting a Bedrock ingestion job is not the same as completing it. The orchestration must keep checking the ingestion status until it succeeds, fails, or reaches its designed stopping condition. This polling behavior still has to be verified on the real AWS environment.

## 4. Google synchronization

Meeting changes are persisted before Google synchronization runs.

```text
Meeting change
   ↓
DynamoDB + sync state
   ↓
DynamoDB Stream
   ↓
GoogleSyncWorker
   ↓
Google Calendar API
```

External failures should produce a retryable or actionable sync state rather than rolling back the CampusMeet meeting.

## 5. Scheduler-based retry

EventBridge Scheduler can run delayed retries. Retry logic must have limits, identify the revision it is processing, and avoid letting stale work overwrite a newer state.

## 6. Meeting reminders

```text
Meeting reminder
   ↓
EventBridge Scheduler
   ↓
Reminder Lambda
   ↓
in-app notification
   ↓
SES email when configured
```

The in-app notification remains the primary application record. Email delivery failure should not remove an already-created notification.

## 7. AI jobs and Step Functions

Long-running AI work uses an `aiJobId` so the frontend can poll the job instead of holding the original API request open.

```text
API request
  ↓
AIJob QUEUED
  ↓
Step Functions
  ↓
AI Worker
  ↓
COMPLETED / FAILED
```

## 8. Idempotency and stale work

Asynchronous systems can deliver the same work more than once. CampusMeet therefore relies on idempotency, revisions, conditional writes, and state checks so duplicate or stale events become safe no-ops instead of corrupting newer data.

## 9. Features not included in the current core E2E

The architecture includes future or partially designed speech-processing paths, but the workshop does not claim they are production-complete.

- Live streaming transcription: not part of the current core E2E.
- Full recording lifecycle and consent flow: not part of the current core E2E.
- Batch audio transcription: not treated as available until a complete runtime worker is present and verified.

For the current workshop, a small text/PDF document is the recommended ingestion source for AI/RAG testing.

## 10. E2E verification

After deployment, test a direct document upload, attachment completion, and—if AI is enabled—the ingestion job reaching `READY`. Test Google synchronization only after the core Meeting workflow already passes.

Actual AWS/browser results must be recorded separately from source-level implementation status.

## Result

CampusMeet moves file transfer and external-service work out of synchronous API requests, while the workshop stays explicit about which transcription features are not yet part of the verified release.
