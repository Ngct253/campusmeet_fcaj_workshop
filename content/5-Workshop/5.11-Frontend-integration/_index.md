---
title: "Frontend Integration"
date: 2026-08-08
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

# Frontend Integration

## Goal

This section explains how the React/Vite frontend connects to Cognito and the CampusMeet HTTP API, restores sessions, protects routes, sends JWTs, and keeps server state in sync after mutations.

## 1. Frontend stack

The web application lives under `apps/web/` and uses React, TypeScript, Vite, React Router, TanStack Query, Cognito authentication, and a shared API client.

The browser never reads DynamoDB directly.

## 2. Public configuration

The current frontend uses public values such as:

```dotenv
VITE_COGNITO_USER_POOL_ID=<UserPoolId>
VITE_COGNITO_USER_POOL_CLIENT_ID=<UserPoolClientId>
VITE_API_BASE_URL=<ApiUrl>
VITE_GOOGLE_CLOUD_PROJECT_NUMBER=<GoogleCloudProjectNumber>
```

`VITE_*` values are visible in the browser bundle. They must never contain passwords, JWTs, AWS access keys, Google client secrets, or refresh tokens.

## 3. Authentication flow

```text
Sign up
  ↓
Confirm email
  ↓
Sign in
  ↓
Cognito session
  ↓
Protected application routes
```

The app must also handle sign-out, forgotten passwords, and session restoration after a browser reload.

## 4. Main routes

The current frontend includes routes such as:

```text
/
/sign-in
/sign-up
/confirm-sign-up
/forgot-password
/app/dashboard
/app/groups
/app/groups/:groupId
/app/groups/:groupId/meetings
/app/meetings/:meetingId
/app/tasks
/app/notifications
/app/invitations
/app/settings
/meet-addon/side-panel
```

Routes under `/app` require an authenticated session. UI role checks improve usability, but backend authorization remains mandatory.

## 5. API calls and JWT

The API client obtains the current token and sends it in the `Authorization` header. It must handle common responses such as 401, 403, 404, 409, and server errors without exposing internal stack traces to users.

## 6. Server state with TanStack Query

After a successful mutation, the application invalidates or refetches the affected query instead of keeping stale data in memory.

For example:

```text
Create Meeting
   ↓
API success
   ↓
invalidate meeting list
   ↓
refetch stored data
```

The same pattern applies to groups, invitations, minutes, tasks, notifications, dashboard summaries, and AI job status.

## 7. Loading, error, and conflict states

Important pages should provide loading, empty, error, and disabled-submit states. When a versioned resource returns `409 Conflict`, the frontend should preserve the user's draft and explain that the server copy has changed.

## 8. Production build

```powershell
npm run build -w @campusmeet/web
```

The generated `dist` directory is the artifact published to S3/CloudFront in the full application environment.

## 9. Browser verification

After deployment, open the real frontend URL, sign up, sign in, reload a protected route, navigate through Group → Meeting → Minutes → Task → Dashboard, then sign out and confirm protected pages require authentication again.

The `/meet-addon/side-panel` route existing in source is not proof that the Google Meet Add-on has been published or browser-verified; that is tested separately.

## Result

The frontend acts as a client of Cognito and the HTTP API, while authorization and persisted data remain server-side responsibilities.
