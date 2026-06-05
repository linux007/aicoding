---
name: openspec-gitnexus-bootstrap
description: Use when the user wants to initialize a new project or repository for recurring OpenSpec plus GitNexus usage, especially when they mention 新项目初始化、搭环境、配置 openspec、配置 gitnexus、setup repo tooling, or want a reusable bootstrap flow across projects.
---

# OpenSpec + GitNexus Bootstrap

为新项目做最小可复用初始化：先跑 preflight，再补齐 OpenSpec、项目级 Superpowers 接入和 GitNexus 索引。

## When to Use

适用于：
- 新项目初始化 OpenSpec + GitNexus
- 想把仓库接成可用 `/opsx:*` 和 GitNexus MCP 的状态
- 想沉淀一个跨项目复用的 bootstrap 流程

不适用于：
- 只实现功能或修 bug
- 只想创建一个 OpenSpec change
- 只想查询 GitNexus，不需要初始化

## Workflow

### 0. Preflight

先执行（所有命令加 `|| echo "UNAVAILABLE"` fallback，避免交互阻塞）：

```bash
git status --short
git rev-parse --is-inside-work-tree 2>/dev/null || echo "NOT_GIT"
ls
which openspec 2>/dev/null || echo "UNAVAILABLE"
which claude 2>/dev/null || echo "UNAVAILABLE"
claude plugin list 2>/dev/null || echo "UNAVAILABLE"
ls openspec 2>/dev/null || echo "UNAVAILABLE"
ls openspec/schemas 2>/dev/null || echo "UNAVAILABLE"
ls .claude/commands 2>/dev/null || echo "UNAVAILABLE"
ls .gitnexus 2>/dev/null || echo "UNAVAILABLE"
```

路径不存在时，允许命令失败；把失败当状态信号。

按下面规则判断：

- **OpenSpec**
  - `which openspec` 返回 UNAVAILABLE → `BLOCKED: OpenSpec CLI missing`（需要用户手动安装）
  - 缺少 `openspec/` → `ACTION: openspec init`（可自动执行）
  - 缺少 `openspec/schemas/superpowers-bridge/` → `ACTION: install superpowers-bridge`（可自动执行）
  - `openspec/config.yaml` 存在但默认 schema 不是 `superpowers-bridge` → `ASK: switch default schema?`

- **GitNexus**
  - GitNexus MCP/CLI 不可用 → `BLOCKED: GitNexus unavailable`（需要用户手动安装）
  - 当前目录不是 git 仓库（`git rev-parse` 返回 `NOT_GIT`）→ 不要判为 BLOCKED；改为 `ACTION: npx gitnexus analyze --skip-git`（可自动执行）
  - 可用且当前目录是 git 仓库，但 repo 未索引或索引陈旧 → `ACTION: npx gitnexus analyze`（可自动执行）
  - 可用且 repo context / repo list 正常 → `OK: GitNexus ready`

- **Superpowers**（项目级优先，不依赖全局）

  先解析 `claude plugin list` 输出，精确判断 enabled/disabled 状态，而非仅判断是否安装：
  ```bash
  claude plugin list 2>/dev/null | grep -A3 'superpowers@' || echo "NOT_INSTALLED"
  ```
  关注 `Status: ✔ enabled` vs `Status: ✘ disabled`。

  - 项目级已启用（`superpowers@...` 在 project scope 下 Status = `✔ enabled`）+ 项目级已有 workflow routing（CLAUDE.md 中有 `/opsx:*` 指引）→ `OK: Superpowers ready`
  - 项目级已安装但 **disabled**（`Status: ✘ disabled`）→ `ACTION: enable superpowers at project scope`（优先尝试 `claude plugin enable superpowers --scope project`，失败则 ASK 用户手动启用）
  - 全局已安装（user scope 或 project scope 任一 enabled）+ 项目级没有 workflow routing → `ASK: add project-level workflow routing to CLAUDE.md?`
  - 全局未安装 → `ACTION: project-level minimal workflow routing`（不 BLOCKED，降级为项目级配置）
    - 在 CLAUDE.md 中追加最小 workflow routing 指引（什么时候用 `/opsx:*`，什么时候 direct PR）
    - 不要尝试全局安装 superpowers 插件

**关键原则：Superpowers 未启用优先尝试在项目级启用（`claude plugin enable`），其次降级为 CLAUDE.md 最小 routing。全局缺失不是 BLOCKED。**

Preflight 后先汇报：

```markdown
## Preflight
- OpenSpec CLI: OK | BLOCKED (需手动安装)
- OpenSpec project files: OK | MISSING (可自动 init)
- GitNexus availability: OK | BLOCKED (需手动安装)
- GitNexus repo mode: GIT | NON_GIT (NON_GIT 时可自动退化为 `--skip-git`)
- GitNexus repo index: OK | MISSING | STALE (可自动 refresh)
- Superpowers: OK | ACTION (项目级启用) | ACTION (项目级配置) | ASK (需确认)
  - 项目级 disabled 会显示 "ACTION: 尝试 claude plugin enable superpowers --scope project"

## Next actions
1. ...
2. ...
3. ...
```

