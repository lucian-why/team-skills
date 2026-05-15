# Sync-Main

## Overview

每天开工前同步 main 分支：拉最新代码 → 报告变动 → 影响分析 → 重建环境 → 启动。只看不动，功能完成后再合并。

**团队架构参考:** https://lucian-why.github.io/team-skills/team-architecture.html

**Core principle:** 先看别人干了什么，再决定怎么开工。不改你的分支，不 rebase，不 merge。

**Announce at start:** "I'm using the sync-main skill to sync with main and start dev environment."

## When to Use

- 每天开工前（继续开发已有分支时）
- 隔了几天没写代码，想看看 main 有什么新东西
- 触发词："开工" / "同步" / "sync" / "看看 main 有什么更新"

## When NOT to Use

- 开新功能 → 用 start-feature（已经包含拉最新 main）
- 功能完成准备推代码 → 用 pre-push（包含 rebase/merge 和冲突处理）

## The Process

### Step 1: Fetch and Compare

```bash
git fetch origin main

BRANCH=$(git branch --show-current)
BEHIND=$(git rev-list HEAD..origin/main --count)
AHEAD=$(git rev-list origin/main..HEAD --count)
```

**If behind is 0:**
```
✅ 你的分支 feature/chat 已经是最新，领先 main 2 个 commit。
   直接开工：docker-compose up
```
Skip to Step 4.

**If behind > 0:** Continue to Step 2.

### Step 2: Generate Sync Report

```bash
# 获取 main 上的变动（自你分支分叉以来的 commit）
git log HEAD..origin/main --pretty=format:"%h|%an|%s" --no-merges
```

输出格式：

```
═══════════════════════════════════════
         sync-main 报告
═══════════════════════════════════════

📍 你的分支 feature/chat 落后 main 5 个 commit

📝 main 变动：
  abc1234 @张三 — feat: 新增学习路径推荐算法
    └ shared/ai/model.ts, features/learning-path/service/recommend.ts
  def5678 @李四 — fix: 登录页样式错位
    └ features/auth/components/LoginForm.tsx
  ghi9012 @王五 — chore: 升级 React 19
    └ package.json, package-lock.json, Dockerfile
```

### Step 3: Impact Analysis

检查变动文件，分类标记影响级别：

```bash
# 检查 shared/ 有没有变动
git diff --name-only HEAD..origin/main | grep "^shared/"

# 检查依赖文件有没有变动
git diff --name-only HEAD..origin/main | grep -E "package\.json|Dockerfile|docker-compose"
```

输出影响分析：

```
⚠️ 影响分析：
  🔴 shared/ai/ 有变动 → 你用到了 ai/ 的接口，注意兼容性
  🔴 shared/frontend/ui/ 有变动 → 通用组件更新了
  🔴 package.json 变了 → 需要 docker-compose build
  🟡 Dockerfile 变了 → 建议 docker-compose build --no-cache
  🟢 features/auth/ 改动 → 不影响你的 feature/chat
```

**判断逻辑：**
- 🔴 红色：shared/ 下有文件变动，或 package.json/Dockerfile 变动
- 🟡 黄色：Dockerfile 变动（建议 --no-cache）
- 🟢 绿色：只改了其他 feature 目录，不影响你

**注意：** 这里只是提醒你别人改了什么。你不需要现在合并或 rebase，功能完成后再用 pre-push 处理合并。

### Step 4: Rebuild and Start

```bash
# 用 main 最新代码重建 Docker 镜像并启动
docker-compose up --build
```

输出：
```
🚀 用 main 最新代码启动环境...
   前端: http://localhost:5173
   后端: http://localhost:3000

✅ 环境就绪，浏览器打开确认一下。
```

> **为什么每次都 build？** main 上可能有新提交，镜像需要更新。这一步保证你跑的是最新代码构建的环境，不是你本地旧镜像。

### Step 5: Verify

```
浏览器打开 http://localhost:5173，确认页面能正常加载。
如果有问题，检查 docker-compose logs 输出。
```

## Quick Reference

| 场景 | 执行 |
|------|------|
| 落后 0 commit | 跳过，直接 docker-compose up |
| 落后 + 无 shared/ 变动 | 看报告 → docker-compose up --build |
| 落后 + shared/ 变动 | 看报告 + 影响分析 → docker-compose up --build |
| 落后 + 依赖变动 | 看报告 → docker-compose up --build |
| 功能完成要合并 | 用 pre-push，不用 sync-main |

## Common Mistakes

**不检查就开工**
- **Problem:** 写了半天代码，不知道 main 上别人改了 shared/，后面合并才发现不兼容
- **Fix:** 每天开工先跑 sync-main

**在 sync-main 里做 rebase/merge**
- **Problem:** 功能没完成就合并 main，引入不必要的冲突和复杂度
- **Fix:** sync-main 只负责看报告，合并是 pre-push 的事

**忽略 🔴 影响警告**
- **Problem:** shared/ 改了你没注意，写了一天代码发现接口不兼容
- **Fix:** 看完影响分析再开工，提前知道哪些接口变了

## Red Flags

**Never:**
- 跳过 sync 直接写代码（不知道 main 变了什么）
- 功能没完成就 rebase/merge main（合并是 pre-push 的事）

**Always:**
- 开工前先 sync-main
- 看完影响分析再决定要不要改自己的代码
- shared/ 有变动时检查自己的 import 是否还兼容

## Integration

**Pairs with:**
- **start-feature** — 开新功能用 start-feature，继续开发用 sync-main
- **sync-shared** — 你改了 shared/ 用 sync-shared 通知别人，别人改了 shared/ 用 sync-main 知道
- **pre-push** — sync-main 负责看报告，pre-push 负责合并和推送
