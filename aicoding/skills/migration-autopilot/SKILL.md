---
name: migration-autopilot
description: Use when migrating functionality between modules, services, or frameworks — including porting old code to new modules, aligning logic across repos, consolidating duplicate implementations, or "rewrite this in the new stack". Triggers on user saying "迁移", "migrate", "port", "对齐", "搬运", "refactor across modules", or asking to move code from one system to another. Even if the user frames it as "just copy over", use this skill to prevent naive translation bugs.
---

# Migration Autopilot

## Core Principle

A migration is **not** a line-by-line translation. It is:
1. Extract the **contract** the old code fulfills.
2. Re-implement in the target stack using its idioms.
3. Prove equivalence with **runtime evidence**, not eyeballing.

If you skip step 1, you port bugs and workarounds. If you skip step 3, you find out in production.

## When to Use

Any time you're asked to move behavior across module/framework/language boundaries. Even casual "just copy this" requests — that's a risk signal.

## The Workflow

Work through these phases in order. Stop at each gate until the user confirms.

## First Response Template

When the user starts a migration task, your first response should:
1. State that you are using the migration-autopilot flow.
2. Name the capability or capabilities being migrated.
3. Draft the smallest useful Gate 1 contract you can.
4. Name one migration risk that makes direct translation unsafe.
5. State what Gate 2 or the next clarification will do after the user confirms.
6. Mention that the migration must end behind a reversible switch, even if implementation starts with only one capability.
7. Not start coding yet.

### Gate 1: Contract (Phase 1)

Before reading implementation details, write a **capability contract** covering:
- **Inputs**: shape, units, required vs optional, null/empty semantics.
- **Outputs**: shape, error representations.
- **Side effects**: what it writes (DB, cache, MQ, external API), in what order.
- **Error semantics**: what propagates, what's swallowed, retry behavior.
- **Concurrency & idempotency**: parallel-safety, retry-safety.
- **Essential vs Incidental**: business rules that must be preserved vs framework artifacts that should not.

**Gate check**: Present the contract. If the user says "looks right", proceed. If you can't produce this contract, ask targeted questions first.

### Gate 2: Difference Matrix (Phase 2)

Enumerate known-hazard differences between source and target stacks:

| Dimension | Source behavior | Target behavior | Migration decision |
|-----------|-----------------|-----------------|--------------------|
| Error propagation | | | |
| Null/missing semantics | | | |
| Time/timezone handling | | | |
| Logging/tracing convention | | | |
| Auth/context propagation | | | |
| Type coercion | | | |

**Gate check**: Present the matrix. Any row where the decision is "not sure" must be resolved with the user before coding.

### Gate 3: Layered Migration (Phase 3)

Migrate in this order, **never mixed in one change**:
1. **Data contracts** — DTOs, error codes, enums. Align types first.
2. **Pure business logic** — rules, calculations, no framework deps.
3. **Adapter layer** — thin shell bridging target entry points to layer 2. **No business logic here.**
4. **Infrastructure** — logging, metrics, tracing. Use the **target project's idioms**.

If the user asks for controller, service, and data to be changed together in one step, explicitly refuse to plan that as a single mixed-layer migration. Split the work by capability and by layer first.

After each layer, pause for user review.

### Gate 4: Equivalence Evidence (Phase 4)

Before claiming migration is complete, provide runtime evidence. Prefer this order:
- **Golden case tests** — black-box tests derived from Phase 1 contract. Run against source first, then target.
- **Traffic diff** — capture real traffic, run both implementations, compare responses.
- **Invariant assertions** — for critical quantities, write explicit equivalence checks.

If the user says "I'll test later" or "I'll verify it myself later", explicitly say that manual follow-up testing is not sufficient migration equivalence evidence and does not replace this gate.

If a stronger method is feasible, do not stop at a weaker one just because it already passed.

**Never claim equivalence based on "looks right" or "tests pass".**

### Gate 5: Reversible Cutover (Phase 5)

Every migrated capability ships behind a config switch that can route to old or new. Keep the old implementation live for observation. Delete old code only after the switch has been at 100% new with no incidents.

## Meta-Rules

These are the rules you'll be most tempted to break. Reread when you feel pressure to "just get it done".

- **No line-by-line translation.** The source is a reference, not a spec. Target code must read like native target-stack code.
- **No porting source dependencies.** Use what the target project already uses.
- **One capability at a time.** Finish Gate 4 for capability N before starting N+1.
- **Unclear source semantics → stop and ask.** Guessing canonizes bugs.
- **Always ship behind a reversible switch.**
- **Preserve observability continuity.** New implementation must emit equivalent-or-better logs/metrics/traces.

## Red Flags — STOP and Reassess

| Symptom | What it means |
|---------|---------------|
| "It's basically the same, just copy" | Semantic gap underestimated. Write contract first. |
| "This source code is confusing" | Don't guess — ask. Unexplained source = bug or implicit convention. |
| "Tests pass, looks equivalent" | Not sufficient. Need traffic diff or golden cases. |
| "I'll test it later myself" | Explicitly state that manual later testing is not sufficient migration equivalence evidence. Keep the evidence gate. |
| "I'll add the switch later" | Migration without rollback path is unacceptable. |
| "Let me port controller, service, and data all at once" | Mixed-layer migration hides root cause and blocks safe review. Split it first. |
| "Let me port this whole module at once" | Batch migrations hide root cause of diffs. One capability at a time. |

## Repo-Specific Branch: classme/liveme

When migrating between liveme (source) and classme (target), apply these additional constraints:

1. **GitNexus impact analysis**: Before modifying any symbol, run `mcp__gitnexus__impact` to check blast radius. Warn the user if HIGH or CRITICAL.
2. **Traffic diff**: Use the bundled traffic diff tooling to verify equivalence before merging.
3. **Target idioms**: Use `zlog` for logging, `helpers.*` for cache/redis/kms, existing `api/` packages for downstream calls — never port liveme's utility wrappers.
4. **Worktree isolation**: Use `superpowers:using-git-worktrees` for each capability migration.
5. **Reversible switch**: Every capability must be gated by a feature flag in classme's config system.
