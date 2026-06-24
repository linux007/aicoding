---
name: trace-full-service-chain
description: Use when the user gives a route, controller, service method, or business path and wants the complete end-to-end chain across route, controller, service, storage, downstream services, async workflows, callbacks, and final state writes rather than only local function calls.
license: MIT
compatibility: Code graph tools strongly recommended for route, callback, and downstream boundary tracing.
metadata:
  author: local
  version: "0.2"
---

这个 skill 用来做一件事：**从一个入口追出完整业务链路，并覆盖所有经过的服务**。

适用于：
- “分析这个接口的完整链路”
- “把这个流程涉及的所有服务都找出来”
- “不要只看函数调用，要覆盖所有服务”
- “给我一版 Mermaid / 时序图”

不适用于：
- 单函数解释
- 局部字段溯源
- 纯实现改动

## 核心规则

### 1. 先找入口，再找实现
- 用户给 URL/path：先找 `route -> controller -> service`
- 用户给函数：先反查 route / caller，再追下游
- 不要直接从 service 开始讲

### 2. 先用图谱，再读文件
优先用图谱工具找：
- 入口定义
- 调用链
- callback 入口
- HTTP 边界

再读：
- 路由文件
- controller
- 主 service
- `data/api/models`
- `conf/mount/api.yaml` 或同类配置文件

### 3. 不要停在函数名
完整链路至少要回答：
- 谁发起
- 谁编排
- 谁执行
- 谁 callback
- 谁写状态
- 谁入最终业务库
- 谁负责发布或关系更新

如果答案里只有函数名，没有服务名、接口 path、存储类型、状态流转和最终写回位置，说明分析还不完整。

### 4. 必须覆盖这些边界
每次都检查是否经过：
- 当前服务
- MySQL
- Redis
- 对象存储
- 下游 HTTP 服务
- MQ / workflow / callback 来源
- 最终成品落库服务

如果用户明确说“所有服务都要覆盖到”，不能省略 DB、Redis、对象存储。

### 5. 看到异步必须追完
如果入口只是提交任务，继续确认：
- taskId / workflowId / transData / businessId 怎么传
- callback 路由在哪
- callback 怎么处理
- 成功 / 失败怎么落状态
- 最终资源写回哪里

callback 没追完，不要结束。

## 固定输出
必须按下面顺序输出：

### 1. 入口定位
- route
- controller
- 主 service
- 关键请求结构（如有必要）

### 2. 服务级完整链路
按阶段写：
- 发起阶段
- 异步阶段
- callback 阶段
- 成品写回 / 发布阶段

优先写经过哪些服务，不要先堆函数名。

### 3. 所有涉及服务清单
单独列出：
- 当前服务
- 下游业务服务
- MySQL
- Redis
- 对象存储
- callback 来源服务
- 最终落库服务

### 4. Mermaid 时序图
至少包含：
- 调用方
- 当前服务
- MySQL / Redis
- 关键下游服务
- callback 来源
- 最终写回服务

有成功 / 失败 / reset / retry 时，优先用 `alt` / `opt`。

### 5. 一句话总结
格式尽量像：
`调用方 -> 编排服务 -> 生产服务 -> callback -> 入库服务 -> 发布/状态回写`

## 常见漏项
交付前自查：
- route 找了吗
- controller 找了吗
- 主 service 找了吗
- callback 路由找了吗
- 配置里的真实下游服务找了吗
- MySQL / Redis / 对象存储写了吗
- 接口 path 写了吗
- 最终状态和写回位置写了吗
- Mermaid 图画了吗

## 推荐收尾话术
> 以上是按服务边界整理后的完整链路。
> 如果需要，我可以继续细化成更详细的 Mermaid，或按接口名展开成逐跳调用清单。
