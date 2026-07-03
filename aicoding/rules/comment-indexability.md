---
description: Make controller/service/router comments searchable and useful for codebase-memory
paths:
  - "controllers/http/**/*.go"
  - "service/**/*.go"
  - "router/**/*.go"
alwaysApply: false
---

# Comment Indexability

- **Controller, callback, and consumer entry functions MUST have a one-line leading comment**
  - Put the comment immediately above the function so codebase-memory captures it as `docstring`
  - State the business purpose, not the syntax
  - Prefer stable domain words: callback, publish, bind, retry, review, room, resource, signal, workflow

- **Leading comments for entry functions SHOULD answer one of these**
  - Who calls this entry
  - What business action it performs
  - What state or downstream side effects it triggers

- **Good leading comment examples**
  - `// 视频互动资源生成回调`
  - `// 流媒体转码完成后的回调入口，负责发布资源并回写状态`
  - `// 数字人工作流回调，处理资源状态和结果落库`

- **Router registration lines SHOULD keep trailing comments**
  - Use them to describe endpoint purpose in one short phrase
  - This text is searchable by `search_code`, even if it is not stored as function `docstring`
  - Prefer interface purpose over vague labels

- **Good route trailing comment examples**
  - `// 流媒体资源加工后回调接口`
  - `// 视频-转码回调`
  - `// 数字人剪辑回调`

- **Complex service branches SHOULD have short internal comments only when the branch is not obvious**
  - Write comments for idempotency, retry policy, fallback behavior, cross-service writeback, and state transitions
  - Explain why the branch exists or what invariant it protects

- **Good internal comment examples**
  - `// 幂等：重复回调直接忽略，避免重复发布`
  - `// 转码失败最多重试 3 次，超过后停止自动补偿`
  - `// 先更新原始关系状态，再绑定审核 room，避免读取到未发布资源`

- **DO NOT write narration comments**
  - Avoid comments like `// 参数校验`, `// 调用 service`, `// 返回成功`
  - These add noise and do not improve codebase-memory retrieval

- **Use searchable nouns in comments**
  - Prefer real business terms and service names: `playback-bus`, `interactme`, `roomdata`, `resource`, `businessId`, `segmentTimeMap`
  - When a branch writes state externally, mention the target system or state keyword

- **For callback and async entrypoints, ALWAYS mention the upstream source or trigger**
  - Examples: transcode callback, workflow callback, MQ consumer, internal callback
  - This makes later call-chain search more reliable
