---
title: "Workshop"
date: 2026-07-27
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building and Deploying the CampusMeet Meeting Management Platform on AWS

## Overview

CampusMeet is a platform for managing the activities that take place before, during, and after meetings for study groups, student projects, and small collaborative teams.

The system brings user management, groups, memberships, invitations, meetings, notifications, minutes, and follow-up work into one application. CampusMeet uses managed AWS services to reduce server administration, scale with demand, and align operating costs with actual usage.

This workshop covers the design and deployment of the main CampusMeet components, including:

- User authentication with Amazon Cognito.
- API protection with an Amazon API Gateway JWT authorizer.
- Application logic implemented with AWS Lambda.
- A five-table Amazon DynamoDB data model.
- Group, membership, invitation, and notification workflows.
- Meeting management and related data.
- Integration between the web application and backend services.
- Infrastructure management with AWS SAM and AWS CloudFormation.
- Logging, monitoring, and operational visibility with Amazon CloudWatch.
- IAM permissions and security controls.
- End-to-end testing.
- Cost control and resource cleanup.

## Problem Statement

Meeting information is often scattered across messaging applications, calendars, documents, and task-management tools. This makes it difficult for teams to manage membership, enforce access rules, track meeting records, and maintain follow-up work.

CampusMeet addresses this problem through a centralized system in which users are authenticated, application data is authorized within group boundaries, and the activities before, during, and after a meeting are managed as one connected workflow.

## High-Level Architecture

![CampusMeet AWS Architecture](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png)

CampusMeet uses the following core AWS services:

| Service | Role |
| --- | --- |
| Amazon Cognito | User authentication and identity management |
| Amazon API Gateway | HTTP APIs and JWT validation |
| AWS Lambda | Application and business logic |
| Amazon DynamoDB | Application data stored in a five-table model |
| Amazon S3 | Files, audio recordings, and large content |
| AWS Step Functions | Orchestration of asynchronous processing |
| Amazon Transcribe | Speech-to-text processing |
| Amazon Bedrock | AI-assisted processing and retrieval |
| Amazon CloudWatch | Logging, monitoring, and alerts |
| AWS IAM | Access control for users and AWS resources |
| AWS SAM and CloudFormation | Infrastructure definition and deployment |

## Workshop Contents

1. [CampusMeet Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [System Architecture](5.3-Architecture/)
4. [IAM and Environment Configuration](5.4-IAM/)
5. [Authentication with Amazon Cognito](5.5-Authentication/)
6. [DynamoDB Data Foundation](5.6-Data-foundation/)
7. [API Gateway and AWS Lambda](5.7-Api-lambda/)
8. [Groups, Memberships, and Invitations](5.8-Collaboration/)
9. Meeting Management
10. Minutes and Follow-up Work
11. Frontend Integration
12. Meeting Transcripts and Asynchronous Processing
13. AI Integration
14. Monitoring and Security
15. End-to-End Testing
16. Cost Control
17. Resource Cleanup
