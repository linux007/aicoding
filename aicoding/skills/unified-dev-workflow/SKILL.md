---
name: unified-dev-workflow
description: "统一的开发工作流指南，整合 OpenSpec（规范层）和 Superpowers（执行层）。当用户开始任何开发任务、提到'新功能'、'bug修复'、'重构'、'迁移'、'开发流程'、'怎么做这个需求'时必须使用此 skill。即使用户没有明确说'流程'，只要是开发相关任务也应触发。"
---

# 统一开发工作流

整合 OpenSpec 和 Superpowers 的开发工作流指南。

## 核心分工

```
OpenSpec = 规范层（记录决策、团队共享、历史追溯）
Superpowers = 执行层（TDD 强制、质量保证、并行执行）

一句话：用 OpenSpec 定义"建什么"，用 Superpowers 保证"建对"
```

---

## 场景选择器

根据用户需求判断场景类型：

| 场景 | 判断条件 | 流程 |
|------|---------|------|
| **新功能开发** | 用户说"实现"、"添加"、"开发"、"新增" | Flow A |
| **Bug 修复** | 用户说"报错"、"失败"、"异常"、"bug"、"修复" | Flow B |
| **小需求修改** | 修改量 < 20 行 且 无需归档 | Flow C |
| **代码重构** | 用户说"重构"、"优化"、"清理" 且 不改变行为 | Flow D |
| **代码迁移** | 用户说"迁移"、"从 A 到 B" | Flow D |

### 判断规则

```
IF 影响范围 > 3 个文件 OR 需要团队评审 OR 需要历史追溯:
    使用 OpenSpec
ELSE:
    跳过 OpenSpec

IF 涉及业务逻辑代码:
    使用 Superpowers TDD
ELSE:
    可简化流程
```

---

## Flow A: 新功能开发

### 流程

```
Phase 1: 需求探索 (OpenSpec)
    /opsx:propose <feature-name>
    → 创建 proposal.md + specs/ + design.md + tasks.md
    → 用户评审确认

Phase 2: 实现规划 (桥接)
    基于 tasks.md 创建 Superpowers plan
    → docs/superpowers/plans/YYYY-MM-DD-<feature>.md

Phase 3: TDD 实现 (Superpowers)
    对每个任务执行 RED-GREEN-REFACTOR 循环
    → Write failing test → Implement → Test pass → Commit
    → Code Review

Phase 4: 归档 (OpenSpec)
    /opsx:archive
    → 更新 specs/，归档变更历史
```

### 检查清单

**Phase 1**:
- [ ] proposal.md 包含目标、范围、排除项
- [ ] specs/ 包含 Given/When/Then 场景
- [ ] design.md 包含架构图、数据流
- [ ] tasks.md 任务粒度适中（每任务 1-2 小时）

**Phase 2**:
- [ ] 计划与 tasks.md 一一对应
- [ ] 每个任务有明确文件路径
- [ ] 每个任务有测试验证命令

**Phase 3**:
- [ ] 所有测试先行（RED 验证）
- [ ] 覆盖率 ≥ 80%
- [ ] 无 CRITICAL/HIGH 问题

**Phase 4**:
- [ ] specs 已更新
- [ ] archive 目录已创建

---

## Flow B: Bug 修复

### 流程

```
Phase 1: 问题定位 (Superpowers systematic-debugging)
    [1] 收集信息（错误日志、复现步骤、影响范围）
    [2] 假设原因（≥2个）
    [3] 验证假设
    [4] 确定根因

Phase 2: 记录问题 (OpenSpec)
    /opsx:propose fix-<bug-name>
    → proposal.md（根因分析）
    → specs/（修复后预期行为）
    → tasks.md（修复步骤）
    注意：design.md 可省略

Phase 3: TDD 修复
    [1] 写复现测试（必须 FAIL）
    [2] 实现修复
    [3] 测试通过
    [4] 回归测试

Phase 4: 归档
    /opsx:archive
    Commit message 包含 Root Cause、Fix、Test
```

