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

## Information managed by CampusMeet

CampusMeet connects user profiles, groups, members, meetings, documents, minutes, and tasks. Each meeting belongs to a group; documents and minutes belong to a meeting; follow-up work has an owner, due date, and status.

Documents remain in private storage and are visible only to authorized users. For minutes, transcripts, and AI-assisted content, the system must distinguish drafts from user-confirmed information.

## Reporting principle

The presence of a screen does not prove that the complete feature behind it is finished. A locally working flow may also require verification in a shared environment. The report therefore records only what has been observed and clearly identifies remaining work.
