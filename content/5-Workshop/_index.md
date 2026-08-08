---
title: "Workshop"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

## Introducing CampusMeet

CampusMeet helps study groups and small project teams manage information before, during, and after meetings in one place. Instead of separating schedules, documents, minutes, and follow-up work across different tools, CampusMeet connects them to the relevant group and meeting.

This workshop presents CampusMeet from a product perspective, focusing on the problem, objectives, high-level architecture, current features, and meeting journey. Source code, commands, configuration values, and API paths have been removed.

### Objectives

After reading the workshop, readers should be able to:

- Explain the problem and objectives that CampusMeet addresses.
- Follow the journey from creating a group to completing a meeting and tracking work.
- Understand the purpose of the AWS components in the diagram at a high level.
- Distinguish existing functionality from items requiring further verification or development.
- Recognize principles for access control, information protection, and human review of AI suggestions.

### Architecture overview

![CampusMeet high-level architecture](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

The diagram shows a simple flow: users access CampusMeet, sign in, perform actions through the central application, and store information with optional support from calendar, notification, transcription, and AI services. Some branches represent the target architecture and are not presented as complete features.

### Workshop contents

1. [CampusMeet overview](5.1-Workshop-overview/)
2. [Objectives and access](5.2-Prerequiste/)
3. [High-level architecture](5.3-Architecture/)
4. [Current feature scope](5.4-IAM/)
5. [Meeting workflow](5.5-Authentication/)
6. [Evaluation and next steps](5.6-Data-foundation/)

Together, these six sections provide a complete product story rather than a technical installation guide.
