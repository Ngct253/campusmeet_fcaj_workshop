---
title: "Frontend and Production Deployment"
date: 2026-08-08
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Frontend and Production Deployment

CampusMeet uses React and Vite for the web application. The production build is published through Amazon S3 and CloudFront so the project has a real HTTPS link for the final demo and submission.

## Main screens

The production interface includes:

- sign-up and sign-in;
- Dashboard;
- Groups and Invitations;
- Meeting detail;
- Minutes and Tasks;
- Notifications and Settings.

The frontend connects to Cognito and the production API through public environment values.

## Production preparation

Before publishing the frontend, the AWS environment should provide:

- the data stack;
- the user-content/orchestration stack when those features are used;
- the full application stack;
- outputs such as API URL, Cognito IDs, frontend bucket, and CloudFront domain.

Use a separate production environment such as `campusmeet-prod` instead of mixing final demo data with development resources.

## Build and publish

After the CloudFormation outputs are available, build the frontend:

```powershell
npm run build -w @campusmeet/web
```

Upload the generated build to the frontend S3 bucket and serve it through CloudFront.

## Production link

The key result is an HTTPS URL such as:

```text
https://<cloudfront-domain>.cloudfront.net
```

The link must work without `localhost`. Users should be able to register, sign in, and use the core CampusMeet workflow against the real AWS backend.

CORS settings for the API and user-content bucket should use the actual CloudFront origin in production.

E2E testing begins only after the frontend, API, Cognito, and CloudFormation stacks are all available in the production environment.