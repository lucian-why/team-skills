# Team Skills

跨平台 Agent Skill 库，兼容 Claude Code 和 OpenAI Codex。

## Skills

| Skill | 用途 | 触发时机 |
|-------|------|---------|
| **using-superpowers** | 工作流总纲 — Skill 优先级、开发流程、团队协作原则 | 会话启动时自动加载 |
| **start-feature** | 开始新功能 — 建分支、建目录、拉最新代码、接口契约模板 | 开始新功能/修 bug |
| **sync-shared** | 公共代码变更 — 通知团队、生成变更说明、检查下游影响 | 改了 shared/ 代码 |
| **pre-push** | 推送前 — AI review + 人审架构 + PR 描述 + 推送 + 检查点 | 推送前（核心 skill） |
| **neat-freak** | 文档同步 — 更新 API 文档、同步 CLAUDE.md、清理过时文档 | 功能完成后 |

### 开发周期中的 Skill 触发点

```
start-feature → 写代码 → sync-shared（改公共代码时）→ pre-push → neat-freak
```

## 安装

### Claude Code

```bash
git clone https://github.com/lucian-why/team-skills.git ~/team-skills

# Symlink 方式（推荐，自动更新）
ln -sf ~/team-skills/using-superpowers ~/.claude/skills/using-superpowers
ln -sf ~/team-skills/start-feature ~/.claude/skills/start-feature
ln -sf ~/team-skills/sync-shared ~/.claude/skills/sync-shared
ln -sf ~/team-skills/pre-push ~/.claude/skills/pre-push
ln -sf ~/team-skills/neat-freak ~/.claude/skills/neat-freak
```

**重要：** Claude Code 用户需要在项目的 `CLAUDE.md` 顶部添加 `@AGENTS.md` 来导入跨 Agent 共享规则：

```markdown
# CLAUDE.md

@AGENTS.md

# 以下是 Claude Code 专属指令...
```

### OpenAI Codex

```bash
ln -sf ~/team-skills/using-superpowers ~/.codex/skills/using-superpowers
ln -sf ~/team-skills/start-feature ~/.codex/skills/start-feature
ln -sf ~/team-skills/sync-shared ~/.codex/skills/sync-shared
ln -sf ~/team-skills/pre-push ~/.codex/skills/pre-push
ln -sf ~/team-skills/neat-freak ~/.codex/skills/neat-freak
```

Codex 原生读取项目根目录的 `AGENTS.md`，无需额外配置。

### 项目级共享

把 skill 复制到项目的 `.claude/skills/` 下，随 git 一起分发：

```bash
cp -r ~/team-skills/* your-project/.claude/skills/
```

## 多 Agent 团队协同

当团队成员使用不同 AI Agent 时，保持规则一致：

### 方案：AGENTS.md 单一源 + @AGENTS.md 导入

```
AGENTS.md              ← 单一事实源（Linux Foundation 开放标准）
  ↑                        ↑
  │                        │
CLAUDE.md              Codex
@AGENTS.md 导入         原生读取 AGENTS.md
```

**Claude Code** — 在 `CLAUDE.md` 顶部加 `@AGENTS.md`，自动导入全部内容
**Codex** — 原生读取 `AGENTS.md`，零配置

### AGENTS.md 内容建议

```markdown
# AGENTS.md

## 项目概览
一句话说清楚项目是什么

## 模块分工
谁负责哪个 feature/目录

## 协作规范
分支命名、PR 流程、代码风格

## 红线
AI 绝对不能做的事（如直接推 main）

## 阅读顺序
先读什么、再读什么的指引
```

### 同步脚本

```bash
#!/bin/bash
# sync-skills.sh — 一键同步 skills 到所有 agent

SKILLS="using-superpowers start-feature sync-shared pre-push neat-freak"
TEAM_SKILLS=~/team-skills

for skill in $SKILLS; do
  # Claude Code
  ln -sf "$TEAM_SKILLS/$skill" ~/.claude/skills/$skill
  # Codex
  mkdir -p ~/.codex/skills
  ln -sf "$TEAM_SKILLS/$skill" ~/.codex/skills/$skill
done

echo "✅ Skills synced to Claude Code and Codex"
```

## 团队架构可视化

访问 [team-architecture.html](https://lucian-why.github.io/team-skills/team-architecture.html) 查看完整的团队协作架构图。

## 更新

```bash
cd ~/team-skills && git pull
```

Symlink 方式安装的 skill 会自动获取最新版本。

## 检查点机制

`pre-push` skill 内置检查点去重，避免重复运行：

- 检查点文件：`.claude/.skill-checkpoints`（gitignored）
- 格式：`skill名|分支名|时间戳|commit hash`
- 同分支 + 同 commit = 跳过，新 commit = 自动重跑
