# Product Requirements Document (PRD) - X-put Conversational Forms MVP
## 1. Product Overview
X-put is a hosted, embeddable conversational form system that lets integrators define a form once (hybrid schema + AI guidance), embed a lightweight two-pane widget (chat + dynamic form/summary), and receive validated, normalized submissions server-side. The MVP is single-tenant, ships as a secure web widget hosted on a single origin, and prioritizes schema-first collection with optional AI-suggested fields under explicit integrator control.

Key pillars
- Single-origin hosted widget with tight CSP, SRI, sandboxed iframe, and strict postMessage origin checks.
- Two-pane UX: left chat planner, right dynamic form/summary with live validity status and suggestion chips.
- Simple lifecycle: collecting (ephemeral) → submitted (persisted); latest valid submission overwrites prior ones.
- Server-to-server data access; UI can only create/overwrite submissions and read the form definition.
- Observability via standardized events stored in DB; no public Metrics API in MVP.

## 2. User Problem
Integrators need a fast, secure way to collect structured data via a conversational interface without building custom UIs, validation, or AI orchestration. Respondents need a clear, accessible experience that minimizes back-and-forth and ensures data is captured correctly the first time. Organizations need reliable, low-latency APIs with strict security and predictable token spend.

## 3. Functional Requirements
3.1 Form management
- Create form: POST /forms with hybrid spec { metadata, schema (JSON Schema-lite), guidance } returns formUuid (UUIDv4/5/7, implementation detail; externally opaque).
- Read form: GET /forms/{uuid} with ETag and caching (max-age=60, must-revalidate). ETag must include schema.version which increments on changes.
- JSON Schema-lite supported keywords: type, enum, format (email, uri, date, datetime), minLength, maxLength, minimum, maximum, pattern, items, required, properties.
- Guidance guardrails: max 8k characters; lint at publish-time for secrets/PII patterns; reject on violations with actionable details.

3.2 Authentication and authorization
- JWT-only auth using tenant JWKS; required claims: sub (≤128 chars), aud, iss.
- Token TTL 90 minutes; clock skew ±60 seconds; SDK handles 401 → onAuthNeeded() → single retry.
- Server-to-server scopes: submissions.history:read (elevated), submissions.latest:read (implicit via endpoint permissions). Enabling transcript inclusion requires a scope flag gated by DPA sign-off.

3.3 Widget and UX
- Two-pane layout: chat planner on the left; form/summary on the right with required/optional badges.
- Planner behavior: temperature=0.2, maxClarifications=2, maxTokens=1024; required fields grouped first; after 2 failed clarifications for any required field, switch to manual form-only input.
- AI-suggested fields appear as suggestion chips only when allow_ai_fields=true and keys are whitelisted; suggestions live under _ai.suggestions; apply shows diff and records approval in _ai.approvals[].
- Accessibility: WCAG 2.2 AA; full keyboard navigation and screen-reader labels; error text mapped from machine-readable details[].
- Localization: en-US only; normalize dates to ISO 8601 (UTC); honor Accept-Language and tz where present.

3.4 Submission handling
- Atomic submit: latest valid submission overwrites previous for the form+actor pair; history preserved immutably for 30 days (configurable) with metadata-only by default.
- Idempotency and de-duplication: client sends submissionClientId (UUIDv7) and Idempotency-Key; server de-duplicates within a fixed 10-minute window.
- Submission envelope: submissionId (UUIDv7), formUuid, actorId (from JWT sub), occurredAt, implicit schemaVersion and aiVersion, confidence, rawTranscript, normalizedPayload.
- Token budget handling: when exhausted, show banner with Retry-After and next reset timestamp; disable chat; keep manual form usable; emit token_budget_exhausted event.

3.5 Data access and caching
- UI reads: GET /forms/{uuid}; UI creates/overwrites submissions.
- Server-to-server reads: GET /forms/{uuid}/submission returns latest valid submission. Include ETag and If-None-Match support.
- History: GET /forms/{uuid}/submission/history?cursor&limit (internal or elevated) returns metadata by default; include=data requires elevated scope.
- Storage indexing: primary (formUuid), secondary (formUuid, updatedAt DESC), forms_latest_submission pointer table.

