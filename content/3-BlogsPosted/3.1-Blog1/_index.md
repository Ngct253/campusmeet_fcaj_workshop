---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# CAMPUSMEET – MEETING MANAGEMENT AND KNOWLEDGE RETRIEVAL WITH AWS SERVERLESS & GENERATIVE AI

Starting from a common problem: after every meeting, information is often scattered across multiple places — calendars in Calendar, meetings on Google Meet, documents in Drive, meeting minutes written separately, and follow-up tasks managed in another tool.

From that problem, we built **CampusMeet** to manage the end-to-end meeting lifecycle (before, during, and after meetings), while turning meeting data into a searchable knowledge base powered by AI. CampusMeet targets study groups, project teams, and small project groups.

Instead of building a new video calling platform, CampusMeet focuses on post-meeting value: agenda, participants, reminders, transcripts, minutes, action items, tasks, and knowledge retrieval, while integrating Google Calendar/Google Meet for conferences.

Key AWS Serverless highlights:
- **End-to-End Meeting Lifecycle**: Amazon Cognito for authentication, API Gateway + AWS Lambda for business logic, and DynamoDB storing groups, meetings, minutes, tasks, and AI jobs across 5 physical tables designed by access patterns.
- **Google Calendar/Meet Integration with Fallbacks**: Internal meeting data is independent of external artifacts.
- **Asynchronous Data Uploads & Processing**: Direct S3 Presigned URL uploads for large files/audio, processed asynchronously via Step Functions and AI jobs.
- **Generative AI with Amazon Bedrock & Multi-Meeting RAG**: Citation-backed QA across meetings filtered by group access control.
- **Human-in-the-Loop AI**: AI proposes minutes/action items, but humans must review and confirm before backend execution.
- **Operations & Cost Optimization**: Managed serverless services without EC2/RDS/NAT Gateway overhead, monitored via CloudWatch and defined with AWS SAM / CloudFormation IaC.

### Post Link on AWS Study Group
🔗 **Post Link**: [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2237833836981576](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2237833836981576)

### Publication Evidence
![Publication Evidence on AWS Study Group](images/3-BlogsPosted/3.1-Blog1/blog-evidence.png?v=2)