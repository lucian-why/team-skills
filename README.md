# Team Skills

4 个团队自定义 Skill，覆盖功能开发全流程。兼容 Claude Code 和 Codex。

推荐搭配 [obra/superpowers](https://github.com/obra/superpowers) 使用 — superpowers 提供 14 个基础 Skill（TDD、调试、代码审查等），team-skills 在此基础上叠加团队协作层。

## 安装

> **两个独立安装，互不依赖，随便先装哪个。**

### 1. superpowers（推荐，14 个基础 Skill）

```bash
# Claude Code
/plugin marketplace add obra/superpowers
/plugin install superpowers@superpowers

# Codex
/plugins → 搜索 superpowers → Install Plugin
```

详见 [obra/superpowers](https://github.com/obra/superpowers)

### 2. team-skills（4 个团队 Skill）

<details>
<summary><b>Claude Code</b></summary>

```bash
# 插件方式（推荐）
/plugin marketplace add lucian-why/team-skills
/plugin install team-skills@team-skills

# 或者手动 symlink
git clone https://github.com/lucian-why/team-skills.git ~/team-skills
for skill in ~/team-skills/skills/*/; do
  ln -sf "$skill" ~/.claude/skills/$(basename "$skill")
done
```

**重要：** Claude Code 用户需要在项目的 `CLAUDE.md` 顶部添加 `@AGENTS.md`：

```markdown
# CLAUDE.md

@AGENTS.md

# 以下是 Claude Code 专属指令...
```

</details>

<details>
<summary><b>Codex</b></summary>

```bash
# 插件方式（推荐）
/plugins
# 搜索 team-skills，选 Install Plugin

# 或者手动 symlink
git clone https://github.com/lucian-why/team-skills.git ~/team-skills
for skill in ~/team-skills/skills/*/; do
  mkdir -p ~/.codex/skills
  ln -sf "$skill" ~/.codex/skills/$(basename "$skill")
done
```

Codex 原生读取项目根目录的 `AGENTS.md`，无需额外配置。

</details>

## Skills

### 团队自定义（5 个）

| Skill | 用途 | 什么时候用 |
|-------|------|---------|
| **sync-main** | 同步 main — 拉最新代码、变动报告、影响分析、自动重建环境 | **每天开工前**，了解别人干了什么，确保基于最新代码开发 |
| **start-feature** | 开始新功能 — 建分支、建目录、拉最新代码、接口契约模板 | **每次开发新功能前**，确保工作区干净、分支正确 |
| **sync-shared** | 公共代码变更 — 通知团队、生成变更说明、检查下游影响 | **改了 shared/ 或公共模块后**，通知下游避免踩坑 |
| **pre-push** | 推送前 — AI review + 人审架构 + PR 描述 + 推送 + 检查点 | **每次推代码前必跑**，AI 审细节 + 人审架构，不能跳过 |
| **neat-freak** | 文档同步 — 更新 API 文档、同步 CLAUDE.md、清理过时文档 | **功能完成后**，确保文档跟代码同步，不留技术债 |

### 推荐搭配 superpowers（14 个）

superpowers 提供基础开发流程 Skill：brainstorming、writing-plans、TDD、systematic-debugging、code-review 等。

详见 [obra/superpowers](https://github.com/obra/superpowers)

### 开发周期中的 Skill 触发点

```
sync-main（每天开工前）→ brainstorming → writing-plans → start-feature → 写代码（TDD / 子代理）
    → sync-shared（改公共代码后）→ pre-push（推代码前必跑）→ neat-freak（功能完成后）
```

> brainstorming / writing-plans / TDD 等来自 superpowers，需独立安装。

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

插件安装的 skill 会自动获取最新版本。

## 致谢

基础 Skill 框架来自 [obra/superpowers](https://github.com/obra/superpowers)。
