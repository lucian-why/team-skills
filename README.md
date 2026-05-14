# Team Skills

跨平台 Agent Skill 库，兼容 Claude Code 和 OpenAI Codex。

基于 [obra/superpowers](https://github.com/obra/superpowers) 框架，额外添加 4 个团队自定义 Skill。

## Skills

### 团队自定义（4 个）

| Skill | 用途 | 触发时机 |
|-------|------|---------|
| **start-feature** | 开始新功能 — 建分支、建目录、拉最新代码、接口契约模板 | 开始新功能/修 bug |
| **sync-shared** | 公共代码变更 — 通知团队、生成变更说明、检查下游影响 | 改了 shared/ 代码 |
| **pre-push** | 推送前 — AI review + 人审架构 + PR 描述 + 推送 + 检查点 | 推送前（核心 skill） |
| **neat-freak** | 文档同步 — 更新 API 文档、同步 CLAUDE.md、清理过时文档 | 功能完成后 |

### 来自 superpowers 框架（14 个）

| Skill | 用途 | 触发时机 |
|-------|------|---------|
| **using-superpowers** | 工作流总纲 — Skill 优先级、开发流程、团队协作原则 | 自动（每次会话） |
| **brainstorming** | 头脑风暴 — 把模糊想法变成完整设计和规格说明 | 手动 |
| **writing-plans** | 写实施计划 — 任务拆解、依赖关系、验收标准 | 手动 |
| **executing-plans** | 执行计划 — 逐任务执行，跟踪进度 | 手动 |
| **subagent-driven-development** | 子代理驱动 — 每个任务派发独立子代理，任务间双轮 review | 手动 |
| **dispatching-parallel-agents** | 并行派发 — 多个独立任务同时执行 | 手动 |
| **test-driven-development** | TDD — 先写测试看它失败，再写代码让它通过 | 手动 |
| **systematic-debugging** | 系统化调试 — 收集证据、形成假设、逐一验证 | 手动 |
| **requesting-code-review** | 请求审查 — 派发审查子代理 | 手动 |
| **receiving-code-review** | 接收审查 — 分类处理审查反馈 | 自动 |
| **verification-before-completion** | 完成前验证 — 测试/lint/类型检查全过 | 自动 |
| **finishing-a-development-branch** | 完成分支 — 合并/建 PR/保留/丢弃 | 手动 |
| **using-git-worktrees** | Git Worktree — 一个仓库同时开多个分支 | 手动 |
| **writing-skills** | 写新 Skill — 把最佳实践编码成可复用 Skill | 手动 |

### 开发周期中的 Skill 触发点

```
brainstorming → writing-plans → start-feature → 写代码（TDD / 子代理）
    → sync-shared（改公共代码时）→ pre-push → neat-freak
```

## 安装

### Claude Code

```bash
git clone https://github.com/lucian-why/team-skills.git ~/team-skills

# Symlink 方式（推荐，自动更新）
for skill in ~/team-skills/*/; do
  ln -sf "$skill" ~/.claude/skills/$(basename "$skill")
done
```

**重要：** Claude Code 用户需要在项目的 `CLAUDE.md` 顶部添加 `@AGENTS.md`：

```markdown
# CLAUDE.md

@AGENTS.md

# 以下是 Claude Code 专属指令...
```

### OpenAI Codex

```bash
for skill in ~/team-skills/*/; do
  mkdir -p ~/.codex/skills
  ln -sf "$skill" ~/.codex/skills/$(basename "$skill")
done
```

Codex 原生读取项目根目录的 `AGENTS.md`，无需额外配置。

## 多 Agent 协同

```
AGENTS.md              ← 单一事实源（Linux Foundation 开放标准）
  ↑                        ↑
  │                        │
CLAUDE.md              Codex
@AGENTS.md 导入         原生读取 AGENTS.md
```

## 团队架构可视化

访问 [team-architecture.html](https://lucian-why.github.io/team-skills/team-architecture.html) 查看完整的团队协作架构图。

## 更新

```bash
cd ~/team-skills && git pull
```

Symlink 方式安装的 skill 会自动获取最新版本。

## 致谢

Skill 框架来自 [obra/superpowers](https://github.com/obra/superpowers)。
