---
name: dive-add-calculator
description: 当需要在 dive-engine-g 中新增或修改 calculator(计算器)、调整 DefaultPrams、变更计算器逻辑或参数，或修改奖励计算逻辑时使用此 skill。尤其适用于 calculator 改动同时伴随 `${}` 占位符、process 取值、默认参数注入或新增参数来源的场景。在设计方案与实施计划时自动触发。
---

# 新增 Calculator

当任务是在 `dive-engine-g` 中新增一种 calculator，或对现有 calculator 做较大改动时，使用此 skill。

## 联动触发规则

出现以下任一信号时，除了使用本 skill，还应同时使用 `dive-add-param-supplier`：

- 修改 `DefaultPrams()` 中的参数列表、`Type`、`Args` 或 `${}` 占位符
- calculator 新增了原本不存在的入参
- calculator 所需参数不能直接由 process 原样提供，需要做结构转换、字段映射或第三方取数
- 需要在 `engine/param/param.go` 注册新的 param type
- 需要新增或修改 `engine/param/*.go`

如果任务既改 `engine/calculator/`，又改 `engine/param/`，默认视为“calculator + param supplier”复合改动，不能只使用本 skill。

## 目标
- param 实现可参考 `dive-add-param-supplier` skill。
- 是否命中复合改动信号：
  - 若本次不仅改计算公式，还改了 `DefaultPrams()`、`${}` 占位符、参数名称、参数结构或下游取数链路，必须联动启用 `dive-add-param-supplier`。
- 参数注入方式：根据计算器特性判断使用默认 param 还是任务入参 param。

如果存在以下情况，不要仅停留在“补 supplier”这句话，直接切换到 `dive-add-param-supplier` 的完整流程：

- `Args` 需要改为 `${order}`、`${this}`、`"${...}"` 等占位符形式
- supplier 需要调用 `thirdproxy`
- 参数返回值类型需要重新设计
- 需要补充 supplier 的占位符替换测试或 thirdproxy 组装测试


## 优先检查的文件

- `engine/calculator/calculator.go`
- `engine/param/param.go`
- `engine/engine.go`
- `engine/context/context.go`
- `engine/extra/*`
- `engine/risk/*`
- `controller/api/task/os2_action.go`
- `models/service/task.go`

## 计算器设计风格

在计算器模块中，优先采用“自底向上”的设计方式：上层计算器应尽量复用或组合已有底层计算器（例如 grade 计算器）。

优先设计通用型计算器；仅在业务强绑定（如 hour-base 计算器）且无法抽象复用时，才设计业务专用计算器。

## 工作流

### 1. 确认 calculator 契约

开始编码前，先确认以下信息：

- 计算器功能边界：优先抽象为可复用公共能力；仅在强业务关联时新增业务专用计算器。
- 计算所需参数及其类型。
- 参数来源：
  - 若上游可提供，放入 process 入参。
  - 若无需额外处理，可直接使用 `${}` 占位符替换。
- 是否需要新增 param 读取链路：
  - 若上游参数需要加工，或上游无法提供且需从第三方获取，考虑扩展 param 方案。
  - param 实现可参考 `dive-add-param-supplier` skill。
- 参数注入方式：根据计算器特性判断使用默认 param 还是任务入参 param。
  - 每次计算都必需的参数，优先使用默认 param。
  - 仅特定场景需要的参数，优先使用任务入参传入。
- calculator 输入参数结构设计（可参考标准 Calculator 类）。
- reward 输出结构定义。

如果上述信息不完整，先暂停编码并向用户进一步确认。优先参考 `engine/calculator/` 下相近实现。

### 2. 实现 calculator

在 `engine/calculator/` 下新增文件，并实现：

- 配置解码结构体（如需要）
- `Compute(config json.RawMessage, input []param.ComputeData) (ComputeResult, error)`

当 calculator 需要将具名参数映射到强类型结构体时，优先使用 `toParam(...)`。

在 `init()` 中注册：

```go
func init() {
    Register("your_calculator_type", YourCalculator{})
}
```

如果 calculator 依赖隐式默认参数，实现：

```go
func (YourCalculator) DefaultPrams() []context.CalcParam
```

### 3. 补充或复用 param supplier

检查所有所需参数类型是否已存在于 `engine/param/param.go`：

- 已存在：直接复用
- 不存在：在 `engine/param/` 下新增 supplier
- 将新类型注册到 `ParamsMap`

参数命名必须与 calculator 配置中的名称完全一致。

### 4. 检查 engine 集成点

确认新增 calculator 是否还需要修改：

- `engine/risk/*` 或 `engine/risk/reward/*`：当前 biz 与 reward type 是否需要风控钩子
- `engine/engine.go`：是否需要加入前置风控顺序

不要默认修改这些位置，仅在业务流程明确需要时才修改。

### 5. 验证 API 与任务流兼容性

calculator 由任务配置选择，通常不需要修改 controller。

但仍需确认：

- `controller/api/task/os2_action.go` 可原样透传新的 calculator 类型与参数
- `models/service/task.go` 中 compute 流程可正确持久化并重新加载任务
- 任务从 Fusion 或 Redis 重载后，异步 compute 仍可正常运行

### 6. 补充测试

测试尽量贴近实现：

- calculator 单测放在 `engine/calculator/*_test.go`
- 参数解析较复杂时，在 `engine/param/*_test.go` 补 supplier 测试

至少覆盖：

- 合法配置与预期 reward 输出
- 缺失参数或非法参数
- 边界值
- 空结果或零结果场景

## 完成标准

仅当满足以下条件时，calculator 改动才算完成：

- calculator type 已完成注册
- 所有必需的 param type 均可正确解析
- 合法输入与边界输入下 compute 输出稳定
- 已检查 extra 与 risk 钩子
- 测试覆盖新增行为
- 最终说明中明确写清修改文件与验证方式

结束时要明确说明：按照 `dive-add-calculator` 技能完成执行。
