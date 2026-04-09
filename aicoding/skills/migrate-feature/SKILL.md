---
name: migrate-feature
description: |
  模块接口功能迁移 skill。当你需要将某个功能从老模块迁移到新模块实现时使用。
  典型场景：新模块需要实现老模块已有的某个功能，PHP 老实现 → Go 新实现，或 Go 旧实现 → Go 新实现。
  当用户提到功能迁移、模块迁移、接口迁移、从老模块实现新模块、从 A 模块迁移到 B 模块时使用此 skill。
  确保使用 brainstorming 先理解迁移目标，openspec 管理 spec.md，tdd 工作流实现。
---

# 模块接口功能迁移 Skill

## Concept & Trigger

**触发场景：** 当你需要将某个功能从老模块迁移到新模块实现时使用。

**典型场景：**
- 新模块需要实现老模块已有的某个功能
- 功能迁移：老模块 A 的功能 → 新模块 B
- PHP 老实现 → Go 新实现，或 Go 旧实现 → Go 新实现
- 需要复用老模块的数据源和业务逻辑

**目标用户：** 个人开发者（自用）

## Tech Stack
- 源语言：PHP、Go
- 目标语言：Go
- 遵循目标模块的架构和代码风格

## Workflow

### Phase 1: 探索与理解

1. **Brainstorming** — 使用 `superpowers:brainstorming` 理解迁移目标
   - 确定源模块（老功能）和目标模块（新实现）
   - 理解功能的输入、输出、依赖关系

2. **创建/检查 OpenSpec** — 使用 `/opsx:new` 或检查现有 `spec.md`
   - 记录源模块的接口契约
   - 列出需要迁移的功能点
   - 确认目标模块的接口定义

3. **代码探索** — 按以下优先级调研老模块实现：

   ```
   优先级 1: LSP 工具
   ├── hover <变量名>           → 直接查看 struct 完整定义
   ├── goToDefinition <方法名>   → 跳转到方法实现
   └── findReferences <符号>     → 查找所有引用位置

   优先级 2: Agent Explore
   └── 使用 Explore agent 广泛搜索老模块实现

   优先级 3: Grep
   └── 精确文本匹配（备选）

   优先级 4: Read
   └── 已知文件路径时使用（最后备选）
   ```

### Phase 2: 实现

1. **TDD 循环** — 使用 `superpowers:test-driven-development`
   - 先写测试用例（RED）
   - 实现功能迁移代码（GREEN）
   - 重构优化（IMPROVE）

2. **更新 spec.md** — 使用 `/opsx:continue` 记录实现决策

### Phase 3: 验证与收尾

1. **Code Review** — 使用 `superpowers:requesting-code-review`
   - 检查功能一致性
   - 验证数据填充逻辑

## 功能迁移流程

### Step 1: 定位源功能

```
1. 在老模块中搜索目标功能
   └── Grep "func " + 功能名关键词

2. 找到功能入口方法
   └── LSP: goToDefinition 跳转到实现

3. 分析输入参数和返回值
   └── LSP: hover 查看结构体定义
```

### Step 2: 分析数据来源

```
1. 找到数据访问入口
   └── LSP: goToDefinition GetXxxById

2. 查看 Context 结构体
   └── LSP: hover *Ctx

3. 理解数据依赖链
   └── LSP: findReferences 查找字段引用
```

### Step 3: 在新模块中实现

```
1. 遵循目标模块的架构风格
   ├── 保持现有 package 结构
   ├── 保持错误处理模式
   └── 保持接口定义风格

2. 移植业务逻辑
   └── 从老模块迁移到新模块
```

## 工具优先级详解

| 工具 | 适用场景 | 示例 |
|------|----------|------|
| LSP hover | 查看 struct 定义、类型信息 | `LSP: hover widgetsCtx` → 直接看到完整字段 |
| LSP goToDefinition | 跳转方法实现 | `LSP: goToDefinition GetXxxById` → 跳转数据访问入口 |
| LSP findReferences | 查找符号引用 | 查找某字段在哪些地方被使用 |
| Agent Explore | 未知区域探索（PHP/Go 混合） | 并行搜索多文件 |
| Grep | 精确文本匹配 | 备选方案 |
| Read | 已知路径文件 | 最后备选 |

## 强制规则
- [ ] Phase 1 每个步骤必须完成才能进入 Phase 2
- [ ] 未完成 brainstorming 禁止修改任何代码
- [ ] 未创建 OpenSpec 禁止进入实现阶段

## 验收标准

- [ ] 功能逻辑已迁移，与源模块行为一致
- [ ] 接口契约保持不变
- [ ] 遵循目标模块架构和风格
- [ ] 相关测试通过
- [ ] spec.md 已更新
- [ ] Code review 通过

## 不要做

- 不要改变现有接口契约（除非明确要求）
- 不要直接复制老模块代码到新模块
- 不要破坏老模块的现有功能
- 不要忽略目标模块的架构风格
- 不要自动进行git提交