### 简化条件

```
IF bug 影响范围 < 1 个文件 AND 修复时间 < 30 分钟:
    可跳过 Phase 2（OpenSpec）
```

---

## Flow C: 小需求修改

### 适用条件

- 修改量 < 20 行
- 影响文件 < 3 个
- 无需团队评审
- 无需历史追溯

典型场景：配置调整、文案修改、日志添加、小样式调整

### 流程

```
[1] 理解需求（简短对话）
[2] 判断是否需要测试
    ├── 业务逻辑 → 写测试 → 实现 → 验证
    └── 配置/文案/日志/样式 → 直接实现 → 手动验证
[3] Commit
```

### 是否需要测试

| 修改类型 | 需要测试 | 原因 |
|---------|---------|------|
| 业务逻辑 | ✓ | 防止回归 |
| 配置修改 | ✗ | 手动验证 |
| 文案修改 | ✗ | 无逻辑风险 |
| 日志添加 | ✗ | 不影响业务 |
| 样式调整 | ✗ | 视觉验证 |

---

## Flow D: 代码重构/迁移

### 流程

```
Phase 1: 现状分析 (OpenSpec)
    /opsx:explore → 分析依赖、影响、风险
    /opsx:propose refactor-<name> 或 migrate-<name>
    → specs/ 定义"行为不变"的验证标准

Phase 2: 安全网构建 (Superpowers)
    [1] 确保现有测试通过
    [2] 补充缺失的测试（行为锚点）
    [3] 记录当前行为快照
    
    原则：没有测试覆盖的代码不允许重构

Phase 3: 分步重构
    每步：测试通过 → 小幅修改 → 测试仍通过 → Commit
    原则：每次提交后系统可正常运行

Phase 4: 验证与归档
    全量测试 → 覆盖率未降 → 性能无退化 → /opsx:archive
```

### 重构 vs 迁移

| 维度 | 重构 | 迁移 |
|------|------|------|
| 目标 | 改善内部结构 | 从 A 到 B 系统 |
| 行为 | 必须不变 | 可能有变化 |
| 验证 | 测试不变 | 对比验证 |

### 迁移额外检查

- 源系统行为已记录
- 数据映射关系已定义
- 并行运行期验证（双写双读）
- 回滚方案已测试

---

## 工具命令速查

### OpenSpec

| 命令 | 用途 | 适用 |
|------|------|------|
| `/opsx:propose <name>` | 创建变更提案 | A, B, D |
| `/opsx:explore` | 探索性问题分析 | B, D |
| `/opsx:apply` | 执行任务 | A, B, D |
| `/opsx:archive` | 归档变更 | A, B, D |

### Superpowers

| Skill | 用途 | 适用 |
|-------|------|------|
| `systematic-debugging` | Bug 定位 | B |
| `test-driven-development` | TDD 实现 | A, B, D |
| `subagent-driven-development` | 并行执行 | A, D |
| `requesting-code-review` | 代码审查 | A, B, D |

---

## 反模式警告

| 反模式 | 正确做法 |
|--------|---------|
| 同时运行 `/opsx:propose` 和 `brainstorming` | 只用 OpenSpec |
| 实现时不参考 OpenSpec tasks | 以 tasks.md 为准 |
| 跳过 TDD 直接实现 | 强制使用 Superpowers TDD |
| 不归档就结束 | 必须 `/opsx:archive` |
| 设计变更不同步 | 更新 OpenSpec design.md |

---

## 执行步骤

1. **声明场景**：告诉用户检测到的场景类型和将使用的流程
2. **执行流程**：按照对应 Flow 的步骤执行
3. **检查清单**：每个 Phase 完成后确认检查点
4. **报告进度**：关键节点向用户汇报

开始任何开发任务时，首先声明：
> "检测到 [场景类型] 场景，使用 Flow [X]。"
