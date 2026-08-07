---
title: "Minutes and Follow-up Work"
date: 2026-08-08
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# Minutes and Follow-up Work

## Goal

This section covers the post-meeting workflow in CampusMeet: saving minutes, recording decisions, creating action items, converting an action item into an official task, and reflecting task state on the dashboard.

```text
Meeting
  ↓
Minutes
  ↓
Decision + Action Item
  ↓
Action Item → Task
  ↓
TODO → DOING → DONE
  ↓
Dashboard
```

## 1. Meeting minutes

```http
GET /meetings/:meetingId/minutes
PUT /meetings/:meetingId/minutes
```

Minutes belong to a meeting, so the backend first resolves the meeting and checks group access. A minutes document can contain a summary, decisions, action items, assignees, due dates, and a version number.

## 2. Version conflicts

CampusMeet protects minutes from stale writes. If two users both read version 3 and one saves first, the second user must not silently overwrite version 4 with an older copy. The stale request should receive a conflict response so the UI can keep the local draft and ask the user to reconcile the newer version.

## 3. Decisions and action items

A decision records an agreed outcome from the meeting. An action item records follow-up work, usually including description, assignee, and optional deadline.

The assignee must still be a valid member of the group when the backend validates the request.

## 4. Convert an action item into a task

```http
POST /meetings/:meetingId/minutes/action-items/:actionItemId/task
```

The backend validates the meeting, group, assignee, and action-item state before creating the task. The operation must be safe against retries so one action item does not accidentally create multiple tasks.

## 5. Task management

```http
GET /tasks
POST /tasks
PATCH /tasks/:taskId/status
```

The normal state flow is:

```text
TODO → DOING → DONE
```

The backend validates transitions and permissions; hiding an option in the frontend is not an authorization control.

## 6. Dashboard

The personal dashboard summarizes task state, including totals, TODO, DOING, DONE, and overdue work when applicable.

A useful E2E check is to assign a task to User B, update it from TODO to DOING and then DONE, reload the dashboard, and verify the counters reflect the stored data.

## 7. AI proposals are not official tasks

CampusMeet can generate AI task proposals, but a proposal is not an official task by itself. The reliable E2E path in the current system is still:

```text
Minutes Action Item
→ user review
→ Convert to Task
```

The workshop does not claim that AI proposals are automatically committed as tasks unless that confirmation flow is verified end to end.

## 8. Verification

Run the normal quality checks before deployment:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
```

After deployment, verify minutes persistence, version conflicts, action-item assignment, one-time task conversion, task status transitions, cross-group denial, and dashboard updates.

## Result

Minutes and follow-up work are connected through a versioned workflow, and task state is stored and reflected on the dashboard instead of living only in frontend state.
