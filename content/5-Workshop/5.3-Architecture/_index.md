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

## Existing and target areas

| Scope | Meaning at the workshop milestone |
| --- | --- |
| Accounts, groups, invitations, meetings, and notifications | Existing product foundation |
| Meeting forms, documents, minutes, and tasks | Implementation and related tests exist in selected areas; each flow still needs appropriate verification |
| Google Calendar and Google Meet | An integration direction exists, with selected parts verified locally |
| Transcription and AI | Evolving areas that are not presented as complete in a realistic environment |
| Monitoring and cost awareness | Ongoing requirements as the system expands |

Arrows in the diagram show intended connections. They do not mean that every branch has reached the same level of completion.
