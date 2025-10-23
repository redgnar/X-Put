# Junie Guidelines for X‑Put

Purpose: Provide a concise, actionable guide for Junie (JetBrains autonomous programmer) to work safely and effectively in this repository with minimal changes and high signal output.

## Core Principles
- Make minimal, targeted changes to satisfy the specific issue description; avoid broad refactors.
- Prefer additive edits over destructive changes; keep diffs small and well‑scoped.
- Preserve existing conventions and align with project‑specific rules in GUIDELINES.md.
- Ask for clarification when requirements are ambiguous before implementing risky assumptions.

## Project Context
- Product context: .ai/prd.md
- Technology constraints and policies: .ai/tech-stack.md
- Consolidated contributor guidelines: GUIDELINES.md (converted from .cursor/rules)

## Project Structure (quick reference)
- src/ — source code
  - src/layouts — Astro layouts
  - src/pages — Astro pages
    - src/pages/api — API endpoints
  - src/components — UI components (Astro & React)
    - src/components/ui — shadcn/ui components
  - src/lib — services and helpers
  - src/db — Supabase client and types (optional)
  - src/styles — global styles
- public — public assets

Keep this structure when adding files. If you change it, update GUIDELINES.md accordingly.

## Coding Practices (quick reference)
- Early‑return on errors; keep the happy path last. Avoid unnecessary else.
- Handle edge cases and log errors clearly; prefer typed errors where helpful.
- Use TypeScript strictly; keep types close to usage.
- Follow lint and format rules (see package.json → lint-staged).

## Astro/React Notes
- API routes: export uppercase handlers (GET/POST), export const prerender = false.
- Validate input with zod in API routes.
- Extract logic into src/lib/services; keep endpoints thin.
- Favor Astro islands to keep JS small; lazy‑load heavy UI/AI pieces.

## shadcn/ui
- Components in src/components/ui; import via the @/ alias as configured in components.json.
- Add components with: npx shadcn@latest add <component-name>.

## Supabase (optional)
- If integrating: follow the snippets in GUIDELINES.md (Supabase section) and ensure env/types are set up.

## Junie Operational Rules in This Repo
- Tooling discipline:
  - Prefer specialized tools (file search, structured editing, rename) over raw shell ops.
  - When renaming any code element, exclusively use the rename_element tool (it updates all references safely).
  - Use search_project to locate files/symbols or exact text; don’t rely on fuzzy search.
  - Use get_file_structure to understand a file before editing large sections.
  - Use search_replace for precise, uniquely‑matching edits; keep replacements minimal and exact.
  - Avoid creating or modifying files via shell redirection; use the create/edit tools.
- Communication:
  - Keep the user informed using update_status at major checkpoints (plan, progress, decisions).
  - If blocked by missing context or conflicting requirements, ask_user to clarify before proceeding.
- Verification:
  - If an issue describes a bug, reproduce it with a small script or by running the relevant tool/command.
  - After changes, re‑run reproduction and check for regressions; lint/format Markdown and code as needed.

## When in Doubt
- Prefer the least‑intrusive change that fully satisfies the issue.
- Link back to GUIDELINES.md sections instead of duplicating content.
- Escalate with a clarification question if multiple interpretations exist.

References
- GUIDELINES.md — project‑specific contributor guidance
- .cursor/rules — original rule sources (kept for AI tooling compatibility)
- README.md — quick start and AI tooling notes
