---
name: project-rule-extraction
description: 基于真实代码模式提炼项目私有规则，并产出可直接落地到 `.claude/rules/` 的小规则文件。适用于“帮我提炼项目规则”“把现有代码模式固化成 Claude 规则”“从 controller/service/model/test 代码里总结规范”“给这个 Go 仓生成项目私有规则”这类请求；尤其适合多仓 workspace，需要先选目标子仓、再用 codebase-memory 缩范围、最后回填 CLAUDE.md 规则索引的场景。
license: MIT
compatibility: Codebase-memory MCP strongly recommended.
metadata:
  author: local
  version: "0.1"
---

这个 skill 只做一件事：**从真实代码里抽稳定模式，生成项目私有规则。**

适用于：
- “帮我总结这个项目的 Claude 规则”
- “把现有 Go 代码约定提炼成 `.claude/rules`”
- “从 controller/service/model/test 里抽规则”
- “多仓 workspace 里，帮我先选目标仓再提炼规则”
- “把本次规则提炼流程做成可复用动作”

不适用于：
- 用户已经明确给出规则内容，只需要机械写入文件
- 只想要通用语言风格，不关心项目私有模式
- 当前任务是实现功能或修 bug，而不是沉淀规则

## 核心规则

### 1. 先确认目标仓，不要混提多仓风格
如果当前 workspace 有多个 repo 或多个 Go module，先确认：
- 目标 repo
- 解释基线：当前分支还是主干
- 规则落点：目标 repo 的 `.claude/rules/` 还是 workspace 根目录
- 是否允许新增规则文件
- 第一处预计编辑位置

如果用户没有指定，必须先问。

### 2. 先用 codebase-memory 缩范围，再少量读文件
优先使用：
- `list_projects`
- `get_architecture`
- `search_graph`
- `search_code`
- `get_code_snippet`

只在已经定位到高代表性样本后，才用 `Read` 补充少量精确代码块。
不要一上来顺序读很多文件。

### 3. 样本按层抽，不按目录扫全量
至少覆盖这些层中的 3 到 4 类：
- controller / handler
- service
- model / storage / data
- request DTO / response DTO
- test

每层只读 1 到 3 个代表性样本，目标是找到“重复模式”，不是覆盖全仓。

优先抽这些信息：
- 参数绑定和校验在哪里做
- 业务编排落在哪层
- DTO 和 model 的边界在哪里
- 错误如何包装和透传
- context 如何传递
- 测试 seam 如何建立

### 4. 只提炼稳定重复模式，不提炼偶发实现细节
只有满足下列条件之一的模式，才写成规则：
- 在多个文件中重复出现
- 是仓内明确的层边界
- 能阻止 Claude 常见漂移
- 与现有 `CLAUDE.md` / memory 明确一致

不要把单个函数的偶发写法提升为项目规则。
不要把你个人偏好写成项目规则。

### 5. 规则文件要小而聚焦
优先拆成多个小规则文件，而不是一个总纲。
建议拆分方向：
- controller / service 边界
- DTO / model 边界
- error / testing 模式
- callback / MQ / async 入口
- 领域专属规则（如果某个业务域足够稳定）

每个规则文件只负责一个主题。

### 6. 写完规则后，回填 CLAUDE.md 索引
如果 repo 内已有 `CLAUDE.md`，补一个简短的本地规则索引段，说明：
- 有哪些规则文件
- 各自约束什么
- 修改该 repo 时应优先遵循这些本地规则

只加索引说明，不重写已有架构文档。

## 固定流程

### Step 1. 确认约束
按下面顺序确认或说明：
- 目标 repo
- 基线分支
- 规则文件落点
- 是否允许新增规则文件
- 第一处预计编辑位置

### Step 2. 用图谱定位候选样本
先做：
- `list_projects`
- `get_architecture`
- `search_graph` / `search_code`

目标是缩到少量高价值入口，而不是立刻写规则。

### Step 3. 读取代表性样本
优先读：
- 一个薄 controller
- 一个 service orchestration 入口
- 一个 DTO mapping helper
- 一个 model/data 查询入口
- 一个 controller 或 service test

### Step 4. 提炼规则候选
按下面格式整理：
- pattern:
- evidence:
- scope:
- why_it_matters:
- rule_file_candidate:

如果证据不足，标记为“待确认”，不要直接成文。

### Step 5. 生成 `.claude/rules/*.md`
规则文件必须：
- 有 frontmatter
- `description` 说明触发场景
- `paths` 尽量收敛到相关目录
- `alwaysApply` 按实际需要设置
- 内容短、硬、可执行

### Step 6. 更新 `CLAUDE.md`
只新增一个本地规则索引段，说明规则入口。
不要顺手改其他段落。

## 输出要求

最终输出至少包含：

### 1. 目标仓与约束
- target_repo:
- baseline:
- rule_location:
- first_edit_location:

### 2. 提炼依据
- controller sample:
- service sample:
- model/data sample:
- dto sample:
- test sample:

### 3. 新增规则文件
- path:
- purpose:

### 4. 已回填的 CLAUDE.md 入口
- path:
- section:

### 5. 待确认点
- 

## 验收清单

交付前自查：
- 是否先确认了目标 repo，而不是混提多仓风格
- 是否先用 codebase-memory 缩范围
- 是否只读了少量高代表性样本
- 是否所有规则都能追溯到真实代码证据
- 是否拆成了小规则文件，而不是一份大总纲
- 是否把规则入口补回了 `CLAUDE.md`

## Guardrails

- 不要跳过目标仓确认，尤其是多仓 workspace
- 不要从通用最佳实践直接推导项目私有规则
- 不要在没有图谱定位前连续 `Read` 多个代码文件
- 不要把单次实现细节、临时 workaround、历史脏代码写成规则
- 不要顺手改业务代码；这个 skill 的产物是规则和索引，不是功能实现
