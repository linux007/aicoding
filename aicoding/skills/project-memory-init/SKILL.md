---
name: project-memory-init
description: 初始化项目记忆系统，自动扫描代码结构并生成可快速检索的模式文件。当用户说"初始化项目记忆"、"project-memory-init"、"生成项目记忆"、"扫描项目模式"时使用此 skill。支持 Go/TypeScript/Python/PHP 等多语言项目。
---

# Project Memory Init

为项目自动初始化一套可快速检索的记忆文件系统，让 AI 在后续开发中能自动参考项目模式，减少 grep 调用，提升实现准确性。

## 核心原则

1. **CLAUDE.md 是权威来源** — 构建命令、目录结构、错误码范围、路由分组等已在 CLAUDE.md 中的内容，记忆文件**不要重复**，只需引用
2. **记忆文件记录 CLAUDE.md 没有的代码级模式** — 函数签名、调用模板、参数校验流程、缓存策略等
3. **从代码中发现，不预设** — 约束和模式必须来自实际代码扫描，不能套用其他项目的规则

---

## 第一步：读取 CLAUDE.md

扫描前先读取项目根目录 `CLAUDE.md`（如果存在），记录以下内容，后续生成时**跳过这些已覆盖的信息**：
- 构建/测试命令
- 目录结构说明
- 已列出的路由分组
- 已列出的错误码范围
- 已列出的关键依赖

如果 CLAUDE.md 不存在，继续扫描，这些基础信息会写入 MEMORY.md。

---

## 第二步：项目扫描（并发探索）

根据项目语言自动识别（通过 `go.mod` / `package.json` / `requirements.txt` / `composer.json` 等），同时启动后台 explore agent：

```
Agent 1: 外部调用扫描 — 找出所有 HTTP/RPC 客户端 wrapper，提取方法名、协议、响应结构
Agent 2: 入口/路由扫描 — 找出路由注册文件，提取所有端点、Handler、中间件链
Agent 3: 分层结构扫描 — 识别项目的层级组织（如 controller/service/data 或 routes/services/models 等），提取函数签名风格、调用链
Agent 4: 数据/缓存扫描 — 找出数据库/缓存访问模式
Agent 5: 特征扫描 — MQ consumer、错误定义、中间件、数据结构定义
```

主线程同时执行：
```bash
# 项目规模
find . -type f -not -path '*/\.*' -not -path '*/node_modules/*' -not -path '*/vendor/*' -not -path '*/target/*' | wc -l

# 路由文件定位（多语言）
find . -type f \( -name "http.go" -o -name "routes.*" -o -name "router.*" -o -name "*route*" -o -name "urls.py" -o -name "*controller*index*" \) -not -path '*/\.*'

# 全局项目记忆路径（用于检测是否已有记忆）
GLOBAL_MEM_DIR="$(dirname "$(pwd)")/.claude/projects/$(echo "$(pwd)" | md5sum | cut -d' ' -f1)/memory"
```

---

## 第三步：确定需要哪些记忆文件

不是所有项目都有相同层次。按**实际扫描结果**动态决定。

### 通用（几乎所有项目都有）

| 文件 | 记录内容 | 何时省略 |
|------|---------|---------|
| `api-calls.md` | 外部 HTTP/RPC 调用方式、协议、响应结构 | 项目完全不调用外部服务时省略 |
| `error-handling.md` | 错误码范围、错误定义方式、包装模式 | 所有项目都有，不应省略 |

### 有条件（扫描到对应特征时创建）

| 文件 | 创建条件 | 记录内容 |
|------|---------|---------|
| `controller.md` | 存在 HTTP handler/web 路由层 | Handler 签名、参数校验、响应封装、端点清单 |
| `service.md` | 存在独立的业务逻辑层 | 函数签名风格、跨层调用链、目录组织 |
| `data-access.md` | 存在独立的数据访问层/缓存逻辑 | 查询模式、缓存策略、锁模式 |
| `mq.md` | 存在消息队列消费者 | 消费者注册、消息处理流程、重试策略 |
| `auth.md` | 有自定义认证/中间件链 | 认证流程、session 处理、灰度逻辑 |
| `schema.md` | 有独立的 schema/数据结构定义 | 请求/响应结构定义规范 |

---

## 第四步：写入全局项目记忆路径