3.6 Observability, metrics, and SLOs
- P0 events: session_started, field_validated, clarification_prompted, submission_saved, token_budget_exhausted, auth_refreshed, rate_limited. Each includes tenantId, formUuid, sessionId, actorId (if available), timestamp (ISO 8601 UTC).
- Metrics retained in DB only (no public Metrics API in MVP); internal dashboards read from DB.
- SLOs: availability 99.9%; API p95 < 1.5s; data retention compliance per policy; events delivery ≥ 99.9%.
- Rate limits: 10 RPS per token (burst 20). Schema size ≤ 256 KB. Transcript cap 8k tokens. Exclude repeated idempotent retries from rate counting during the de-dup window.

3.7 Security and privacy
- CSP and embedding: sandboxed iframe, strict frame-ancestors allowlist (static for single tenant via configuration), X-Frame-Options: DENY on non-widget routes, strict origin checks for postMessage, no third-party cookies.
- Data protection: encryption at rest (KMS), audit trails for transcript access, masking in logs by default (emails/phones).
- Error taxonomy and i18n: 400_VALIDATION, 401_UNAUTHORIZED, 403_FORBIDDEN, 404_NOT_FOUND, 409_CONFLICT, 422_UNPROCESSABLE, 429_RATE_LIMIT, 429_TOKEN_BUDGET, 5xx. details[] with JSON Pointers; i18n keys mapped to default strings in SDK dictionary.
- DPA: transcript inclusion only after DPA sign-off and enabling scope flag.

3.8 Performance and delivery
- Widget performance budget: bundle ≤ 120 KB gzipped; first interaction ≤ 2.5s on 4G; lazy-load AI modules; single-file embed with SRI.
- Browser support: last 2 versions of Chromium/Firefox/Safari and Safari ≥ 16.
- Environments: dev/stage/prod with separate KMS/JWKS; sealed secrets; health/readiness probes; change control for schema tables.
- Release gates: OWASP ASVS L2, WCAG 2.2 AA audit, dependency audit clean, load test to 10 RPS/token (burst 20), OpenAPI + Postman + mock server, runbook. CI blocks merges on gate failures with assigned DRIs (RACI).

## 4. Product Boundaries
In scope (MVP)
- Hosted widget and embeddable script on single origin with CSP, SRI, sandboxed iframe.
- Two-pane chat + form/summary UX with inline validation and suggestion chips.
- Schema-first forms with optional AI proposals under explicit whitelist.
- JWT-only auth with refresh hook; server-to-server reads for saved data.
- DB-only metrics; standardized event stream; 30-day immutable submission history (metadata by default).
- Token budget enforcement with graceful fallback to manual form.

Out of scope (MVP)
- File uploads and autosave/drafts.
- Public Metrics API or hosted customer-facing dashboard.
- Multi-language UI beyond en-US; RTL.
- Third-party cookies and non-web channels (native iOS/Android SDKs).
- Payments, roles/teams UI.

Assumptions
- Single-tenant deployment for MVP with static frame-ancestors allowlist.
- At-least-once webhook semantics if webhooks are configured; idempotency required on receiver side.
- All dates normalized to ISO 8601 (UTC).

Dependencies
- Tenant JWKS for JWT validation; KMS for encryption; internal event pipeline and DB for metrics; CI/CD for guardrails.

## 5. User Stories
US-001
Title: Create form with hybrid spec
Description: As an integration engineer, I want to define a form with metadata, schema, and guidance so that the system can render and drive a conversational intake.
Acceptance Criteria:
- Given a valid JSON Schema-lite and guidance ≤ 8k chars, when I POST /forms, then I receive formUuid and schema.version = 1.
- Given guidance contains disallowed tokens (e.g., secrets), when I POST /forms, then I receive 422_UNPROCESSABLE with details[] pointing to offending segments.
- Given I update the schema or guidance, when I PUT /forms/{uuid}, then schema.version increments and the ETag changes.

US-002
Title: Read form with caching
Description: As a widget client, I want to GET the form with ETag so that I can cache and revalidate efficiently.
Acceptance Criteria:
- Given I have ETag E1, when I GET /forms/{uuid} with If-None-Match E1 and nothing changed, then I receive 304.
- Given schema.version changed, when I GET /forms/{uuid}, then I receive 200 with new ETag containing the updated version.

US-003
Title: Embed secure widget
Description: As an integration engineer, I want to embed the widget in a sandboxed iframe restricted by frame-ancestors so that only allowed origins can host it.
Acceptance Criteria:
- Given the tenant allowlist contains https://partner.example, when the iframe loads from elsewhere, then it is blocked.
- Given postMessage is used, when an unexpected origin posts, then the widget ignores the message.

