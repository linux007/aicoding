---
name: Bootstrap Project
description: Run the reusable OpenSpec + GitNexus bootstrap flow for the current repository
category: Setup
---

Bootstrap the current project for recurring OpenSpec + GitNexus usage.

Use the global skill `openspec-gitnexus-bootstrap` for the actual workflow.

If the user provided extra arguments after `/bootstrap-project`, treat them as extra constraints for the bootstrap run, such as:
- only run preflight
- don't modify CLAUDE.md
- prefer audit mode
- initialize fully

Default behavior when no extra arguments are given:
1. Run the preflight from `openspec-gitnexus-bootstrap`
2. Report `BLOCKED` / `ACTION` / `ASK` / `OK`
3. If not blocked, continue with minimal initialization
4. Stop and ask before overwriting existing project-level configuration

## 交互约定

- Preflight 完成后，如果有 `BLOCKED` 项，明确告知用户："当前被 BLOCKED，请手动处理后回复 **继续**，或回复 **跳过** 跳过该步骤"
- 如果所有项都是 `OK` 或 `ACTION`（可自动执行），自动继续，不要等待用户确认
- `ASK` 项需要用户确认后再执行
- 一旦用户是在回答某个 `ASK` 项（例如“不要追加”“跳过这个”“先不改 CLAUDE.md”），处理完该 `ASK` 的决定后必须暂停，等待用户明确指示是否继续其他 `ACTION`；不要把这次回复解释为对剩余步骤的默认同意
