---
description: Enforce concise Uber-style Go generation in this project
paths:
  - "**/*.go"
alwaysApply: true
---

# Go General Uber Style

- **Keep code small and explicit**
  - Prefer the simplest working implementation.
  - Do not add helpers, wrappers, builders, factories, or config unless the task needs them now.
  - Match surrounding code style exactly.

- **Use clear names**
  - Names must be descriptive and searchable.
  - Avoid meaningless abbreviations.
  - Keep receiver names short and consistent.
  - Do not name interfaces with `Interface` suffix.

- **Keep functions focused**
  - Prefer short functions with one responsibility.
  - Use early returns to reduce nesting.
  - Avoid variable-function patterns.

- **Use interfaces sparingly**
  - Introduce interfaces only when multiple implementations, caller decoupling, or testing clearly require them.
  - Prefer small consumer-side interfaces.
  - Do not create an interface for a single implementation.

- **Handle errors explicitly**
  - Never swallow errors.
  - Error messages should be lowercase and concise.
  - Wrap errors only when adding meaningful context.
  - Do not use panic for normal business errors.

- **Be strict with pointers and collections**
  - Check pointers for nil before dereferencing.
  - Check slice length before indexing or slicing.
  - Keep nil vs empty slice/map behavior consistent inside the same layer.

- **Comment only when needed**
  - Explain why, not what.
  - Use GoDoc style for exported symbols.
  - Avoid obvious comments.
