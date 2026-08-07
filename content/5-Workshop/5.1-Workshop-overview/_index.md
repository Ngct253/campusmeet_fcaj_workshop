---
title: "CampusMeet Overview"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# CampusMeet Overview

CampusMeet is a meeting and follow-up management platform for study groups, student projects, and small teams. It connects information that would otherwise be spread across calendars, documents, chat tools, and task trackers.

## The problem

Small teams often struggle with three things:

- meeting information is split across several tools;
- decisions and follow-up work are easy to lose;
- access control becomes inconsistent as more members participate.

CampusMeet connects these activities into one flow:

```text
Create group
  ↓
Invite members
  ↓
Create meeting
  ↓
Save minutes and action items
  ↓
Convert to tasks
  ↓
Track progress
```

Google Calendar/Meet, document storage, and AI are additional integrations around this core workflow.

## High-level architecture

![CampusMeet AWS Architecture](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

| Component | Role |
| --- | --- |
| React/Vite | User interface |
| Amazon Cognito | Sign-up and sign-in |
| API Gateway | HTTP API |
| AWS Lambda | Business logic and authorization |
| DynamoDB | Main application data |
| Amazon S3 | Files and larger content |
| EventBridge Scheduler / Step Functions | Asynchronous work |
| Amazon Bedrock | AI-assisted retrieval and drafting |
| CloudWatch | Logs and monitoring |

CampusMeet does not implement its own video-conferencing system. Google Calendar and Google Meet are external integrations.

## Workshop scope

The workshop focuses on the parts that directly support the final deployment:

- authentication and API;
- group-based data and authorization;
- Group, Invitation, Meeting, Minutes, Task, and Dashboard;
- a production frontend served through CloudFront;
- document upload and asynchronous processing;
- Google and AI integration where appropriate;
- monitoring, security, E2E testing, and cost control.

Live transcription, recording, and batch audio transcription are not required for the current core production E2E.

## Project status

The workshop distinguishes between:

1. implemented in source;
2. checked by automated tests or local builds;
3. verified through a real AWS/browser E2E run.

The final submission prioritizes the third level: a production URL and a core workflow that actually runs on AWS.