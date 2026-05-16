# dive-engine-g-skills

把 `dive-engine-g` 仓库中的 8 个开发辅助 skills 打包成 Codex / Claude Code 插件。

## 包含的 Skills

| Skill | 用途 |
| --- | --- |
| `dive-engine-g` | 仓库通用开发/排障入口 |
| `dive-add-calculator` | 新增/修改 calculator |
| `dive-add-param-supplier` | 新增/修改 param supplier |
| `dive-add-reward-type` | 新增/修改奖励类型 |
| `dive-add-risk-hook` | 对接反作弊/风控 |
| `dive-code-review` | 提交前代码审查 |
| `dive-debug-task-flow` | 任务流排障 |
| `didi-code-jungui` | i18n、CAC、防腐、时间和货币相关代码军规 |

## Codex 安装

本插件包含 Codex manifest:

- `/Users/didi/test/dive-engine-g/plugins/dive-engine-g-skills/.codex-plugin/plugin.json`
- `/Users/didi/test/dive-engine-g/.agents/plugins/marketplace.json`

在 Codex 中使用仓库级 marketplace 后，插件入口为 `dive-engine-g-skills`。

## Claude Code 安装

把本仓库 clone 到本地后：

```bash
# 在 Claude Code 中执行
/plugin marketplace add /absolute/path/to/dive-engine-g/plugins/dive-engine-g-skills
/plugin install dive-engine-g-skills@dive-engine-g-skills
```

或者直接通过 git 远端安装:

```bash
/plugin marketplace add <git-url-to-this-repo>
/plugin install dive-engine-g-skills@dive-engine-g-skills
```

安装后 `/plugin` 列表里会出现 dive-engine-g-skills,触发词命中时 Claude 会自动加载对应 skill。

## 目录结构

```
plugins/dive-engine-g-skills/
├── .claude-plugin/
│   ├── plugin.json        # 插件元数据
│   └── marketplace.json   # 单插件市场清单
├── .codex-plugin/
│   └── plugin.json        # Codex 插件元数据
├── skills/
│   ├── dive-engine-g/
│   ├── dive-add-calculator/
│   ├── dive-add-param-supplier/
│   ├── dive-add-reward-type/
│   ├── dive-add-risk-hook/
│   ├── dive-code-review/
│   ├── dive-debug-task-flow/
│   └── didi-code-jungui/
└── README.md
```

## 维护说明

仓库内 `/Users/didi/test/dive-engine-g/skills/` 是“源”，本目录下的副本是发布版本。
源 skill 改动后需要同步:

```bash
for s in dive-engine-g dive-add-calculator dive-add-param-supplier didi-code-jungui \
         dive-add-reward-type dive-add-risk-hook dive-code-review dive-debug-task-flow; do
  rm -rf /Users/didi/test/dive-engine-g/plugins/dive-engine-g-skills/skills/$s
  cp -R /Users/didi/test/dive-engine-g/skills/$s /Users/didi/test/dive-engine-g/plugins/dive-engine-g-skills/skills/
done
```

并在 `.codex-plugin/plugin.json` 与 `.claude-plugin/plugin.json` 中升 version。
