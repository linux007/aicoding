# 代码探索工具选择策略

**Extracted:** 2026-04-02
**Context:** Go 代码迁移时，从 stub 实现调研可用数据

## Problem
当需要从 stub 实现完善功能时，如何高效找到所需的数据来源？使用 Grep/Read 多次迭代效率低。

## Solution

### 工具优先级（从高到低）

1. **LSP 工具**（hover/goToDefinition/findReferences）
   - 直接查看类型定义和导航
   - 无需猜测行号

2. **Agent Explore**
   - 广泛搜索未知区域
   - 一次探索多文件

3. **Grep**
   - 精确文本匹配
   - 备选方案

4. **Read**
   - 已知文件路径时使用
   - 最后备选

### 低效模式（避免）
```
Grep "widgetsCtx" → 很多结果 → 再 Grep "lessonInfos" → 再 Read 文件
```

### 高效模式（推荐）
```
LSP: hover widgetsCtx → 直接看到 struct 完整定义
LSP: goToDefinition GetLessonInfoByLessonId → 直接跳到方法实现
```

## Stub 实现处理流程

当发现 `// stub:` 或 `// TODO: 无法获取` 注释时：

1. 搜索 `*Ctx` 结构体 - 可能已包含所需数据但未使用
2. 搜索对应的 `GetXxxById` 方法 - 数据访问入口
3. 检查 `widgetsCtx` 等聚合上下文 - 已填充的相关数据

## Example

**目标：** 从 `tutoringBackup` stub 实现完善功能

**低效方式：**
```bash
Grep "tutoringBackup" → 找到 stub 位置
Grep "widgetsCtx" → 找到很多结果
Grep "lessonInfos" → 再找
Read teacherwidgets.go → 找到结构体定义
```

**高效方式：**
```bash
# 1. LSP hover 查看 teacherFeatureCtx 定义
LSP: hover teacherFeatureCtx

# 2. 直接查看 widgetsCtx 结构体
LSP: hover widgetsCtx

# 3. 找到 GetLessonInfoByLessonId 方法
LSP: goToDefinition GetLessonInfoByLessonId
```

## When to Use

- 当需要从 stub 实现完善功能时
- 当需要调研某个 Context 结构体包含哪些数据时
- 当需要找到某个方法的定义时
- 当 Grep/Read 多次迭代效率低时
