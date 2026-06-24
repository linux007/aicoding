---
name: go-architecture-wiki
description: Use when the user wants a system design wiki, architecture overview, service map, module interaction contract, onboarding architecture doc, or repo-level design summary for a Go codebase. Trigger for both single-service and multi-service Go repositories, especially when they ask to read the codebase or GitNexus graph, explain service boundaries, map entrypoints, add Mermaid diagrams, or trace key IDs and lifecycle flows.
---

# Go Architecture Wiki

Generate an evidence-based architecture wiki for a Go repository.

The goal is not to dump package names. The goal is to explain how the system is shaped, where requests enter, how modules collaborate, which IDs and contracts stabilize cross-module behavior, and which boundaries are risky to change.

## When to Use

Use this skill when the user asks for any of the following:

- A system design wiki or architecture document
- A repository overview for onboarding or maintenance
- A service map for a Go codebase
- Module interaction contracts or cross-service boundaries
- Entry maps for HTTP, MQ, command, or cron/task flows
- Key ID lifecycle analysis such as `taskId`, `businessId`, `resourceId`, `roomId`
- A Markdown architecture doc with Mermaid diagrams

Do not use this skill for:

- Small bugfixes or feature implementation
- API-only reference extraction without architecture context
- PR review or change impact analysis
- Non-Go repositories unless the user explicitly wants to reuse the process

## Core Principle

Always prefer **contract-first architecture writing** over generic system prose.

A good wiki answers:

1. What are the main service or module boundaries?
2. How does work enter the system?
3. Which layer owns orchestration versus access?
4. Which IDs and payloads cross boundaries?
5. Which flows best represent the real runtime behavior?

If the document cannot answer those five questions, it is not finished.

## Workflow

### 1. Lock scope before writing

Ask concise clarifying questions, one at a time, until you know:

- Scope: one service or the whole repository
- Audience: onboarding, maintainers, reviewers, operators
- Output form: Markdown only, or Markdown plus Mermaid diagram

Default to:

- Whole repository if the user asks for a “system” or “architecture” wiki
- Maintainer + onboarding audience if unspecified
- Markdown + Mermaid if unspecified

### 2. Classify repository shape

Determine which mode applies:

- **Single-service Go repo**: one main business service with one `main.go` / `go.mod`
- **Multi-service Go workspace**: multiple service roots, multiple `main.go`, multiple `go.mod`, or clearly separated business services

This choice changes the wiki structure.

### 3. Use GitNexus first when available

If GitNexus is available, read `references/gitnexus-mode.md` and follow it.

Do this before broad grepping:

1. Read repo context
2. Read cluster list
3. Query for architecture concepts
4. Read key process traces
5. Read symbol context for representative flows
6. Read source files only after graph evidence identifies where to look

If GitNexus is not available, fall back to source exploration:

- `README*`, `CLAUDE.md`, `AGENTS.md`
- `main.go`, `go.mod`, `go.work`
- `router/`, `controllers/`, `service/`, `data/`, `models/`, `api/`, `helpers/`, `middleware/`, `conf/`
- MQ / command / task registration files

## What to Extract

For every repo, extract these seven views:

1. **Repository inventory**
   - single service or multi service
   - business domains or service catalog
2. **Layered model**
   - entry layer
   - orchestration layer
   - access layer
   - infrastructure layer
3. **Functional clusters / domains**
   - from GitNexus clusters if available
   - otherwise inferred from package structure and call paths
4. **Interaction contracts**
   - controller ↔ service
   - service/data ↔ models/api
   - service ↔ service
   - cross-service call boundaries
5. **Entry maps**
   - HTTP groups
   - MQ callbacks
   - command / cron / task entrypoints
6. **Key IDs and lifecycle anchors**
   - `taskId`, `businessId`, `resourceId`, `roomId`, or repo-specific equivalents
7. **Representative execution flows**
   - 2–4 flows that best explain how the system really behaves

## Required Sections

Always include these sections in the final wiki:

- Goal / document purpose
- Repository or service overview
- High-level architecture diagram
- Layered model
- Core modules or clusters
- Interaction contracts
- Representative execution flows
- High-risk boundaries or change-sensitive areas

### Add these sections whenever evidence exists

- Entry map
- Key ID lifecycle
- Callback or async workflow model
- Configuration / initialization model
- Single-service vs multi-service comparison note

## Single-Service Template

Read `references/output-templates.md` and use the single-service template when the repo is centered around one service.

Key emphasis:

- request lifecycle
- package responsibilities
- orchestration path
- data ownership
- async or callback points if any

## Multi-Service Template

Read `references/output-templates.md` and use the multi-service template when the repo contains multiple services.

Key emphasis:

- service catalog
- cross-service boundaries
- common layering conventions
- which service owns what
- shared IDs and resource lifecycles

## Writing Rules

- Prefer evidence from code and GitNexus over assumptions
- State inferred claims carefully; mark them as inference if not directly confirmed
- Use file paths and symbols to ground explanations
- Explain module purpose in terms of collaboration, not only directory names
- Prefer a few strong representative flows over exhaustive endpoint dumps
- Prefer Mermaid diagrams that show responsibility flow, not decorative topology

## Anti-Patterns

Avoid these common failures:

- Writing a generic “system overview” that never explains module contracts
- Listing packages without explaining ownership and interaction
- Skipping entry maps for HTTP / MQ / command / task flows
- Missing key ID lifecycle anchors in async or cross-service systems
- Using only grep when GitNexus clusters and process traces are available
- Focusing only on deployment or infrastructure while ignoring code-level boundaries
- Treating single-service and multi-service repos with the same outline

## Quality Gate

Before finishing, verify the wiki answers all of these:

- Can a new maintainer find the main entrypoints quickly?
- Can a reviewer identify which layer owns orchestration?
- Can someone trace a key business flow end to end?
- Can someone tell which IDs stabilize async or cross-service collaboration?
- Can someone spot the riskiest architectural boundaries to change?

If any answer is no, revise the wiki.

## Output Format

Default output is:

- A Markdown wiki
- One Mermaid high-level architecture diagram
- Optional entry map appendix
- Optional key ID lifecycle appendix

When the user asks for a reusable wiki artifact, write the document into the repository rather than only replying in chat.

## References

- Read `references/workflow-checklist.md` for the full step-by-step checklist
- Read `references/output-templates.md` for single-service and multi-service section templates
- Read `references/gitnexus-mode.md` when GitNexus is available
