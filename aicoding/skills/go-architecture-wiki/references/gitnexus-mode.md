# GitNexus Mode

Use this mode when GitNexus is available for the repository.

## Why

GitNexus gives process traces, clusters, and symbol context that are usually better architecture evidence than raw grep.

Without it, architecture docs often miss:

- real execution flows
- functional clusters
- cross-module collaboration hotspots
- hidden async/callback paths

## Preferred Order

1. Read repo context
2. Read cluster list
3. Query for architecture concepts
4. Read process traces
5. Read symbol context
6. Read source files for confirmation

## Suggested Queries

Use repo-specific wording, but these concepts usually help:

- `system architecture core modules service http controller data model api cache middleware`
- `router http mq commands main httpServer route register consumer builder`
- `cross service interactions callback task workflow resource production bind`
- `taskId businessId resourceId roomId callback transData`

## What to Capture from GitNexus

### From repo context
- repository size
- process count
- whether the repo looks single-service or multi-service

### From clusters
- top functional clusters
- likely domain ownership areas
- where cohesion is highest

### From process traces
- representative end-to-end flows
- async or callback paths
- service-to-service choreography

### From symbol context
- incoming callers
- outgoing dependencies
- process participation
- whether a symbol is a good contract anchor

## Important Rule

Do not stop at GitNexus summaries.

Always read the source files for the final claims you plan to write into the wiki. Use GitNexus to narrow and prioritize where to read.

## Fallback

If the index is stale or unavailable, fall back to normal source exploration. The skill still applies; only the evidence collection mode changes.
