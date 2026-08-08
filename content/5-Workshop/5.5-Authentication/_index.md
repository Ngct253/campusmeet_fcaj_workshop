---
title: "Meeting Workflow"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Before the meeting

A coordinator creates the meeting within a group and provides the time, purpose, and necessary information. Members review the schedule, prepare content, and access related documents. When calendar integration is available, CampusMeet can support event synchronization and an online meeting link.

## During the meeting

The group uses prepared content to guide the discussion. Important points, decisions, and follow-up actions are recorded for later review. CampusMeet does not replace video conferencing; it organizes and preserves information related to the meeting.

## After the meeting

Minutes are reviewed for accuracy. Action items become tasks with an owner, due date, and status. Members continue updating progress so the meeting outcome does not remain only as a note.

| Stage | Main information |
| --- | --- |
| Before | Purpose, time, participants, preparation, and documents |
| During | Notes, decisions, and action items |
| After | Minutes, tasks, owners, due dates, and progress |

## End-to-end example

A student project team holds a weekly progress review. After an administrator creates the group and members accept their invitations, the coordinator schedules a meeting to review completed work and agree on the next responsibilities. Members review preparation notes and attach relevant documents; the calendar event and Google Meet link may also be synchronized when the integration is available.

During the meeting, decisions and follow-up actions are recorded with the discussion. A transcript, when used with appropriate consent, remains supporting material. An authorized person later reviews the minutes or transcript, corrects inaccurate content, and confirms the appropriate version.

Agreed action items become tasks with owners and due dates. Members update progress, and the team can trace each task to its meeting. Approved sources may support citation-grounded questions or summary drafts, but users still decide what becomes an official outcome.

## Workflow control points

- **Before the meeting:** the purpose, time, owning group, and preparation material should be clear before members are notified.
- **During the meeting:** notes and transcripts remain supporting information; collection requires appropriate consent and does not automatically create official minutes.
- **After the meeting:** content is reviewed before confirmation, while each task has an owner, due date, and link to its source meeting.
- **With AI assistance:** sources stay within the requester's access, answers remain grounded, and business changes still require confirmation.

## Technical processing

### Document upload

Files do not pass through the API payload or live directly in DynamoDB. Upload is separated into the following steps:

```text
Frontend requests an upload
  → Backend checks membership, file type, size, and checksum
  → An object key is created for the correct group/meeting
  → A short-lived presigned URL is issued
  → The browser uploads directly to private S3
  → The backend verifies the object and stores its metadata
```

Knowing an object key does not grant download access. When a user needs the file again, the backend rechecks permission before issuing another short-lived URL. If the file requires further processing, upload completion creates an idempotent job so retries do not create multiple jobs for the same source.

### Google and reminders

The CampusMeet Meeting remains the primary record. Under the current design, Calendar API synchronization and the Google Meet link occur after internal data is stored; Meet REST API retrieves participants, recordings, or transcripts only when the artifact exists and the granted OAuth scopes allow it. If Google fails, the synchronization state can be retried without deleting or rolling back the CampusMeet Meeting.

In-app notification is also the primary reminder record. A reminder may attempt email delivery through SES, but an email failure does not remove a notification already created. The Google adapter and verification with a real account still require further work in the shared environment.

### Transcript lifecycle

Each meeting has one canonical transcript within the current edit and approval scope. Partial segments support temporary display only, while final segments are persisted in a stable order. After a live session, the producer moves through finalization and reaches `READY` or records `FAILED` when processing cannot complete.

An active member may read the permitted transcript. Editing and approval require the meeting organizer or an active group administrator. Each edit increments the current version, and approval refers to the exact version reviewed by the user. Approved content must be frozen as an immutable source before knowledge processing so downstream AI cannot read newer content while labeling it as an earlier approved version.

### Knowledge and AI assistance

A document, minutes record, or transcript becomes a knowledge source only after the appropriate approval. The source retains metadata for the group, meeting, content type, version, and approval state. Retrieval filters access and source scope before any content is passed to the model.

AI may support citation-grounded questions and answers, summaries, minutes drafts, or task proposals. The result remains a draft. A task proposal becomes a Task only after preview and confirmation by an authorized user; idempotency and a transaction prevent retries from creating duplicate tasks. These flows have source and tests in selected areas, while audio processing and complete cloud end-to-end verification still require further work.

## Expected outcome

A meeting has a clear result when members can identify what was agreed, who is responsible, when work is due, and where progress will be tracked.
