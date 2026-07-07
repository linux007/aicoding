---
name: solution-to-openspec-changes
description: Turn a complex cross-system solution document, design discussion, or decomposition request into a sequenced OpenSpec change plan and ready-to-use /opsx:propose prompts. Use this whenever the user wants to 拆需求、拆任务、拆 changes、生成 proposal 文案、按 OpenSpec/OPSX 推进，especially for multi-subsystem work with shared contracts, request/response 协议、字段命名、示例 JSON、兼容旧接口, or end-to-end chains like save/init/production/callback/signal/preview. If the user mentions “接口协议要对齐文档”, “字段不能改”, “init/detail/list/test 语义要保留”, “不要直出 model”, or provides example request/response JSON, strongly prefer this skill. This skill stops at change planning and /opsx:propose drafting; later OpenSpec execution is handled by OPSX itself.
---

# solution-to-openspec-changes

把“复杂跨系统方案”整理成“一组可顺序推进的 OpenSpec changes”，并生成对应的 `/opsx:propose` 文案。

只做三件事：
1. 判断是否需要拆成多 change
2. 产出 change 规划
3. 生成 `/opsx:propose` 文案

到此为止。`/opsx:propose` 之后交给 OPSX。

---

## When to use

优先用于这些场景：

- 用户给了方案文档、技术设计、长篇需求
- 需求涉及多个子系统、旧链路复用、共享协议、兼容性约束
- 有明确端到端主线，例如 `save -> production -> callback -> signal -> preview`
- 用户想拆需求、拆 changes、产 proposal、按 phase 推进、安排并行开发
- 用户强调：
  - 接口协议要对齐文档
  - 字段名不能变
  - `init/detail/list/test` 语义不能丢
  - 不要直接返回 model
  - 示例 JSON 就是验收基准

---

## When NOT to use

不要用于：

- 单接口 CRUD
- 单文件修改
- 小型 bugfix
- 纯实现请求
- 用户已经有明确 change，只需要开始实现
- 没有跨系统复杂性，只是在问“这个怎么实现”

这类情况直接走正常实现或更轻量的 OpenSpec / implementation skill。

---

## Core principles

1. **按业务闭环和系统责任拆**，不要按 `controller/service/model` 拆
2. **共享约束先冻结**，尤其是协议、状态机、ID 规则、兼容性边界
3. **高风险项单列**，例如 ID 冲突、主链回接、删除治理、幂等、补偿
4. **联调主线单列**，不要缩成一句“最后联调”
5. **一个 change = 一个可评审闭环**，必须能说清 scope、non-scope、依赖、交付物

---

## Contract-first rule

如果文档里出现以下任一内容，必须先把它们当成“外部契约”：

- request / response 示例 JSON
- 字段命名要求（如 `resourceId` vs `id`）
- `init/detail/list/test/publish/delete` 的接口语义
- “Signal 顶层结构不变”之类的协议边界
- “禁止 model 直出 / 必须 DTO 组装”之类的实现边界

先整理 contract matrix，再决定怎么拆 change。

contract matrix 至少包含：
- 接口路径
- 接口语义
- request 字段
- response 字段 / 层级
- 示例 JSON 是否为验收基准
- 禁止事项（如不得直出 model）
- 是否建议冻结成前置 change

如果多个后续 changes 都依赖这些契约，优先建议开：
- `freeze-xxx-contracts`
- `freeze-xxx-api-shapes`
- `freeze-xxx-compat-rules`

---

## Workflow

### Step 1: 判断是否适合用本 skill
满足以下任意两项，就按本 skill 执行：
- 涉及 2 个以上子系统
- 有共享协议 / 契约 / 状态机
- 要兼容旧链路 / 老模板 / 老接口
- 有明确端到端主线
- 需要多人并行或分阶段推进

不满足时，明确建议：
- 单 change
- 或直接改代码

### Step 2: 收集输入并看 OpenSpec 现状
优先从对话提取：
- 文档路径、标题、目标、范围 / 非范围
- 已确认的拆分结论
- 端到端主线
- 共享约束
- 已给出的接口示例 JSON / 字段命名要求
- 是否存在“init 不是 detail”“不能直出 model”之类的协议警告

如果用户明确要对接 OpenSpec，检查是否已有相关 change：

```bash
openspec list --json
```

目标：
- 避免重复开 change
- 判断是新开还是补充已有 change

### Step 3: 提炼 contract matrix
如果文档里有接口协议、示例 JSON、字段命名或兼容性要求，先输出 contract matrix：
- path
- 语义（init / detail / list / test / publish / delete）
- request contract
- response contract
- 示例 JSON 是否为验收基准
- 禁止事项
- 是否建议冻结

如果 contract 不完整，也要明确指出缺口。

### Step 4: 先做需求拆分
至少输出：
1. 一句话拆分原则
2. 子系统拆分
3. 共享约束
4. 接口契约矩阵
5. 高风险专项
6. 联调主线
7. phase 顺序

如果用户已经给了人工拆分结果：
- 优先承接
- 不重复发散
- 只补缺失的共享约束、契约、风险和主线

### Step 5: 映射成 OpenSpec changes
切 change 时遵守：
- 共享契约 / 兼容性 / ID 规则优先前置单列
- 如果多个 changes 依赖同一批 API shape，优先先切 `freeze-xxx-contracts`
- 资产层、编排层、运行链路、预览层优先分开
- 联调 / 灰度 / 上线收口可以独立成 change
- 不要把多个大闭环塞进一个 change
- 不要按代码分层切 change

