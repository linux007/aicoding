# Workflow Checklist

Use this checklist when generating an architecture wiki.

## 1. Scope Lock

- Confirm single-service or multi-service
- Confirm audience
- Confirm output form
- Confirm whether to include Mermaid diagram

## 2. Evidence Collection

### If GitNexus is available
- Read repo context
- Read cluster list
- Query for architecture / service / router / workflow concepts
- Read 2–4 representative process traces
- Read context for key symbols
- Read source files that confirm the graph findings

### If GitNexus is not available
- Read `README*`, `CLAUDE.md`, `AGENTS.md`
- Find `main.go`, `go.mod`, `go.work`
- Inspect `router`, `controllers`, `service`, `data`, `models`, `api`, `helpers`, `middleware`, `conf`
- Identify MQ / command / task registration

## 3. Extract the Architecture Model

- Service inventory or package inventory
- Unified layering model
- Domain clusters or logical modules
- Entry map
- Cross-module contracts
- Key ID lifecycle anchors
- Representative runtime flows

## 4. Choose Document Shape

### Single-service
- purpose
- request lifecycle
- module responsibilities
- contracts
- async / callback flows
- risks

### Multi-service
- service catalog
- service map
- shared layering conventions
- cross-service contracts
- key IDs and lifecycle
- representative flows
- risks

## 5. Writing Rules

- Prefer explanation over enumeration
- Prefer contract boundaries over directory listings
- Prefer 2–4 flows over endpoint exhaustiveness
- Prefer evidence-backed claims

## 6. Final Quality Check

- Entry map included if entrypoints are non-trivial
- Key IDs included if async / callback / cross-service work exists
- Mermaid diagram included unless user asked for text only
- High-risk boundaries called out explicitly
- No placeholders or vague sections remain
