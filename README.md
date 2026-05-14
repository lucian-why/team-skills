# Team Skills

跨平台 Agent Skill 库，兼容 Claude Code、OpenAI Codex、OpenCode、OpenClaw。

## Skills

| Skill | 用途 |
|-------|------|
| **neat-freak** | 会话结束后知识库洁癖级审查与同步 — 项目文档、CLAUDE.md、Agent 记忆对齐 |
| **using-superpowers** | 会话启动时自动发现并加载相关 Skill 的元技能 |

## 安装

### Claude Code

```bash
# 克隆到本地
git clone https://github.com/lucian-why/team-skills.git ~/team-skills

# 用 symlink 安装到 Claude Code skills 目录
ln -sf ~/team-skills/neat-freak ~/.claude/skills/neat-freak
ln -sf ~/team-skills/using-superpowers ~/.claude/skills/using-superpowers
```

### OpenAI Codex

```bash
ln -sf ~/team-skills/neat-freak ~/.codex/skills/neat-freak
ln -sf ~/team-skills/using-superpowers ~/.codex/skills/using-superpowers
```

### OpenCode

```bash
# OpenCode 同时扫描 ~/.claude/skills/ 和 ~/.codex/skills/，任选其一即可
ln -sf ~/team-skills/neat-freak ~/.config/opencode/skills/neat-freak
ln -sf ~/team-skills/using-superpowers ~/.config/opencode/skills/using-superpowers
```

### OpenClaw

```bash
ln -sf ~/team-skills/neat-freak ~/.openclaw/skills/neat-freak
ln -sf ~/team-skills/using-superpowers ~/.openclaw/skills/using-superpowers
```

### 项目级共享

也可以直接把需要的 skill 目录复制到项目的 `.claude/skills/` 下，随 git 一起分发：

```bash
cp -r ~/team-skills/neat-freak your-project/.claude/skills/
```

## 更新

```bash
cd ~/team-skills && git pull
```

Symlink 方式安装的 skill 会自动获取最新版本。
