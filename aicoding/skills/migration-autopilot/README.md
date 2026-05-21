# migration-autopilot 使用说明

## 这是什么

`migration-autopilot` 是一份专门面向“迁移类开发任务”的 workflow skill，适合这类场景：

- 把逻辑从 `liveme` 迁到 `classme`
- 老模块和新模块之间做行为对齐
- 在模块 / 服务 / 框架之间迁移功能
- 把旧实现 rewrite 到目标栈里，但**不做逐行照抄**

它的核心目标不是“帮你抄代码”，而是让 Claude 像一个稳妥的资深工程师一样，先收边界、控风险、定合同、做验证，再逐步落地。

## 怎么用

大多数情况下，你**不需要手动执行 skill 里的每一步**。你只要正常给 Claude 一个迁移任务，这个 skill 就会在下面这些词出现时自动触发：

- `迁移`
- `migrate`
- `port`
- `对齐`
- `搬运`
- `rewrite 到新模块`

例如：

- `把 liveme 里的 jzlOneVsMany 逻辑迁到 classme`
- `把 teacherwidgets 这块从老模块搬到新模块`
- `对齐 liveme 和 classme 的 teacherfeature 行为`
- `把这个老接口 rewrite 到 classme 现有链路里`

如果你想**显式强制** Claude 使用这套流程，可以直接这样说：

- `用 migration-autopilot 来做这个迁移`
- `按 migration-autopilot workflow 执行`
- `先走 Gate 1/2，不要直接改`

## 它会怎么工作

这份 skill 不会直接上来改代码，而是会按一套带门禁的流程推进：

### Gate 1：能力合同（Contract）
先定义：
- 输入是什么
- 输出是什么
- 有哪些副作用
- 错误语义是什么
- 哪些业务规则必须保留
- 哪些只是旧工程实现细节，不应该照搬

### Gate 2：差异矩阵（Difference Matrix）
对 source 和 target 的行为差异做显式比较，例如：
- 错误传播方式
- 空值 / 默认值语义
- 时间 / 时区处理
- 日志 / trace 方式
- 鉴权 / 上下文传播
- 类型转换行为

### Gate 3：分层迁移（Layered Migration）
按顺序推进：
1. data contracts
2. pure business logic
3. adapter layer
4. infrastructure integration

它会尽量避免把 controller / service / data 一次性混在一起改。

### Gate 4：等价性证据（Equivalence Evidence）
优先要求这些验证方式：
- golden-case tests
- traffic diff
- invariant assertions

它不会接受：
- “看起来一样”
- “测试过了应该没问题”
- “我回头自己测”

作为迁移等价性的充分证据。

### Gate 5：可回滚切换（Reversible Cutover）
迁移完成后，要求挂在一个可回滚开关后面，而不是直接硬切。

## 推荐的提问方式

这份 skill 在你给出的任务里如果包含下面 3 个信息，效果最好：

1. **source**
2. **target**
3. **这次只迁哪一个 capability**

推荐写法：

```text
用 migration-autopilot，把 liveme/appvendor/teacherwidgets.go 里的 tiyuSummary 对齐到 classme/service/teacherwidgets.go。
本次只迁这个 widget 能力，不扩散到其他 teacherwidgets。
```

另一个例子：

```text
把 liveme 的 teacherfeature 逻辑迁到 classme，对齐返回语义，不要直接照抄工具层。
```

## 你应该期待 Claude 表现成什么样

当这份 skill 工作正常时，Claude 通常会：

- 明确说自己正在使用迁移 workflow
- 先识别这次迁移的 capability
- 在编码前先起一个最小合同
- 说出至少一个迁移风险
- 说明确认后下一步要做什么 Gate
- 要求最终带可回滚开关

也就是说，它应该更像一个在控风险的 senior，而不是一个直接照着老代码开抄的助手。

## 在 classme / liveme 场景下的额外要求

当 source 是 `liveme`、target 是 `classme` 时，这份 workflow 还会额外要求：

- 改 symbol 前先做 GitNexus impact analysis
- 尽量使用目标侧 idiom，例如：`zlog`、现有 `api/` 包、项目内 helper
- 在宣称语义对齐前做 traffic diff 或等价验证
- 上线前挂 classme 侧的可回滚 feature switch

## 适合什么时候用

它特别适合这些任务：

- 老模块实现很杂，担心历史包袱一起被搬过来
- 新旧模块框架不同，但业务看起来“差不多”
- 需要灰度 / 开关 / 回滚
- 不能只靠“看起来一样”判断完成
- 需要把迁移拆成 capability-by-capability 的小闭环

## 一句话总结

如果你希望 Claude 在迁移任务里像一个稳妥的资深工程师一样工作，就直接正常提迁移需求；如果你想更稳，就在需求里明确写：

```text
用 migration-autopilot 做这次迁移。
```
