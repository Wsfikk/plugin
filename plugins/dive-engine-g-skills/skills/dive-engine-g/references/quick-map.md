# Quick Map

## 仓库主线

```text
HTTP/MQ 入口
  -> controller
  -> models/service/task.go
  -> engine/engine.go
  -> param / calculator / risk / reward / extra / filter
  -> models/service/message.go
  -> OS2 callback 或 MQ notice
```

## 先读顺序

1. `main.go`
2. `router/router.go`
3. `controller/api/task/os2_action.go`
4. `models/service/task.go`
5. `engine/engine.go`

## 扩展点

- calculator：`engine/calculator/`
- param supplier：`engine/param/`
- reward sender：`engine/reward/`
- risk 映射：`engine/risk/reward/init.go`
- extra：`engine/extra/extra_data.go`
- filter：`engine/filter/filter.go`

## 仓库文档

- 规则与协作：`../../../AGENTS.md`
- 架构：`../../../docs/ARCHITECTURE.md`
- 开发指南：`../../../docs/DEVELOPMENT_GUIDE.md`
- 系统设计：`../../../docs/SYSTEM_DESIGN.md`
- 技能导航：`../../README.md`
