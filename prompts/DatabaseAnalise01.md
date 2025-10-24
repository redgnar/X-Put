You are an AI assistant whose task is to help plan a PostgreSQL database schema for an MVP (Minimum Viable Product) based on the provided information. Your goal is to generate a list of questions and recommendations that will be used in subsequent prompting to create the database schema, relationships, and row-level security (RLS) policies.

Please carefully review the following information:

<product_requirements>
@prd.md
</product_requirements>

<tech_stack>
@tech-stack.md
</tech_stack>

Analyze the provided information, focusing on aspects relevant to database design. Consider the following issues:

1. Identify key entities and their attributes based on product requirements.
2. Determine potential relationships between entities.
3. Consider data types and constraints that may be necessary.
4. Think about scalability and performance impact.
5. Assess security requirements and their impact on database design.
6. Consider any specific PostgreSQL features that might be beneficial for the project.

Based on your analysis, generate a list of 10 questions and recommendations in a combined form (question + recommendation). These should address any ambiguities, potential issues, or areas where more information is needed to create an effective database schema. Consider questions regarding:

1. Entity relationships and cardinality
2. Data types and constraints
3. Indexing strategies
4. Partitioning (if applicable)
5. Row-level security requirements
6. Performance considerations
7. Scalability concerns
8. Data integrity and consistency

The output should have the following structure:

List your questions and recommendations here, numbered for clarity:

For example:
1. Should the `users` entity have a relationship with `posts`?

Recommendation: Yes, the `users` entity should have a relationship with `posts` because users can create posts.
</questions>

Remember that your goal is to provide a comprehensive list of questions and recommendations that will help create a solid PostgreSQL database schema for the MVP. Focus on clarity, relevance, and accuracy of your outputs. Do not include any additional comments or explanations beyond the specified output format.

Continue this process, generating new questions and recommendations based on the provided context and user responses, until the user explicitly requests a summary.

Remember to focus on clarity, relevance, and accuracy of outputs. Do not include any additional comments or explanations beyond the specified output format.


Questions:
<questions> 
1. Should we include a `tenant_id` column on all core tables even though the MVP is single-tenant? Recommendation: Yes—add `tenant_id` (UUID) to `forms`, `submissions`, `events`, and `forms_latest_submission` to future-proof for multi-tenant and enable strict RLS predicates like `tenant_id = current_setting('app.tenant_id')::uuid`. This aligns with single-origin hosting but preserves isolation for later expansion. :contentReference[oaicite:0]{index=0}

What identifiers and schema storage should forms use to support versioning and ETag?
Recommendation: Use form_uuid UUIDv7 (for time ordering), version int (increments on change), and schema JSONB with a generated etag (e.g., sha256(schema::text || version)). Add a unique (form_uuid, version) constraint; latest pointer lives elsewhere. ETag must include schema.version per PRD.

How do we model "latest overwrites prior" while keeping immutable history for 30 days?
Recommendation: Keep submissions immutable (submission_id UUIDv7, form_uuid, actor_id, occurred_at timestamptz, normalized_payload JSONB, raw_transcript TEXT), and maintain a forms_latest_submission(form_uuid PK, actor_id, submission_id, updated_at) pointer that is atomically swapped on valid submit. Enforce retention with a scheduled job to purge old history per policy.

