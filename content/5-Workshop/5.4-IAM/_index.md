---
title: "Cognito, API, and Data Foundation"
date: 2026-08-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Amazon Cognito configuration

The authentication stack creates a Cognito User Pool that uses email as the username and automatically requires email verification. The current password policy requires at least eight characters with lowercase, uppercase, numeric, and symbol characters. MFA is disabled in the current development configuration; this records the present state and is not a recommendation to omit MFA from a production environment.

The User Pool Client is intended for the web application and therefore does not generate a client secret in the browser. It allows SRP authentication and refresh tokens for session continuity, and user-existence error prevention is enabled. After account confirmation and successful sign-in, Cognito issues tokens for frontend API requests.

![Cognito User Pool owned by the CampusMeet authentication stack](images/5-Workshop/campusmeet-evidence/cognito-auth-user-pool.png)

*The `campusmeet-dev-auth-users` User Pool was matched against the physical resource in the `campusmeet-dev-auth` stack. This ID-based check prevents confusion with another User Pool that remains in the development account.*

## Connecting Cognito to API Gateway

The HTTP API uses a JWT authorizer by default. The authorizer checks two primary token values:

- the `issuer` must belong to the stack's Cognito User Pool;
- the `audience` must match the web application's User Pool Client.

The `/health` endpoint does not require sign-in so it can support service checks. Business routes under `/{proxy+}` use the authorizer, while `OPTIONS` requests remain separate for browser CORS handling. The frontend origin is supplied through `AllowedOrigin`, and permitted methods and headers are restricted in the template.

![JWT authorizer attached to the business route](images/5-Workshop/campusmeet-evidence/api-jwt-routes.png)

*The API Gateway configuration shows that `ANY /{proxy+}` is protected by `JWT Auth`, while `/health`, the Google callback, and `OPTIONS` are handled separately for their respective purposes.*

## From JWT to business authorization

API Gateway rejects invalid tokens before invoking Lambda. For a valid request, Lambda obtains the user identity from the token, maps it to the internal profile, and checks membership and role for the target resource. Cognito therefore performs authentication, while the backend performs authorization.

```text
Frontend signs in through Cognito
  → receives a JWT
  → sends an Authorization header to the HTTP API
  → API Gateway validates issuer and audience
  → Lambda resolves the user and checks access
  → the application layer reads or updates data
```

Authentication failures, denied access, invalid input, and system failures should return different statuses so the interface can guide the user toward the appropriate next action.

## DynamoDB foundation

CampusMeet uses five physical DynamoDB tables organized around access patterns and data boundaries instead of creating one table per entity:

| Table | Main information |
| --- | --- |
| `identity` | Profiles, preferences, integration references, and notifications |
| `collaboration` | Groups, memberships, invitations, and audit events |
| `meeting-data` | Meetings, attendees, agenda, minutes, reminders, file metadata, and transcripts |
| `task-data` | Tasks, history, and views by assignee or meeting |
| `ai-work` | AI jobs, knowledge sources, conversations, citations, proposals, and idempotency |

Binary files and audio are not stored in DynamoDB. The objects remain in private S3 storage, while DynamoDB stores only the metadata and references required to identify the group, meeting, source, and state.

CloudFormation creates the tables in `PAY_PER_REQUEST` mode with server-side encryption and TTL for temporary records. `DeletionPolicy` and `UpdateReplacePolicy` use `Retain`; PITR and deletion protection can be enabled through parameters after considering the environment and cost. The `meeting-data` table has a stream for workflows that react to data changes.

![Five CampusMeet DynamoDB tables](images/5-Workshop/campusmeet-evidence/dynamodb-tables.png)

*All five CampusMeet data tables are `Active`. The screenshot also records the index counts and current deletion-protection state so the assessment does not overstate the deployed configuration.*

## API-to-data path

```text
Interface
  → API client attaches JWT
  → Handler receives the request
  → Application service enforces rules and access
  → Repository executes the access pattern
  → DynamoDB or S3
```

The frontend never accesses DynamoDB directly. Handlers also avoid assembling data queries independently for every case; services and repositories keep authorization rules, transactions, and mapping behavior testable in isolation.

## Collaboration and consistency

The group creator becomes a `GROUP_ADMIN`, while other members join only through a valid flow. Invitations carry states and expiry, and acceptance must match the intended account. The backend verifies active membership before reading or changing group-scoped information.

Several data principles apply throughout the product:

- retryable requests use idempotency to avoid duplicate results;
- concurrent updates use versions or conditional writes;
- multi-item changes that require atomicity use transactions;
- timestamps are stored in UTC and displayed in the user's timezone;
- TTL is reserved for temporary data and does not replace retention rules for primary content;
- normal business requests use defined access patterns or indexes rather than scanning an entire table.

These principles explain the implementation approach without reproducing every PK, SK, GSI, or transaction expression in the workshop.

## Significance of this implementation area

Cognito, API Gateway, Lambda, and DynamoDB provide a common identity, access boundary, and persistence approach for CampusMeet capabilities. A substantial part of the internship scope focused on this foundation, from infrastructure templates to authentication integration and the table model. The workshop presents the method and main decisions without replacing the project's full endpoint catalogue or PK, SK, and GSI specification.
