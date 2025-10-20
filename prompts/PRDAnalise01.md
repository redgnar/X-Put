You are an experienced product manager whose task is to help create a comprehensive Product Requirements Document (PRD) based on the provided information. Your goal is to generate a list of questions and recommendations that will be used in subsequent prompting to create a complete PRD.

Please carefully review the following information:

<project_description>
# Application X-Put

### Main problem
There are too many places to enter data into apps. We need to unify them and create a single place where users can enter data in the simplest way possible.

### Minimum set of functionality
- User (probably external system) creates a from. In return it receives a UUID of the form. Payload of this request is a definition of form. 
- Form definition is a prompt to AI with required data description (some form of definition what we need to get from user).
- User that is identified by e.x. JWT token can use this form (by UUID) to enter data.
- Using added prompt to Handle AI conversation with user to gather data - AI should have some tools to generate simple form (possible fields will be defined by system).
- Persist gathered data in some storage with relation to added propmpt (form definition).
- Fetching data from storage (probably external system) by UUID of the form..

### What is NOT included in the MVP
- No UI for creating and fetching form. API will be used only.
- No user notifications.
- No user management.
- No user roles.
- No user permissions.
- No user data export.
- No user data import.
- No user data backup.

### Success Criteria
- User can enter data in the simplest way possible.

</project_description>

Analyze the information provided, focusing on aspects relevant to PRD creation. Consider the following questions:
<prd_analysis>
1. Identify the main problem that the product is intended to solve.
2. Define the key functionalities of the MVP.
3. Consider potential user stories and paths of product usage.
4. Think about success criteria and how to measure them.
5. Assess design constraints and their impact on product development.
   </prd_analysis>

Based on your analysis, generate a list of 10 questions and recommendations in a combined form (question + recommendation). These should address any ambiguities, potential issues, or areas where more information is needed to create an effective PRD. Consider questions about:

1. Details of the user's problem
2. Prioritization of functionality
3. Expected user experience
4. Measurable success indicators
5. Potential risks and challenges
6. Schedule and resources

<questions>
List your questions and recommendations here, numbered for clarity:

For example:
1. Are you planning to introduce paid subscriptions from the start of the project?

Recommendation: The first phase of the project could focus on free features to attract users, and paid features could be introduced at a later stage.
</questions>

Continue this process, generating new questions and recommendations based on the user's responses, until the user explicitly asks for a summary.

Remember to focus on clarity, relevance, and accuracy of results. Do not include any additional comments or explanations beyond the specified output format.

Analytical work should be done in the thinking block. The final output should consist solely of questions and recommendations and should not duplicate or repeat any work done in the prd_analysis section.

##### **Questions 1:**

1. **Who are the primary integrator personas (e.g., backend service vs. web app) and their top 3 jobs-to-be-done when creating/using a form?**
   **Recommendation:** Define 2–3 concrete integrator personas and document their JTBD with end-to-end sequence diagrams (create → collect → fetch) to anchor scope and API ergonomics.

2. **What form-definition contract do you expect the API to accept (strict JSON schema, prompt-only, or hybrid)?**
   **Recommendation:** Adopt a hybrid spec: `{metadata, schema(JSON Schema-lite), guidance(AI prompt)}` with strict validation on `schema` and free-form `guidance` to keep AI flexible yet predictable.

3. **What data fields must always be captured for every submission (e.g., submissionId, formUuid, userId, occurredAt, version)?**
   **Recommendation:** Standardize a submission envelope (`submissionId UUIDv7, formUuid, actorId, occurredAt, schemaVersion, aiVersion, confidence, rawTranscript, normalizedPayload`) to simplify observability and evolution.

4. **How much autonomy should the AI have in generating fields vs. strictly adhering to the provided schema?**
   **Recommendation:** Default to “schema-first” with AI allowed to propose optional fields; require explicit integrator opt-in (`allow_ai_fields: true`) and tag AI-added fields in the payload for traceability.

5. **What is the desired UX for end users: chat-only, form-only, or chat + dynamic form preview?**
   **Recommendation:** Target chat + dynamic form preview as MVP; expose a minimal UI SDK (web component) that renders AI prompts, field suggestions, and validation messages while your backend stays API-only.

6. **What acceptance criteria define “simplest way possible” (e.g., median time-to-complete, clarifications per session, abandonment rate)?**
   **Recommendation:** Set MVP success metrics: median completion ≤ 2 minutes, ≤ 2 clarification turns, ≥ 80% first-pass validity, ≤ 10% abandonment; instrument events and dashboards from day one.

7. **Which security boundaries are required (authn via JWT, per-form integration token, CORS, rate limits, PII handling, data residency)?**
   **Recommendation:** Enforce server-to-server for read endpoints, require per-form integration tokens, strict allowlisted CORS, per-token rate limits, PII classification flags, and configurable data retention (e.g., 30/90 days).

8. **How will versioning work for forms and models (e.g., schema changes, prompt updates, model upgrades)?**
   **Recommendation:** Implement immutable `formVersion` and `modelVersion`; submissions reference both. Provide `GET /forms/{uuid}/versions`, soft-migrate with adapters, and block breaking changes without explicit version bump.

9. **What failure modes should be handled (LLM refusal, latency spikes, token limits, partial data, validation loops)?**
   **Recommendation:** Add deterministic fallbacks: cap to 2 clarification turns, then return a structured error with partial payload; use streaming with a 95p latency SLO, circuit-breaker to a rules-based validator, and idempotent retries.

10. **What delivery timeline and resources are available (team size/skills, budget for LLM usage, environments)?**
    **Recommendation:** Plan a 3-sprint MVP:

* Sprint 1: core APIs (create form, submit, fetch), auth, storage, observability.
* Sprint 2: AI orchestration (tooling to suggest fields), validation, metrics.
* Sprint 3: minimal UI SDK, hardening (rate limits, privacy), load & cost tests with usage caps.

##### **Answers: 1**

1. Define 2–3 concrete integrator personas and document their JTBD with end-to-end sequence diagrams (create → collect → fetch) to anchor scope and API ergonomics.
2. Adopt a hybrid spec: `{metadata, schema(JSON Schema-lite), guidance(AI prompt)}` with strict validation on `schema` and free-form `guidance` to keep AI flexible yet predictable.
3. Standardize a submission envelope (`submissionId UUIDv7, formUuid, actorId, occurredAt, schemaVersion, aiVersion, confidence, rawTranscript, normalizedPayload`) to simplify observability and evolution.
4. Default to “schema-first” with AI allowed to propose optional fields; require explicit integrator opt-in (`allow_ai_fields: true`) and tag AI-added fields in the payload for traceability.
5. Implement a two-pane layout (chat on left, dynamic form & “live summary” on right) with autosave and an always-visible “Submit” CTA.
6. Set MVP success metrics: median completion ≤ 2 minutes, ≤ 2 clarification turns, ≥ 80% first-pass validity, ≤ 10% abandonment; instrument events and dashboards from day one.
7. Enforce server-to-server for read endpoints, require per-form integration tokens, strict allowlisted CORS, per-token rate limits, PII classification flags, and configurable data retention (e.g., 30/90 days).
8. In MVP ther wont be formVersion or modelVersion. To modify form create new one.
9. Add deterministic fallbacks: cap to 2 clarification turns, then return a structured error with partial payload; use streaming with a 95p latency SLO, circuit-breaker to a rules-based validator, and idempotent retries.
10. Plan a 3-sprint MVP:
* Sprint 1: core APIs (create form, submit, fetch), auth, storage, observability.
* Sprint 2: AI orchestration (tooling to suggest fields), validation, metrics.
* Sprint 3: minimal UI SDK, hardening (rate limits, privacy), load & cost tests with usage caps.

##### Questions 2:

1. **What tenancy model will you support (single-tenant vs. multi-tenant), and how is form ownership established and enforced across tenants?**
   **Recommendation:** Introduce `tenantId` at form and submission level; require JWTs to carry `tenantId` and `issuer`; enforce isolation in storage queries and logs; add `X-Request-Id` and `X-Tenant-Id` headers for traceability.

2. **Which subset of JSON Schema will “JSON Schema-lite” include (types, enums, formats, min/max, regex, arrays, nested objects, conditionals)?**
   **Recommendation:** Start with: `type`, `enum`, `format (email, uri, date, datetime)`, `minLength`, `maxLength`, `minimum`, `maximum`, `pattern`, `items`, `required`, and `properties`; document unsupported features with clear validation errors.