每个 change 至少给出：
- change 名称
- 一句话目标
- 范围
- 非范围
- 前置依赖
- 是否可并行
- 是否承担契约冻结责任

### Step 6: 生成 `/opsx:propose` 文案
默认只生成：
- 第一个 change 的完整 `/opsx:propose` 文案

如果用户明确要更多，再继续生成后续 changes 的 proposal 文案。不要默认一次性铺满。

---

## Output template

默认按这个结构输出：

### 1. 拆分原则
一句话说明本次拆分的核心原则。

### 2. 子系统拆分
按业务闭环列出：
- 模块名
- 目标
- 范围
- 产物

### 3. 共享约束
列出必须先冻结的规则。

### 4. 接口契约矩阵
按接口列出：
- path
- 语义
- request contract
- response contract
- 示例 JSON 是否视为验收基准
- 禁止事项（如不得直出 model）
- 是否建议冻结

### 5. 高风险专项
列出高风险改造点和原因。

### 6. 联调主线
至少给出：
- 1 条端到端主线
- 2~4 个最小联调样例

### 7. Change 规划
按顺序列出：
- change 名
- 目标
- 依赖
- 是否可并行
- 是否负责冻结契约

### 8. `/opsx:propose` 文案
至少生成第一个 change 的完整 proposal 文案。

---

## Good change shapes

优先这些命名：
- `freeze-xxx-contracts`
- `add-xxx-assets`
- `support-xxx-orchestration`
- `extend-xxx-signal-pipeline`
- `support-xxx-preview`
- `verify-xxx-e2e`

避免这些命名：
- `add-xxx-models`
- `update-controllers-and-services`
- `refactor-everything-for-phase3`
- `implement-full-solution`

常见顺序：

```text
Change 0 共享约束 / 接口协议冻结
   ↓
Change 1 资产层
   ↓
Change 2 编排/save/init
   ↓
Change 3 production/callback/signal
   ↓
Change 4 preview/回显
   ↓
Change 5 联调/上线收口
```

需求小的时候，不必强行拆这么多。

---

## Proposal drafting rules

生成 `/opsx:propose` 文案时，除了常规的背景、范围、非范围、依赖、验收标准，还必须写清：

### A. 接口协议硬约束
至少写出：
- request/response 是否必须逐字段对齐文档
- 示例 JSON 是否视为验收基准
- 字段命名是否允许调整
- `init/detail/list/test` 的语义是否必须保留

### B. DTO / model 边界约束
至少写出：
- controller/service 对外返回是否必须 DTO 化
- 是否禁止直接返回数据库 model
- 哪些接口最容易被“数据库直出”偷懒实现

### C. Contract-based acceptance
至少写 2~5 条接口级验收标准，例如：
- `POST /xxx/init` 的 request / response shape 与文档示例一致
- `resourceId` 不得被替换成 `id`
- `init` 不得被简化成 detail by id
- `detail/list/test` 必须包含文档规定字段

写 proposal 时要能回答：
- 为什么单独开这个 change
- 本 change 不做什么
- 后续哪些 changes 依赖它
- 哪些协议如果不冻结，后续实现最容易跑偏

---

## Example direction

如果用户说：
“根据这份方案文档，先拆需求，再产出任务拆分对应的 changes，最后帮我生成 `/opsx:propose` 文案。”

好的输出方向通常是：
- 共享契约 / 接口协议冻结
- 资产层
- 编排层
- signal 链路
- preview 层
- 联调收口

映射成 changes 可能是：
- `freeze-digital-phase3-contracts`
- `add-digital-phase3-assets`
- `support-digital-phase3-orchestration`
- `extend-digital-phase3-signal-pipeline`
- `support-digital-phase3-preview`
- `verify-digital-phase3-e2e`

如果文档已经给了 `question/init`、`question/detail`、`interactive-config/detail` 的 JSON 示例，就应该在第一个 change 中先冻结这些 contract。

---

## Guardrails

### 不要做的事
- 不要跳过需求拆分直接列 changes
- 不要按 controller/service/model 拆
- 不要把兼容性只写成一句“注意兼容旧逻辑”
- 不要把联调缩成一句“最后联调”
- 不要默认延伸到 `/opsx:propose` 之后的执行流程
- 不要把所有 change proposal 一次性铺满，除非用户明确要求
- 不要只列接口名，不写 request/response contract
- 不要把 `init` 默认理解成 “detail by id”
- 不要默认允许 controller/service 直接返回 model
- 不要把“按文档实现接口”简化成“有 CRUD 就行”
- 如果文档给了示例 JSON，不要省略它在 proposal 中的约束表达
- 如果字段命名是兼容性边界，不要写成“实现时再决定”

### 要坚持的事
- 明确共享约束
- 明确前置依赖
- 明确每个 change 的 scope / non-scope
- 明确阶段顺序和并行关系
- 明确接口契约矩阵
- 明确 DTO / model 边界
- 停在 change 规划和 `/opsx:propose` 文案阶段

---

## Final reminder

这个 skill 的价值不是“把方案复述一遍”，而是把复杂方案转成：
- 一组边界清楚的 changes
- 一套清晰的顺序
- 一份可直接执行的 `/opsx:propose` 文案
- 一套能防止后续实现把接口协议做偏的硬约束

如果用户看完就能知道：
- 先开哪个 change
- 为什么先开它
- 哪些接口 contract 要先冻结
- 后面怎么排
- proposal 怎么写
- 哪些实现偏差必须提前拦住

那这个 skill 就成功了。
