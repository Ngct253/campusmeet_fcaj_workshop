---
title: "Workshop"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

## Introducing CampusMeet

CampusMeet helps study groups and small project teams manage information before, during, and after meetings in one place. Instead of separating schedules, documents, minutes, and follow-up work across different tools, CampusMeet connects them to the relevant group and meeting.

This workshop presents CampusMeet from both product and implementation perspectives. Alongside the meeting journey, it explains the authentication and data foundations addressed during the internship: IAM, CloudFormation, Amazon Cognito, API Gateway, Lambda, and DynamoDB. Details are selected to show how the system is built without replacing the complete API documentation or deployment runbook.

### Objectives

After reading the workshop, readers should be able to:

- Explain the problem and objectives that CampusMeet addresses.
- Follow the journey from creating a group to completing a meeting and tracking work.
- Understand the purpose of the AWS components and the boundaries between deployment stacks.
- Understand how IAM, CloudFormation, Cognito, the API, and DynamoDB cooperate in authentication and persistence.
- Understand how to prepare the environment, connect the frontend to AWS, and run quality checks before deployment.
- Distinguish existing functionality from items requiring further verification or development.
- Recognize principles for access control, information protection, and human review of AI suggestions.

### Architecture overview

![CampusMeet high-level architecture](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

The diagram shows a simple flow: users access CampusMeet, sign in, perform actions through the central application, and store information with optional support from calendar, notification, transcription, and AI services. Some branches represent the target architecture and are not presented as complete features.

### Workshop contents

1. [CampusMeet overview and scope](5.1-Workshop-overview/)
2. [Environment preparation and deployment architecture](5.2-Prerequiste/)
3. [IAM, CloudFormation, and AWS configuration](5.3-Architecture/)
4. [Cognito, API, and data foundation](5.4-IAM/)
5. [Frontend, meeting workflow, and integrations](5.5-Authentication/)
6. [Verification, operations, and evaluation](5.6-Data-foundation/)

The six sections keep the workshop concise while making the authentication and DynamoDB implementation work visible. Supporting integrations remain described at their verified level rather than being presented as more complete than the available evidence.