3. **Will submissions support file uploads (e.g., receipts, IDs) and structured media (images/audio) in the chat?**
   **Recommendation:** Add an optional `attachments[]` field in the envelope with `{ id, type, mime, size, sha256, storageUrl }`; restrict MIME allowlist; scan with AV; store only pointers, not blobs, in the submission payload.

4. **How should drafts and session resume work with autosave (timeouts, resume tokens, concurrency)?**
   **Recommendation:** Create `draftId` separate from `submissionId`; autosave every N seconds or on field change; expire drafts after 24h; promote to final on “Submit”; handle last-writer-wins via `If-Match` with `draftEtag`.

5. **What authentication trust model for JWTs (first-party issuer only vs. bring-your-own IdP)?**
   **Recommendation:** Support BYO IdP with JWKS URL per tenant; validate `aud`, `iss`, `exp`, `sub`; map `sub → actorId`; require a per-form integration token for server-to-server reads as a second factor.

6. **How will you expose integration events (created, draft_saved, validation_failed, submitted) to external systems?**
   **Recommendation:** Provide webhooks with HMAC signature and exponential backoff; add `GET /events` polling fallback; include minimal PII in webhook payloads, requiring a separate fetch for full data.

7. **What exact rate limits and payload limits will you enforce (RPS, burst, max schema size, max transcript tokens)?**
   **Recommendation:** Start with 10 RPS per token (burst 20), 256 KB schema limit, 2 MB total attachments per submission, and 8k token transcript cap; return `429` with `Retry-After` and emit a quota event.

8. **How will you measure and control LLM cost and quality (provider selection, fallback models, evaluation datasets)?**
   **Recommendation:** Implement a model router with per-tenant caps, nightly offline evals on a small golden set (accuracy, turns, validity), and real-time cost telemetry per form; define hard fallbacks to a cheaper model when latency/cost spikes.

9. **What privacy posture for transcripts and prompts (retention, redaction, access controls, audit)?**
   **Recommendation:** Encrypt at rest with KMS, redact PII in `rawTranscript` via detectors before storage, default retention 30 days configurable per form, and keep an immutable audit log for access and changes.

10. **How will errors be communicated to integrators and end users (taxonomy, localization, accessibility)?**
    **Recommendation:** Standardize error codes (`400_VALIDATION`, `401_UNAUTHORIZED`, `403_FORBIDDEN`, `404_NOT_FOUND`, `409_CONFLICT`, `422_UNPROCESSABLE`, `429_RATE_LIMIT`, `5xx`), return machine-readable `details[]` with field paths, provide i18n keys for UI SDK, and meet WCAG 2.2 AA for the two-pane layout.

##### Answers 2:

1. Single tenant model
2. Start with: `type`, `enum`, `format (email, uri, date, datetime)`, `minLength`, `maxLength`, `minimum`, `maximum`, `pattern`, `items`, `required`, and `properties`; document unsupported features with clear validation errors.
3. No file support in MVP
4. No autosave in MVP
5. Support BYO IdP with JWKS URL per tenant; validate `aud`, `iss`, `exp`, `sub`; map `sub → actorId`; require a per-form integration token for server-to-server reads as a second factor.
6. In MVP provide only API to fetch saved data. 404 if data isn't provided
7. Start with 10 RPS per token (burst 20), 256 KB schema limit, and 8k token transcript cap; return `429` with `Retry-After` and emit a quota event.
8. Introduce token limit per form (inform user about reaching token limit). After reaching limit allow only manual entry of the data.
9. Encrypt at rest with KMS, redact PII in `rawTranscript` via detectors before storage, default retention 30 days configurable per form, and keep an immutable audit log for access and changes.
10. Standardize error codes (`400_VALIDATION`, `401_UNAUTHORIZED`, `403_FORBIDDEN`, `404_NOT_FOUND`, `409_CONFLICT`, `422_UNPROCESSABLE`, `429_RATE_LIMIT`, `5xx`), return machine-readable `details[]` with field paths, provide i18n keys for UI SDK, and meet WCAG 2.2 AA for the two-pane layout.

##### Questions 3:

1. **Will the end-user data entry surface be included in MVP (two-pane chat+form), or is MVP strictly API-only for collection and fetch?**
   **Recommendation:** If MVP is API-only, explicitly defer the UI SDK and two-pane UX to a post-MVP milestone; if included, add a thin hosted widget with WCAG 2.2 AA and i18n keys to avoid partner reimplementation.

2. **What are the canonical submission states and transitions (e.g., `initiated → collecting → submitted → fetched`), and do you need a `draft` state despite “no autosave”?**
   **Recommendation:** Define a minimal lifecycle without drafts for MVP: `collecting` (in-session, ephemeral) → `submitted` (persisted); reject partial saves and require atomic `POST /forms/{uuid}/submission`.

3. **How should idempotency be handled for submission create to prevent duplicates on client retries?**
   **Recommendation:** Require `Idempotency-Key` header and store a short-lived key hash (24h). On repeat, return the original `201` body to ensure safe retries.

4. **What exact fetch patterns are needed by integrators (single submission vs. list, filtering, pagination)?**
   **Recommendation:** Provide: `GET /forms/{uuid}/submission/{submissionId}` and `GET /forms/{uuid}/submissions?limit&after` (cursor-based). Return stable ordering by `occurredAt` and include `nextCursor`.

5. **Do you need strict schema validation modes (fail-fast vs. coerce/normalize) and how should unknown fields be treated when `allow_ai_fields: true` is off?**
   **Recommendation:** MVP: fail-fast on validation errors; reject unknown fields when AI-fields are disabled; provide `details[]` with JSON Pointer paths for each violation.

6. **What formats and constraints must `actorId` and identifiers follow (UUID vs. opaque string), and should `submissionId` be UUIDv7?**
   **Recommendation:** Use UUIDv7 for `submissionId`; allow `actorId` as opaque string (from JWT `sub`), max 128 chars; validate and log both with correlation header `X-Request-Id`.

7. **How will conversation orchestration be configured (system prompt template, temperature, max turns, stop conditions) given the 8k token cap and per-form token limit?**
   **Recommendation:** Ship a per-form `llmConfig` with safe defaults: `temperature=0.2`, `maxClarifications=2`, `maxTokens=1024`, and a hard token budget; on budget exhaust, degrade to rules-based form fill only and emit a quota event.

8. **What observability is required (logs, metrics, traces) and which SLOs will you publish beyond 95p latency?**
   **Recommendation:** Instrument: request rate, 95p/99p latency, error rate by code, completion time, clarification turns, validation pass rate, token usage per form; publish SLOs for availability (99.9%), p95 latency (<1.5s API), and data retention compliance.

9. **What environments and deployment strategy are planned (dev/stage/prod), and how will KMS keys and JWKS configs be managed per environment?**
   **Recommendation:** Separate dev/stage/prod with distinct KMS keys and JWKS URLs; manage config via sealed secrets; add health checks and readiness probes; require change tickets for schema of system tables.

10. **Do you need DPA/consent artifacts or field-level PII flags for privacy, and should transcripts be redacted before or after persistence?**
    **Recommendation:** Add field-level `pii: {category: enum, redact: boolean}` in schema; run redaction pre-persistence for transcripts; store raw only if `allowRaw=true` per form with stricter access audit and shorter retention.

##### Answers 3:

