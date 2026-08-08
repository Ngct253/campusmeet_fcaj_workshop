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

## Why the architecture is organized this way

The architecture separates responsibilities so that supporting capabilities do not become mixed into the core meeting workflow. The interface focuses on the user experience; identity and central processing enforce access rules; business information and files are stored according to their different characteristics; and external integrations connect through clear boundaries. A change to calendar synchronization, transcription, or AI therefore does not change the meaning of groups, meetings, minutes, and tasks.

Large files and audio remain in private file storage, while CampusMeet manages only the references and context needed to connect them to the correct meeting. Longer work such as audio processing, document normalization, or content generation is tracked through visible states instead of forcing a user to wait on one screen. This approach requires careful state and failure handling, but it keeps the main journey responsive and allows a supporting operation to be retried after a temporary failure.

## Main architecture principles

- Sign-in verifies identity, while authorization is still checked for each group and meeting resource.
- Google Meet remains an external meeting service; CampusMeet manages the surrounding workflow and outcomes rather than building video conferencing.
- Meeting information, files, and AI-assisted content retain links to the group and original source for traceability.
- Only documents or transcripts approved through the appropriate flow become official knowledge sources.
- AI output remains a cited draft; an authorized user confirms changes to minutes or tasks.
- Errors, processing states, and cost require monitoring so advanced capabilities do not hide operational problems.
