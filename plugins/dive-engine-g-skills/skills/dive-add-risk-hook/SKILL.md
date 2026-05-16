---
name: dive-add-risk-hook
description: 当你需要在 dive-engine-g 仓库中新增或修改反作弊对接部分时使用此 skill。触发词：”对接反作弊”、”对接风控”、”修改/新增反作弊”。在设计方案与实施计划时自动触发。
---

# 新增 Risk Hook

当任务是在 `dive-engine-g` 中新增风控实现，或调整某个业务与 reward type 的风控流程时，使用此 skill。

## 目标

安全扩展反作弊链路：

1. 明确目标 `biz + rewardType`
2. 实现并注册 risk handler
3. 决定前置或后置风控
4. 按需增加异步 reward 或 recall 通知钩子
5. 校验作弊与非作弊行为

## 优先检查的文件

- `engine/risk/risk.go`
- `engine/risk/reward/*`
- `engine/engine.go`
- `engine/context/context.go`
- `models/service/thirdproxy/*`
- `models/service/message.go`
- `engine/reward/reward.go`

## 工作流

### 1. 确认 risk 契约

开始编码前，先确认以下信息：

- 目标 business 与 reward type
- 是复用已有反作弊实现，还是新增实现
- 反作弊所需参数与下游接口
- 接口行为可参考已有反作弊实现
- 参数来源：
  - 是否都可通过 process 传入
  - 是否需要从第三方接口补充特定参数；若需要，先确认接口协议
- 风控模式：请求时拦截、发奖后通知，还是两者都需要
- 成功、拦截与错误的语义

如果行为不清晰，先暂停并向用户确认。优先参考 `engine/risk/reward/` 下相近实现。

### 2. 实现 risk handler

新增或修改一个满足下述接口的实现：

```go
type Filter interface {
    Pass(ctx context.Context, process json.RawMessage, param []param.ComputeData, reward ...reward.Info) (bool, error)
}
```

如果下游还需要发奖后或撤回后的异步通知，再实现：

- `Reward(ctx, process, rewards...) error`
- `Recall(ctx, process, rewards...) error`

### 3. 注册 handler

使用下面方式注册：

```go
func init() {
    risk.Register("biz", "reward_type", yourImpl{})
}
```

注册值必须与任务配置中的 `Activity.Business` 与 `Activity.RewardType` 完全一致。

### 4. 决定前置或后置风控

默认情况下，risk 可在计算后执行。若业务要求在奖励计算完成前拦截，需要检查 `engine/engine.go`，确认该组合是否应加入前置风控表。

仅在存在明确业务需求或正确性需求时，才加入前置风控。

### 5. 检查相关钩子

确认风控实现是否还需要：

- `thirdproxy` 请求构造
- 基于 Redis 的风控结果缓存
- 发奖后的 reward 通知
- 撤回后的 recall 通知
- 自定义日志或监控 tag

若逻辑属于具体业务或具体 reward，不要直接堆到 `risk.go`，应放到对应实现文件中。

### 6. 补充测试

测试尽量贴近实现：

- risk 实现测试放在新文件旁边
- 仅在风控顺序本身是重点时补端到端测试

至少覆盖：

- pass 场景
- cheat 或 blocked 场景
- 下游错误场景
- 若支持则覆盖 reward 通知路径
- 若支持则覆盖 recall 通知路径

## 完成标准

仅当满足以下条件时，risk hook 改动才算完成：

- handler 已按正确的 `biz + rewardType` 注册
- pass、block、error 语义清晰
- 已检查前置或后置执行策略
- 已检查异步 reward 与 recall 钩子
- 测试覆盖新增行为
- 最终说明中明确写清 cheat 与 non-cheat 路径的验证方式

结束时要明确说明：按照 `dive-add-risk-hook` 技能完成执行。
