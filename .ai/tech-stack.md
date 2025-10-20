# Technology Stack

## Overview
Goal: Deliver a secure, single-tenant, embeddable conversational forms MVP with strict performance, security, and observability requirements.

## Frontend
- Astro 5 (islands/partial hydration) for minimal JS and fast delivery.
- React 19 for interactive components (chat planner, dynamic form/summary).
- TypeScript 5 for static typing and DX.
- Tailwind CSS 4 for utility-first styling; purge unused CSS to meet bundle budget.
- shadcn/ui (selective, tree-shaken) for accessible React components.
- Accessibility: WCAG 2.2 AA; keyboard nav, ARIA, focus management.
- Performance budget: widget ≤ 120 KB gz; first interaction ≤ 2.5s on 4G.
- Embedding: sandboxed iframe; strict postMessage origin checks; single-file embed with SRI.

## Backend
- Supabase (Postgres) as primary data store: forms, submissions, events, configs.
- API service (containerized) on DigitalOcean:
  - JWT-only auth (BYO IdP via tenant JWKS); TTL 90m; skew ±60s; enforce iss/aud/sub.
  - Endpoints: forms (create/read with ETag), submissions (atomic overwrite, idempotent), history (elevated), metrics (internal DB only).
  - Validation: JSON Schema-lite (type, enum, format: email|uri|date|datetime, min/max, pattern, items, required, properties).
  - Idempotency & de-dup window 10 minutes using Redis or Postgres keys.
  - Caching: ETag/If-None-Match; Cache-Control: max-age=60, must-revalidate.

## AI Integration
- OpenRouter.ai for LLM access with policy-driven routing.
- Defaults per form: temperature=0.2, maxClarifications=2, maxTokens=1024.
- Token budget enforcement: server tracks budget; UI degrades to manual form with Retry-After details.
- Fallbacks: 2× retry with jitter, 4s planning timeout, tenant-level circuit breaker.

## Security
- Single origin hosting; tight CSP and SRI:
  - CSP: default-src 'none'; script-src 'self' 'strict-dynamic'; style-src 'self' 'unsafe-inline'; connect-src 'self'; img-src 'self' data:; frame-ancestors allowlist.
  - X-Frame-Options: DENY on non-widget routes.
- Iframe sandboxing; strict frame-ancestors allowlist (static for single tenant).
- postMessage strict origin validation.
- Encryption at rest (DB); audit trails for transcript/data access.
- Logs: mask emails/phones by default.
- No third-party cookies.

## Observability
- P0 events: session_started, field_validated, clarification_prompted, submission_saved, token_budget_exhausted, auth_refreshed, rate_limited.
- Store events in DB; 30-day retention; internal dashboards read directly from DB.
- SLOs: availability 99.9%; API p95 < 1.5s; events delivery ≥ 99.9%.

## Rate Limiting and Idempotency
- 10 RPS per token (burst 20).
- Exclude repeated Idempotency-Key requests from rate counting within 10 minutes.
- Client provides submissionClientId (UUIDv7) + Idempotency-Key.

## CI/CD and Hosting
- GitHub Actions:
  - CI gates: bundle size ≤ 120 KB gz, 4G synthetic first interaction ≤ 2.5s, dependency audit clean, lint/tests, accessibility checks.
  - Security checks aligned with OWASP ASVS L2.
- Containerized deploys on DigitalOcean (app + reverse proxy).
- Environments: dev/stage/prod; sealed secrets; health/readiness probes.

## Data Model and Indexing
- Tables: forms, submissions, events, forms_latest_submission pointer.
- Indexes: (formUuid) primary; (formUuid, updatedAt DESC) secondary.
- Submission envelope: submissionId (UUIDv7), formUuid, actorId, occurredAt, implicit schemaVersion/aiVersion, confidence, rawTranscript, normalizedPayload.

## Internationalization and Formats
- en-US only for MVP; Accept-Language honored.
- Dates normalized to ISO 8601 (UTC); tz hints respected.

## Constraints and Limits
- Schema size ≤ 256 KB.
- Transcript cap 8k tokens.
- Guidance guardrails: ≤ 8k chars; lint for secrets/PII at publish-time.

## Implementation Notes
- Keep React usage minimal via Astro islands; lazy-load chat/AI modules.
- Tree-shake shadcn/ui and icons; avoid heavy date/utility libs.
- Centralize security headers and CSP at the edge/reverse proxy.
