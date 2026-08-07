---
title: "AI Integration"
date: 2026-08-08
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

# AI Integration

## Goal

CampusMeet uses Amazon Bedrock to help users search meeting knowledge, draft minutes, and generate task proposals. The important part is not only calling a model: the system must limit retrieval to authorized sources and keep AI-generated output reviewable.

## 1. Architecture

```text
CampusMeet API
   ↓
AIJob
   ↓
Step Functions
   ↓
AI Worker
   ↓
Amazon Bedrock
   ↓
Knowledge Base / S3 Vectors
   ↓
result + citations
```

The API returns an `aiJobId` for asynchronous operations instead of keeping the original HTTP request open.

## 2. AIJob state

Typical states are:

```text
QUEUED
PROCESSING
COMPLETED
FAILED
```

The stored job identifies the group, optional meeting, operation type, request, attempts, and timestamps so processing can be tracked independently from the browser session.

## 3. Current AI operations

The source currently contains flows for:

- Meeting Chat.
- Group Search.
- Minutes Draft.
- Task Proposals.
- Progress Analysis.
- Document ingestion through `INGEST_SOURCE`.

The presence of a source implementation is not treated as proof that the corresponding cloud integration has already passed an end-to-end test.

## 4. Knowledge-source ingestion

```text
Attachment
  ↓
INGEST_SOURCE
  ↓
AI Worker
  ↓
Bedrock ingestion
  ↓
READY
```

A source still in `PROCESSING` is not considered ready for normal retrieval. The final deployment test must verify that ingestion polling actually moves the source to a terminal state in AWS.

## 5. Authorization before retrieval

Before sending retrieved content to a model, CampusMeet verifies that:

- the user is an active member of the group;
- selected meetings belong to the same group;
- the source belongs to the authorized scope;
- the source is approved/usable according to the current contract;
- ingestion is ready.

This prevents Group A content from leaking into Group B queries simply because both groups use the same underlying AI infrastructure.

## 6. Citations

Model-generated citations are not trusted automatically. Returned citations must correspond to chunks or sources the backend actually retrieved for the request.

When the source material does not support a confident answer, the system should report insufficient context rather than inventing a plausible response.

## 7. Human review

AI output is assistance, not an automatic business decision.

- AI minutes are drafts.
- Task proposals are proposals.
- Important writes still require a user-controlled path.

For the current core E2E, the reliable official-task flow is still:

```text
Minutes Action Item
→ user review
→ Convert to Task
```

The workshop does not claim AI proposals are automatically committed as tasks unless that confirmation path is verified end to end.

## 8. Failure handling

AI failures should affect the AI job, not corrupt Meeting or Task data. The system should handle unavailable models, timeouts, missing retrieval context, invalid citations, non-ready sources, and authorization changes with explicit job failure states.

## 9. Safe logging

Do not log complete prompts, private documents, transcripts, OAuth tokens, or sensitive model responses merely for debugging. Prefer job IDs, operation types, latency, result status, and error codes.

## 10. Verification

Before deployment run the normal lint, typecheck, test, and build commands. After deployment, upload a small known document, wait for the knowledge source to become `READY`, ask one answerable question and one unsupported question, and inspect the returned citations.

Current documentation status should be read as:

```text
Source implementation: present
Automated/local coverage: present for important paths
AWS production E2E: must be verified during the final deployment run
```

## Result

CampusMeet AI is designed around authorized retrieval, explicit job state, validated citations, and human review rather than unrestricted model access.