1. Yes Include it to the MVP (It isn't API only). In fact it should be first thing to implement
2. Define a minimal lifecycle without drafts for MVP: `collecting` (in-session, ephemeral) → `submitted` (persisted); reject partial saves and require atomic `POST /forms/{uuid}/submission`.
3. Last Save is the only one that matters.
4. Provide: `GET /forms/{uuid}/submission` handle only single submission in MVP
5. MVP: fail-fast on validation errors; reject unknown fields when AI-fields are disabled; provide `details[]` with JSON Pointer paths for each violation.
6. Use UUIDv7 for `submissionId`; allow `actorId` as opaque string (from JWT `sub`), max 128 chars; validate and log both with correlation header `X-Request-Id`.
7. Ship a per-form `llmConfig` with safe defaults: `temperature=0.2`, `maxClarifications=2`, `maxTokens=1024`, and a hard token budget; on budget exhaust, degrade to rules-based form fill only and emit a quota event.
8. Instrument: request rate, 95p/99p latency, error rate by code, completion time, clarification turns, validation pass rate, token usage per form; publish SLOs for availability (99.9%), p95 latency (<1.5s API), and data retention compliance.
9. Separate dev/stage/prod with distinct KMS keys and JWKS URLs; manage config via sealed secrets; add health checks and readiness probes; require change tickets for schema of system tables.
10. No redaction is needed.

##### Questions 4:

1. **What exact scope of the MVP UI do you expect (hosted widget, embeddable SDK, or both) and supported platforms (web only vs. mobile web)?**
   **Recommendation:** Start with a hosted web widget + lightweight embeddable script; expose a single `<div id="xput-form" data-form-uuid="...">` mount API to cover both hosted and embedded use cases.

2. **How will end-user sessions be identified during `collecting` (cookie, JWT, or ephemeral `sessionId`) and what is the session timeout?**
   **Recommendation:** Issue an ephemeral `sessionId` (HTTP-only cookie) scoped to a form; expire after 30 minutes of inactivity; require re-auth (JWT) on transition to `submitted`.

3. **“Last Save is the only one that matters” — should a newer submission overwrite the previous one, and how to prevent race conditions?**
   **Recommendation:** Define `PUT /forms/{uuid}/submission` as idempotent upsert; require `If-Match: <etag>` to guard against concurrent writes; server resolves by `occurredAt` with tie-break on UUIDv7.

4. **Since `GET /forms/{uuid}/submission` returns only one record, which one is canonical (latest successful, or last valid by schema)?**
   **Recommendation:** Always return the latest `submitted` that passed validation; include headers `ETag`, `X-Submission-Id`, and `X-Occurred-At` for cacheability and traceability.

5. **What UX patterns are required in the two-pane layout (validation surfacing, inline tips, example templates, keyboard navigation, screen reader flow)?**
   **Recommendation:** Show inline field errors with JSON Pointer mapping; provide one-click “Apply AI suggestion” chips; ensure full keyboard traversal and ARIA live regions for chat updates to meet WCAG 2.2 AA.

6. **How will authentication flow work in the UI (where does the JWT come from and when is it refreshed)?**
   **Recommendation:** Accept BYO JWT via `Authorization: Bearer`; validate on page load and before submit; support `401` refresh hook for integrators to reissue tokens without losing session.

7. **Should server-to-server reads remain mandatory for fetch despite having a UI (to avoid exposing PII via browser fetches)?**
   **Recommendation:** Keep reads server-to-server only for MVP; the UI should only create/overwrite submissions; partners fetch data server-side via integration token to minimize client PII exposure.

8. **What localization and formatting are needed (languages, date/time/number formats, time zone handling)?**
   **Recommendation:** Start with English + configurable locale; normalize date/time inputs to ISO 8601 (UTC) at submit; render using user locale in UI; include `Accept-Language` and `tz` hints.

9. **Given no transcript redaction, who can access `rawTranscript` and how will you gate it?**
   **Recommendation:** Restrict `rawTranscript` to server-to-server fetch with elevated scope; mask it by default in logs/metrics; add an explicit `include=transcript` query param with audit logging.

10. **What observability surfaces will integrators receive (dashboards, webhooks later, or API-only metrics)?**
    **Recommendation:** For MVP, expose `GET /forms/{uuid}/metrics` with aggregates (completions, p95/p99 latency, clarifications, validation pass rate, token usage) and a daily rollup; add a simple hosted dashboard in a post-MVP.

##### Answers 4:

1. Start with a hosted web widget + lightweight embeddable script; expose a single `<div id="xput-form" data-form-uuid="...">` mount API to cover both hosted and embedded use cases.
2. Only JWT
3. newer submission overwrite the previous one
4. Always return the latest `submitted` that passed validation
5. Show inline field errors with JSON Pointer mapping; provide one-click “Apply AI suggestion” chips; ensure full keyboard traversal and ARIA live regions for chat updates to meet WCAG 2.2 AA.
6. Accept BYO JWT via `Authorization: Bearer`; validate on page load and before submit; support `401` refresh hook for integrators to reissue tokens without losing session.
7. UI should only create/overwrite submissions and reads form definition `GET /forms/{uuid}`. 
8. Start with English + configurable locale; normalize date/time inputs to ISO 8601 (UTC) at submit; render using user locale in UI; include `Accept-Language` and `tz` hints.
9. Restrict `rawTranscript` to server-to-server fetch with elevated scope; mask it by default in logs/metrics; add an explicit `include=transcript` query param with audit logging.
10. For MVP, expose `GET /forms/{uuid}/metrics` with aggregates (completions, p95/p99 latency, clarifications, validation pass rate, token usage) and a daily rollup; add a simple hosted dashboard in a post-MVP.


Questions 5:

1. **Where will the hosted widget and embeddable script be served from, and what are the CORS/CSP rules?**
   **Recommendation:** Host on a dedicated subdomain (e.g., `widget.xput.app`) with strict CSP (`default-src 'none'; connect-src https://api.xput.app; frame-ancestors <allowlist>;`) and Subresource Integrity for the embed script; version assets with cache-busting.

2. **What JWT characteristics do you require (TTL, refresh strategy, clock skew tolerance)?**
   **Recommendation:** Use short-lived JWTs (15–30 min), allow ±60s skew, and define a `401 → refresh → retry` hook in the SDK; reject tokens without `iss`, `aud`, `sub`, and tenant-bound `kid` in JWKS.

3. **Given “newer submission overwrites the previous one,” do you still need a recoverable audit trail for compliance/debugging?**
   **Recommendation:** Keep an immutable submission history (write-once store) gated by elevated server-to-server scope; the UI and standard fetch still return only the latest valid submission.

4. **How should `GET /forms/{uuid}` be authorized and cached for the UI?**
   **Recommendation:** Require JWT with `scope=forms.read`; return `ETag` and `Cache-Control: max-age=60, must-revalidate`; support `If-None-Match` to minimize payload and keep clients in sync with definition updates.

5. **What exact behavior should AI field suggestions follow relative to the strict schema?**
   **Recommendation:** Keep `schema-first`; when `allow_ai_fields=false`, AI may only propose values/tips; when `true` (opt-in), tag AI-added optional fields under a distinct namespace (e.g., `_ai.suggestions`) and strip them at submit unless whitelisted by the integrator.

6. **What granularity and retention do you expect from `GET /forms/{uuid}/metrics`?**
   **Recommendation:** Support `?from=ISO&to=ISO&groupBy=day|hour`, return UTC ISO timestamps, a `nextCursor` for pagination, and a 30-day rolling retention; include totals plus series for completions, p95/p99, clarifications, validation pass rate, and token usage.

7. **How should the UI communicate and handle the per-form token budget exhaustion?**
   **Recommendation:** Show a persistent banner (“AI assistance disabled—token budget reached”), disable chat input, auto-switch to manual form mode, and emit a client event; the API returns a typed `429_TOKEN_BUDGET` with remaining reset info.

8. **How will clients avoid duplicate/competing overwrites during flaky networks or double-clicks?**
   **Recommendation:** Require a client-generated `submissionClientId` (UUIDv7) and accept an `Idempotency-Key`; the server de-duplicates within a short window (e.g., 10 minutes) and returns the latest persisted result.

9. **What additional browser security controls are needed for the embedded widget (clickjacking, XSS, third-party cookies)?**
   **Recommendation:** Deliver the widget in a sandboxed iframe with `frame-ancestors` allowlist, `X-Frame-Options: DENY` on non-widget routes, postMessage API with strict origin checks, and no third-party cookies (use same-site, HTTP-only cookies only if absolutely required).

10. **What engineering resources and release readiness artifacts will you commit for MVP?**
    **Recommendation:** Staff at minimum PM + BE + FE + DevOps; publish an OpenAPI spec, SDK (TypeScript), a mock server, Postman collection, three reference form definitions, and E2E tests (happy path, token budget reached, validation fail, JWT refresh).

Answers 5:

1. Host it all in one keep it as simple as possible
2. Use short-lived JWTs (1-2h), allow ±60s skew, and define a `401 → refresh → retry` hook in the SDK; reject tokens without `iss`, `aud`, `sub`, and tenant-bound `kid` in JWKS.
3. Keep an immutable submission history (write-once store) gated by elevated server-to-server scope; the UI and standard fetch still return only the latest valid submission.
4. Require JWT; return `ETag` and `Cache-Control: max-age=60, must-revalidate`; support `If-None-Match` to minimize payload and keep clients in sync with definition updates.
5. Keep `schema-first`; when `allow_ai_fields=false`, AI may only propose values/tips; when `true` (opt-in), tag AI-added optional fields under a distinct namespace (e.g., `_ai.suggestions`) and strip them at submit unless whitelisted by the integrator.
6. Support `?from=ISO&to=ISO&groupBy=day|hour`, return UTC ISO timestamps, a `nextCursor` for pagination, and a 30-day rolling retention; include totals plus series for completions, p95/p99, clarifications, validation pass rate, and token usage.
7. Show a persistent banner (“AI assistance disabled—token budget reached”), disable chat input, auto-switch to manual form mode, and emit a client event; the API returns a typed `429_TOKEN_BUDGET` with remaining reset info.
8. Require a client-generated `submissionClientId` (UUIDv7) and accept an `Idempotency-Key`; the server de-duplicates within a short window (e.g., 10 minutes) and returns the latest persisted result.
9. Deliver the widget in a sandboxed iframe with `frame-ancestors` allowlist, `X-Frame-Options: DENY` on non-widget routes, postMessage API with strict origin checks, and no third-party cookies (use same-site, HTTP-only cookies only if absolutely required).
10. Staff at minimum PM + BE + FE + DevOps; publish an OpenAPI spec, SDK (TypeScript), a mock server, Postman collection, three reference form definitions, and E2E tests (happy path, token budget reached, validation fail, JWT refresh).

Questions 6

1. **Since everything is hosted on one domain, what is the target hostname and how strict should CSP be without breaking the widget?**
   **Recommendation:** Use a single origin (e.g., `app.xput.app`) with a tight CSP: `default-src 'none'; script-src 'self' 'strict-dynamic'; style-src 'self' 'unsafe-inline'; connect-src 'self'; img-src 'self' data:; frame-ancestors <allowlist>;` and immutable asset versioning (`/assets/app.<hash>.js`).

2. **Who issues refreshed JWTs and how should the SDK’s `401 → refresh → retry` hook interface look?**
   **Recommendation:** Define a pluggable `onAuthNeeded(): Promise<string>` callback the integrator implements; SDK pauses requests, calls the hook, swaps token, replays the last request exactly once, then fails fast if still `401`.

3. **What is the retention policy for the immutable submission history and who can access it?**
   **Recommendation:** Keep history for 90 days by default (configurable per form), accessible only via server-to-server scope `submissions.history:read`; expose `GET /forms/{uuid}/submission/history?cursor&limit` returning metadata only (no PII fields) unless `include=data` is explicitly passed.

4. **Should the widget enforce a maximum bundle size and performance budget (TTFB/TTI) for MVP?**
   **Recommendation:** Cap JS at ≤ 120 KB gzipped, first interaction ≤ 2.5s on 4G, and lazy-load AI/chat modules; ship a single-file embed with integrity hash and defer non-critical CSS.

5. **What are the canonical examples for JSON Schema-lite and validation error messaging?**
   **Recommendation:** Publish a schema cookbook with 5–7 examples (contact form, address, invoice line, date range, enum multiselect) and map each violation to concise user-facing copy + `details[]` JSON Pointer for integrators.

6. **How should the AI orchestrator map schema fields to conversational turns (ordering, grouping, clarifications) given `maxClarifications=2`?**
   **Recommendation:** Implement a deterministic planner: group required fields first by semantic clusters (e.g., “contact → company → dates”), ask one cluster per turn, and emit a single structured summary before submit; fall back to manual form after 2 clarifications.

7. **In “manual-only” mode when token budget is exhausted, should any assistive logic remain (format masks, basic autocomplete)?**
   **Recommendation:** Keep non-LLM assists: input masks, client-side validation, and deterministic suggestions from schema enums; show a persistent banner with a link to integrator docs on raising the budget.

8. **How will rate limits interact with idempotent upserts and retries from flaky clients?**
   **Recommendation:** Exclude repeated `Idempotency-Key` hits from rate counting within a 10-minute window; include `X-RateLimit-*` headers and a `429_RETRY_WINDOW` hint so clients back off gracefully.

9. **What indexes/partitions are needed in storage for single-tenant but high-volume forms?**
   **Recommendation:** Primary index on `(formUuid)` with a covering secondary index on `(formUuid, updatedAt DESC)`; store latest pointer in a small `forms_latest_submission` table to accelerate `GET /forms/{uuid}/submission`.

10. **What release gates and checklists define MVP readiness (security, accessibility, docs)?**
    **Recommendation:** Require: OWASP ASVS L2 review, dependency audit clean, WCAG 2.2 AA audit on widget, load test to 10 RPS/token with burst 20, OpenAPI + Postman + mock server published, and a runbook (JWT refresh, token budget, incident contacts).

Answers 6

1. Use a single origin (e.g., `app.xput.app`) with a tight CSP: `default-src 'none'; script-src 'self' 'strict-dynamic'; style-src 'self' 'unsafe-inline'; connect-src 'self'; img-src 'self' data:; frame-ancestors <allowlist>;` and immutable asset versioning (`/assets/app.<hash>.js`).
2. Define a pluggable `onAuthNeeded(): Promise<string>` callback the integrator implements; SDK pauses requests, calls the hook, swaps token, replays the last request exactly once, then fails fast if still `401`.
3. Keep history for 30 days by default (configurable per form), accessible only via server-to-server scope `submissions.history:read`; expose `GET /forms/{uuid}/submission/history?cursor&limit` returning metadata only (no PII fields) unless `include=data` is explicitly passed.
4. Cap JS at ≤ 120 KB gzipped, first interaction ≤ 2.5s on 4G, and lazy-load AI/chat modules; ship a single-file embed with integrity hash and defer non-critical CSS.
5. Publish a schema cookbook with 5–7 examples (contact form, address, invoice line, date range, enum multiselect) and map each violation to concise user-facing copy + `details[]` JSON Pointer for integrators.
6. Implement a deterministic planner: group required fields first by semantic clusters (e.g., “contact → company → dates”), ask one cluster per turn, and emit a single structured summary before submit; fall back to manual form after 2 clarifications.
7. Keep non-LLM assists: input masks, client-side validation, and deterministic suggestions from schema enums; show a persistent banner with a link to integrator docs on raising the budget.
8. Exclude repeated `Idempotency-Key` hits from rate counting within a 10-minute window; include `X-RateLimit-*` headers and a `429_RETRY_WINDOW` hint so clients back off gracefully.
9.  Primary index on `(formUuid)` with a covering secondary index on `(formUuid, updatedAt DESC)`; store latest pointer in a small `forms_latest_submission` table to accelerate `GET /forms/{uuid}/submission`.
10. Require: OWASP ASVS L2 review, dependency audit clean, WCAG 2.2 AA audit on widget, load test to 10 RPS/token with burst 20, OpenAPI + Postman + mock server published, and a runbook (JWT refresh, token budget, incident contacts).

---

You are an AI assistant whose task is to summarize a conversation about PRD (Product Requirements Document) planning for MVP and prepare a concise summary for the next development stage. In the conversation history you will find the following information:
1. Project description
2. Identified user problem
3. Conversation history containing questions and answers
4. Recommendations regarding PRD content

Your task is to:
1. Summarize the conversation history, focusing on all decisions related to PRD planning.
2. Match the model's recommendations to the answers given in the conversation history. Identify which recommendations are relevant based on the discussion.
3. Prepare a detailed conversation summary that includes:
   a. Main functional requirements of the product
   b. Key user stories and usage paths
   c. Important success criteria and ways to measure them
   d. Any unresolved issues or areas requiring further clarification
4. Format the results as follows:

<conversation_summary>
<decisions>
[List decisions made by the user, numbered].
</decisions>

<matched_recommendations>
[List of the most relevant recommendations matched to the conversation, numbered]
</matched_recommendations>

<prd_planning_summary>
[Provide a detailed summary of the conversation, including the elements listed in step 3].
</prd_planning_summary>

<unresolved_issues>
[List any unresolved issues or areas requiring further clarification, if any exist]
</unresolved_issues>
</conversation_summary>

The final result should contain only content in markdown format. Ensure that your summary is clear, concise, and provides valuable information for the next stage of creating the PRD.


Summary:

<conversation_summary> <decisions>

1. MVP includes a hosted web widget + lightweight embeddable script; single-origin hosting (e.g., `app.xput.app`) with tight CSP and immutable asset versioning.
2. Two-pane UI (chat + dynamic form/summary) is in scope and prioritized first.
3. API lifecycle is minimal: `collecting` (ephemeral) → `submitted` (persisted); only the last valid submission matters and overwrites previous.
4. Single-tenant model.
5. Form definition uses a hybrid spec: `{ metadata, schema (JSON Schema-lite), guidance (AI prompt) }`.
6. JSON Schema-lite supports: `type`, `enum`, `format (email, uri, date, datetime)`, `minLength`, `maxLength`, `minimum`, `maximum`, `pattern`, `items`, `required`, `properties`.
7. Submission envelope: `submissionId (UUIDv7)`, `formUuid`, `actorId` (from JWT `sub`, ≤128 chars), `occurredAt`, `schemaVersion` (implicit as new form), `aiVersion` (implicit), `confidence`, `rawTranscript`, `normalizedPayload`.
8. “Schema-first” by default; optional AI-added fields only when `allow_ai_fields=true`, tagged under `_ai.suggestions` and stripped unless whitelisted.
9. JWT-only auth (BYO IdP via tenant JWKS): short-lived (1–2h), ±60s skew; SDK provides `401 → refresh → retry` via `onAuthNeeded(): Promise<string>`.
10. Server-to-server reads for saved data; UI only creates/overwrites submissions and reads form definition via `GET /forms/{uuid}`.
11. Fetch: `GET /forms/{uuid}/submission` returns the latest valid submission; include `ETag`, caching (`max-age=60`, `must-revalidate`), and `If-None-Match` support.
12. Historical submissions stored immutably for 30 days (configurable); access gated by elevated scope; metadata-only by default with optional `include=data`.
13. No files in MVP; no autosave/drafts.
14. Rate limits: 10 RPS/token (burst 20); schema limit 256 KB; transcript cap 8k tokens; exclude repeated `Idempotency-Key` calls from rate counting within 10 minutes.
15. Idempotency & de-dupe: client provides `submissionClientId` (UUIDv7) and `Idempotency-Key`; server deduplicates within ~10 minutes.
16. Token budget per form: on exhaustion, show banner, disable chat, auto-switch to manual form mode; API returns `429_TOKEN_BUDGET` with reset info.
17. LLM config defaults per form: `temperature=0.2`, `maxClarifications=2`, `maxTokens=1024`, hard token budget; deterministic planner groups required fields, summary before submit; after 2 clarifications → manual form.
18. Localization: start with English; configurable locale; normalize dates to ISO 8601 (UTC); honor `Accept-Language` and `tz`.
19. Privacy & security: encrypt at rest (KMS), access-audited transcripts; no redaction required; mask transcripts in logs/metrics by default and require explicit `include=transcript`.
20. Observability & metrics: instrument request rate, p95/p99 latency, error rate, completion time, clarification turns, validation pass rate, token usage; expose `GET /forms/{uuid}/metrics?from&to&groupBy=day|hour` with rollups and 30-day retention.
21. SLOs: availability 99.9%, API p95 < 1.5s, data retention compliance.
22. Storage indexing: primary `(formUuid)`, secondary `(formUuid, updatedAt DESC)`, plus `forms_latest_submission` pointer table.
23. Widget performance budget: bundle ≤120 KB gzipped; first interaction ≤2.5s on 4G; lazy-load AI modules; single-file embed with SRI.
24. Security headers & embedding: sandboxed iframe, strict `frame-ancestors` allowlist, `X-Frame-Options: DENY` on non-widget routes, strict `postMessage` origin checks; no third-party cookies.
25. Error taxonomy: `400_VALIDATION`, `401_UNAUTHORIZED`, `403_FORBIDDEN`, `404_NOT_FOUND`, `409_CONFLICT`, `422_UNPROCESSABLE`, `429_RATE_LIMIT` (+ `429_TOKEN_BUDGET`), `5xx`; machine-readable `details[]` with JSON Pointers; i18n keys; WCAG 2.2 AA.
26. Environments & ops: dev/stage/prod with distinct KMS/JWKS; sealed secrets; health/readiness probes; change control for schema tables.
27. Delivery plan: three sprints (core APIs/auth/storage/observability → AI orchestration/validation/metrics → minimal UI SDK/hardening/load & cost tests).
28. Release gates: OWASP ASVS L2, dependency audit clean, WCAG 2.2 AA audit, load test to 10 RPS/token (burst 20), OpenAPI + Postman + mock server, runbook (JWT refresh, token budget, incidents).

    </decisions>

<matched_recommendations>

1. Hybrid form spec with strict schema + free-form guidance.
2. Schema-first with optional AI field proposals under `_ai.suggestions`.
3. Standardized submission envelope with UUIDv7 and observability fields.
4. Minimal lifecycle without drafts; atomic submission with last-write-wins.
5. Hosted widget + embeddable script; two-pane UI with inline errors and “Apply AI suggestion” chips.
6. JWT-only auth with pluggable refresh hook in SDK.
7. Server-to-server fetch for data; UI limited to create/overwrite and read form definition.
8. Clear metrics and SLOs; expose metrics API with grouped rollups.
9. Token budget handling: degrade to manual mode and typed error response.
10. Rate limiting, idempotency, and de-duplication strategy.
11. Privacy posture (KMS encryption, audit) and transcript access gating.
12. Tight CSP and iframe sandboxing + origin checks for embed security.
13. Performance budget for widget and lazy loading.
14. Storage indexing and “latest submission” pointer optimization.
15. Release checklist including security, accessibility, load tests, and documentation assets.
    </matched_recommendations>

<prd_planning_summary>
**a. Main functional requirements**

* **Form management (API):** Create form (returns `formUuid`) using hybrid spec; read form (`GET /forms/{uuid}` with caching/ETag).
* **Data collection (UI + API):** Two-pane widget (chat + dynamic form/summary) mounted via single script; conversation planner handles required fields first, up to 2 clarifications, then manual form; client-side validation and masks.
* **Submission handling:** Atomic overwrite of prior submission; client sends `submissionClientId` + `Idempotency-Key`; server dedupes; latest valid submission is canonical; history persisted (30 days) for audits.
* **Fetching data:** `GET /forms/{uuid}/submission` returns latest valid submission (server-to-server); optional history endpoint with scoped access.
* **AI orchestration:** Per-form `llmConfig` defaults; token budget enforced; on exhaustion, AI disabled and form switches to manual mode.
* **Security & privacy:** JWT-only auth via tenant JWKS; KMS encryption; transcripts accessible only via elevated scope with explicit inclusion; tight CSP, iframe sandboxing, origin-checked messaging.
* **Rate limits & quotas:** 10 RPS/token (burst 20); schema ≤256 KB; transcript ≤8k tokens; quota/429 semantics with retry hints; exclude repeated idempotent requests from rate counting.
* **Observability & metrics:** Event instrumentation; `GET /forms/{uuid}/metrics` with day/hour grouping, `nextCursor`, 30-day retention; SLOs (99.9% availability, p95 <1.5s).
* **Performance & accessibility:** Widget ≤120 KB gzipped; TTI ≤2.5s on 4G; WCAG 2.2 AA; internationalization ready; ISO 8601 UTC normalization.
* **Environments & ops:** Dev/stage/prod isolation; sealed secrets; health checks; change control; runbook; OpenAPI + SDK + mock server + Postman collection.

**b. Key user stories & usage paths**

1. **Integrator creates a form** using hybrid spec → receives `formUuid`.
2. **End user opens widget** embedded on partner site → JWT validated → chat begins; planner asks grouped questions; inline validation and suggestion chips.
3. **User submits** → server validates schema (fail-fast) → persists as latest submission (UUIDv7) → confirmation UI; prior submission overwritten.
4. **Integrator fetches submission** server-side → `GET /forms/{uuid}/submission` returns latest valid; may query history (scoped) or metrics for monitoring.
5. **Token budget reached** → user sees banner; chat disabled; manual form remains usable; integrator may raise budget later.
6. **JWT expires** during session → SDK invokes `onAuthNeeded()` → refresh → retry once; if still `401`, show actionable error.

**c. Success criteria & measurement**

* **User efficiency:** Median completion ≤2 minutes; ≤2 clarification turns; ≥80% first-pass validity; ≤10% abandonment (instrumented via events).
* **Reliability:** API p95 <1.5s; availability 99.9%; <1% error rate per day; successful idempotent deduping rate >99.5%.
* **Cost & usage control:** Token usage within budget per form; clear `429_TOKEN_BUDGET` events; ratio of manual-mode completions tracked.
* **Security/compliance:** OWASP ASVS L2 pass; access audits for transcripts; CSP & sandbox checks; dependency audits clean.
* **Performance:** Widget bundle ≤120 KB; first interaction ≤2.5s on 4G.

**d. Areas requiring further clarification integrated into planning**

* Canonical examples for the schema cookbook and error copy are to be authored (agreed as deliverables).
* Exact values for some UI text (banner copy, error strings) and accessible labels to be finalized in i18n files.
* Final decision on metrics API pagination window size and maximum range per request left flexible but to be specified in API docs.

</prd_planning_summary>

<unresolved_issues>

1. Exact CSP `frame-ancestors` allowlist values for embedding partners.
2. Concrete time window for idempotency de-duplication (e.g., exactly 10 minutes vs. configurable).
3. Specifics of metrics API pagination limits (page size defaults, max range per query).
4. Final wording for user-facing errors, validation messages, and the token-budget banner (i18n keys).
5. Any need for non-English locales in near term and the localization rollout plan.
6. Whether to expose a minimal hosted dashboard in MVP or defer strictly to the metrics API.
   </unresolved_issues>
   </conversation_summary>

   
Questions:

1. What specific customer segments and primary jobs-to-be-done are we targeting for the MVP (e.g., SaaS onboarding, lead capture, support intake), and which two use cases are P0?
   Recommendation: Define 2–3 personas and write one-line JTBD + success metrics for each; gate MVP scope to P0 use cases with scenario-based acceptance criteria.

2. How will embedding partners be onboarded to the CSP `frame-ancestors` allowlist, and do we need per-tenant overrides?
   Recommendation: Ship a self-serve allowlist flow (per-tenant domains + wildcard policy with review), expose `GET/PUT /tenants/{id}/embed-origins`, and add runtime checks with clear error UI when an origin is not allowed.

3. What is the exact idempotency de-duplication window we want (fixed 10 minutes vs. configurable), and should it vary by form risk profile?
   Recommendation: Start with a fixed **10-minute** window platform-wide; make it configurable per form in v1.1 and return the effective window in `Rate-Limit-*` style headers.

4. What pagination defaults and maximum query range should the Metrics API enforce for reliability and cost?
   Recommendation: Set `pageSize=500` default (max 1,000), `maxRange=31 days`, `groupBy=hour|day`, and return `nextCursor` + `approxTotal` to guide dashboards.

5. What is the desired UX and copy for token-budget exhaustion (banner, disabled chat, manual mode), and what recovery paths should be shown?
   Recommendation: Standardize a banner with `Retry-After` + next reset timestamp, a “Continue in Form-Only Mode” CTA, and a “Contact Owner” link; log a distinct `token_budget_exhausted` event with tenant/form IDs.

6. Which locales (if any) beyond English are in scope this quarter, and do we need RTL or locale-specific validation (names, phone formats)?
   Recommendation: Launch **en-US** only; internationalize all user-visible strings now, normalize dates to ISO 8601 (UTC), and schedule **pl-PL** as the first expansion with a content freeze + translation QA in Q+1.

7. Do we want a minimal hosted dashboard in MVP for non-engineers (metrics overview + latest submission status), or will API-only access suffice?
   Recommendation: If at least one design partner is non-technical, include a **read-only dashboard** (metrics charts, token usage, recent errors) with tenant-scoped JWT; otherwise defer and provide a ready-made Postman collection + sample Grafana dashboard.

8. How will `_ai.suggestions` be governed—who approves field whitelisting, and how are suggestions surfaced and audited?
   Recommendation: Require explicit integrator whitelisting per field key; show “Apply AI suggestion” chips with change diffs; persist an audit trail (`approved_by`, timestamp) and include an `_ai.approvals[]` array in the envelope.

9. Which LLM providers/regions are allowed (data residency, PII handling), and what is our fallback/timeout/retry policy under load or provider incidents?
   Recommendation: Abstract provider via a policy-driven router (primary: regionally compliant model; fallback: cheaper model); set **2× retry with jitter**, **4s planning timeout**, circuit breaker per tenant, and redact/mask PII in prompts unless `include=transcript` is explicitly set.

10. What are our target platform constraints (browser support, mobile web behavior) and performance enforcement mechanisms?
    Recommendation: Support last 2 versions of Chromium/Firefox/Safari + Safari ≥16; enforce the **≤120 KB gzipped** budget with CI bundle guardrails, ship RUM beacons for TTI/CLS, and add synthetic checks (4G profile) to fail builds above **2.5s** first interaction.


Answers:

1. Define 2–3 personas and write one-line JTBD + success metrics for each; gate MVP scope to P0 use cases with scenario-based acceptance criteria.
2. Single tenant only in MVP
3. Fixed 10 minutes
4. Set `pageSize=500` default (max 1,000), `maxRange=31 days`, `groupBy=hour|day`, and return `nextCursor` + `approxTotal` to guide dashboards.
5. Standardize a banner with `Retry-After` + next reset timestamp, a “Continue in Form-Only Mode” CTA, and a “Contact Owner” link; log a distinct `token_budget_exhausted` event with tenant/form IDs.
6. Launch **en-US** only; internationalize all user-visible strings now, normalize dates to ISO 8601 (UTC)
7. No we dont need it in MVP
8. Require explicit integrator whitelisting per field key; show “Apply AI suggestion” chips with change diffs; persist an audit trail (`approved_by`, timestamp) and include an `_ai.approvals[]` array in the envelope.
9. Abstract provider via a policy-driven router (primary: regionally compliant model; fallback: cheaper model); set **2× retry with jitter**, **4s planning timeout**, circuit breaker per tenant, and redact/mask PII in prompts unless `include=transcript` is explicitly set.
10. Support last 2 versions of Chromium/Firefox/Safari + Safari ≥16; enforce the **≤120 KB gzipped** budget with CI bundle guardrails, ship RUM beacons for TTI/CLS, and add synthetic checks (4G profile) to fail builds above **2.5s** first interaction.


Questions:

1. What concrete P0 use cases (2 max) do we commit to first launch (e.g., “Qualified Lead Capture” and “Trial Onboarding”) and which fields are mandatory for each?
   Recommendation: Freeze two canonical schemas + guidance now; add scenario-based acceptance tests and golden payloads to CI.

2. How will we manage the `frame-ancestors` allowlist for embedding—static list per tenant vs. runtime API?
   Recommendation: For MVP single-tenant, ship a static allowlist in config + env var; document change procedure and add a runtime health check that fails closed on mismatch.

3. What exact JWT lifetime and clock skew do we enforce (within the stated 1–2h, ±60s), and do we require `aud`/`iss` checks?
   Recommendation: Set **TTL=90m**, skew **±60s**, require `aud` and `iss` validation, and log structured auth failures with redaction.

4. Which scopes/roles are required to access Metrics and History APIs, and how are tokens provisioned for server-to-server calls?
   Recommendation: Define two scopes now: `submissions.history:read` and `metrics:read`; issue tenant-scoped service tokens with rotation via JWKS kid rollover.

5. What ETag/TTL and cache-busting rules apply to `GET /forms/{uuid}` and `GET /forms/{uuid}/submission`?
   Recommendation: Keep `max-age=60` with `must-revalidate`; bump schema `version` on any change and include it in ETag; document integrator caching examples.

6. What is the minimum viable event schema for observability (names, required fields), and which events are P0?
   Recommendation: Standardize P0 events: `session_started`, `field_validated`, `clarification_prompted`, `submission_saved`, `token_budget_exhausted`, `auth_refreshed`, `rate_limited`; include tenant/form/session IDs and timestamps (ISO 8601 UTC).

7. What privacy posture do we state in docs (PII categories allowed, DPA needs), given transcripts are gated but not redacted by default?
   Recommendation: Publish a data handling matrix; default to masking emails/phones in logs; require DPA sign-off before enabling transcript inclusion via scope flag.

8. What error copy and i18n keys do we finalize for validation, auth failure, token budget, and rate limit screens?
   Recommendation: Author a small error copy spec now with key → default-string → accessibility note; add to SDK as a replaceable dictionary.

9. Do we need guardrails for AI prompt “guidance” (max length, prohibited tokens), and how do we validate `allow_ai_fields=true` usage?
   Recommendation: Cap guidance at **8k chars**, lint for secrets/PII patterns at publish-time, and require a non-empty whitelist before allowing AI fields in production.

10. What release-readiness exit criteria and owners do we set per sprint (e.g., security, load, accessibility), and who signs off?
    Recommendation: Create a RACI per gate (OWASP ASVS L2, WCAG 2.2 AA, load @ 10 RPS/token, dependency audit clean); assign DRIs and add a checklist to CI that blocks merge on failures.


Answers:

1. Freeze two canonical schemas + guidance now; add scenario-based acceptance tests and golden payloads to CI.
2. For MVP single-tenant, ship a static allowlist in config + env var; document change procedure and add a runtime health check that fails closed on mismatch.
3. Set **TTL=90m**, skew **±60s**, require `aud` and `iss` validation, and log structured auth failures with redaction.
4. Metrics only on database - no acces via API
5. Keep `max-age=60` with `must-revalidate`; bump schema `version` on any change and include it in ETag; document integrator caching examples.
6. Standardize P0 events: `session_started`, `field_validated`, `clarification_prompted`, `submission_saved`, `token_budget_exhausted`, `auth_refreshed`, `rate_limited`; include tenant/form/session IDs and timestamps (ISO 8601 UTC).
7.  Publish a data handling matrix; default to masking emails/phones in logs; require DPA sign-off before enabling transcript inclusion via scope flag.
8. Author a small error copy spec now with key → default-string → accessibility note; add to SDK as a replaceable dictionary.
9.  Cap guidance at **8k chars**, lint for secrets/PII patterns at publish-time, and require a non-empty whitelist before allowing AI fields in production.
10. Create a RACI per gate (OWASP ASVS L2, WCAG 2.2 AA, load @ 10 RPS/token, dependency audit clean); assign DRIs and add a checklist to CI that blocks merge on failures.


---

You are an AI assistant whose task is to summarize a conversation about PRD (Product Requirements Document) planning for MVP and prepare a concise summary for the next development stage. In the conversation history you will find the following information:
1. Project description
2. Identified user problem
3. Conversation history containing questions and answers
4. Recommendations regarding PRD content

Your task is to:
1. Summarize the conversation history, focusing on all decisions related to PRD planning.
2. Match the model's recommendations to the answers given in the conversation history. Identify which recommendations are relevant based on the discussion.
3. Prepare a detailed conversation summary that includes:
   a. Main functional requirements of the product
   b. Key user stories and usage paths
   c. Important success criteria and ways to measure them
   d. Any unresolved issues or areas requiring further clarification
4. Format the results as follows:

<conversation_summary>
<decisions>
[List decisions made by the user, numbered].
</decisions>

<matched_recommendations>
[List of the most relevant recommendations matched to the conversation, numbered]
</matched_recommendations>

<prd_planning_summary>
[Provide a detailed summary of the conversation, including the elements listed in step 3].
</prd_planning_summary>

<unresolved_issues>
[List any unresolved issues or areas requiring further clarification, if any exist]
</unresolved_issues>
</conversation_summary>

The final result should contain only content in markdown format. Ensure that your summary is clear, concise, and provides valuable information for the next stage of creating the PRD.


Summary:


You are an experienced product manager whose task is to create a comprehensive Product Requirements Document (PRD) based on the following descriptions:

<project_description>
- User (probably external system) creates a from. In return it receives a UUID of the form. Payload of this request is a definition of form.
- Form definition is a prompt to AI with required data description (some form of definition what we need to get from user).
- User that is identified by e.x. JWT token can use this form (by UUID) to enter data.
- Using added prompt to Handle AI conversation with user to gather data - AI should have some tools to generate simple form (possible fields will be defined by system).
- Persist gathered data in some storage with relation to added propmpt (form definition).
- Fetching data from storage (probably external system) by UUID of the form..
</project_description>

<project_details>
<conversation_summary> <decisions>

1. MVP ships a hosted web widget + lightweight embeddable script on a single origin with tight CSP and SRI.
2. Two-pane UI (chat + dynamic form/summary) is P0 and prioritized.
3. API lifecycle: `collecting` (ephemeral) → `submitted` (persisted); last valid submission overwrites previous.
4. Single-tenant only in MVP.
5. Form spec is hybrid: `{ metadata, schema (JSON Schema-lite), guidance (AI prompt) }`.
6. JSON Schema-lite supports: `type`, `enum`, `format(email|uri|date|datetime)`, `minLength`, `maxLength`, `minimum`, `maximum`, `pattern`, `items`, `required`, `properties`.
7. Submission envelope includes: `submissionId(UUIDv7)`, `formUuid`, `actorId (JWT sub ≤128)`, `occurredAt`, implicit `schemaVersion` and `aiVersion`, `confidence`, `rawTranscript`, `normalizedPayload`.
8. Schema-first by default; optional AI-added fields only with `allow_ai_fields=true`; proposals live under `_ai.suggestions` and are stripped unless whitelisted.
9. JWT-only auth via tenant JWKS; SDK handles `401 → onAuthNeeded() refresh → single retry`.
10. UI can create/overwrite submissions and read form definitions; saved data is server-to-server only.
11. `GET /forms/{uuid}/submission` returns latest valid submission; supports `ETag`, `If-None-Match`, `max-age=60`, `must-revalidate`.
12. Historical submissions stored immutably for 30 days (configurable); elevated scope required; metadata-only by default with optional `include=data`.
13. No file uploads and no autosave/drafts in MVP.
14. Rate limits: 10 RPS/token (burst 20); schema ≤256 KB; transcript cap 8k tokens; repeated idempotent calls excluded from rate counts within the window.
15. Idempotency & de-dupe: client sends `submissionClientId (UUIDv7)` + `Idempotency-Key`; **fixed 10-minute** de-dupe window.
16. Token budget enforcement: on exhaustion show banner, disable chat, auto-switch to manual form; API returns `429_TOKEN_BUDGET` (where applicable).
17. LLM config defaults per form: `temperature=0.2`, `maxClarifications=2`, `maxTokens=1024`; deterministic planner; after 2 clarifications → manual form.
18. Localization: **en-US only** at launch; respect `Accept-Language`/`tz`; normalize dates to ISO-8601 (UTC).
19. Privacy & security: encrypt at rest (KMS); access-audited transcripts; transcripts masked in logs/metrics by default; explicit `include=transcript` required.
20. Observability events standardized (P0 set below); 30-day retention.
21. SLOs: availability 99.9%, API p95 < 1.5s, data retention compliance.
22. Storage indexing optimized incl. `forms_latest_submission` pointer table.
23. Widget performance: bundle ≤120 KB gzipped; first interaction ≤2.5s on 4G; lazy-load AI modules; single-file embed with integrity hash.
24. Security headers & embedding: sandboxed iframe, strict `frame-ancestors` allowlist, `X-Frame-Options: DENY` on non-widget routes, strict `postMessage` origin checks; no third-party cookies.
25. Error taxonomy: `400_VALIDATION`, `401_UNAUTHORIZED`, `403_FORBIDDEN`, `404_NOT_FOUND`, `409_CONFLICT`, `422_UNPROCESSABLE`, `429_RATE_LIMIT`, `429_TOKEN_BUDGET`, `5xx`; machine-readable `details[]`; i18n keys; WCAG 2.2 AA.
26. Environments: dev/stage/prod with distinct KMS/JWKS; sealed secrets; health/readiness probes; change control for schema tables.
27. Delivery plan: three sprints (core → AI/validation/metrics → UI SDK/hardening/load & cost tests).
28. Release gates: OWASP ASVS L2, dependency audit clean, WCAG 2.2 AA audit, load to 10 RPS/token (burst 20), OpenAPI + Postman + mock server, runbook.
29. Personas finalized: Integration Engineer, Product/Growth Manager, Form Respondent; P0 use cases: Qualified Lead Capture, Trial Onboarding.
30. **Freeze two canonical schemas + guidance now**; add scenario-based acceptance tests and golden payloads to CI.
31. **CSP allowlist**: static per single-tenant via config/env; documented change procedure; runtime health check that fails closed on mismatch.
32. **JWT parameters**: TTL **90 minutes**, skew **±60s**, enforce `aud` & `iss`; log structured/redacted auth failures.
33. **Metrics access**: metrics stored in DB only for MVP (no public Metrics API).
34. **Caching rules**: keep `max-age=60` + `must-revalidate`; bump `schema.version` on any change and include in ETag; provide integrator caching examples.
35. **P0 events**: `session_started`, `field_validated`, `clarification_prompted`, `submission_saved`, `token_budget_exhausted`, `auth_refreshed`, `rate_limited`; include tenant/form/session IDs and ISO-8601 UTC timestamps.
36. **Data handling matrix**: publish; mask emails/phones in logs by default; require DPA sign-off before enabling transcript inclusion via scope flag.
37. **Error copy spec**: key → default string → accessibility note; SDK exposes replaceable dictionary.
38. **Guidance guardrails**: cap guidance at **8k chars**; lint for secrets/PII at publish-time; require non-empty whitelist before enabling AI fields in prod.
39. **LLM routing**: policy-driven router (primary: regionally compliant; fallback: cheaper); **2× retry with jitter**, **4s planning timeout**, tenant-level circuit breaker; redact/mask PII in prompts unless explicitly included.
40. **Platform constraints**: support last 2 versions of Chromium/Firefox/Safari and Safari ≥16; enforce ≤120 KB gz via CI guardrails; RUM for TTI/CLS; synthetic checks (4G) fail builds >2.5s first interaction.
41. **RACI & CI gates**: define DRIs per release gate; CI checklist blocks merges on failures.

    </decisions>

<matched_recommendations>

1. Hybrid form spec with strict schema + free-form guidance.
2. Schema-first with `_ai.suggestions` gated by whitelist and audit trail.
3. Standardized submission envelope (UUIDv7, observability fields).
4. Atomic overwrite lifecycle; no drafts in MVP.
5. Hosted widget + two-pane UI; inline errors and “Apply AI suggestion” chips.
6. JWT-only auth with refresh hook; strict `aud`/`iss` checks.
7. Server-to-server reads; UI limited to create/overwrite + read form definition.
8. Token budget handling (banner, manual fallback, distinct event).
9. Rate limiting, idempotency (fixed 10-minute window), and de-duplication.
10. Privacy posture (KMS, audit, masking; DPA for transcript inclusion).
11. Tight CSP, iframe sandboxing, origin checks; static allowlist + health check.
12. Performance budget and lazy-loading; CI bundle guardrails + synthetic checks.
13. Observability via standardized events; DB-only metrics retention.
14. Release checklist with OWASP/WCAG/load tests; RACI and CI gates.
15. Guidance guardrails (8k cap, linting, whitelist requirement).
16. LLM router with retries, timeout, circuit breaker, and PII masking.
17. Caching strategy with ETag/versioning and integrator examples.
18. Error taxonomy + SDK i18n/error copy dictionary.
19. Personas, P0 use cases, acceptance tests, and golden payloads in CI.
    </matched_recommendations>

<prd_planning_summary>
a. **Main functional requirements**

* **Form management**: Create/read forms using hybrid spec; versioned schemas with ETag; guidance capped and linted.
* **Two-pane collection**: Chat planner prioritizes required fields; live side-pane form/summary; inline validation; max 2 clarifications → manual form fallback.
* **Submission handling**: Schema validation server-side; atomic overwrite; idempotency via `submissionClientId` + `Idempotency-Key` with fixed 10-minute window.
* **Auth & security**: JWT TTL 90m, skew ±60s, enforce `aud`/`iss`; sandboxed iframe; static `frame-ancestors` allowlist; strict `postMessage` checks; no third-party cookies.
* **Observability**: Emit P0 events with tenant/form/session IDs; DB-only metrics (no Metrics API in MVP); 30-day retention.
* **Quotas & budgets**: Rate limit 10 RPS/token (burst 20); token budget banner + manual fallback; schema ≤256 KB; transcript ≤8k tokens.
* **Performance & accessibility**: Widget ≤120 KB gz; first interaction ≤2.5s on 4G; WCAG 2.2 AA; en-US only; ISO-8601 UTC normalization.
* **LLM orchestration**: Router (primary/fallback), 2× retry with jitter, 4s planning timeout, tenant circuit breaker; PII redaction unless explicitly included.

b. **Key user stories & paths**

1. **Integration Engineer** embeds widget and publishes form schema/guidance → validates via ETag/versioning → runs CI acceptance tests with golden payloads.
2. **Respondent** completes chat prompts with inline validation and optional AI suggestion chips → summary pane confirms readiness → submit.
3. **System** validates, dedupes within 10 minutes, persists latest submission, triggers webhooks (if configured), logs `submission_saved`.
4. **Owner** reviews health via DB-backed metrics dashboards (internal), monitors P0 events and token budget exhaustion; adjusts guidance/whitelists.

c. **Success criteria & measurement**

* **User efficiency**: median completion ≤2 min; ≤2 clarification turns; ≥80% first-pass validity; ≤10% abandonment.
* **Reliability**: availability 99.9%; API p95 <1.5s; de-dupe success >99.5%; auth refresh success rate tracked.
* **Cost control**: ≤2% sessions hit token budget; token usage within per-form caps.
* **Compliance/perf**: OWASP ASVS L2 pass; WCAG 2.2 AA audit pass; bundle ≤120 KB gz; TTI ≤2.5s on 4G.
* **Observability**: P0 events delivered ≥99.9%; metrics completeness in DB ≥99%.

d. **Unresolved issues / clarifications**

* **Canonical schemas content**: finalize exact fields and validation rules for the two P0 flows (Lead Capture, Trial Onboarding).
* **CSP allowlist values**: enumerate exact domains (and wildcard policy, if any) for the single tenant.
* **Webhook semantics**: confirm delivery guarantees (at-least-once), retry schedule, and auth mechanism.
* **Token budget reset source of truth**: specify calculation and timezone for banner “next reset”.
* **History retention overrides**: define per-tenant overrides and request/approval process.
* **DB-only metrics ops**: choose internal dashboarding stack, access controls, and refresh cadence; define schema for aggregations.
* **i18n roadmap**: confirm when to add next locale (e.g., pl-PL) and required content freeze/QA steps.
  </prd_planning_summary>

<unresolved_issues>

1. Exact field lists and validation for the two canonical P0 schemas.
2. Concrete `frame-ancestors` allowlist entries for the tenant.
3. Webhook delivery guarantees, retries, and authentication.
4. Token budget reset computation and display rules.
5. Policy for extending/overriding 30-day history retention.
6. Internal metrics dashboard approach and controls (since no public API).
7. Localization expansion timing and process beyond en-US.
   </unresolved_issues>
   </conversation_summary>
</project_details>

Follow these steps to create a comprehensive and well-organized document:

1. Divide the PRD into the following sections:
   a. Project Overview
   b. User Problem
   c. Functional Requirements
   d. Project Boundaries
   e. User Stories
   f. Success Metrics

2. In each section, provide detailed and relevant information based on the project description and answers to clarifying questions. Make sure to:
    - Use clear and concise language
    - Provide specific details and data as needed
    - Maintain consistency throughout the document
    - Address all points listed in each section

3. When creating user stories and acceptance criteria
    - List ALL necessary user stories, including basic, alternative, and edge case scenarios.
    - Assign a unique requirement identifier (e.g., US-001) to each user story for direct traceability.
    - Include at least one user story specifically for secure access or authentication, if the application requires user identification or access restrictions.
    - Ensure that no potential user interaction is omitted.
    - Ensure that each user story is testable.

Use the following structure for each user story:
- ID
- Title
- Description
- Acceptance Criteria

4. After completing the PRD, review it against this checklist:
    - Is each user story testable?
    - Are the acceptance criteria clear and specific?
    - Do we have enough user stories to build a fully functional application?
    - Have we included authentication and authorization requirements (if applicable)?

5. PRD Formatting:
    - Maintain consistent formatting and numbering.
    - Do not use bold formatting in markdown ( ** ).
    - List ALL user stories.
    - Format the PRD in proper markdown.

Prepare the PRD with the following structure:

```markdown
# Product Requirements Document (PRD) - {{app-name}}
## 1. Product Overview
## 2. User Problem
## 3. Functional Requirements
## 4. Product Boundaries
## 5. User Stories
## 6. Success Metrics
```

Remember to fill each section with detailed, relevant information based on the project description and our clarifying questions. Ensure the PRD is comprehensive, clear, and contains all relevant information needed for further product development.

The final output should consist solely of the PRD in the specified markdown format, which you will save in the file .ai/prd.md