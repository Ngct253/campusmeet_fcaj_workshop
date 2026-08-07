---
title: "CampusMeet Overview"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# CampusMeet Overview

## Introduction

CampusMeet is a meeting and follow-up management platform for study groups, student projects, and small teams. The project brings identity, groups, invitations, meetings, minutes, tasks, documents, and AI-assisted workflows into one place so meeting information is not scattered across unrelated tools.

The current implementation focuses on:

- user authentication;
- group and membership management;
- invitations and notifications;
- meeting creation, updates, and cancellation;
- minutes, decisions, and action items;
- converting action items into tasks and tracking task status;
- attachment management;
- asynchronous Google Calendar/Meet synchronization;
- AI-assisted retrieval and drafting with source-aware citations.

CampusMeet does not build its own video-conferencing system. Google Calendar and Google Meet remain external services, while CampusMeet owns the application workflow and business data.

## Problem

Small teams often keep meeting information in several different systems: calendars, chat applications, standalone documents, and separate task trackers. That makes it harder to enforce access boundaries, recover the context of an old decision, and make sure follow-up work is not forgotten.

CampusMeet addresses this by linking groups, meetings, minutes, tasks, and knowledge sources under the same authorization model.

## Workshop goals

This workshop follows the current CampusMeet repository and covers the path from AWS foundations to a deployable E2E core workflow.

The main goals are to:

- configure Amazon Cognito and JWT-protected APIs;
- implement API logic with API Gateway and AWS Lambda;
- use a five-table DynamoDB data model;
- build the Group, Invitation, Meeting, Minutes, Task, and Dashboard flows;
- connect the React frontend to the deployed API;
- store user files in private Amazon S3;
- use EventBridge Scheduler and Step Functions for asynchronous work;
- integrate Google Calendar/Meet without making Google the source of truth;
- use Amazon Bedrock with authorization-aware retrieval and citations;
- monitor the system with CloudWatch and scoped IAM roles;
- run a real browser/AWS E2E test;
- understand the main cost drivers of the environment.

## High-level architecture

![CampusMeet AWS Architecture](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

| Component | Role |
| --- | --- |
| CampusMeet Web | React/Vite user interface |
| Amazon Cognito | Sign-up, account confirmation, and JWT issuance |
| Amazon API Gateway | HTTP API and JWT authorizer |
| AWS Lambda | Business logic and authorization checks |
| Amazon DynamoDB | Business data stored across five physical tables |
| Amazon S3 | User files and larger content |
| EventBridge Scheduler | Reminders and delayed retries |
| AWS Step Functions | AI jobs and longer-running orchestration |
| Amazon Bedrock | Generation, retrieval, and Knowledge Base integration |
| Amazon CloudWatch | Logs, metrics, and alarms |
| AWS IAM | Permissions between components |
| AWS SAM / CloudFormation | Infrastructure definition and deployment |

Amazon Transcribe remains a planned extension for speech processing. Live transcription and batch audio transcription are not treated as required parts of the current core production E2E flow.

## Data model

CampusMeet uses five DynamoDB tables:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

Binary files are stored in S3; DynamoDB keeps metadata, relationships, and workflow state.

## Authentication and authorization

CampusMeet separates identity from application permissions.

Cognito and API Gateway validate who the caller is. The backend still verifies group membership, role, and resource scope before reading or changing data. A valid JWT is therefore not permission to access every group.

## Main workflow

Before a meeting, users create a group, invite members, schedule a meeting, prepare agenda/attendees, and optionally synchronize the event with Google.

During a meeting, the current workshop focuses on the meeting workspace and stored content. Full live recording/transcription is an extension rather than a requirement of the current core E2E release.

After a meeting, users save minutes, record decisions, create action items, convert them into tasks, track progress, upload supporting documents, and optionally use those documents as AI knowledge sources after ingestion is ready.

## AI principles

AI output is treated as assistance rather than unquestioned business data:

- important generated content remains a draft until reviewed;
- retrieval is restricted to authorized groups and meetings;
- knowledge sources must be in a valid state before use;
- citations are checked against retrieved content;
- an AI task proposal is not automatically an official task.

## Implementation status vs. verification status

The workshop distinguishes three different facts:

1. **Implemented in source** — code exists in the repository.
2. **Covered by automated/local checks** — logic has tests or a successful local build.
3. **Verified in AWS/browser E2E** — the feature has actually been deployed and exercised against real services.

This distinction is intentional. A template or unit test is not presented as evidence that production integration has already succeeded.

## Result

After the workshop, learners should understand the CampusMeet architecture, deploy its serverless components, work with Cognito/API Gateway/Lambda/DynamoDB, connect the frontend, explain asynchronous and AI flows, inspect CloudWatch, run a real E2E test, and understand the environment's main cost controls.
