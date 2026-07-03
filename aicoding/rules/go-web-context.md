---
description: Enforce request-scoped context handling for Go web code
paths:
  - "**/*.go"
alwaysApply: true
---

# Go Web Context

- **Pass request context explicitly**
  - All request-scoped operations must receive `ctx` explicitly.
  - In web handlers and request flows, use `*gin.Context` as the first parameter when the surrounding code follows that pattern.
  - Never store request context in struct fields.

- **Do not replace upstream context silently**
  - Do not create a fresh background context in lower layers.
  - Only use `context.WithTimeout` or `context.WithCancel` when the operation truly needs its own boundary.
  - Always call `cancel()` when deriving a cancellable context.

- **Keep layering explicit**
  - Pass context through controller, service, storage, and downstream client calls.
  - Do not hide request-scoped values in globals or long-lived objects.
