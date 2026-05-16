---
name: dive-debug-task-flow
description: 当你需要在 dive-engine-g 仓库中排查任务流问题时使用此 skill。触发词："任务状态异常"、"算奖失败"、"发奖失败"、"任务流排查"、"重试异常"。
---

# 排查任务流问题

当任务是在 `dive-engine-g` 中排查任务异常、算奖结果不对、发奖失败、任务状态不一致或重试异常时，使用此 skill。

## 目标

按阶段缩小问题范围：

1. 先判断问题发生在入口、计算、风控、发奖、持久化还是回调
2. 确认任务原始上下文是否完整
3. 确认任务状态写入了哪里
4. 判断是单次失败、可重试失败还是幂等命中
5. 给出最小修复或最小复现路径

## 优先检查的文件

- `controller/api/task/os2_action.go`
- `controller/api/task/task.go`
- `controller/mq/task.go`
- `models/service/task.go`
- `models/service/message.go`
- `models/service/fusion.go`
- `engine/engine.go`
- `engine/param/param.go`
- `engine/risk/risk.go`
- `engine/reward/reward.go`
- `dao/calc/reward_task.go`
- `dao/calc/send_reward_detail.go`

## 排障顺序

### 1. 先定位问题所在阶段

优先判断属于哪一类问题：

- 请求进不来
- 任务能创建但算奖失败
- 算奖成功但被风控拦截
- 算奖成功但没有进入发奖
- 发奖执行失败
- 发奖成功但状态没落库
- 状态正常但回调或通知失败
- 重试逻辑异常

不要一开始就通读全链路，先缩小到阶段。

### 2. 确认任务原始上下文

优先从以下位置确认原始任务：

- `FusionService.StoreTask` 是否有写入
- `FusionService.Task` 是否能读出任务
- `controller/api/task/task.go` 的 `/task/tool/info` 是否能返回原始任务和阶段状态

如果任务上下文本身就不完整，后面的排查价值很低。

### 3. 检查 compute 阶段

如果问题落在算奖阶段，按顺序检查：

1. `TaskService.Compute` 是否命中了历史成功状态
2. `param.Load` 是否加载失败
3. `risk.IsCheat` 是否拦截
4. `calculator.Compute` 是否返回错误
5. `UpdateComputeResult` 是否写入成功

重点看：

- `task.Data`
- `Activity.Calculator.Type`
- `Activity.Calculator.Params`
- `Activity.Calculator.Detail`
- `response.Param`
- `response.Detail`
- `response.Reward`

### 4. 检查 reward 阶段

如果问题落在发奖阶段，按顺序检查：

1. `TaskService.CreateRewardTask` 是否被调用
2. MQ 是否投递到 reward topic
3. `controller/mq/task.go` 是否成功从 Fusion 取回任务
4. `engine.Reward` 是否真的调用到对应 sender
5. `UpdateRewardResult` 是否成功
6. `NoticeActualReward` 和 `CallBackReward` 是否成功

重点看：

- `task.Reward`
- sender 注册是否生效
- reward result 的 `Code`、`Status`、`Error`
- 是否命中幂等短路

### 5. 检查状态存储分支

这个仓库的状态可能落在不同位置，排障时必须先分辨：

- 是否启用了 `fusion_only`
- 是否命中了 `onlyWriteNotRead`
- 读取时是从 Fusion 命中，还是回源 MySQL

如果不先搞清真实数据源，很容易误判“状态没写”。

### 6. 检查重试链路

如果表现为反复失败或没有重试，检查：

- `controller/mq/task.go` 中当前失败是否会进入重试
- `MaxRetryTimes` 是否已到上限
- `ReCompute` 或 `ReReward` 是否成功投递延迟消息
- 错误是否属于其实不该重试的场景

## 推荐排障输出

排查任务流问题时，最终输出尽量包含：

- 问题所处阶段
- 关键任务标识，例如 `taskID`、`userID`、`biz`、`rewardType`
- 是否命中历史状态或幂等
- 是否命中风控
- 失败发生在哪个函数
- 最小复现方式
- 如果修复，说明是数据问题、配置问题还是代码问题

## 验证

排障类任务通常优先用最小验证：

```bash
go test ./...
```

如果问题集中在单层，优先改用更窄的验证范围，例如：

```bash
go test ./engine/param ./engine/calculator
go test ./engine/reward ./engine/risk/...
```

如果仓库已有调试接口，优先复用 `/task/tool/info`、`/task/tool/redo` 和 OS2 compute 入口做最小复现。

## 完成标准

只有满足以下条件，任务流排查才算完成：

- 已明确问题发生阶段
- 已确认任务原始上下文是否完整
- 已说明真实状态数据源
- 已定位具体失败点或明确无法继续定位的阻塞条件
- 如有修复，已说明验证方式与结果

结束时要明确说明：按照 `dive-debug-task-flow` 技能完成执行。