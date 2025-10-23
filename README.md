# X‑Put — Embeddable Conversational Forms (MVP)

[![Node.js](https://img.shields.io/badge/node-22.14.0-339933?logo=node.js&logoColor=white)](./.nvmrc)
[![Astro](https://img.shields.io/badge/Astro-5.x-ff5d01?logo=astro&logoColor=white)](https://astro.build)
[![React](https://img.shields.io/badge/React-19-61dafb?logo=react&logoColor=black)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![Status](https://img.shields.io/badge/status-MVP%20%7C%20WIP-orange)](#project-status)
[![License](https://img.shields.io/badge/license-MIT-blue)](#license)

A hosted, embeddable conversational form system that lets integrators define a form once (hybrid schema + AI guidance), embed a lightweight two‑pane widget (chat + dynamic form/summary), and receive validated, normalized submissions server‑side.

Links:
- Product Requirements Document (PRD): ./.ai/prd.md
- Technology Stack: ./.ai/tech-stack.md

## Table of Contents
- [Project name](#xput--embeddable-conversational-forms-mvp)
- [Project description](#project-description)
- [Tech stack](#tech-stack)
- [Getting started locally](#getting-started-locally)
- [Available scripts](#available-scripts)
- [Project scope](#project-scope)
- [Project status](#project-status)
- [License](#license)

## Project description
X‑Put prioritizes schema‑first data collection with optional, strictly‑controlled AI guidance. The MVP ships as a secure widget hosted on a single origin with a two‑pane UX (chat planner on the left, dynamic form/summary on the right). Submissions are validated, normalized, and saved server‑side with idempotency and overwrite semantics for the latest valid result per actor/form.

Key pillars (from PRD):
- Single‑origin hosted widget with tight CSP, SRI, sandboxed iframe, and strict postMessage origin checks.
- Two‑pane UX: chat planner + dynamic form/summary with live validity, required/optional badges, and suggestion chips.
- Simple lifecycle: collecting (ephemeral) → submitted (persisted); latest valid submission overwrites prior ones.
- Server‑to‑server reads for saved data; the UI can only create/overwrite submissions and read the form definition.
- Observability via standardized events stored in DB; internal dashboards (no public Metrics API in MVP).
- Performance budgets: widget ≤ 120 KB gz; first interaction ≤ 2.5s on 4G; modern browser support.

Security & compliance highlights:
- JWT‑only auth via tenant JWKS (TTL 90m; skew ±60s; enforce sub/aud/iss).
- Strict CSP, X‑Frame‑Options: DENY on non‑widget routes, frame‑ancestors allowlist, sandboxed iframe, masked logs.
- Data protection: encryption at rest, audit trails for transcript/data access, transcript gating via DPA.

APIs and behavior (MVP):
- Forms: create/read with ETag and caching (max‑age=60; must‑revalidate). Schema‑lite validation (type, enum, format, min/max, pattern, items, required, properties).
- Submissions: atomic overwrite of latest valid; de‑dup with Idempotency‑Key + submissionClientId (UUIDv7) within 10 minutes.
- Latest submission and history endpoints for server‑to‑server (history data access gated by scope).

For detailed user stories, success metrics, SLOs, and release gates, see the PRD.

## Tech stack
From tech‑stack.md and package.json:
- Frontend: Astro 5 (islands/partial hydration), React 19, TypeScript 5, Tailwind CSS 4, shadcn/ui.
- Backend: Supabase (Postgres) for forms, submissions, events, configs; containerized API service on DigitalOcean with ETag caching, JSON Schema‑lite validation, de‑dup window using Redis/Postgres.
- AI: OpenRouter.ai; defaults per form: temperature=0.2, maxClarifications=2, maxTokens=1024; retries with jitter, 4s planning timeout, tenant‑level circuit breaker; token budget enforcement with graceful UI fallback.
- Security: tight CSP/SRI, sandboxed iframe, postMessage origin validation, encryption at rest, masked logs, no third‑party cookies.
- Observability: P0 events (session_started, field_validated, clarification_prompted, submission_saved, token_budget_exhausted, auth_refreshed, rate_limited) stored in DB; SLOs availability 99.9%, API p95 < 1.5s, events ≥ 99.9%.
- Rate limiting & idempotency: 10 RPS per token (burst 20); exclude repeated idempotent retries from rate count; client sends submissionClientId + Idempotency‑Key.
- CI/CD & Hosting: GitHub Actions with gates (bundle size, 4G first interaction, dependency audit, lint/tests, a11y checks), containerized deploys on DigitalOcean; environments: dev/stage/prod with sealed secrets and probes.

Core runtime/tooling versions:
- Node.js 22.14.0 (see .nvmrc)
- Astro ^5.13.7, React ^19.1.1, TypeScript (via eslint tooling), Tailwind CSS ^4.1.13

## Getting started locally
Prerequisites:
- Node.js 22.14.0 (as specified in .nvmrc)
- npm (bundled with Node.js)

Setup:
1. Install dependencies
   - npm install
2. Start the dev server
   - npm run dev
3. Build for production
   - npm run build
4. Preview the production build
   - npm run preview

Notes:
- No external services are required to run the basic UI locally. Backend/API integration and secrets are described at a high level in the PRD and tech‑stack docs.
- Follow repository coding and tooling conventions (lint/format hooks) when committing.

## Available scripts
Defined in package.json:
- dev: astro dev — start the development server
- build: astro build — build for production
- preview: astro preview — preview the production build
- astro: astro — run Astro CLI directly
- lint: eslint . — lint the codebase
- lint:fix: eslint . --fix — auto‑fix lint issues
- format: prettier --write . — format supported files

## Project scope
In scope (MVP):
- Hosted widget and embeddable script on a single origin (CSP, SRI, sandboxed iframe).
- Two‑pane chat + form/summary UX with inline validation and suggestion chips (AI proposals only when whitelisted).
- Schema‑first forms; optional AI proposals under explicit whitelist.
- JWT‑only auth with refresh hook; server‑to‑server reads for saved data.
- DB‑only metrics; standardized event stream; 30‑day immutable submission history (metadata by default).
- Token budget enforcement with graceful fallback to manual form.

Out of scope (MVP):
- File uploads and autosave/drafts.
- Public Metrics API or hosted customer‑facing dashboard.
- Multi‑language UI beyond en‑US; RTL.
- Third‑party cookies and non‑web channels (native iOS/Android SDKs).
- Payments, roles/teams UI.

For complete boundaries, user stories, SLOs, and acceptance criteria, see ./.ai/prd.md.

## Project status
MVP — work in progress. The repository contains the frontend scaffold (Astro + React) and project tooling. Backend/API, security hardening, and CI gates follow the specifications outlined in the PRD and tech‑stack documents and will be implemented/validated per the release gates (OWASP ASVS L2, WCAG 2.2 AA audit, dependency audit clean, bundle size and performance budgets, load tests, OpenAPI + Postman + mock server, runbook).

## License
MIT. If a LICENSE file is present in the repository, its contents govern usage. If not, licensing terms may be updated in a future commit.