汇报规则：
- `BLOCKED` 项标注 "（需手动安装）" 并告知用户："以上 BLOCKED 项需要手动处理，请处理后回复 **继续**，或回复 **跳过** 跳过该步骤"
- `ACTION` 项标注 "（可自动执行）"，仅在用户尚未介入打断时自动继续
- `ASK` 项等待用户确认后再执行
- 如果完全没有 `ASK` 且没有 `BLOCKED` 项，自动进入下一步，不要等待用户
- 如果存在 `ASK`，而用户已经对其中某项给出答复，则处理完该项后必须暂停，等待用户明确说明是否继续剩余 `ACTION`；不要把对单个 `ASK` 的回答视为对其余步骤的授权

### 1. OpenSpec minimal setup

1. 缺少 `openspec/` 时运行：
   ```bash
   openspec init
   ```
2. 缺少 `openspec/schemas/superpowers-bridge/` 时：
   - 克隆 `https://github.com/JiangWay/openspec-schemas` 到临时目录
   - 复制 `superpowers-bridge/` 到 `openspec/schemas/superpowers-bridge/`
   - 清理临时目录
3. 校验：
   ```bash
   openspec schema validate superpowers-bridge
   openspec schemas
   ```
4. 检查 `openspec/config.yaml`
   - 不存在就创建最小配置
   - 已存在但 schema 不是 `superpowers-bridge` 时先问用户

最小配置：

```yaml
schema: superpowers-bridge
context: |
  语言：中文（简体）
  所有产出物必须用简体中文撰写。
```

### 2. Project-level Superpowers routing

**不依赖全局 superpowers 插件，项目级 workflow routing 独立可用。**

#### 2a. 优先尝试启用项目级 Superpowers

如果 Preflight 发现 superpowers 已安装但 disabled（project scope 或 user scope 任一 `✘ disabled`）：

```bash
# 尝试启用 project scope
claude plugin enable superpowers@claude-plugins-official --scope project 2>/dev/null
# 如果上面的失败，尝试 zyb 版本
claude plugin enable superpowers@zyb-plugins --scope project 2>/dev/null
```

- 启用成功 → 验证 `claude plugin list | grep -A3 superpowers` 确认 Status = `✔ enabled`
- 启用失败 → 告知用户失败原因，降级到 2b

#### 2b. 项目级最小 workflow routing（降级方案）

如果没有 `CLAUDE.md`，不要自动生成整份文档；先问用户是否补最小片段。

如果已有 `CLAUDE.md`，只检查并追加最小 workflow routing：
- 什么时候用 `/opsx:*`
- 什么时候 direct PR
- brainstorming 不应写到 `docs/superpowers/specs/`
- 项目采用 `superpowers-bridge` 时优先走项目内工作流

**如果全局没有 superpowers 插件且启用失败，不要尝试安装，只做上述项目级配置即可。**

### 3. GitNexus minimal setup

1. 先判断当前目录是否为 git 仓库：
   - 如果 `git rev-parse --is-inside-work-tree` 返回 `NOT_GIT`，直接走非 git 模式，不要把这一步判为 BLOCKED
   - 如果是 git 仓库，再验证当前仓库是否可从 GitNexus 读取 context 或在 `list_repos` 中找到
2. 若缺失或陈旧，运行：
   ```bash
   npx gitnexus analyze
   ```
   非 git 目录则改为：
   ```bash
   npx gitnexus analyze --skip-git
   ```
3. 完成后至少验证其中一个：
   - repo context 可读
   - `list_repos` 能看到当前 repo

repo 名不明确时，先从 `list_repos` 找到目标 repo 名。

### 4. Final report

输出保持简洁，只包含：
- OpenSpec：已存在 / 新建 / 补齐了什么
- Schema：是否已切到 `superpowers-bridge`
- Superpowers：已启用 / 已尝试启用(成功/失败) / 降级为项目级 routing
- CLAUDE.md：未改 / 已追加片段 / 等待确认
- GitNexus：已索引 / 已刷新 / 仍阻塞
- 下一步建议，最多 3 条

## Guardrails

- 不覆盖已有 `openspec/config.yaml` 的非必要内容
- 不无提示重写整个 `CLAUDE.md`
- 已有配置时优先补缺，不重装
- 影响共享状态或覆盖现有内容前先确认
- 出现 `BLOCKED` 后不要继续自动修改，明确告知用户 BLOCKED 原因和恢复方式
- 当前目录不是 git 仓库时，不要把 GitNexus 判为 BLOCKED；优先退化到 `npx gitnexus analyze --skip-git`
- **Superpowers 未启用优先尝试 `claude plugin enable --scope project`，失败后降级为项目级配置**
