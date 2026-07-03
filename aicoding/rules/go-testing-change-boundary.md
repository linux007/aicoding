---
description: Keep Go tests risk-based and edits tightly scoped
paths:
  - "**/*.go"
alwaysApply: true
---

# Go Testing And Change Boundary

- **Test by risk, not by ritual**
  - Add focused tests for logic, parsing, state transitions, and shared utilities.
  - Do not overbuild tests for simple CRUD, HTTP glue, config, or log changes unless the task explicitly needs them.
  - Prefer table-driven tests when they make behavior clearer.

- **Keep edits precise**
  - Change only code required by the task.
  - Remove only code made obsolete by your change.
  - Do not refactor unrelated code opportunistically.

- **Report only verified results**
  - Do not claim a fix is complete unless you ran the relevant check.
  - If verification was skipped, say so plainly.
