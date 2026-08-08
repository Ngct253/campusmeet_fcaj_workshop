---
title: "Current Feature Scope"
date: 2026-08-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Feature overview

| Feature group | Scope | General status |
| --- | --- | --- |
| Accounts and profiles | Registration, sign-in, and personal information | Main foundation available |
| Groups and collaboration | Groups, members, invitations, and notifications | Core workflows are available |
| Meeting management | Create, view, update, cancel, and prepare meetings | The core workflow exists; selected authorization scenarios still require verification |
| Post-meeting content and work | Documents, minutes, transcript editing and approval, action items, tasks, and progress tracking | Several interface and workflow areas exist; end-to-end testing should continue |
| Integrations and automation | Google Calendar/Meet synchronization, content upload, reminders, and email | Initial flows exist; selected areas are only locally verified or still require verification in a realistic environment |
| Knowledge and AI assistance | Ingesting approved sources, citation-grounded questions and answers, content summaries, minutes/task drafts, and group progress analysis | Selected flows and related tests exist; the scope is not presented as complete, and users must confirm all suggested content |

## Cross-cutting requirements

Every workflow should identify the actor, input information, resulting output, and next step. Access must be checked before content is displayed or changed. Drafts, confirmed information, and completed work need distinct states so members do not confuse them. When a data source or integration fails, the interface should explain what is unavailable and preserve the rest of the journey whenever possible.

## End-to-end journey across feature groups

An account identifies the actor, membership places that person in the correct group, and a meeting provides context for preparation and documents. Minutes and action items continue into tasks with owners, due dates, and states. Only documents or transcripts approved through the appropriate flow move into the knowledge process, and AI results return to user confirmation. If calendar or AI support is unavailable, the team must still be able to open a meeting, record its outcome, and track the resulting tasks.

## Information managed by CampusMeet

CampusMeet uses five physical DynamoDB tables organized around access patterns and data boundaries instead of creating one table per entity:

| Table | Main information |
| --- | --- |
| `identity` | Profiles, preferences, integration references, and notifications |
| `collaboration` | Groups, memberships, invitations, and audit events |
| `meeting-data` | Meetings, attendees, agenda, minutes, reminders, file metadata, and transcripts |
| `task-data` | Tasks, history, and views by assignee or meeting |
| `ai-work` | AI jobs, knowledge sources, conversations, citations, proposals, and idempotency |

Binary files and audio are not stored in DynamoDB. The objects remain in private S3 storage, while DynamoDB stores only the metadata and references required to identify the group, meeting, source, and state.

## Data-processing path

```text
Interface
  → API client attaches JWT
  → Handler receives the request
  → Application service enforces rules and access
  → Repository executes the access pattern
  → DynamoDB or S3
```

The frontend never accesses DynamoDB directly. Handlers also avoid assembling data queries independently for every case; services and repositories keep authorization rules, transactions, and mapping behavior testable in isolation.

## Collaboration and consistency

The group creator becomes a `GROUP_ADMIN`, while other members join only through a valid flow. Invitations carry states and expiry, and acceptance must match the intended account. The backend verifies active membership before reading or changing group-scoped information.

Several data principles apply throughout the product:

- retryable requests use idempotency to avoid duplicate results;
- concurrent updates use versions or conditional writes;
- multi-item changes that require atomicity use transactions;
- timestamps are stored in UTC and displayed in the user's timezone;
- TTL is reserved for temporary data and does not replace retention rules for primary content;
- normal business requests use defined access patterns or indexes rather than scanning an entire table.

These principles explain the implementation approach without reproducing every PK, SK, GSI, or transaction expression in the workshop.

## Reporting principle

The presence of a screen does not prove that the complete feature behind it is finished. A locally working flow may also require verification in a shared environment. The report therefore records only what has been observed and clearly identifies remaining work.
