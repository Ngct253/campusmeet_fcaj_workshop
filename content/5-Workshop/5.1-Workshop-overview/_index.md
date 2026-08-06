---
title: "CampusMeet Overview"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# CampusMeet Overview

## Introduction

CampusMeet is a meeting and teamwork management system for study groups, student projects, and small collaborative teams.

The system is designed to connect the activities that take place before, during, and after a meeting in one workspace:

- Creating groups and managing members.
- Sending and responding to group invitations.
- Scheduling, updating, and cancelling meetings.
- Managing agendas, attendees, and notifications.
- Recording minutes, decisions, and follow-up actions.
- Assigning work to responsible members and tracking progress.
- Supporting meeting transcripts, source-grounded questions, and draft content with citations.

CampusMeet does not provide its own video conferencing system and is not intended to replicate Google Meet. Google Calendar and Google Meet are external integrations, while CampusMeet manages the workflow, application data, and authorization rules surrounding each meeting.

## Problem Statement

Meeting information is often spread across several disconnected tools:

- Schedules are stored in calendar applications.
- Reminders and discussions take place in messaging platforms.
- Minutes are maintained in separate documents.
- Follow-up work is tracked in another system.
- Documents, transcripts, and decisions are difficult to associate with the correct meeting.

This fragmented workflow creates several problems:

- Team members cannot easily find a complete meeting record.
- Follow-up actions may be forgotten.
- Access control is inconsistent across tools.
- Team administrators cannot clearly track upcoming meetings, overdue work, or overall progress.
- Speech-processing and AI features may expose data when access boundaries are not enforced correctly.

CampusMeet addresses these issues through a centralized system in which groups, meetings, minutes, tasks, and supporting sources are connected. Every operation is governed by the user's active membership and role within the relevant group.

## Workshop Objectives

This workshop presents the design and deployment of CampusMeet on a serverless AWS architecture, covering authentication, data storage, application logic, operations, and resource cleanup.

The main objectives are to:

- Implement user registration, account confirmation, and sign-in with Amazon Cognito.
- Protect HTTP APIs with an Amazon API Gateway JWT authorizer.
- Process application logic with AWS Lambda.
- Enforce membership and role checks in the backend.
- Design Amazon DynamoDB data around defined access patterns.
- Implement group, membership, invitation, notification, and meeting workflows.
- Connect the React frontend to AWS backend services.
- Manage infrastructure with AWS SAM and AWS CloudFormation.
- Process files, audio, and transcripts through asynchronous workflows.
- Apply citations and user confirmation to AI-assisted output.
- Collect logs, failures, and operational signals with Amazon CloudWatch.
- Apply least-privilege IAM permissions.
- Run end-to-end tests, control costs, and clean up resources.

## Intended Audience

This workshop is suitable for:

- Students learning to build an end-to-end serverless AWS application.
- Developers with basic knowledge of web interfaces, backend services, and databases.
- Learners who want to use Amazon Cognito, API Gateway, Lambda, and DynamoDB in one practical project.
- Teams studying authentication, authorization, Infrastructure as Code, and asynchronous processing.

## High-Level Architecture

CampusMeet is organized into the following layers:

```text
User
  |
  v
CampusMeet Web
React + TypeScript + Vite
  |
  +--------------------+
  |                    |
  v                    v
Amazon Cognito    Amazon API Gateway
                       |
                       v
                  AWS Lambda
                       |
            +----------+----------+
            |                     |
            v                     v
      Amazon DynamoDB        Amazon S3
            |                     |
            |                     v
            |              Files and audio
            |
            v
     Application data

Asynchronous processing
  |
  +--- EventBridge Scheduler
  +--- AWS Step Functions
  +--- Amazon Transcribe
  +--- Amazon Bedrock
  +--- Bedrock Knowledge Bases / S3 Vectors

Operations
  |
  +--- Amazon CloudWatch
  +--- Amazon SNS
  +--- Amazon SES

Infrastructure
  |
  +--- AWS IAM
  +--- AWS SAM
  +--- AWS CloudFormation
```

## Core Components

