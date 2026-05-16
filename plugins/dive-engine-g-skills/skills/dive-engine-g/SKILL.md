---
name: dive-engine-g
description: Use when working in the dive-engine-g codebase or when the user explicitly mentions dive-engine-g. 触发词：开发需求、编写代码、修复bug、分析这段日志/代码.
---

# Dive Engine G

用于 `dive-engine-g` 仓库的通用开发、排障。

## 先读这些

1. `../../AGENTS.md`
2. `../README.md`
3. `references/quick-map.md`
4. `references/change-checklist.md`

如果任务是新增活动、奖励链路或反作弊对接，按需继续读取：

- `../dive-add-param-supplier/SKILL.md`
- `../dive-add-calculator/SKILL.md`
- `../dive-add-reward-type/SKILL.md`
- `../dive-add-risk-hook/SKILL.md`

如果任务是 review、提交前检查或回归排查，继续读取 `../dive-code-review/SKILL.md`。

如果任务是任务流异常定位，继续读取 `../dive-debug-task-flow/SKILL.md`。

如果任务涉及 i18n、CAC、防腐边界、时间时区或货币金额合规，继续读取 `../didi-code-jungui/SKILL.md`。

## Workflow

1. 先定位入口，再定位实现。
   - HTTP 从 `router/router.go` 和 `controller/api/` 看起
   - MQ 从 `controller/mq/` 或 `common/handler/` 看起
   - 大多数链路最后都会落到 `models/service/task.go`
2. 意图分析，找到任务属于哪一类：
   - 只改接口
   - 只改算奖
   - 只改风控
   - 只改发奖
   - 同时影响 MQ / callback / 存储
3. 读取同目录已有实现，参考现有命名、结构和注册方式
4. 判断是否需要加载功能性skill，是否需要组合
5. 计划确定后，使用superpower:writing-plans技能驱动文档生成，如果没有则自主归档
6. 将可执行方案拆分出尽量独立的几个子步骤，可以让agent分步执行或多个子agent并行执行。
7. 优先做静态验证：
   - 查注册点是否补齐
   - 查 `business + rewardType` 的散点集合是否补齐
   - 查 `models/service/message.go` 与 `config/ddmq.yaml` 是否同步
8. 改了 Go 文件时执行 `gofmt`。
9. 开启新的子agent加载dive-code-review技能对代码改动做CR审查，并给出CR审查报告
10. 除非用户明确要求，不运行 `go test`、`go build`、项目二进制、服务启动脚本或依赖内网环境的动作

## 何时加载更多上下文

- 要理解全链路：打开 `../../docs/SYSTEM_DESIGN.md`
- 要看日常接入与扩展：打开 `../../docs/DEVELOPMENT_GUIDE.md`
- 要看仓库级入口与规范：打开 `../../docs/ARCHITECTURE.md` 和 `../../AGENTS.md`

## Final Response Rule

结束时要明确说明：按照 `dive-engine-g` 技能约束，编译、单测和运行态验证被有意跳过，除非用户在本轮明确要求执行。
