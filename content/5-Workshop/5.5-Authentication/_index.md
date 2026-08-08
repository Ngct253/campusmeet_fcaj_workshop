---
title: "Frontend, Meeting Workflow, and Integrations"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Connecting the frontend to authentication and the API

The React interface uses the environment configuration prepared in Section 5.2 to connect to the correct Cognito and API resources. The authentication module manages registration, account confirmation, sign-in, and session recovery, while the API client attaches the JWT centrally instead of leaving token handling to individual screens.

Protected routes require a valid session. A signed-out error should remain distinct from a signed-in user lacking permission; after a page reload, the interface should restore the session and real data instead of relying only on temporary browser state. This section focuses on interface behavior; Section 5.4 already explains server-side JWT validation and authorization.

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

![Minutes, decisions, and post-meeting work in CampusMeet](images/5-Workshop/campusmeet-evidence/meeting-minutes.png)

*The minutes screen links the summary, decisions, and follow-up work. The screenshot contains development-environment demonstration data rather than a required template for every meeting.*

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

In the development environment, the `campusmeet-uploads-dev` bucket blocks all public access and uses default encryption with Amazon S3 managed keys. These settings protect the storage layer; the backend must still authorize every upload or download URL it issues.

![Public access is blocked for the CampusMeet upload bucket](images/5-Workshop/campusmeet-evidence/s3-block-public-access.png)

*Block Public Access is enabled for the user-content bucket.*

![Default encryption for the upload bucket](images/5-Workshop/campusmeet-evidence/s3-default-encryption.png)

*New objects receive SSE-S3 server-side encryption. Storage configuration does not replace application-layer authorization checks.*

### Google and reminders

The CampusMeet Meeting remains the primary record. Under the current design, Calendar API synchronization and the Google Meet link occur after internal data is stored; Meet REST API retrieves participants, recordings, or transcripts only when the artifact exists and the granted OAuth scopes allow it. If Google fails, the synchronization state can be retried without deleting or rolling back the CampusMeet Meeting.

In-app notification is also the primary reminder record. A reminder may attempt email delivery through SES, but an email failure does not remove a notification already created. The Google adapter and verification with a real account still require further work in the shared environment.

![Upcoming-meeting notification in CampusMeet](images/5-Workshop/campusmeet-evidence/reminder-notification.png)

*The “meeting starts soon” notification confirms that a reminder reached the interface. It does not independently prove SES delivery or every scheduler invocation.*

A controlled check created a meeting and recorded a Google synchronization request. The interface then moved to a reconnect-required state because the test account was not included in the OAuth Test users list, so no Google Meet URL was created. This result shows that CampusMeet preserved the internal meeting and surfaced the integration failure, but it is not considered a complete Google flow.

![Meeting details with Google reconnect-required state](images/5-Workshop/campusmeet-evidence/google-reconnect-state.png)

*The meeting remains available in CampusMeet while Google synchronization is incomplete, allowing the user to understand the state and reconnect.*

### Transcript lifecycle

Each meeting has one canonical transcript within the current edit and approval scope. Partial segments support temporary display only, while final segments are persisted in a stable order. After a live session, the producer moves through finalization and reaches `READY` or records `FAILED` when processing cannot complete.

An active member may read the permitted transcript. Editing and approval require the meeting organizer or an active group administrator. Each edit increments the current version, and approval refers to the exact version reviewed by the user. Approved content must be frozen as an immutable source before knowledge processing so downstream AI cannot read newer content while labeling it as an earlier approved version.

### Knowledge and AI assistance

A document, minutes record, or transcript becomes a knowledge source only after the appropriate approval. The source retains metadata for the group, meeting, content type, version, and approval state. Retrieval filters access and source scope before any content is passed to the model.

In the development environment, retrieval and generation use two Regions. Approved sources are normalized under the `kb/` prefix in S3 in Singapore; Bedrock Knowledge Base uses Cohere Embed Multilingual v3 and S3 Vectors for indexing, while the AI Worker is configured to call Bedrock Mantle in N. Virginia with `openai.gpt-oss-20b`. Only authorization-filtered context proceeds to generation.

```text
AIJob in DynamoDB
  → Step Functions manages state and retries
  → AI Worker normalizes or retrieves an approved source
  → Knowledge Base and S3 Vectors find relevant passages
  → validate group, meeting, source, and version
  → Bedrock Mantle generates an answer or draft
  → the interface shows sources for user review and confirmation
```

![State machine coordinating CampusMeet AI jobs](images/5-Workshop/campusmeet-evidence/ai-state-machine.png)

*The `campusmeet-dev-ai-jobs` state machine is `Active`. This state proves that the orchestration resource exists; it does not prove that every execution or AI result is correct.*

AI may support citation-grounded questions and answers, summaries, minutes drafts, or task proposals. The result remains a draft. A task proposal becomes a Task only after preview and confirmation by an authorized user; idempotency and a transaction prevent retries from creating duplicate tasks. These flows have source and tests in selected areas, while audio processing and complete cloud end-to-end verification still require further work.

![Meeting-scoped AI assistant interface](images/5-Workshop/campusmeet-evidence/ai-assistant-interface.png)

*The assistant interface lets a user ask questions from group sources or summarize a selected section. Its presence confirms that the experience has been built, while retrieval quality, citations, and responses remain separate verification concerns.*

## Expected outcome

A meeting has a clear result when members can identify what was agreed, who is responsible, when work is due, and where progress will be tracked.
