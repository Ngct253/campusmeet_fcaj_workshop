---
title: "Data and Group Collaboration"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Data and Group Collaboration

CampusMeet uses DynamoDB as its main application database. This workshop focuses on how the data is separated by purpose rather than documenting every key and index.

## Main data areas

| Table | Main content |
| --- | --- |
| `identity` | user profiles and notifications |
| `collaboration` | groups, members, and invitations |
| `meeting-data` | meetings, minutes, and related content |
| `task-data` | tasks and task status |
| `ai-work` | AI jobs, knowledge sources, and citations |

The tables are managed through CloudFormation so development and production environments stay consistent.

## Groups and members

The group creator becomes `GROUP_ADMIN`. Invited users join as `MEMBER` after accepting an invitation.

- `GROUP_ADMIN` manages the group and invitations.
- `MEMBER` uses the group features available to normal participants.

The backend checks active membership before returning group data.

## Invitation flow

```text
Admin creates Group
  ↓
Invite member by email
  ↓
CampusMeet creates invitation
  ↓
Recipient signs in
  ↓
Accept / Decline
```

Notifications help the recipient find and respond to the invitation.

## Data principles

CampusMeet keeps a few important rules:

- group data is limited to authorized members;
- administrative actions require the appropriate role;
- large files are stored in S3 rather than DynamoDB;
- important writes are designed to avoid partial or duplicate data.

Low-level PK/SK, GSI, and transaction details remain implementation details and are intentionally omitted from the main workshop narrative.