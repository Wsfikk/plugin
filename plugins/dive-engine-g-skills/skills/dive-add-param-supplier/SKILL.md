---
name: dive-add-param-supplier
description: 当需要在 dive-engine-g 中新增或修改 param supplier、调整 `engine/param/*.go`、改动 `ParamsMap` 注册、修改 calculator 的参数来源或 `${}` 占位符、将 process 字段转换为计算器参数，或通过第三方接口获取计算器所需参数时使用此 skill。在设计方案与实施计划时自动触发。
---

# 新增 Param Supplier

当任务是在 `dive-engine-g` 中新增参数类型，或修改现有 param supplier 的取数行为时，使用此 skill。

## 联动触发规则

出现以下任一信号时，应优先判定本 skill 已命中；如果同时修改了 `engine/calculator/`，还应联动使用 `dive-add-calculator`：

- 新增或修改 `engine/param/*.go`
- 修改 `engine/param/param.go` 中的 `ParamsMap`
- calculator 的 `DefaultPrams()` 里新增、删除或替换 param `Type`
- `Args` 中使用 `${order}`、`${this}`、`"${...}"` 等占位符，且本次改动涉及占位符结构变化
- 需要把 process 中的对象、列表或嵌套字段转换为 supplier 入参
- 需要通过 `thirdproxy`、DAO 或其他下游接口补齐计算器参数
- 需要让 calculator 从“上游直传参数”切到“supplier 动态获取参数”

如果需求描述里出现“根据 process 组装参数”“从第三方接口获取计算器所需字段”“新增实际付款金额/排行榜/订单信息参数”这类表述，默认应该触发本 skill，而不是只当作 calculator 改动处理。

## 目标
- 是否需要从第三方接口取数；若需要，确认接口入参与出参
- 当前 calculator 是否也会同步改动：
    - 如果会修改 `engine/calculator/*.go` 中的 `DefaultPrams()`、参数名或 `toParam(...)` 对应结构，必须联动启用 `dive-add-calculator`

- calculator 配置中的参数名必须与 supplier 最终产出的 `ComputeData.Name` 一致
- 如果 `Args` 是对象、数组或嵌套 JSON，优先确认是否应直接使用 `${order}`、`${this}`、`"${@this}"` 等现有写法，而不是手工再包一层 JSON
- 若 supplier 的入参结构与占位符替换结果不匹配，先调整契约，再编码

- 占位符替换后的边界场景
- calculator 真实 `DefaultPrams()` 配置下的最小联动场景，避免只测 supplier 内部 helper 而漏掉 `${}` 替换问题

## 优先检查的文件

- `engine/param/param.go`
- `engine/param/*.go`
- `engine/context/context.go`
- `engine/calculator/*.go`
- `engine/engine.go`
- `models/service/thirdproxy/*`

## 工作流

### 1. 确认参数契约

开始编码前，先确认以下信息：

- 参数类型名，例如 `order_info`、`leader_board`
- 参数在 calculator 配置中的名称
- `args` 的 JSON 结构
- 返回值的 Go 类型
- 是否依赖任务上下文中的用户、活动、时间或 process 数据
- 参数处理逻辑
- 是否需要从第三方接口取数；若需要，确认接口入参与出参

如果上述信息不完整，暂停询问用户。
优先查看现有 supplier 实现及其对应 calculator 的使用方式。

### 2. 实现 supplier

在 `engine/param/` 下新增或修改实现，遵循：

```go
type GetComputeData func(ctx context.Context, args json.RawMessage) (ComputeData, error)
```

实现时应保证：

- `Body` 是 calculator 真正消费的结构
- `Resp` 尽量保留原始响应，便于排障
- 错误信息可定位到具体参数加载失败点

### 3. 注册到 ParamsMap

在 `engine/param/param.go` 的 `ParamsMap` 中注册新类型：

```go
"your_param_type": wrapper(exampleValue, YourSupplier),
```

其中 `exampleValue` 用于声明返回值类型，必须与真实返回类型一致。

### 4. 检查占位符与配置兼容性

确认该参数是否依赖 `args` 中的路径替换：

- `${...}` 会在 `param.ReplacePath` 中使用 process 数据替换
- calculator 配置中的参数名必须与 supplier 最终产出的 `ComputeData.Name` 一致

如果参数结构较复杂，优先在本地构造最小配置和 process 样例，验证替换结果。

### 5. 检查关联影响

确认新增参数类型是否还会影响：

- calculator 的 `toParam(...)` 结构映射
- `binding` 校验逻辑
- 下游 `thirdproxy` 的请求组装
- mock 或测试入口

不要默认修改这些位置，仅在存在真实依赖时才修改。

### 6. 补充测试

测试尽量贴近实现：

- supplier 单测放在 `engine/param/*_test.go`

至少覆盖：

- 合法 args 与正常返回
- args 缺失或非法
- 下游失败
- 占位符替换后的边界场景

## 完成标准

仅当满足以下条件时，param supplier 改动才算完成：

- 新参数类型已注册到 `ParamsMap`
- 返回值类型与 `wrapper` 示例一致
- calculator 可正确消费该参数
- 占位符替换逻辑已验证
- 测试覆盖正常与异常路径
- 最终说明中明确写清修改文件与验证方式

结束时要明确说明：按照 `dive-add-param-supplier` 技能完成执行。
