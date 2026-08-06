---
title: "Groups, Memberships, and Invitations"
date: 2026-07-27
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# Groups, Memberships, and Invitations

## Objectives

This section covers the core CampusMeet collaboration flow: creating groups, listing a user's groups, viewing members, updating group details, creating invitations, accepting or declining invitations, and managing related notifications.

The implementation uses:

- `campusmeet-dev-collaboration` for groups, memberships, invitations, and audit events.
- `campusmeet-dev-identity` for user profiles and notifications.
- Cognito JWT claims to identify the current user.
- The shared roles `GROUP_ADMIN` and `MEMBER`.

## Group Roles

| Role | Main permissions |
| --- | --- |
| `GROUP_ADMIN` | View and update the group, manage invitations, and perform administrative operations |
| `MEMBER` | View groups they belong to and use member-level features |

A group must retain at least one active Group Admin. The current member-removal API rejects removal of any `GROUP_ADMIN` membership.

## 1. Create a Group

Route:

```http
POST /groups
```

The request requires a valid JWT. The repository creates a group through one DynamoDB transaction:

1. Group metadata:

   ```text
   PK=GROUP#<groupId>
   SK=META
   ```

2. Membership for the creator with role `GROUP_ADMIN`:

   ```text
   PK=GROUP#<groupId>
   SK=MEMBER#<userId>
   ```

3. A `GROUP_CREATED` audit event.

The `groupId` is derived deterministically from the user and `Idempotency-Key`, so a retried request does not create several equivalent groups.

Example request body:

```json
{
  "name": "CampusMeet Project Team",
  "description": "Team for project development and testing"
}
```

Do not send `createdBy`, `role`, or `userId` and expect the backend to trust them. Identity and ownership come from the JWT and server-side rules.

## 2. List the Current User's Groups

Route:

```http
GET /groups
```

Membership items use:

```text
GSI1PK=USER#<userId>
GSI1SK=GROUP#<joinedAt>#<groupId>
```

The repository queries `GSI1` and then retrieves the matching group metadata. It does not scan the collaboration table.

Each group summary includes the current user's role and join time.

## 3. View Group Details and Members

Route:

```http
GET /groups/:groupId
```

The backend:

1. Loads group metadata.
2. reads the viewer's membership with `GetItem`.
3. Rejects the request when the membership does not exist or is inactive.
4. Queries items whose sort key begins with `MEMBER#`.
5. Batch-loads user profiles from the identity table when available.

An authenticated user who is not a member must not receive the group's member list.

## 4. Update a Group

Route:

```http
PATCH /groups/:groupId
```

Only a `GROUP_ADMIN` can update the group name and description. The update also writes a `GROUP_UPDATED` audit event.

Example:

```json
{
  "name": "CampusMeet Project Team",
  "description": "Team building the meeting management application"
}
```

The backend checks the stored membership role rather than a client-provided role.

## 5. Remove a Member

Route:

```http
DELETE /groups/:groupId/members/:userId
```

Current rules:

- The caller must be a Group Admin.
- Only a membership with role `MEMBER` can be removed through this route.
- The API rejects removal of any `GROUP_ADMIN` membership.
- The change is recorded in audit data.

This prevents accidental removal of every administrator from a group.

## 6. Create an Invitation

Route:

```http
POST /groups/:groupId/invitations
```

Only a Group Admin can create an invitation.

Example request:

```json
{
  "email": "member@example.com"
}
```

The repository:

1. Trims and lowercases the email address.
2. Checks that the same group does not already have a valid `PENDING` invitation for that email.
3. Generates a random 32-byte token.
4. Stores only the SHA-256 hash of the token in DynamoDB.
5. Sets a seven-day expiration.
6. Writes a `MEMBERSHIP_INVITED` audit event.
7. Creates an in-application notification in the same transaction when the email already belongs to a CampusMeet profile.

Invitation key:

```text
PK=GROUP#<groupId>
SK=INVITE#<invitationId>
```

Indexes:

```text
GSI1PK=EMAIL#<normalizedEmail>
GSI1SK=INVITE#<expiresAt>#<invitationId>

GSI2PK=TOKEN#<tokenHash>
GSI2SK=INVITE#<invitationId>
```

The raw token is not stored in a notification. The notification links to:

```text
/app/invitations?invitationId=<invitationId>
```

## 7. Invitation States

| State | Meaning |
| --- | --- |
| `PENDING` | Awaiting a response and not expired |
| `ACCEPTED` | The recipient joined the group |
| `DECLINED` | The recipient declined |
| `EXPIRED` | The invitation expired |
| `REVOKED` | A Group Admin revoked the invitation |

TTL on `expiresAtEpoch` supports eventual cleanup, but application logic must still validate status and expiration time before processing a response.

## 8. List and Revoke Group Invitations

Routes:

```http
GET  /groups/:groupId/invitations
POST /groups/:groupId/invitations/:invitationId/revoke
```

Both operations require the `GROUP_ADMIN` role.

Revocation changes the status to `REVOKED`, invalidates the old token, and records an audit event.

## 9. User Invitation Inbox

Route:

```http
GET /invitations
```

The backend reads the verified Cognito email and queries invitations by normalized email. A user sees only invitations sent to the email associated with their account.

CampusMeet supports two response paths.

In-application response by invitation ID:

```http
POST /invitations/by-id/:invitationId/accept
POST /invitations/by-id/:invitationId/decline
```

Token-based fallback:

```http
GET  /invitations/:token
POST /invitations/:token/accept
POST /invitations/:token/decline
```

The backend still compares the signed-in user's verified email with the invitation email. Possessing a token does not allow one account to accept an invitation for another person.

## 10. Accept an Invitation

Acceptance requires the invitation to be:

- `PENDING`.
- Not expired.
- Addressed to the current user's verified email.
- Not already represented by an active duplicate membership.

A successful transaction updates the invitation to `ACCEPTED`, creates a `MEMBER` membership, and writes an audit event. These related changes are atomic so the system does not leave a partially updated invitation flow.

## 11. Decline an Invitation

Declining changes the invitation from `PENDING` to `DECLINED` and does not create a membership. A declined invitation is not reused as a new invitation. A Group Admin must create another invitation when the user should be invited again.

## 12. Invitation Notifications

When the invitation email already belongs to a CampusMeet profile, the system creates an `INVITATION` notification in the identity table:

```text
PK=USER#<userId>
SK=NOTIFICATION#<createdAt>#invitation-<invitationId>
```

Unread notifications use:

```text
GSI2PK=USER#<userId>#UNREAD
```

After acceptance or decline, the corresponding `invitation-<invitationId>` notification is marked as read. Invitation processing does not fail only because an old notification is missing.

Notification routes:

```http
GET  /notifications
POST /notifications/:notificationId/read
```

## 13. Test the Collaboration Flow

Prepare two confirmed accounts:

- Account A: group creator.
- Account B: invite recipient.

Test sequence:

1. A signs in and creates a group with an `Idempotency-Key`.
2. A views the group and confirms the `GROUP_ADMIN` role.
3. A invites B's email.
4. A views the group's invitation list.
5. B signs in and opens the invitation inbox.
6. B accepts the invitation.
7. B appears in the member list with role `MEMBER`.
8. B's invitation notification becomes read.
9. B attempts to update the group and is rejected.
10. Retry a relevant create request to verify duplicate protection.

Failure cases to test:

- A non-member reads group details.
- A normal member creates an invitation.
- The same email is invited while a valid invitation is still pending.
- An expired or revoked invitation is accepted.
- A different email account attempts to accept the invitation.
- A Group Admin is removed through the member-removal route.

## Expected Result

- Group creation atomically creates the Group Admin membership and audit event.
- Users can list and view only groups they belong to.
- Only Group Admins update groups, manage invitations, and remove normal members.
- Invitations use normalized email, hashed tokens, explicit expiration, and clear states.
- Acceptance creates membership atomically.
- Notifications open the correct invitation and never contain the raw token.