**关键：必须写入全局项目记忆系统路径，禁止在项目中创建目录**

全局项目记忆路径格式：
```
~/.claude/projects/<项目目录hash>/memory/
```

获取全局路径：
```bash
GLOBAL_MEM_PATH="~/.claude/projects/$(echo "$(pwd)" | md5sum | cut -d' ' -f1)/memory"
```

示例：对于 `/Users/foo/work/myproject`，全局路径为：
```
~/.claude/projects/<hash>/memory/
```

---

## 第五步：生成记忆文件

### 通用文件结构

每个记忆文件都遵循同一结构，只是内容不同：

```markdown
---
name: {{文件名}}
description: {{一句话描述}}
type: reference
---

# {{标题}}

{{核心模式描述 + 真实代码示例}}

{{按需添加子章节：方法矩阵 / 缓存层级 / 响应 Helpers 等}}

## 端点清单（如适用）

仅在该文件涉及端点/路由时添加。表格格式：

| 端点 | 方法 | Handler | 中间件 | 功能 | 文件位置 |
|------|------|---------|--------|------|---------|
```

```markdown
## 核心实体映射

该文件涉及的关键 struct/class/函数：

| 实体名 | 类型 | 文件路径 | 功能 | 关键方法/字段 |
|--------|------|---------|------|--------------|
```

```markdown
## 约束

从代码中发现的强制性规则，使用"必须"/"禁止"等词：

- {{约束 1}}
- {{约束 2}}
```

### 生成要点

- **代码示例必须是真实项目代码**，不要用占位符
- **约束必须从代码模式中发现**，不能套用模板里的示例约束
- **端点清单从路由注册文件提取**，列出所有端点
- **实体映射列出该文件涉及的关键类型和函数**
- 每个文件不超过 200 行，保持精简

---

## 第六步：生成 MEMORY.md

MEMORY.md 放在全局记忆路径根目录，作为索引入口：

```markdown
# {{项目名}} 项目记忆

## 索引

- [API 调用模式](api-calls.md) — {{一句话}}
- [错误处理模式](error-handling.md) — {{一句话}}
- [Controller 模式](controller.md) — {{一句话}}（如存在）
...

## 补充说明

详见项目根目录 CLAUDE.md（构建命令、目录结构、路由分组、错误码范围等）
```

MEMORY.md 保持在 50 行以内，只作为索引。基础信息引导读者去看 CLAUDE.md。

---

## 第七步：质量检查

- [ ] 每个文件包含 frontmatter（name/description/type）
- [ ] 每个文件包含真实代码示例（非占位符）
- [ ] 涉及端点的文件包含端点清单表格
- [ ] 每个文件包含核心实体映射表
- [ ] 每个文件包含约束区块
- [ ] 没有重复 CLAUDE.md 已有的内容（构建命令、目录结构等）
- [ ] 没有泛泛建议（"写好代码"、"遵循最佳实践"）
- [ ] 约束是从代码中发现的，不是套用的
- [ ] 所有文件都写入全局项目记忆路径（`~/.claude/projects/<hash>/memory/`），没有在项目目录中创建

---

## 语言发现映射表

| 识别文件 | 语言 | 路由文件常见位置 | 外部调用常见位置 |
|---------|------|----------------|----------------|
| `go.mod` | Go | `router/http.go`, `cmd/*/main.go` | `api/`, `internal/api/`, `pkg/client/` |
| `package.json` | TS/JS | `src/routes/`, `pages/`, `app/api/` | `src/api/`, `src/services/`, `lib/` |
| `requirements.txt` / `pyproject.toml` | Python | `urls.py`, `views.py`, `routers.py` | `api/`, `clients/`, `services/` |
| `composer.json` | PHP | `routes/`, `app/Http/routes.php` | `app/Services/`, `app/Clients/` |

## 自动更新

初始化完成后，后续开发中：
1. 发现新模式 → 自动写入对应 memory 文件（全局路径）
2. 模式变化 → 自动更新已有文件（全局路径）
3. 发现端点清单、实体映射表相关变化→自动更新已有文件
4. 用户说"记住这个" → 写入全局项目记忆路径

**注意：所有记忆文件必须写入全局路径 `~/.claude/projects/<hash>/memory/`，禁止在项目目录中创建**
