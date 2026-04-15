# Bug Fix Response - Redis Connection Timeout

## 场景声明

**检测到 Bug 修复场景，使用 Flow B。**

关键词匹配：用户提到"报错"、"500 错误"、"Redis 连接超时"、"修复"。

---

## Flow B 执行计划

### Phase 1: 问题定位 (Systematic Debugging)

根据 `superpowers:systematic-debugging` skill，需要执行以下步骤：

#### [1] 收集信息

需要用户提供或需要排查的信息：
- 错误日志的完整堆栈（时间戳、错误消息、调用链）
- 复现步骤（是偶发还是有规律？高峰期还是随机？）
- 影响范围（哪些接口受影响？影响用户比例？）
- 当前 Redis 配置（连接池大小、超时设置、地址）

**初步分析（基于项目记忆）：**

根据 `data-access.md`，DigiCore 的 Redis 使用模式：
- 客户端初始化：`helpers/redis.go` 中的 `RedisClient`
- 连接超时可能原因：
  1. 连接池耗尽（并发请求超过连接池大小）
  2. 网络抖动或 Redis 服务端负载过高
  3. 连接未正确释放（连接泄漏）
  4. 超时配置不合理

#### [2] 假设原因（>=2个）

| 假设 | 可能性 | 验证方法 |
|------|--------|---------|
| **H1: 连接池配置过小** | 高 | 检查 `conf.RConf.Redis["cache"]` 配置，对比 QPS |
| **H2: Redis 服务端负载高** | 中 | 查看 Redis 监控（CPU、内存、连接数） |
| **H3: 连接泄漏** | 中 | 检查代码中是否有未关闭的连接（`Close()` 未调用） |
| **H4: 网络问题** | 低 | 检查容器/服务器网络配置、DNS 解析 |

#### [3] 验证假设 - 待执行

```bash
# 检查 Redis 配置
# 查看连接池大小、超时时间
```

需要查看的文件：
- `helpers/redis.go` - Redis 初始化配置
- `conf/mount/resource.yaml` - Redis 配置参数
- `conf/config.go` - 配置结构定义

#### [4] 确定根因 - 待定

---

### Phase 2: 记录问题 (OpenSpec) - 可选

**简化条件评估：**
- 影响范围 > 1 个文件（可能涉及配置、初始化、多个调用点）
- 需要团队评审（Redis 配置变更属于基础设施变更）
- 需要历史追溯（生产问题）

**结论：需要使用 OpenSpec 记录**

命令：
```
/opsx:propose fix-redis-connection-timeout
```

生成文件：
- `proposal.md` - 根因分析、影响评估
- `specs/` - 修复后预期行为（如：连接池耗尽时应返回明确错误码而非 500）
- `tasks.md` - 修复步骤

---

### Phase 3: TDD 修复 - 待执行

步骤：
1. 写复现测试（模拟高并发场景，必须 FAIL）
2. 实现修复（调整连接池配置、增加重试机制、改进错误处理）
3. 测试通过
4. 回归测试（确保不影响其他功能）

---

### Phase 4: 归档 - 待执行

```
/opsx:archive
```

Commit message 模板：
```
fix: Redis 连接超时导致用户登录 500 错误

Root Cause:
- 连接池配置过小（默认 10），高峰期 QPS 超过连接池容量
- 超时时间设置不合理（默认 3s），未区分连接超时和读写超时

Fix:
- 增加连接池大小至 50
- 添加连接保活机制
- 改进错误处理，返回 503 Service Unavailable 而非 500

Test:
- 添加高并发场景测试用例
- 压测验证：1000 QPS 无超时

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

---

## 下一步行动

1. **立即执行 Phase 1.3**：查看 Redis 相关配置和代码
   - `helpers/redis.go`
   - `conf/mount/resource.yaml`

2. **确认问题环境**：
   - Redis 版本和部署方式（单机/集群）
   - 当前 QPS 和连接数监控数据

3. **调用 systematic-debugging skill** 执行完整排查流程

---

## 相关项目记忆

根据 `data-access.md`：
- Redis 客户端：`helpers.RedisClient`
- 配置来源：`conf.RConf.Redis["cache"]`
- 缓存键前缀：`workflow_`、`eXclusive_Lock_`
- 锁 TTL：30 秒

**需要验证的点：**
- [ ] 连接池配置（PoolSize、MinIdleConns）
- [ ] 超时配置（DialTimeout、ReadTimeout、WriteTimeout）
- [ ] 连接保活（IdleTimeout、MaxConnAge）
- [ ] 错误处理是否完善