US-004
Title: Authenticate via JWT
Description: As a system, I want to validate JWTs so that only authorized actors interact with the form.
Acceptance Criteria:
- Given JWT TTL 90m and skew ±60s, when an expired token is presented, then I return 401_UNAUTHORIZED.
- Given aud or iss mismatches, when a request arrives, then I return 401_UNAUTHORIZED and log a redacted failure.

US-005
Title: Refresh on 401 in SDK
Description: As a respondent using the widget, I want seamless auth refresh so that my session continues without data loss.
Acceptance Criteria:
- Given a 401 from any API call, when onAuthNeeded() returns a new token, then the SDK retries once and continues.
- Given the retry also fails, when the error is displayed, then actionable guidance is shown without exposing PII.

US-006
Title: Planner asks required fields first
Description: As a respondent, I want the chat to prioritize required fields so that I complete the form quickly.
Acceptance Criteria:
- Given a schema with required fields, when the session starts, then the first prompts target required groups.
- Given a required field fails validation twice, when maxClarifications=2 is reached, then the planner stops and the manual field becomes editable with helper text.

US-007
Title: Inline validation and errors
Description: As a respondent, I want immediate validation and clear errors so that I can correct mistakes.
Acceptance Criteria:
- Given a field format=email, when I type an invalid email, then I see an accessible error and an example.
- Given details[] from backend, when errors are present, then UI maps to user-friendly copy via the i18n dictionary.

US-008
Title: Apply AI suggestion chip
Description: As a respondent, I want to apply AI-suggested values when appropriate so that I can speed up completion.
Acceptance Criteria:
- Given allow_ai_fields=true and a whitelisted key, when the chip is clicked, then the value is applied with a visible diff.
- Given the value is applied, when the submission is saved, then _ai.approvals[] records approver and timestamp.

US-009
Title: Token budget exhaustion fallback
Description: As a respondent, I want the system to degrade gracefully when token budget is hit so that I can still finish the form.
Acceptance Criteria:
- Given the budget is exhausted mid-session, when the next model call is attempted, then a banner shows Retry-After and next reset and chat is disabled.
- Given chat is disabled, when I continue in form-only mode, then I can submit successfully if all validations pass.

US-010
Title: Atomic submit with overwrite
Description: As a system, I want to persist the latest valid submission and overwrite previous so that the latest is canonical.
Acceptance Criteria:
- Given a previous submission exists for actorId+formUuid, when a new valid submission arrives, then it overwrites the pointer in forms_latest_submission and history stores both immutably.
- Given validation fails, when submit is attempted, then 400_VALIDATION returns with details[] and nothing is persisted.

US-011
Title: Idempotent submission
Description: As a client, I want idempotent submits so that retries do not create duplicates.
Acceptance Criteria:
- Given submissionClientId and Idempotency-Key are provided, when the same request is retried within 10 minutes, then the original result is returned.
- Given the retry window passes, when the same request is sent, then a new submission is created.

US-012
Title: Latest submission fetch (server-to-server)
Description: As an external system, I want to fetch the latest valid submission by formUuid so that I can process it.
Acceptance Criteria:
- Given a valid service token, when I GET /forms/{uuid}/submission, then I receive the latest normalizedPayload with ETag.
- Given If-None-Match matches, when I GET again, then I receive 304.

US-013
Title: History access with elevated scope
Description: As a compliance auditor, I want to list prior submissions so that I can review history.
Acceptance Criteria:
- Given submissions.history:read scope, when I GET /forms/{uuid}/submission/history, then I receive metadata-only unless include=data is explicitly set.
- Given include=data is requested without scope, when I call the endpoint, then I receive 403_FORBIDDEN.

US-014
Title: Accessibility compliance
Description: As a respondent using assistive tech, I want a WCAG-compliant experience so that I can complete the form.
Acceptance Criteria:
- Given keyboard navigation, when I tab through interactive elements, then focus order is logical and visible.
- Given screen readers, when errors occur, then they are announced and linked to inputs.

US-015
Title: Performance budgets enforced
Description: As an engineering team, I want guardrails so that widget performance meets targets.
Acceptance Criteria:
- Given CI checks, when bundle size exceeds 120 KB gz, then the build fails.
- Given synthetic 4G tests, when first interaction exceeds 2.5s, then the build fails.