What cardinality and constraints ensure one canonical latest per (form, actor)?
Recommendation: Add a unique index on forms_latest_submission(form_uuid, actor_id) and a FK to submissions(submission_id). In submissions, create a partial unique index to enforce idempotency per window (see #7). This supports the lifecycle “collecting → submitted (persisted); latest valid submission overwrites prior ones)”.

How should we express RLS so the UI can only create/overwrite, while server-to-server can read?
Recommendation: Enable RLS on all tables. For submissions, define:

INSERT policy requiring jwt_claims ->> 'sub' = actor_id and aud/iss checks via a SECURITY DEFINER function; no SELECT for UI role.

SELECT policies only for a service role with scopes like submissions.latest:read and submissions.history:read (optionally include=data). Use has_scope(text) helpers reading JWT.

Where do AI suggestions/approvals live, and how do we prevent accidental leakage without DPA?
Recommendation: Store user-confirmed values only in normalized_payload. Keep _ai data (suggestions, approvals, ai_version, confidence) in a separate submission_ai_meta(submission_id PK, ai_meta JSONB) table with stricter RLS: SELECT allowed only if a “transcript/data inclusion” scope is present (gated by DPA). Avoid embedding _ai under user payload by default.

How do we enforce a 10-minute de-dup window for idempotent submits?
Recommendation: Create an idempotency table with (form_uuid, actor_id, idempotency_key, created_at) and a unique index on (form_uuid, actor_id, idempotency_key); delete rows older than 10 minutes, or store an expires_at and use a partial unique index WHERE now() < expires_at. Alternatively, add these columns to submissions and enforce a partial unique (form_uuid, actor_id, idempotency_key) WHERE occurred_at >= now() - interval '10 minutes'.

What indexing and partitioning strategy supports SLOs for reads and high-volume events?
Recommendation: Follow tech-stack indices—PK on (form_uuid) where applicable and secondary (form_uuid, updated_at DESC)—and add:

submissions: index (form_uuid, actor_id, occurred_at DESC); GIN on normalized_payload for specific keys if needed.

events (P0 stream): declarative partitioning by day or month on timestamp; BRIN on timestamp plus B-tree on (form_uuid, session_id). Consider COMPRESS/TOAST and retention jobs.

How do we store and protect transcripts and PII per policy?
Recommendation: Place raw_transcript in its own table submission_transcripts(submission_id PK, transcript TEXT) encrypted at rest (KMS) with RLS allowing SELECT only when a scope flag is set (after DPA sign-off). Default logs/events must mask emails/phones; add CHECKs or triggers to scrub event payloads before insert.

What data types, limits, and validations should be enforced at the DB layer?
Recommendation: Use UUIDv7 for ordered IDs; timestamptz (UTC) for all times; JSONB for schema and normalized_payload. Add CHECKs for schema size ≤ 256 KB, transcript/token caps, and occurred_at present. Consider generated columns (e.g., extracting common fields from normalized_payload) for queryable indexes. Validate inputs against “JSON Schema-lite” in the app layer, but persist normalized, validated data only.

Answers:
1. No I prefer to keep it simple. I prefer separate DB for tenant in the future.
2. Use form_uuid UUIDv7 (for time ordering), version int (increments on change), and schema JSONB with a generated etag (e.g., sha256(schema::text || version)). Add a unique (form_uuid, version) constraint; latest pointer lives elsewhere. ETag must include schema.version per PRD.
3. Keep submissions immutable (submission_id UUIDv7, form_uuid, actor_id, occurred_at timestamptz, normalized_payload JSONB, raw_transcript TEXT), and maintain a forms_latest_submission(form_uuid PK, actor_id, submission_id, updated_at) pointer that is atomically swapped on valid submit. Enforce retention with a scheduled job to purge old history per policy.
4. Add a unique index on forms_latest_submission(form_uuid, actor_id) and a FK to submissions(submission_id). In submissions, create a partial unique index to enforce idempotency per window (see #7). This supports the lifecycle “collecting → submitted (persisted); latest valid submission overwrites prior ones)”.
5. No check for now all endpoints will check only if token is valid.
6. Keep it simple.
7. Create an idempotency table with (form_uuid, actor_id, idempotency_key, created_at) and a unique index on (form_uuid, actor_id, idempotency_key); delete rows older than 10 minutes, or store an expires_at and use a partial unique index WHERE now() < expires_at. Alternatively, add these columns to submissions and enforce a partial unique (form_uuid, actor_id, idempotency_key) WHERE occurred_at >= now() - interval '10 minutes'.
8. Follow tech-stack indices—PK on (form_uuid) where applicable and secondary (form_uuid, updated_at DESC)—and add:

submissions: index (form_uuid, actor_id, occurred_at DESC); GIN on normalized_payload for specific keys if needed.

events (P0 stream): declarative partitioning by day or month on timestamp; BRIN on timestamp plus B-tree on (form_uuid, session_id). Consider COMPRESS/TOAST and retention jobs.
9. Place raw_transcript in its own table submission_transcripts(submission_id PK, transcript TEXT) encrypted at rest (KMS) with RLS allowing SELECT only when a scope flag is set (after DPA sign-off). Default logs/events must mask emails/phones; add CHECKs or triggers to scrub event payloads before insert.
0Use UUIDv7 for ordered IDs; timestamptz (UTC) for all times; JSONB for schema and normalized_payload. Add CHECKs for schema size ≤ 256 KB, transcript/token caps, and occurred_at present. Consider generated columns (e.g., extracting common fields from normalized_payload) for queryable indexes. Validate inputs against “JSON Schema-lite” in the app layer, but persist normalized, validated data only.


Questions:

1. Do we persist `schemaVersion` and `aiVersion` alongside each submission or derive them at read time? Recommendation: Persist both as immutable columns on `submissions` (`schema_version int`, `ai_version text`) at insert to ensure reproducible reads and caching with ETag on latest submission responses. :contentReference[oaicite:0]{index=0}

Where should we store submissionClientId and Idempotency-Key to support the 10-minute de-dup window and observability?
Recommendation: Add submission_client_id UUIDV7 and idempotency_key text to submissions with a unique partial index on (form_uuid, actor_id, idempotency_key) filtered to occurred_at >= now() - interval '10 minutes', and mirror inserts into the dedicated idempotency table for fast lookups and rate-limit exclusion.

Should forms have a separate forms_current pointer or a is_current flag to locate the latest version quickly?
Recommendation: Create a forms_current(form_uuid PK, version int, updated_at) pointer table to avoid wide scans over (form_uuid, version) and keep the history table append-only. Enforce FK to forms(form_uuid, version).

How should we structure events for SLO-grade analytics and retention?
Recommendation: Define events as a partitioned table on occurred_at (monthly), with columns {event_id UUIDV7 PK, form_uuid, session_id, actor_id nullable, name enum, occurred_at timestamptz, attrs JSONB}; add BRIN on occurred_at, B-tree on (form_uuid, session_id), and a 30-day retention job.

What JSONB indexing should submissions.normalized_payload use to balance flexibility and performance?
Recommendation: Start with a GIN index on normalized_payload jsonb_path_ops; for known hot keys (e.g., email, date), add generated columns (e.g., email_generated text GENERATED ALWAYS AS ((normalized_payload->>'email')) STORED) with selective B-tree indexes.

How do we handle transcripts given “keep it simple” but policy requires gating and masking?
Recommendation: Keep submission_transcripts(submission_id PK, transcript text) as a separate table with a boolean is_included flag and default false; only set true when higher-level checks pass. Add BEFORE INSERT trigger on events to mask emails/phones in attrs.

Do we need DB-side validation artifacts to map backend details[] to UI errors?
Recommendation: Store validation_errors in a lightweight submission_errors(submission_id, path text, code text, message text, created_at) for failed attempts (not persisted to canonical submissions), enabling accurate i18n mapping and debugging while keeping final submissions clean. Purge rows older than 24h.

How should we compute and store etag for forms and latest submission responses?
Recommendation: For forms, store etag as sha256(schema::text || ':' || version) materialized on write; for latest submission, compute sha256(submission_id::text || ':' || schema_version) and persist on forms_latest_submission to support fast GET /forms/{uuid}/submission with If-None-Match.

What minimal DB roles and permissions align with “token-valid-only” checks while staying safe?
Recommendation: Create two Postgres roles: app_rw (INSERT to submissions, events; SELECT to forms, forms_current, forms_latest_submission) and svc_ro (SELECT to all read endpoints). Defer RLS for MVP but isolate DDL in a separate schema_migrations role and deny PUBLIC on all tables.

How do we enforce global limits (schema size ≤ 256 KB; transcript cap) at the DB layer to protect SLOs?
Recommendation: Add CHECK constraints on forms to enforce octet_length(schema::text) <= 262144, and on submission_transcripts to cap octet_length(transcript). Add a statement_timeout and idle_in_transaction_session_timeout at the DB for the API user to guard p95 latency.


Answers:
1. Persist both as immutable columns on `submissions` (`schema_version int`, `ai_version text`) at insert to ensure reproducible reads and caching with ETag on latest submission responses. :contentReference[oaicite:0]{index=0}
2. Add submission_client_id UUIDV7 and idempotency_key text to submissions with a unique partial index on (form_uuid, actor_id, idempotency_key) filtered to occurred_at >= now() - interval '10 minutes', and mirror inserts into the dedicated idempotency table for fast lookups and rate-limit exclusion.
3. Create a forms_current(form_uuid PK, version int, updated_at) pointer table to avoid wide scans over (form_uuid, version) and keep the history table append-only. Enforce FK to forms(form_uuid, version).
4. No we dont need events in MVP
5. No indexing JOSNB in MVP.
6. No special handling for transcripts.
7. Store validation_errors in a lightweight submission_errors(submission_id, path text, code text, message text, created_at) for failed attempts (not persisted to canonical submissions), enabling accurate i18n mapping and debugging while keeping final submissions clean. Purge rows older than 24h.
8. For forms, store etag as sha256(schema::text || ':' || version) materialized on write; for latest submission, compute sha256(submission_id::text || ':' || schema_version) and persist on forms_latest_submission to support fast GET /forms/{uuid}/submission with If-None-Match.
9. Create two Postgres roles: app_rw (INSERT to submissions, events; SELECT to forms, forms_current, forms_latest_submission) and svc_ro (SELECT to all read endpoints). Defer RLS for MVP but isolate DDL in a separate schema_migrations role and deny PUBLIC on all tables.
10. Add CHECK constraints on forms to enforce octet_length(schema::text) <= 262144, and on submission_transcripts to cap octet_length(transcript). Add a statement_timeout and idle_in_transaction_session_timeout at the DB for the API user to guard p95 latency.

Questions:

<questions>
1. What data type and length constraints should we enforce for `actor_id` to align with JWT `sub`?
Recommendation: Use `actor_id TEXT CHECK (octet_length(actor_id) <= 128)` and require it on `submissions` and `forms_latest_submission`, since `sub` is required and limited to ≤128 chars by auth policy. :contentReference[oaicite:0]{index=0}

2. Which columns and checks belong on `forms` to honor JSON Schema-lite and guidance limits?
   Recommendation: Add `{ form_uuid UUID, version INT, schema JSONB, guidance TEXT, etag TEXT, created_at, updated_at }` with `CHECK (octet_length(guidance) <= 8000)` and validate schema keywords (type, enum, format, min/max, etc.) in the app before insert/update; persist the ETag computed from `schema` + `version`.

3. How do we guarantee atomicity between a `forms` change and the `forms_current` pointer + ETag?
   Recommendation: Update `forms`, compute `etag = sha256(schema::text || ':' || version)`, and upsert `forms_current(form_uuid, version, updated_at)` in a single transaction; add FK `forms_current(form_uuid, version) → forms(form_uuid, version)` to avoid stale pointers.

4. Should `submissions` record `confidence` and enforce ISO/UTC normalization at the DB layer?
   Recommendation: Include `confidence NUMERIC(3,2) CHECK (confidence BETWEEN 0 AND 1)` and store all timestamps as `timestamptz` (UTC). Add a CHECK or write-path validation that payload date/datetime fields are ISO 8601 normalized per requirement.

5. Do we enforce referential integrity from `submissions(schema_version)` to `forms(form_uuid, version)`?
   Recommendation: Yes—persist `schema_version INT NOT NULL` on `submissions` and add an FK `submissions(form_uuid, schema_version) → forms(form_uuid, version)` so reads are reproducible against the exact schema version used at write time.

6. What delete semantics should `forms_latest_submission` use when pruning 30-day history?
   Recommendation: Add `occurred_at` to `forms_latest_submission` and set `ON DELETE CASCADE` from `submissions(submission_id)` so the scheduled purge can’t leave dangling pointers; repoint the pointer to the newest remaining submission in the same transaction if the latest is purged. Also, consider `forms.history_retention_days INT DEFAULT 30`.

7. Where do we persist the idempotent “original result” to replay within 10 minutes?
   Recommendation: In addition to `(form_uuid, actor_id, idempotency_key)`, store `request_hash TEXT`, `first_submission_id UUID`, and `status_code INT` in the `idempotency` table so the API can return the exact prior outcome (including 4xx validation) during the window.

8. How do we reference failed validation attempts if they aren’t canonical submissions?
   Recommendation: Generate an `attempt_id UUIDv7` at the API layer and persist `submission_errors(attempt_id PK, form_uuid, actor_id, path TEXT, code TEXT, message TEXT, created_at)` with a 24h TTL job; when a submission finally succeeds, include `attempt_id` in the response so the UI can reconcile messages.

9. Do we need to persist additional caching hints for the latest-submission endpoint?
   Recommendation: Alongside the decided `latest_etag = sha256(submission_id || ':' || schema_version)` on `forms_latest_submission`, add `last_modified timestamptz` for `Cache-Control: must-revalidate` workflows and easier conditional GET handling in downstream services.

10. Where should UUIDv7s be generated to keep Postgres simple in MVP?
    Recommendation: Generate all UUIDv7 IDs (`form_uuid`, `submission_id`, `submission_client_id`, `attempt_id`) in the application layer using a vetted library; store in Postgres as `UUID` to avoid extension dependencies now, matching the PRD’s UUIDv7 envelope.

</questions>

Answers: 

1. Use `actor_id TEXT CHECK (octet_length(actor_id) <= 128)` and require it on `forms`, `submissions` and `forms_latest_submission`, since `sub` is required and limited to ≤128 chars by auth policy. :contentReference[oaicite:0]{index=0}
2. Add `{ form_uuid UUID, version INT, schema JSONB, guidance TEXT, etag TEXT, created_at, updated_at, actor_id }` with `CHECK (octet_length(guidance) <= 8000)` and validate schema keywords (type, enum, format, min/max, etc.) in the app before insert/update; persist the ETag computed from `schema` + `version`.
3. Update `forms`, compute `etag = sha256(schema::text || ':' || version)`, and upsert `forms_current(form_uuid, version, updated_at)` in a single transaction; add FK `forms_current(form_uuid, version) → forms(form_uuid, version)` to avoid stale pointers.
4. Include `confidence NUMERIC(3,2) CHECK (confidence BETWEEN 0 AND 1)` and store all timestamps as `timestamptz` (UTC). Add a CHECK or write-path validation that payload date/datetime fields are ISO 8601 normalized per requirement.
5. Yes—persist `schema_version INT NOT NULL` on `submissions` and add an FK `submissions(form_uuid, schema_version) → forms(form_uuid, version)` so reads are reproducible against the exact schema version used at write time.
6. Add `occurred_at` to `forms_latest_submission` and set `ON DELETE CASCADE` from `submissions(submission_id)` so the scheduled purge can’t leave dangling pointers; repoint the pointer to the newest remaining submission in the same transaction if the latest is purged. Also, consider `forms.history_retention_days INT DEFAULT 30`.
7. In addition to `(form_uuid, actor_id, idempotency_key)`, store `request_hash TEXT`, `first_submission_id UUID`, and `status_code INT` in the `idempotency` table so the API can return the exact prior outcome (including 4xx validation) during the window.
8. Generate an `attempt_id UUIDv7` at the API layer and persist `submission_errors(attempt_id PK, form_uuid, actor_id, path TEXT, code TEXT, message TEXT, created_at)` with a 24h TTL job; when a submission finally succeeds, include `attempt_id` in the response so the UI can reconcile messages.
9. Alongside the decided `latest_etag = sha256(submission_id || ':' || schema_version)` on `forms_latest_submission`, add `last_modified timestamptz` for `Cache-Control: must-revalidate` workflows and easier conditional GET handling in downstream services.
10. Generate all UUIDv7 IDs (`form_uuid`, `submission_id`, `submission_client_id`, `attempt_id`) in the application layer using a vetted library; store in Postgres as `UUID` to avoid extension dependencies now, matching the PRD’s UUIDv7 envelope.

---

You are an AI assistant whose task is to summarize the conversation about database planning for MVP and prepare a concise summary for the next stage of development. In the conversation history, you will find the following information:
1. Product Requirements Document (PRD)
2. Information about the tech stack
3. Conversation history containing questions and answers
4. Model recommendations

Your tasks are:
1. Summarize the conversation history, focusing on all decisions related to database planning.
2. Match model recommendations to the answers given in the conversation history. Identify which recommendations are relevant based on the discussion.
3. Prepare a detailed conversation summary that includes:
   a. Main requirements for the database schema
   b. Key entities and their relationships
   c. Important security and scalability concerns
   d. Any unresolved issues or areas requiring further clarification
4. Format the results as follows:

<conversation_summary>
<decisions>
[List decisions made by the user, numbered].
</decisions>

<matched_recommendations>
[List of the most relevant recommendations matched to the conversation, numbered]
</matched_recommendations>

<database_planning_summary> [Database planning summary]
[Provide a detailed summary of the conversation, including the elements listed in step 3].
</database_planning_summary>

<unresolved_issues>
[List any unresolved issues or areas requiring further clarification, if any exist]
</unresolved_issues>
</conversation_summary>

The final output should contain only the content in markdown format. Ensure that your summary is clear, concise, and provides valuable information for the next stage of database planning.

Summary:

<conversation_summary> <decisions>

1. Multi-tenancy deferred: keep MVP single-tenant; plan separate DB per tenant in the future (no tenant_id columns).
2. Forms: `form_uuid UUIDv7`, `schema JSONB`, `guidance TEXT (≤8k)`, `etag TEXT`, `actor_id TEXT (≤128)`, timestamps. ETag = `sha256(schema::text)`; unique `(form_uuid)`;
3. Submissions immutable: `submission_id UUIDv7`, `form_uuid`, `actor_id TEXT (≤128)`, `occurred_at timestamptz`, `normalized_payload JSONB`, `raw_transcript TEXT`, `ai_version TEXT`, `confidence NUMERIC(3,2) CHECK 0–1`; FK `(form_uuid) → forms(form_uuid)`.
4. Latest pointer: `forms_latest_submission(form_uuid PK, actor_id, submission_id, occurred_at, last_modified timestamptz, latest_etag TEXT)`; overwrite atomically on valid submit; `ON DELETE CASCADE` from `submissions(submission_id)`; repoint if latest purged.
5. History retention: keep immutable submission history; purge policy configurable (default 30 days).
6. Idempotency: create `idempotency(form_uuid, actor_id, idempotency_key, request_hash, first_submission_id, status_code, created_at/expires_at)` with unique on `(form_uuid, actor_id, idempotency_key)` (10-minute window). Client supplies `submissionClientId UUIDv7` + `Idempotency-Key`.
7. Errors: track failed attempts in `submission_errors(attempt_id UUIDv7 PK, form_uuid, actor_id, path, code, message, created_at)`; TTL 24h.
8. Caching: store `forms.etag`; store `forms_latest_submission.latest_etag = sha256(submission_id)`; support `If-None-Match` and `Cache-Control: must-revalidate` on latest submission endpoint.
10. Roles & security (MVP): defer RLS; create roles `app_rw` (INSERT submissions; SELECT forms, forms_current, forms_latest_submission) and `svc_ro` (read endpoints). Deny PUBLIC on all tables; separate `schema_migrations` role. JWT-only auth with required claims `sub (≤128), aud, iss`.
11. Constraints & limits: `octet_length(schema::text) ≤ 262144`; timestamps stored as `timestamptz` (UTC); ISO 8601 normalization enforced in app; statement and idle-in-transaction timeouts for API user.
12. Out-of-scope for MVP DB: no `events` table; no JSONB GIN indexing; no special transcript handling beyond storage field.

    </decisions>

<matched_recommendations>

1. Persist `ai_version` on `submissions` for reproducible reads — **Accepted** (Decision 4).
2. Add `submission_client_id` and `idempotency_key` with a partial unique index / idempotency table — **Accepted** (Decision 7).
3. Exclude `events` from MVP schema — **Accepted** (Decision 12).
4. Avoid JSONB GIN indexing in MVP — **Accepted** (Decision 12).
5. Keep transcripts simple; no special gating in DB — **Accepted** (Decision 12).
6. Create `submission_errors` for failed attempts with TTL — **Accepted** (Decision 8).
7. Store ETags for forms and latest submission and support conditional requests — **Accepted** (Decision 2 & 9).
8. Minimal roles; defer RLS in MVP; deny PUBLIC — **Accepted** (Decision 10).
9. Enforce size/time limits and UTC normalization — **Accepted** (Decision 11).
    </matched_recommendations>

<database_planning_summary>
[Database planning summary]
a. Main requirements:

* Single-tenant Postgres (Supabase) with core tables: `forms`, `submissions`, `forms_latest_submission`, `idempotency`, `submission_errors`.
* Forms lifecycle and strong caching: `ETag`; unique `(form_uuid)`;
* Submissions are immutable; latest pointer per `(form_uuid, actor_id)` determines canonical record; 30-day (configurable) immutable history.
* Idempotency: deduplicate for 10 minutes using `(form_uuid, actor_id, idempotency_key)` and return original result for retries; `submissionClientId` required.
* Caching for reads: ETag + If-None-Match on `GET /forms/{uuid}` and `GET /forms/{uuid}/submission`.

b. Key entities & relationships:

* `forms(form_uuid PK, schema, guidance, etag, actor_id, created_at, updated_at)`; unique `(form_uuid)`.
* `submissions(submission_id PK, form_uuid, actor_id, occurred_at, normalized_payload, raw_transcript, ai_version, confidence)`; FK `(form_uuid) → forms(form_uuid)`.
* `forms_latest_submission(form_uuid PK, actor_id, submission_id, occurred_at, last_modified, latest_etag)`; FK to `submissions(submission_id)`.
* `idempotency(form_uuid, actor_id, idempotency_key, request_hash, first_submission_id, status_code, created_at, expires_at)`; unique `(form_uuid, actor_id, idempotency_key)`.
* `submission_errors(attempt_id PK, form_uuid, actor_id, path, code, message, created_at)` (TTL 24h).

c. Security & scalability concerns:

* Auth: JWT-only; enforce required claims `sub (≤128)`, `aud`, `iss`; `actor_id TEXT` capped at 128 bytes.
* Roles: minimal (`app_rw`, `svc_ro`, `schema_migrations`); RLS deferred; deny PUBLIC privileges.
* Data protection: encryption at rest; logs masked by default; transcripts stored but unmanaged beyond size caps in MVP.
* Performance: JSONB GIN indexes and analytics `events` omitted for MVP; consider later if SLOs require. Constraints on schema size (≤256 KB) and DB timeouts guard p95 latency.

d. Unresolved/clarifications:

* History retention exact default (suggested 30 days) and whether per-form overrides are required at schema level.
* Exact columns for `normalized_payload` generated fields (if any) are postponed; currently no JSONB indexes.
* Transcript storage is present but special access gating is deferred; revisit when DPA-based inclusion is enabled.
  </database_planning_summary>

<unresolved_issues>

1. Confirm default `history_retention_days` and need for per-form override vs. global setting.
2. Define canonical set of “hot” fields (if any) to extract from `normalized_payload` later when indexes are introduced.
3. Decide whether `raw_transcript` lives inside `submissions` or a separate table before adding size caps and purging policies.
4. Specify exact cache headers for `GET /forms/{uuid}/submission` (e.g., `Cache-Control` directives) alongside ETag usage.
   </unresolved_issues>
   </conversation_summary>
