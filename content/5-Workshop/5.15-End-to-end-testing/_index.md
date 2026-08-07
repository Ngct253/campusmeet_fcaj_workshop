---
title: "End-to-End Testing"
date: 2026-08-08
weight: 15
chapter: false
pre: " <b> 5.15. </b> "
---

# End-to-End Testing

## Goal

This section proves that the core CampusMeet flow works across the browser, Cognito, API Gateway, Lambda, and persisted data. Unit tests and successful builds remain important, but they are not substitutes for a real deployed E2E run.

## 1. Test users

Use two real email addresses:

- User A becomes the `GROUP_ADMIN`.
- User B joins as a `MEMBER`.

Using two users is necessary to verify role boundaries and membership behavior.

## 2. Basic service checks

Before the business flow:

```text
GET /health
→ expected 200

GET /me without JWT
→ expected 401
```

Then sign up, confirm, and sign in through the browser. Reload a protected route to verify session restoration.

## 3. Core E2E flow

```text
A signs in
  ↓
A creates Group
  ↓
A invites B
  ↓
B accepts invitation
  ↓
A creates Meeting
  ↓
A updates Meeting
  ↓
A saves Minutes
  ↓
A creates Action Item assigned to B
  ↓
A converts Action Item → Task
  ↓
B changes TODO → DOING → DONE
  ↓
B's Dashboard reflects the stored task state
```

This is the minimum production-demo path for the current release.

## 4. Core result table

| Step | Expected | Actual | Status |
| --- | --- | --- | --- |
| Open deployed frontend | HTTPS page loads | TBD | Not verified |
| `/health` | 200 | TBD | Not verified |
| Protected API without JWT | 401 | TBD | Not verified |
| User A sign-up/confirm | Account confirmed | TBD | Not verified |
| User B sign-up/confirm | Account confirmed | TBD | Not verified |
| Create group | Group stored, A is admin | TBD | Not verified |
| Invite B | Invitation pending | TBD | Not verified |
| B accepts | B becomes member | TBD | Not verified |
| Create meeting | Meeting appears in list/detail | TBD | Not verified |
| Update meeting | Data survives reload | TBD | Not verified |
| Save minutes | Versioned minutes stored | TBD | Not verified |
| Add action item | B is assigned | TBD | Not verified |
| Convert to task | One task created | TBD | Not verified |
| B views tasks | Task is visible | TBD | Not verified |
| B updates status | TODO/DOING/DONE persists | TBD | Not verified |
| Dashboard | Counters reflect task | TBD | Not verified |

Do not replace `TBD` with `PASS` until the corresponding deployed/browser test is actually run.

## 5. Authorization checks

Also verify that unauthenticated users cannot open protected pages, users outside a group cannot read its resources, normal members cannot perform admin-only actions, and stale version writes return a conflict rather than overwriting newer data.

## 6. Persistence checks

Reload or sign out/sign back in after important mutations. Group membership, meeting updates, minutes, task state, and dashboard data must come back from persisted backend data instead of surviving only in local frontend state.

## 7. Optional Google E2E

Only after the core path passes:

1. Connect Google in Settings.
2. Complete OAuth.
3. Create or update a meeting.
4. Wait for sync state.
5. Verify the real Calendar event and Meet URL when available.

A Google failure should be recorded as an integration failure, not as proof that the internal Meeting workflow is broken.

## 8. Optional document RAG E2E

Use a small text document with an easy-to-check fact. Upload it, wait for the knowledge source to become `READY`, ask one supported question and one unsupported question, and verify the citation and insufficient-context behavior.

| Step | Expected | Actual | Status |
| --- | --- | --- | --- |
| Document upload | Attachment completes | TBD | Not verified |
| Ingestion | Source becomes `READY` | TBD | Not verified |
| Supported question | Answer with citation | TBD | Not verified |
| Unsupported question | Safe insufficient-context result | TBD | Not verified |

## 9. Diagnosing failures

When an E2E step fails, record the time and safe request/resource identifier, check the corresponding CloudWatch logs, and determine whether the failure belongs to frontend configuration, API logic, IAM, persisted data, or an external service before changing code.

Do not copy tokens or secrets into the test report.

## Acceptance criteria

The core release is accepted when the deployed frontend, Cognito authentication, two-user group flow, meeting persistence, minutes, action-item-to-task conversion, task status updates, dashboard, and basic authorization all work without relying on localhost.

Google and RAG remain optional integrations for this release and must be documented honestly if they have not yet passed.