US-016
Title: Event emission for observability
Description: As an operator, I want standardized events so that I can monitor health and funnels.
Acceptance Criteria:
- Given a session starts, when the widget loads, then session_started is emitted with IDs and timestamp.
- Given token budget exhaustion, when it occurs, then token_budget_exhausted is emitted and stored in DB.

US-017
Title: Error taxonomy surfaced to users
Description: As a respondent, I want clear, localized error messages so that I understand what to do next.
Acceptance Criteria:
- Given a 429_RATE_LIMIT from API, when shown in UI, then I see retry guidance without leaking internals.
- Given 401 after refresh, when displayed, then I see a message to reauthenticate and a safe retry action.

US-018
Title: Data handling and masking
Description: As a security lead, I want sensitive data masked in logs by default so that we minimize exposure.
Acceptance Criteria:
- Given email or phone appears in logs, when events are written, then they are masked unless elevated debug is enabled for a bounded incident window.
- Given transcript inclusion is requested, when DPA is not signed, then inclusion is denied by policy with an audit record.

US-019
Title: Caching and revalidation examples for integrators
Description: As an integration engineer, I want documented caching examples so that I implement ETag and If-None-Match correctly.
Acceptance Criteria:
- Given sample code, when I follow it, then subsequent GET /forms/{uuid} return 304 when unchanged.
- Given schema.version bumps, when I redeploy, then clients pick up the change on next revalidation.

US-020
Title: Localization baseline
Description: As a product manager, I want en-US only in MVP with normalized dates so that we reduce complexity.
Acceptance Criteria:
- Given locale headers, when Accept-Language is non-en, then UI still renders en-US but dates in payloads are ISO 8601 UTC.
- Given tz hints, when date inputs are provided, then normalization is consistent and tested.

US-021
Title: Security headers and cookie policy
Description: As a platform, I want strict headers so that the widget is resilient to clickjacking and cross-site exploits.
Acceptance Criteria:
- Given non-widget routes, when framed, then X-Frame-Options: DENY blocks rendering.
- Given all requests, when CSP is evaluated, then only self and allowed inline hashes are permitted; no third-party cookies are set.

US-022
Title: LLM routing and resilience
Description: As a system, I want policy-driven model routing so that I meet latency, cost, and compliance goals.
Acceptance Criteria:
- Given primary region is degraded, when requests are sent, then the router fails over to a cheaper fallback with 2× retry and jitter and a 4s planning timeout.
- Given repeated failures, when thresholds are crossed, then a tenant-level circuit breaker opens and errors are surfaced with guidance to switch to manual mode.

US-023
Title: Rate limiting behavior
Description: As a client, I want predictable rate limit responses so that I can back off correctly.
Acceptance Criteria:
- Given 10 RPS/token burst 20 is exceeded, when calls arrive, then 429_RATE_LIMIT is returned with retry hints while repeated idempotent calls are excluded from counting within the window.
- Given sustained overload, when backoff is respected, then service recovers without throttling compliant clients.

US-024
Title: Two canonical schemas frozen
Description: As a team, I want two canonical P0 schemas and guidance frozen so that CI can run golden-path tests.
Acceptance Criteria:
- Given the schemas are committed, when CI runs, then scenario-based acceptance tests validate prompts, clarifications, and normalized payloads match golden fixtures.
- Given schema changes, when PRs are opened, then CI fails unless fixtures and version numbers are updated.

US-025
Title: Internal metrics dashboards from DB
Description: As an internal stakeholder, I want dashboards reading from DB so that I can monitor without a public API.
Acceptance Criteria:
- Given DB access, when the dashboard queries events, then it renders funnel and latency charts with ≥ 99% completeness.
- Given permissions, when an unauthorized user attempts access, then it is denied and audited.

## 6. Success Metrics
User efficiency
- Median completion time ≤ 2 minutes.
- Median clarification turns ≤ 2.
- First-pass validity ≥ 80%.
- Abandonment ≤ 10%.

Reliability and performance
- Availability 99.9% monthly.
- API p95 latency < 1.5 seconds.
- Idempotent de-duplication success > 99.5%.
- Widget bundle ≤ 120 KB gz; first interaction ≤ 2.5s on 4G.

Cost and usage control
- Sessions hitting token budget ≤ 2%.
- Spend within per-form token caps.

Security and compliance
- OWASP ASVS L2 pass.
- WCAG 2.2 AA audit pass.
- Data retention compliance; transcripts gated by policy/DPA.

Observability
- P0 events delivered and stored ≥ 99.9%.
- Metrics completeness in DB ≥ 99%.
