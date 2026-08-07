---
title: "Meeting Management"
date: 2026-08-08
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# Meeting Management

## Goal

This section explains how CampusMeet manages the lifecycle of a meeting after users and groups are already available. It follows the current API services, repositories, frontend pages, and automated tests rather than treating the original design document as proof that every feature is already running in AWS.

## 1. Meeting data

Meeting records belong to a group and are stored in the `meeting-data` table. A meeting keeps information such as `meetingId`, `groupId`, `organizerId`, title, start/end time, status, version, agenda, attendees, and Google synchronization state when that integration is enabled.

The group is the authorization boundary. Knowing a `meetingId` is not enough to read that meeting.

## 2. Create a meeting

```http
POST /groups/:groupId/meetings
```

The backend validates the authenticated user, verifies group membership and the required permission, validates the input, and then persists the meeting. When the contract requires an idempotency key, clients should reuse the same key for the same create intent so a retry does not create duplicate meetings.

## 3. List meetings

```http
GET /groups/:groupId/meetings
GET /meetings
```

The first route provides a group timeline. The second supports the current user's personal meeting view. Repositories use the designed keys and indexes instead of scanning the full table for normal requests.

## 4. Read meeting details

```http
GET /meetings/:meetingId
```

The backend resolves the meeting's group and checks active membership before returning the resource. The meeting page then becomes the workspace for details, minutes, action items, attachments, Google sync status, and AI features that are available in the deployed environment.

## 5. Update and version control

```http
PATCH /meetings/:meetingId
```

CampusMeet uses version-aware writes to avoid silently overwriting a newer change.

```text
Client reads version 4
        ↓
sends expected version 4
        ↓
server updates only if current version is still 4
        ↓
new version = 5
```

A stale update should return a conflict instead of replacing newer data.

## 6. Cancel a meeting

```http
POST /meetings/:meetingId/cancel
```

Cancellation is a business-state transition, not a physical deletion. Keeping the meeting record preserves history and links to minutes or follow-up work.

## 7. Authorization

The backend, not the UI, makes the final authorization decision. Tests should cover at least:

- users outside the group cannot read a meeting;
- members can perform only allowed actions;
- group admins or organizers receive the additional permissions defined by the service;
- cross-group access is rejected.

## 8. Google Calendar / Meet synchronization

Google is an external integration, so its state is kept separate from the core meeting record.

Typical states include:

```text
PENDING
SYNCED
FAILED
ACTION_REQUIRED
```

The intended flow is asynchronous:

```text
Meeting change
   ↓
persist CampusMeet state
   ↓
Google sync worker
   ↓
Google Calendar API
```

A Google failure must not erase the CampusMeet meeting. Real OAuth, Calendar event creation, and Meet URL verification are covered later in the E2E section.

## 9. Verification

Before deployment run:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
```

After deployment, use at least two real users to create, read, update, reload, and cancel a meeting, and verify that a user outside the group is denied.

## Result

At the end of this section, meeting CRUD is connected to group authorization and version control, while Google synchronization remains an asynchronous integration that is verified separately from the core workflow.
