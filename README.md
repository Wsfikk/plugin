# plugin
dive留存插件市场

## 快速安装教程

### 1. 让 Agent 自动帮助添加插件市场

将以下 prompt 发送给 Claude Code 或 Codex，Agent 会自动帮你配置插件市场：

```
请帮我添加增长留存团队的插件市场。

插件市场配置文件路径：
- Claude Code: ~/.claude/plugins/marketplaces.json
- Codex: ~/.codex/plugins/marketplaces.json

插件市场仓库地址：https://github.com/didi/plugin

配置格式示例：
{
  "marketplaces": [
    {
      "name": "didi-engagement",
      "source": "git@github.com:didi/plugin.git",
      "branch": "main"
    }
  ]
}
```

### 2. 手动添加插件市场

#### Claude Code

```bash
claude plugins add didi-engagement git@github.com:didi/plugin.git
```

或者手动编辑配置文件：

```bash
# 创建配置目录（如果不存在）
mkdir -p ~/.claude/plugins

# 添加插件市场配置
cat > ~/.claude/plugins/marketplaces.json << 'EOF'
{
  "marketplaces": [
    {
      "name": "didi-engagement",
      "source": "git@github.com:didi/plugin.git",
      "branch": "main"
    }
  ]
}
EOF
```

#### Codex

```bash
codex plugins add didi-engagement git@github.com:didi/plugin.git
```

或者手动编辑配置文件：

```bash
# 创建配置目录（如果不存在）
mkdir -p ~/.codex/plugins

# 添加插件市场配置
cat > ~/.codex/plugins/marketplaces.json << 'EOF'
{
  "marketplaces": [
    {
      "name": "didi-engagement",
      "source": "git@github.com:didi/plugin.git",
      "branch": "main"
    }
  ]
}
EOF
```

## 可用插件

- **dive-engine-g-skills**: dive-engine-g 开发/排障/CR 全套 skills

## 配置说明

插件市场配置文件位置：
- Claude Code: `~/.claude/plugins/marketplaces.json`
- Codex: `~/.codex/plugins/marketplaces.json`

配置文件格式：
```json
{
  "marketplaces": [
    {
      "name": "marketplace-name",
      "source": "git@github.com:org/repo.git",
      "branch": "main"
    }
  ]
}
```

## 常见问题

**Q: 如何验证插件市场是否安装成功？**

A: 在 Claude Code 或 Codex 中执行 `/plugins` 命令，查看是否列出 `didi-engagement` 插件市场。

**Q: 如何更新插件市场？**

A: 进入插件市场目录执行 `git pull`：
```bash
cd ~/.claude/plugins/didi-engagement && git pull
```

**Q: 如何删除插件市场？**

A: 删除配置文件中的对应条目，或执行：
```bash
claude plugins remove didi-engagement
```