| Component | Responsibility in CampusMeet |
| --- | --- |
| CampusMeet Web | Provides the user interface for groups, meetings, minutes, tasks, transcripts, and AI-assisted features |
| Amazon Cognito | Authenticates users and issues JWTs |
| Amazon API Gateway | Exposes HTTP APIs and validates JWTs before invoking Lambda |
| AWS Lambda | Executes application use cases and enforces resource-level authorization |
| Amazon DynamoDB | Stores application data in a five-table physical model |
| Amazon S3 | Stores files, audio, recordings, and other large content |
| EventBridge Scheduler | Runs one-time reminders at scheduled times |
| AWS Step Functions | Coordinates long-running and asynchronous processing |
| Amazon Transcribe | Converts speech into text |
| Amazon Bedrock | Supports source-grounded questions, summaries, and draft generation |
| Amazon CloudWatch | Collects logs, metrics, and operational information |
| AWS IAM | Restricts access for users and AWS resources |
| AWS SAM and CloudFormation | Define, validate, and deploy infrastructure |

## Data Model

CampusMeet uses five physical DynamoDB tables designed around application access patterns:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

| Table | Main data domains |
| --- | --- |
| `identity` | Users, preferences, integration references, and notifications |
| `collaboration` | Groups, memberships, invitations, and audit events |
| `meeting-data` | Meetings, attendees, agendas, minutes, transcripts, and related records |
| `task-data` | Tasks and task history |
| `ai-work` | AI processing jobs, knowledge sources, conversations, citations, and proposals |

Binary files and audio are not stored directly in DynamoDB. Large content is stored in Amazon S3, while DynamoDB stores metadata and application state.

## Authentication and Authorization

CampusMeet separates authentication from application authorization.

### Authentication

Amazon Cognito authenticates the user and issues a JWT. Amazon API Gateway validates the token before forwarding the request to Lambda.

### Application Authorization

A valid JWT does not grant access to all application data. The backend must also verify:

- That the user is an active member of the group.
- That the user has the required role for the requested operation.
- That the resource belongs to the correct group or meeting.
- That the user is allowed to read, create, update, or cancel the resource.

This separation prevents authenticated users from accessing information outside their authorized scope.

## Meeting Lifecycle

CampusMeet is designed around a complete meeting lifecycle.

### Before the Meeting

- Create a group and manage its members.
- Schedule a meeting.
- Select attendees.
- Prepare the agenda and supporting documents.
- Create reminder notifications.
- Synchronize a Google Calendar event and request a Google Meet link when the organizer has connected a Google account.

### During the Meeting

- Display meeting information and agenda items.
- Obtain consent before recording or processing speech.
- Produce a live meeting transcript.
- Store final transcript segments.
- Support authorized questions and catch-up summaries based on permitted sources.

### After the Meeting

- Complete the meeting minutes.
- Record decisions.
- Create follow-up action proposals.
- Require user review and confirmation before creating official tasks.
- Track assignees, due dates, and status.
- Support questions over authorized documents, transcripts, and minutes.

## AI Principles

CampusMeet applies the following controls to AI-assisted features:

- AI-generated content remains a draft until a user confirms it.
- Answers and summaries must include citations to supporting sources.
- Retrieval is limited to groups and meetings the user is authorized to access.
- AI-generated tasks or actions require preview and confirmation.
- The system does not infer the real identity of a speaker.
- AI cannot commit sensitive application changes without authorization and human approval.

## Workshop Outcomes

After completing the workshop, learners will be able to:

- Explain the overall CampusMeet architecture.
- Deploy serverless components with AWS SAM and CloudFormation.
- Configure authentication with Amazon Cognito.
- Protect APIs with JWT validation.
- Structure Lambda code around application use cases and data-access boundaries.
- Design DynamoDB data for known access patterns.
- Implement group, meeting, and task workflows.
- Connect a frontend application to AWS APIs and services.
- Process files and transcripts through asynchronous workflows.
- Apply citations and user confirmation to AI-assisted features.
- Inspect logs, metrics, and failures with CloudWatch.
- Apply IAM, data protection, and cost-control practices.
- Run end-to-end tests and clean up AWS resources.
