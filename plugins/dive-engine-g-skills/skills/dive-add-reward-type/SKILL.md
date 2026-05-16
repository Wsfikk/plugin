---
name: dive-add-reward-type
description: 当你需要在 dive-engine-g 仓库中新增或修改 reward type（奖励类型）时使用此 skill。触发词：”新增奖励类型”、”对接新下游”、”修改奖励类型”。在设计方案与实施计划时自动触发。
---

# 新增 Reward Type

当任务是在 `dive-engine-g` 中新增 reward type，或对现有 reward sender 做较大改动时，使用此 skill。

## 目标

安全扩展发奖链路：

1. 明确 reward payload 结构
2. 实现并注册 sender
3. 按需增加 recall 支持
4. 校验持久化、通知与风控钩子
5. 补充聚焦测试

## 优先检查的文件

- `engine/reward/reward.go`
- `engine/engine.go`
- `engine/filter/filter.go`
- `engine/risk/*`
- `engine/risk/reward/*`
- `engine/extra/*`
- `models/service/task.go`
- `models/service/message.go`
- `dao/calc/send_reward_detail.go`

## 工作流

### 1. 确认 reward 契约

开始编码前，先确认以下信息：

- reward type 名称与对应整数值
- `reward.Info.Data` 的预期结构
- 单个任务是否可能发放多个该类型奖励
- 新增奖励类型对应的下游发奖接口：
  - 根据接口入参/出参判断，参数由上游传入还是由算奖侧在 Apollo 维护
  - 明确幂等键及其构建方式
- 根据下游参数要求，判断是否需要特殊 `extra` 产出
  - 发奖阶段无法直接获取 process 参数，需要提前评估是否通过 extra 提取
- 奖励结果是否需要落库、预算通知等附加逻辑
- 成功与失败 code 的语义
- 该 reward 是否支持 recall

如果上述信息不完整，先暂停并向用户确认。优先参考 `engine/reward/` 下相近实现。

### 2. 实现 sender

在 `engine/reward/` 下新增文件，并实现：

- sender 结构体
- `send(ctx context.Context, info []Info) []Result`

在 `init()` 中注册：

```go
func init() {
    register(Type("your_type"), 99, yourSender{}, MockConfig{})
}
```

如果需要标准状态包装、mock 能力或记录更新，优先复用现有 `basic` helper 模式。

### 3. 按需增加 recall 支持

如果该 reward 可撤回，再实现：

```go
func (yourSender) recall(ctx context.Context, info InfoWithUUID) Result
```

如果下游系统不支持撤回，不要默认增加 recall。

### 4. 检查持久化与通知行为

确认新增 reward 是否还需要修改：

- 发奖明细持久化
- 重复发放保护
- 实际奖励通知 payload
- recall 通知 payload
- mock 行为

如果该 reward 不应写发奖记录，检查是否应加入 skip set。若需要特殊下游格式，检查 `models/service/message.go`。

### 5. 检查 filter 与 risk 钩子

确认是否还需要修改：

- `engine/filter/filter.go`：是否存在发奖前拦截逻辑
- `engine/risk/reward/*`：当前 biz 与 reward type 是否需要反作弊钩子
- `engine/extra/*`：发奖阶段是否依赖 compute 产出的额外字段

不要默认修改这些位置，仅在业务流程明确需要时才修改。

### 6. 补充测试

测试尽量贴近实现：

- reward sender 测试放在 `engine/reward/*_test.go`
- 仅在跨多层行为时补 service 层测试

至少覆盖：

- 发奖成功
- 下游失败
- 不支持或非法 payload
- 若存在则覆盖 no-retry 行为
- 若支持则覆盖 recall 路径

## 完成标准

仅当满足以下条件时，reward type 改动才算完成：

- reward type 已完成注册
- send 路径返回稳定的 `Result`
- recall 支持已明确处理，或已明确说明不支持
- 已检查持久化与下游通知行为
- 已检查 risk 与 filter 钩子
- 测试覆盖新增行为
- 最终说明中明确写清修改文件与验证方式

结束时要明确说明：按照 `dive-add-reward-type` 技能完成执行。
