---
title: "Workshop"
date: 2026-07-27
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building and Deploying CampusMeet on AWS

## Overview

CampusMeet is a meeting and follow-up management platform for study groups, student projects, and small collaborative teams. It brings identity, groups, invitations, meetings, minutes, tasks, documents, and AI-assisted workflows into one application.

This workshop follows the current CampusMeet source code and takes the project from AWS foundations to a deployable end-to-end core flow. Features that exist only in source or have not yet been verified in AWS/browser testing are labeled accordingly instead of being presented as production-complete.

The main services used in the workshop include Amazon Cognito, API Gateway, AWS Lambda, DynamoDB, Amazon S3, EventBridge Scheduler, Step Functions, Amazon Bedrock, CloudWatch, and IAM. Google Calendar/Meet is treated as an external integration and verified separately.

## Workshop Contents

1. [CampusMeet Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [System Architecture](5.3-Architecture/)
4. [IAM and Environment Configuration](5.4-IAM/)
5. [Authentication with Amazon Cognito](5.5-Authentication/)
6. [DynamoDB Data Foundation](5.6-Data-foundation/)
7. [API Gateway and AWS Lambda](5.7-Api-lambda/)
8. [Groups, Memberships, and Invitations](5.8-Collaboration/)
9. [Meeting Management](5.9-Meeting-management/)
10. [Minutes and Follow-up Work](5.10-Minutes-tasks/)
11. [Frontend Integration](5.11-Frontend-integration/)
12. [Meeting Content and Asynchronous Processing](5.12-Async-content-processing/)
13. [AI Integration](5.13-AI-integration/)
14. [Monitoring and Security](5.14-Monitoring-security/)
15. [End-to-End Testing](5.15-End-to-end-testing/)
16. [Cost Control](5.16-Cost-control/)

The workshop ends with cost control. There is no separate resource-cleanup chapter because the CampusMeet environment remains available for later deployment and E2E verification.
