# AGENTS.md

Common, provider-agnostic execution baseline for AI coding agents (Claude Code, Cursor, Codex,
Antigravity). This file holds **only the shared baseline** — project-specific rules live in each
tool's project file (e.g. `CLAUDE.md`), which references this baseline.

<!-- AI-ENGINEERING:BEGIN AGENTS_BASELINE -->
## AI Engineering Baseline

Read this file before any task.

### Priority (resolve all conflicts in this order)

1. User request → 2. Project-specific rules (e.g. `CLAUDE.md`) → 3. This baseline (`AGENTS.md`)
   → 4. Tool/skill instructions → 5. General AI knowledge

If rules conflict with each other, choose the **safest change that still satisfies the user
request**, and state the assumption.

### Rules

1. **Read before writing.** Inspect the relevant code/config/tests (and this file) before editing.
   Don't bulk-scan protected paths or external references.
2. **Surgical changes.** Smallest clear change that satisfies the request; touch only files for the
   current goal; preserve unrelated user work. No new abstraction unless it removes real complexity
   or matches an existing pattern.
3. **Match conventions.** Follow local naming/layout/validation/commit/protected-path rules before
   introducing a new one.
4. **Surface conflicts.** When instructions/configs/tests disagree, name the conflict and resolve by
   Priority; ask only when the decision is **risky** (irreversible, outward-facing, destructive) and
   cannot be inferred.
5. **Verify intent.** Run deterministic validation (validators, selected tests) proving the requested
   *behavior/invariant*, not just that code runs.
6. **Fail loud.** No completion claim without evidence. Report blockers, skipped checks, uncertainty,
   and residual risk. Never present unverified output as fact.
7. **Comment sparingly.** Only where purpose/flow isn't obvious.

> **Long / ambiguous tasks:** define the success condition up front; checkpoint after
> plan/edit/verify/publish; prefer targeted reads over broad scans.

### Safety — never act without explicit confirmation

- **Don't push to protected branches** (`main`/`master`/`dev`/`develop`). Work on a feature branch;
  leave the diff/PR for human review.
- **Don't run destructive or schema-changing operations** (migration/DDL, data deletion, `reset --hard`,
  force-push, history rewrite) without explicit confirmation.
- **Don't read, print, or modify secret files** (`.env`, `*.env`, credentials, `*.jks`/`*.keystore`,
  key material).
- **Stop for human diff review before committing** (1 task = 1 commit).

> The project file may add more specific boundaries (e.g. generated dirs, source-of-truth paths).

### Command routing (if the tool supports commands/workflows)

Prefer an installed command/workflow entry point over ad-hoc skill choice. Treat a command file as
an *orchestration contract* (intent → required skills → steps → output). If none matches, fall back
to the most specific skill and **say so**.

> The catalog path is **tool-specific**, defined by the provider (e.g. Codex:
> `.codex/workflows/commands.md`). Other tools ignore this line.

### Language

- User-facing output in **Vietnamese** with correct UTF-8 diacritics: PR/MR descriptions, release
  notes, user docs, and the Vietnamese body of commit messages.
- Commit header stays `type(scope): summary`; follow the repo's existing header language.
- Code identifiers, this baseline file, and config keys stay in **English**.

### Definition of Done (affirm before claiming complete)

- Relevant files were read before editing.
- Edits are limited to the requested scope; unrelated work preserved.
- Validators/tests ran after any code/config/structure/doc change (state results); skipped checks
  and residual risk reported.
- Conflicts, assumptions, and uncertainty surfaced.
<!-- AI-ENGINEERING:END AGENTS_BASELINE -->
