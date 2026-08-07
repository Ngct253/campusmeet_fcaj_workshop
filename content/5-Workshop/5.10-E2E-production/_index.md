---
title: "E2E Testing and Production Result"
date: 2026-08-08
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# E2E Testing and Production Result

The final workshop section records what has actually been deployed and verified. A feature is marked complete here only after it has run against the real AWS environment.

## Production link

After the frontend is published through CloudFront, record the final URL here:

```text
Production URL: TBD
Region: ap-southeast-1
```

The URL must work over HTTPS without relying on `localhost`.

## Core E2E flow

Use two real accounts so authorization is tested as well as functionality:

```text
User A signs up / signs in
  ↓
A creates Group
  ↓
A invites User B
  ↓
B accepts
  ↓
A creates Meeting
  ↓
A saves Minutes + Action Item
  ↓
Action Item → Task for B
  ↓
B changes TODO → DOING → DONE
  ↓
Dashboard updates
```

Reload the browser after important steps to confirm the data is really persisted in the backend.

## Actual result

| Area | Status |
| --- | --- |
| CloudFront production URL | Not verified |
| Cognito sign-up / sign-in | Not verified |
| Group and Invitation | Not verified |
| Meeting | Not verified |
| Minutes and Task | Not verified |
| Dashboard | Not verified |
| Basic authorization | Not verified |

Do not change a row to `PASS` only because a unit test or build succeeded.

## Optional integrations

After the core flow is stable, verify additional integrations when time allows:

- Google OAuth and Calendar/Meet synchronization;
- document upload;
- RAG and citations;
- other AI features enabled in the production environment.

An optional integration may remain unverified without invalidating a successful core E2E result.

## Submission evidence

Keep a small set of screenshots showing:

- production home or sign-in page;
- a Group with two members;
- Meeting detail;
- Minutes or Task;
- Dashboard after task completion;
- production CloudFormation resources.

The workshop should finish with the real production URL and the actual E2E status, not another long implementation checklist.