---
title: "Meetings, Minutes, and Tasks"
date: 2026-08-08
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Meetings, Minutes, and Tasks

This is the main CampusMeet workflow after a group has been created.

## Meeting management

Authorized users can create meetings, choose time, agenda, and attendees, then update or cancel the meeting when needed. CampusMeet stores the meeting first; Google Calendar synchronization is optional and handled separately.

## Minutes

After the meeting, users can save:

- a short summary;
- decisions;
- action items;
- assignees and due dates.

Minutes remain attached to the Meeting so follow-up work keeps its original context.

## Action items and tasks

```text
Meeting
  ↓
Minutes
  ↓
Action Item
  ↓
Task
  ↓
TODO → DOING → DONE
```

An Action Item can be converted into an official Task and assigned to a valid group member.

## Dashboard

The Dashboard summarizes task status so each user can quickly see what still needs attention and what has already been completed.

The production E2E uses this complete flow to prove that data moves from Meeting to Minutes to Task and remains available after reload.

Live recording and transcription are not required for this core workflow.