---
title: "Week 7 Worklog"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives

- Standardize the authentication configuration used by the team.
- Finalize the shared DynamoDB data foundation.
- Align the infrastructure definitions, access permissions, and project documentation.
- Continue building the core CampusMeet features.

### Completed Tasks

| Day | Task Details | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Monday | Updated the instructions for retrieving the Cognito User Pool ID, Client ID, and API URL.<br>Standardized the frontend configuration and clarified feature ownership across the team.<br>Replaced the 17-table DynamoDB plan with a five-table physical model designed around application access patterns.<br>Updated CloudFormation, IAM permissions, PK/SK conventions, GSIs, TTL settings, validation scripts, and deployment documentation.<br>Ran infrastructure validation, linting, type checking, tests, and the production build. | 27/07/2026 | 27/07/2026 | <https://github.com/Ngct253/CampusMeet> |
| Friday | Aligned the architecture documentation, team plan, and frontend environment setup.<br>Completed the foundational group, membership, invitation, authorization, notification, and dashboard features.<br>Implemented meeting creation, viewing, updating, and cancellation.<br>Added group-level authorization checks and persisted meeting data in the `meeting-data` table.<br>Refined the scheduling and meeting detail interfaces.<br>Ran the test suite, linting, type checking, and production build. | 31/07/2026 | 02/08/2026 | <https://github.com/Ngct253/CampusMeet> |

### Week 7 Achievements

- Completed the shared Amazon Cognito configuration guide for the team.
- Defined and locally validated the five-table DynamoDB model.
- Aligned the CloudFormation templates, IAM permissions, and deployment documentation.
- Completed the core CampusMeet foundation features.
- Implemented the meeting management flow for creating, viewing, updating, and cancelling meetings.
- Refined the scheduling and meeting detail user interfaces.
- Completed linting, type checking, automated tests, and build verification.
