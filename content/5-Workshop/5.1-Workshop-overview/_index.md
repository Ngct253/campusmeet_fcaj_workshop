---
title: "CampusMeet Overview"
date: 2026-08-08
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## What is CampusMeet?

CampusMeet is a meeting-management platform for study groups and small project teams. It brings together information that is often scattered across different tools, including members, schedules, documents, discussion notes, minutes, and follow-up tasks.

CampusMeet does not replace video-conferencing software. Meetings may still take place through Google Meet or in person. CampusMeet helps the team prepare, record outcomes, and continue tracking work after the meeting ends.

## The problem

Group information is often split across a calendar, shared drives, chat messages, and separate task lists. Team members may struggle to identify the latest information, understand who owns a task, or recall what was agreed.

CampusMeet connects this information through one continuous journey:

1. Create a group and invite members.
2. Schedule a meeting and prepare its content.
3. Share relevant documents.
4. Record minutes, decisions, and action items.
5. Track progress after the meeting.
6. Retrieve previous information when needed.

## Scope-definition approach

CampusMeet is scoped from the meeting journey rather than a technology list. The essential flow must help a team bring members together, organize a meeting, preserve outcomes, and track follow-up work. Calendar synchronization, transcription, and AI are added later to reduce repeated work or support retrieval. If an integration is unavailable, the product foundation still supports the meeting workflow instead of depending entirely on Google services, audio processing, or AI.

## Implementation direction

CampusMeet is developed as complete functional slices instead of building every screen first and connecting the backend later. For each scope, the team identifies the user journey, data contract, access rules, processing boundary, persistence, and relevant tests. A capability is therefore assessed through the complete path of its data rather than only by whether a screen exists.

The general sequence contains four layers:

1. Standardize the repository, shared types, and quality process.
2. Establish identity, API, and data foundations with consistent access boundaries.
3. Complete the core journey from groups and meetings to minutes and tasks.
4. Add upload, Google synchronization, transcription, and AI as recoverable supporting branches with their own states.

This sequence allows advanced capabilities to continue evolving without changing the meaning of meeting information already stored in CampusMeet.

## Success criteria

CampusMeet is not evaluated only by the number of screens. A journey is valuable when members receive enough information to prepare, post-meeting outcomes identify decisions, owners, and due dates, records remain traceable to the correct group and meeting, and unauthorized people cannot access the content. Advanced capabilities are successful when they reduce effort or improve retrieval without removing the user's responsibility to confirm results.

## How the parts connect

A group is the shared workspace and access boundary. Meetings connect documents, transcripts, minutes, and tasks, allowing members to trace a task or decision back to its meeting.

## Main value

- **Centralized information:** group and meeting information stays in one place.
- **Clear ownership:** follow-up work has an owner and a visible status.
- **Traceable history:** decisions and meeting outcomes can be reviewed later.
- **Better collaboration:** members work from a shared source of information.
- **Controlled intelligent support:** AI suggestions remain drafts until a user reviews them.

## Current scope

CampusMeet has a foundation for accounts, groups, invitations, meetings, and notifications. Meeting forms, documents, minutes, and task workflows have also been built or refined in selected areas. Google integration, transcription, and AI have working flows and tests within defined scopes, but their level of cloud verification varies. The workshop therefore identifies both available capabilities and remaining verification.
