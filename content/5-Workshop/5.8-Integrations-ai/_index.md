---
title: "Asynchronous Processing, Google, and AI"
date: 2026-08-08
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# Asynchronous Processing, Google, and AI

Some CampusMeet work should not be completed inside a single short HTTP request, especially when files, external services, or AI are involved.

## Files and Amazon S3

User documents are uploaded directly to a private S3 bucket using temporary upload URLs. The backend validates access and stores metadata while the file itself remains in S3.

## Google Calendar and Meet

When an organizer connects Google, CampusMeet can synchronize meeting information with Google Calendar and receive Meet information when available.

```text
Meeting changes
  ↓
CampusMeet stores data
  ↓
Google sync worker
  ↓
Sync status is updated
```

A Google failure does not remove the Meeting stored in CampusMeet.

## AI with Amazon Bedrock

CampusMeet uses Bedrock to support features such as:

- meeting questions and search;
- group-level search;
- draft minutes;
- suggested follow-up actions.

Documents must complete ingestion before they are used for retrieval. Important AI output remains a draft until a user reviews it.

## Authorization and citations

AI retrieval is restricted to groups and meetings the current user is allowed to access. Source-based answers should include citations so the user can verify the supporting content.

If the available sources do not support an answer, the system should communicate missing context rather than inventing a confident response.

## Current scope

Live transcription, recording lifecycle, and batch audio transcription are not required for the current core production E2E.