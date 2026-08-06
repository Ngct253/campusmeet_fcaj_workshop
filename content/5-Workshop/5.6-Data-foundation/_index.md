---
title: "DynamoDB Data Foundation"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# DynamoDB Data Foundation

## Objectives

This section deploys the independent CampusMeet data stack, verifies the five DynamoDB tables, and explains why the physical model follows access requirements rather than creating one table for every business entity.

## Five-Table Model

`infra/data-foundation.yaml` creates exactly five tables:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

| Table | Main data |
| --- | --- |
| `identity` | User profiles, preferences, integration references, OAuth state, and notifications |
| `collaboration` | Groups, memberships, invitations, and audit events |
| `meeting-data` | Meetings, attendees, agendas, minutes, reminders, attachments, recordings, and transcripts |
| `task-data` | Tasks and task history |
| `ai-work` | AI jobs, knowledge sources, conversations, citations, and proposals |

Using five physical tables does not remove logical entities. Items are distinguished through `entityType`, `PK`, `SK`, and key prefixes.

Example keys:

```text
PK=GROUP#<groupId>       SK=META
PK=GROUP#<groupId>       SK=MEMBER#<userId>
PK=GROUP#<groupId>       SK=INVITE#<invitationId>

PK=MEETING#<meetingId>   SK=META
PK=MEETING#<meetingId>   SK=ATTENDEE#<userId>
```

## Shared Table Properties

All five tables use:

- String partition key `PK` and sort key `SK`.
- `PAY_PER_REQUEST` billing.
- Server-side encryption.
- TTL on `expiresAtEpoch`.
- Tags including `Project=CampusMeet`, `ManagedBy=CloudFormation`, and `DataModelVersion=2`.
- `DeletionPolicy: Retain` and `UpdateReplacePolicy: Retain`.
- Optional point-in-time recovery and deletion protection.

Secondary-index counts:

| Table | GSIs |
| --- | ---: |
| `identity` | 2 |
| `collaboration` | 2 |
| `meeting-data` | 3 |
| `task-data` | 3 |
| `ai-work` | 2 |

## 1. Validate the Data Template

```powershell
sam validate `
  --template-file infra/data-foundation.yaml `
  --lint `
  --region ap-southeast-1
```

Or use the project command:

```powershell
npm run sam:validate:data -- --region ap-southeast-1
```

Expected logical resources:

```text
IdentityTable
CollaborationTable
MeetingDataTable
TaskDataTable
AIWorkTable
```

## 2. Preview the Change Set

```powershell
sam deploy `
  --template-file infra/data-foundation.yaml `
  --stack-name campusmeet-dev-data `
  --resolve-s3 `
  --parameter-overrides `
    Environment=dev `
    TablePrefix=campusmeet-dev `
    EnablePointInTimeRecovery=false `
    EnableDeletionProtection=false `
  --no-execute-changeset `
  --region ap-southeast-1
```

Before execution, verify:

- Exactly five `AWS::DynamoDB::Table` resources.
- No unexpected delete or replacement action.
- Stable `campusmeet-dev` names.
- `PAY_PER_REQUEST` billing.
- `PK/SK` primary keys.
- Correct GSI counts.
- TTL on `expiresAtEpoch`.
- Encryption and retention policies.

Do not execute a change set that contains unreviewed actions.

## 3. Deploy the Data Stack

After review, execute the change set in CloudFormation or rerun the command without `--no-execute-changeset`:

```powershell
sam deploy `
  --template-file infra/data-foundation.yaml `
  --stack-name campusmeet-dev-data `
  --resolve-s3 `
  --parameter-overrides `
    Environment=dev `
    TablePrefix=campusmeet-dev `
    EnablePointInTimeRecovery=false `
    EnableDeletionProtection=false `
  --region ap-southeast-1
```

When deployment fails, inspect CloudFormation **Events** before retrying. Do not manually change the tables to bypass a template error.

## 4. Verify the Five Tables

```powershell
$AccountId = aws sts get-caller-identity --query Account --output text

powershell -NoProfile -File scripts/verify-data-foundation.ps1 `
  -Region ap-southeast-1 `
  -TablePrefix campusmeet-dev `
  -ExpectedAccountId $AccountId
```

The verification script checks table state, billing mode, primary keys, GSIs, TTL, and the `DataModelVersion=2` tag.

The equivalent project command is:

```powershell
npm run aws:verify:data -- `
  -Region ap-southeast-1 `
  -TablePrefix campusmeet-dev `
  -ExpectedAccountId $AccountId
```

## 5. Read CloudFormation Outputs

```powershell
aws cloudformation describe-stacks `
  --stack-name campusmeet-dev-data `
  --query "Stacks[0].Outputs" `
  --output table `
  --region ap-southeast-1
```

The stack exports the name and ARN of every table. Do not hard-code account IDs or table ARNs in application source code.

## 6. Data Design Rules

- Store timestamps in UTC ISO 8601 format.
- Store `groupId` on group-scoped items for authorization and auditing.
- Never trust a client-provided `userId`, `groupId`, or role.
- Use `GetItem` and `Query` for normal application requests; avoid `Scan`.
- Use conditional writes for state, version, and uniqueness checks.
- Use DynamoDB transactions when several items must change atomically.
- Use TTL only for temporary data such as OAuth state, expired invitations, idempotency records, or retained notifications.
- Store files and audio in S3; DynamoDB keeps metadata and references.

## 7. Access Examples

Group membership lookup:

```text
GetItem: PK=GROUP#<groupId>, SK=MEMBER#<userId>
Query:   PK=GROUP#<groupId>, begins_with(SK, "MEMBER#")
```

User group listing uses `GSI1`:

```text
GSI1PK=USER#<userId>
GSI1SK=GROUP#<joinedAt>#<groupId>
```

Invitation lookup uses normalized email or a token hash:

```text
GSI1PK=EMAIL#<normalizedEmail>
GSI2PK=TOKEN#<tokenHash>
```

The raw invitation token is not stored in an index or notification.

## 8. Stack Boundary and Data Safety

The data stack is independent from Cognito and the API stack. `infra/auth-integration.yaml` and `infra/template.yaml` reference the tables but do not recreate them.

Because the tables use `Retain` policies:

- Deleting the stack does not guarantee table deletion.
- Retained tables can continue to incur charges.
- Data and dependencies must be reviewed before manual deletion.
- The shared data stack must not be removed while other developers still use it.

## Expected Result

- `campusmeet-dev-data` deploys successfully.
- Exactly five tables are `ACTIVE` in `ap-southeast-1`.
- Keys, GSIs, TTL, tags, and billing mode pass the verification script.
- No table or index is created manually outside CloudFormation.
