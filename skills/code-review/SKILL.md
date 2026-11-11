---
name: code-review
description: >-
  Performs a general-purpose code review: change-impact (ripple effect) check,
  dead-code and stale-doc cleanup, test adequacy, and production-readiness
  audit for any codebase. Use when asked to review code or diffs, audit
  quality, clean up redundant code or outdated docs/comments, assess test
  adequacy, confirm a change was applied everywhere, or verify release
  readiness. Do not use for routine feature work or debugging a single isolated
  runtime error unless a broader review is requested.
license: MIT
---

# General-Purpose Code Review & Readiness Audit

Keep this skill stateless on disk: never write counters, caches, review logs,
or memory files to the skill folder or workspace. Within a conversation, reuse
prior context and user decisions, but re-verify against live files and command
output.

Default to **read-only analysis** unless the user asks for fixes. Work through
the sections in order; diagnostic commands from section 5 may run earlier as
needed.

## 1. Principles

- **Ground truth, no guessing.** Read actual definitions, callers, configs, and
  dependency sources before making claims. Verify library/framework behavior via
  pinned manifests, local sources, or web search, not memory. Re-read exact
  lines right before reporting or editing. If domain rules, intentional
  trade-offs, or target environments can't be proven, ask the user when it
  blocks the work; otherwise record it as an open question in the report.
- **Proportional and high-signal.** Apply each lens only where it fits the
  project. Never assume a tool, framework, or practice exists; use what the
  project provides and skip the rest. Never invent filler findings. Leave pure
  formatting to configured linters/formatters unless style hides bugs or hurts
  readability.

## 2. Scope

Infer the project's nature, conventions, and tooling from the workspace. Honor
prior conversation decisions and documented architectural trade-offs, unless
the code, tests, or docs contradict them or they cause a bug or security flaw.

- **Change review:** identify the changed code from the diff. With no VCS or
  diff, use the user's description or recent file modifications, or ask.
- **Whole-project audit:** map entrypoints, high-risk areas, and overall
  structure first, then audit module by module (parallel read-only subagents if
  useful). Report what was inspected and what was not.
- Read generated and third-party files to verify contracts, but direct findings
  and edits at maintained sources or generators.

## 3. Trace Ripple Effects

For any change being reviewed or fixed, find everything that depends on or
describes the changed behavior, and check it is updated or still correct.
Search the workspace for references (symbols, config keys, string literals,
error messages, paths, CLI flags, env vars) instead of trusting the diff.

- **Companions:** callers, overrides, and other implementations of an
  interface; tests, fixtures, mocks; comments and docstrings; READMEs, docs,
  examples; config, schema, and API specs; build/CI scripts; migrations;
  changelogs; translations; generated artifacts.
- Flag companions that should have changed but didn't (e.g., a renamed
  parameter still in a docstring, a changed default still in the README).

## 4. Review Lenses

### A. Design & Simplicity
- Right level of abstraction: no premature over-engineering, no leaky
  boundaries exposing storage/transport details.
- Reuse existing utilities; consolidate duplication only when it simplifies
  maintenance without forced coupling.
- Clear control flow, named constants/types over magic values, a single source
  of truth with no derived state that can drift.
- **Structure and placement:** code lives where it best fits the project's
  layout conventions: with related code, or in a new unit when it has its own
  responsibility. Flag unrelated responsibilities mixed in one unit, tangled
  dependencies, or needless fragmentation (e.g., tiny single-use units,
  pass-through wrappers). In a change review, only flag what the change
  introduces or worsens. Size alone is not a verdict.

### B. Correctness & Test Adequacy
- Trace data flow, state transitions, boundary inputs, concurrency/atomicity
  (check-then-act races, retries, locks/transactions), error propagation (fail
  fast, keep context, no silent swallowing), and cleanup on success and failure.
- For each defect, find the root cause and search for the same pattern
  elsewhere.
- Map each changed or critical behavior (branches, boundaries, error paths) to
  a test that would actually fail if the behavior broke, not just happy paths
  or stale mocks. Use coverage output on changed code if available; treat gaps
  as leads, not a numeric target, and covered lines as no proof of meaningful
  assertions. If there are no tests, flag untested critical behavior only where
  the project's scale or risk warrants it.

