---
title: "Verification, Operations, and Evaluation"
date: 2026-08-08
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Evaluation approach

CampusMeet is evaluated through its user journey and observable evidence, not only by counting screens or source files. Each feature group is considered against the following criteria:

| Criterion | How it is considered |
| --- | --- |
| Workflow continuity | Whether users can move from groups, meetings, and preparation to minutes, tasks, and progress |
| Access control | Whether information keeps the correct group, role, and meeting boundary for every action |
| Information reliability | Whether drafts, confirmed content, and source versions remain clearly distinguished |
| Verification level | Whether a result exists only in source, has local tests, or has also been verified in a shared environment and realistic browser flow |
| Operational readiness | Whether failures, processing states, retention needs, and cost can be observed as usage expands |

A capability is complete only when its related parts work in a realistic context; a screen or cloud resource alone does not prove the whole flow is ready.

## Evidence used for the summary

Workshop conclusions are cross-checked at several levels instead of relying on description alone:

1. The interface and user journey show which steps a person can perform.
2. Source code and tests show where rules, permissions, and failure cases are covered.
3. Results in the shared environment show how components connect and persist information.
4. Browser checks with appropriate accounts, roles, and data confirm the end-to-end experience.

When evidence exists at only some levels, the workshop reports only what was observed and keeps the remainder under further verification. This lets readers distinguish design, local implementation, and behavior demonstrated under realistic conditions.

## Technical checks before handover

The repository quality gates should run from the workspace root:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
npm run infra:validate
```

`lint` and `typecheck` catch rule and type issues; `test` covers components, services, handlers, repositories, and adapters within their respective scopes; `build` confirms that the workspaces can produce artifacts; and `infra:validate` checks repository-defined infrastructure constraints. A successful run does not prove production readiness, but it is a necessary condition before deployment or handover.

In AWS, verification continues with CloudFormation outputs, stack state, runtime configuration, the five data tables, and the `/health` endpoint. In the browser, the frontend should point to the intended User Pool and API, protected routes should require sign-in, errors should provide appropriate feedback, and persisted information should remain after a reload. Only integrations enabled in that environment should be tested; a screen or an undeployed template is not enough to mark a flow complete.

## Operations, security, and cost

Before an environment is used for a demonstration or assessment, review the following:

- Lambda functions and workers use resource-scoped execution roles rather than long-lived access keys.
- User-content buckets remain private, with upload and download access issued through short-lived URLs after authorization.
- CORS permits only the configured frontend origin, while secrets and tokens remain server-side.
- Logs exclude JWTs, OAuth tokens, presigned URLs, complete transcripts, and document contents.
- CloudWatch observes API failures, workflow failures, stuck jobs, incomplete uploads, and external-service errors.
- Retries are bounded and idempotent, while AWS Budgets provides early warning for unexpected cost growth.
- Retention, PITR, and deletion protection are enabled by environment after reviewing cost and data-protection needs.

![CampusMeet Lambda operational metrics in CloudWatch](images/5-Workshop/campusmeet-evidence/cloudwatch-lambda-metrics.png)

*CloudWatch has collected invocation, duration, error, and throttling metrics for CampusMeet Lambda functions over the selected period. These metrics confirm resource activity, but they do not independently prove that every business flow completed successfully.*

## Current AI infrastructure evidence

Read-only checks in the development environment recorded the following components:

| Component | Verified state or configuration |
| --- | --- |
| Step Functions `campusmeet-dev-ai-jobs` | `Active`, `Standard` type |
| Lambda `campusmeet-dev-ai-worker` | `Active`, latest update successful, Node.js 22, 1024 MB, 300-second timeout |
| Bedrock Knowledge Base | `campusmeet-dev-knowledge-v2`, `ACTIVE` |
| Data source | `campusmeet-dev-sources`, `AVAILABLE` |
| Vector store | S3 Vectors in `ap-southeast-1`, declared by the application stack |
| Generation | AI Worker configured for `https://bedrock-mantle.us-east-1.api.aws/v1` with `openai.gpt-oss-20b` |
| Model credential | A Secrets Manager reference exists; the workshop neither reads nor displays the secret value |

At the verification point, the Mantle dashboard recorded requests and tokens for `openai.gpt-oss-20b`. This usage shows that the model endpoint has been called in the AWS account, but it cannot attribute every request to CampusMeet or confirm the full retrieval–citation–authorization flow.

The AI Worker, Knowledge Base, S3 Vectors, IAM roles, and alarms are managed through CloudFormation/SAM in the application stack. The team still needs to review the overall state and change set before subsequent infrastructure updates; active child resources do not by themselves prove that the complete workflow is ready.

## Current result

The authentication, group, invitation, notification, and meeting CRUD interfaces are connected to APIs. The five DynamoDB tables have been deployed and verified in `ap-southeast-1`. Auth/API and the meeting core exist in the development environment, while the application stack and selected CRUD and authorization smoke tests in shared data still require stabilization and further verification.

Upload, transcripts, knowledge, and AI assistance now include source, contracts, interface behavior, tests, and AWS resources for transcript read/edit/approval, immutable sources, citation-grounded answers, drafts, and progress analysis. The Google check reached a reconnect-required state without creating a Meet URL; AI has model usage and deployed control-plane components but lacks complete evidence for retrieval, citations, and confirmation. The full system is therefore not considered production-ready.

## Lessons learned

- Starting with the user journey produces a clearer scope than starting with a service list.
- Authentication and authorization remain separate, while external integrations must not interrupt the core journey.
- Transcription and AI provide value only when sources are traceable, users confirm results, and each verification level is reported accurately.

## Areas requiring further verification

- Access scenarios across multiple groups and roles.
- Document upload in the intended environment.
- Complete application-stack stabilization and review the next change set before another deployment.
- Google Calendar and Google Meet synchronization after the test account receives the correct OAuth access.
- End-to-end audio processing and transcription.
- End-to-end AI flow from an approved source through retrieval, citations, Mantle response, and user confirmation.
- Cross-Region Mantle impact on outbound context, latency, quotas, and cost.
- Error visibility, cost awareness, and information retention as usage grows.

## Development direction

1. Stabilize the journey from groups to meetings and follow-up tasks.
2. Verify each flow in the shared environment before presenting it as complete.
3. Complete the flow from approved documents and transcripts into knowledge sources and verify AI assistance end to end.
4. Keep AI in a supporting role and let users review sources and confirm results.
5. Improve error messages and capability states so users understand the next step.
6. Maintain appropriate access and protect each group's information.

## Conclusion

At the project-summary milestone, CampusMeet should be understood as a platform with an established core and a clear direction for expansion, while still requiring further verification before the whole system can be considered complete. This assessment reflects the product accurately without overstating the work performed.
