---
title: "High-level Architecture"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## CampusMeet architecture

![CampusMeet high-level architecture diagram](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

The CampusMeet architecture contains six main parts:

1. **User experience:** where users sign in, manage groups, view meetings, and update work.
2. **Identity and access protection:** confirms who the user is before allowing an action.
3. **Central processing:** receives requests, checks permissions, and applies CampusMeet rules.
4. **Storage:** keeps group, meeting, task, and file information.
5. **Supporting services:** connect calendars, send notifications, transcribe content, or provide AI suggestions when appropriate.
6. **Operational visibility:** supports monitoring of errors, service health, and cost.

## A simple information flow

When a member opens a meeting, CampusMeet first confirms the account and access to the group. It then retrieves the appropriate information and displays it. When a user updates minutes or a task, the change is stored so other authorized members can follow it.

Documents are uploaded to private file storage rather than placed directly inside meeting records. CampusMeet links each file to the correct group and meeting.

## Extended content processing

Reviewed or approved documents and transcripts can become knowledge sources while retaining their group, meeting, and source-version context. AI answers and drafts remain limited by user access, include citations, and require confirmation before becoming official minutes or tasks.

## Current implementation status

| Scope | Status at the workshop milestone |
| --- | --- |
| Accounts, groups, invitations, meetings, and notifications | The product foundation and primary user journeys are available |
| Meeting forms, documents, minutes, and tasks | Several interface, workflow, and related test areas exist; end-to-end verification should continue in the shared environment |
| Google Calendar and Google Meet | Connection, synchronization, and local verification flows exist; complete AWS and browser verification with a real account is still required |
| Transcripts | Reading, pagination, editing, approval, and handoff of approved sources are available in selected areas; complete audio processing and cloud end-to-end verification still require further work |
| Knowledge and AI assistance | Approved-source ingestion, citation-grounded questions and answers, content summaries, minutes/task drafts, group progress analysis, and related tests exist; cloud end-to-end verification is still required before operational readiness |
| Monitoring and cost awareness | Included in the architecture and infrastructure and should continue as usage expands |

Arrows in the diagram show how the components work together, while the table states what has been implemented and what still requires verification.
