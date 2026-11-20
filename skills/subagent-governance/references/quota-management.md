# 子代理并发配额与治理规范 (Quota & Governance)

在 Antigravity 运行环境中，并发子代理的派发受到系统底层配额和资源监控组件（如 `host-governor`）的严格约束。

---

## 1. 基础配额约束

1. **并发上限**：
   - 默认情况下，每个底层模型分配的活跃并发子代理上限为 **3 个**。
   - 超过配额后继续调用 `invoke_subagent` 将触发排队、限流或直接被 `PreToolUse` Hook 拒绝。
2. **资源消耗感知**：
   - 每个子代理均维护独立的推理会话，高并发派发会成倍放大 API 速率限制（Rate Limit / TPM / RPM）与宿主机器内存压力。

---

## 2. 与 Host Governor 联动流程

当确实需要处理大规模合规任务且已获得授权时，应按序调度：

```javascript
// 1. 查询当前模型配额状态
const status = await call_mcp_tool("host-governor", "governor_status", {});

// 2. 按需申请槽位
await call_mcp_tool("host-governor", "governor_acquire_slot", { model: "inherit" });

// 3. 执行任务并在完成后立即释放槽位
await call_mcp_tool("host-governor", "governor_release_slot", { model: "inherit" });
```

---

## 3. 异常处理与孤儿回收

1. **超时与挂起处理**：
   - 若子代理未在合理时间内返回有效结果，主 Agent 应当通过 `manage_subagents` 的 `kill` 操作主动终止，严禁无休止等待。
2. **退出前清理**：
   - 会话结束或主任务汇报前，确保所有衍生的临时工作区或子会话均已妥善释放。