### C. Dead Code & Hygiene
- Find unused symbols, files, exports, routes, feature flags, manifest
  dependencies, commented-out code, debug leftovers, scratch files,
  prototype/mock scaffolding, and tests that are obsolete, disabled, or skipped
  without reason, including anything a change orphaned.
- Before calling something dead, check dynamic references, reflection/DI,
  configs, build scripts, entrypoints, and public exports. Use available
  analysis tooling (linters, unused-code detectors) to surface candidates,
  alongside reference searches.

### D. Comments & Contract Sync
- Comments and docstrings should explain *why*, not *what*, and match current
  signatures, invariants, defaults, and behavior. Flag stale or contradictory
  ones and triage `TODO`/`FIXME`.
- Check that code, config (including examples), schemas, CI files, help text,
  and docs agree, including producer ↔ consumer contracts.

### E. Security & Production Readiness *(when applicable)*
- **Security/supply chain:** exposed secrets, missing ignore/package
  exclusions, untrusted input and auth boundaries, leaky error responses,
  dependency health, and pinning suited to how the project is distributed.
- **Resilience:** fail-fast config validation (no insecure production
  fallbacks), bounded timeouts and idempotent retries on external I/O,
  constraints backing critical invariants (no external side effects inside
  uncommitted transactions), backward-safe migrations/contracts, clean
  shutdown/health semantics, logs free of PII and secrets.

## 5. Verify

Run available build, type-check, lint, test, coverage, or audit commands when
safe and non-destructive, and avoid or clean up any artifacts they generate.
Report exact commands and outcomes. Never claim a check passed from reading
code alone, and note any that couldn't run.

## 6. Report

Group verified findings as **Blocker**, **Important**, or **Cleanup / Polish**,
each with exact `file:line` (or the affected units for structural findings),
concrete impact, and a minimal fix. Rate by demonstrated impact: structural
findings are Cleanup / Polish unless they cause concrete harm (e.g., circular
dependencies, risky coupling), and untested critical behavior is at least
**Important**. Keep it short when the code is clean.

## 7. Fix (only when asked)

- Separate safe, behavior-preserving cleanups from breaking contract, schema,
  or behavior changes, and confirm trade-offs with the user before the latter.
  Large structural refactors (split, move, merge) must still preserve
  behavior, but also confirm with the user first.
- Update ripple-effect companions in the same pass. Confirm first if the
  updates reach into other code modules (beyond the change's own tests and
  docs) or alter a public interface.
- After any removal, move, or rename, re-search references to the changed
  target; remaining hits must be intentional (e.g., a compatibility shim for a
  public interface), and immutable history (applied migrations, past changelog
  entries) stays untouched. Remove what the change orphaned or made redundant
  (stale imports/re-exports, duplicates, emptied files without a structural
  role) and repeat until a pass finds nothing new, subject to the confirmation
  rule above.
- Never weaken or rewrite a test just to make it pass. If test and code
  disagree, determine which is correct first.
- When adding tests, follow the project's existing conventions (ask before
  introducing a framework). Keep them deterministic and focused on one
  behavior each; where practical, confirm a bug-fix test fails without the fix.
- Re-run section 5 after edits.

## Maintaining This Skill

Applies only when editing this file, not when performing a review.

- **Concise.** Add content only for genuinely new points; merge into an
  existing bullet instead of repeating it. Prefer rewording over adding lines.
- **General.** Keep instructions project-, language-, and tool-agnostic: no
  named tools or numeric thresholds, and never assume a tool, framework, or
  scenario exists. Use conditional wording ("if available", "where applicable").
- **Linear flow.** Sections read top-down in work order; avoid unnecessary
  forward references. Keep numbering and cross-references correct after any
  edit.
- **Consistent.** Check each edit for conflicts with existing rules (e.g.,
  read-only default, statelessness) and keep the `description` trigger accurate.
- **Delivery.** Before presenting changes, look for remaining gaps and
  conflicts and fix them first. Then update the file in place, or output the
  complete paste-ready file if asked.
