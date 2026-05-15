# Sync-Main

## Overview

每天开工前同步 main 分支：拉最新代码 → 报告变动 → 影响分析 → rebase → 重建环境 → 启动。

**Core principle:** 先看别人干了什么，再决定怎么开工。

**Announce at start:** "I'm using the sync-main skill to sync with main and start dev environment."

## When to Use

- 每天开工前（继续开发已有分支时）
- 隔了几天没写代码，想看看 main 有什么新东西
- 触发词："开工" / "同步" / "sync" / "看看 main 有什么更新"

## When NOT to Use

- 开新功能 → 用 start-feature（已经包含拉最新 main）
- 刚提完 PR → 用 pre-push

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
Skip to Step 5.

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

### Step 4: Rebase

```bash
git rebase origin/main
```

**如果有冲突：**
```
⚠️ Rebase 有冲突！冲突文件：
  - shared/ai/model.ts

请手动解决冲突后：
  git add <resolved-files>
  git rebase --continue

或者放弃 rebase：
  git rebase --abort
```

**如果无冲突：**
```
✅ Rebase 成功，你的分支已基于最新 main。
```

### Step 5: Rebuild and Start

```bash
# 检测是否需要重建
CHANGED=$(git diff --name-only HEAD~$BEHIND..HEAD 2>/dev/null | grep -E "package\.json|Dockerfile" || true)

if [ -n "$CHANGED" ]; then
  echo "📦 依赖/构建配置有变动，执行 docker-compose up --build"
  docker-compose up --build
else
  echo "🚀 无依赖变动，直接启动"
  docker-compose up
fi
```

输出：
```
📦 检测到 package.json 变动，执行 docker-compose up --build
🚀 启动中...
   前端: http://localhost:5173
   后端: http://localhost:3000

✅ 环境就绪，浏览器打开确认一下。
```

### Step 6: Verify

```
浏览器打开 http://localhost:5173，确认页面能正常加载。
如果有问题，检查 docker-compose logs 输出。
```

## Quick Reference

| 场景 | 执行 |
|------|------|
| 落后 0 commit | 跳过，直接 docker-compose up |
| 落后 + 无 shared/ 变动 | rebase → docker-compose up |
| 落后 + shared/ 变动 | rebase → 检查兼容 → docker-compose up |
| 落后 + 依赖变动 | rebase → docker-compose up --build |
| 落后 + Dockerfile 变动 | rebase → docker-compose build --no-cache → up |
| rebase 冲突 | 手动解决 → git add → rebase --continue |

## Common Mistakes

**不检查就开工**
- **Problem:** 写了半天代码，发现基于旧 main，rebase 一堆冲突
- **Fix:** 每天开工先跑 sync-main

**每次都 build**
- **Problem:** 没改依赖也 docker-compose build，浪费时间
- **Fix:** 只在 package.json 或 Dockerfile 变了才 build

**冲突了就放弃**
- **Problem:** git rebase --abort 回到旧状态，继续写旧代码
- **Fix:** 先看冲突文件，通常只是 import 路径变了，改一下就好

## Red Flags

**Never:**
- 跳过 sync 直接写代码（基于旧 main 的代码迟早要重写）
- 冲突时直接 force push（会覆盖别人的工作）

**Always:**
- 开工前先 sync-main
- 看完影响分析再决定要不要改自己的代码
- shared/ 有变动时检查自己的 import 是否还兼容

## Integration

**Pairs with:**
- **start-feature** — 开新功能用 start-feature，继续开发用 sync-main
- **sync-shared** — 你改了 shared/ 用 sync-shared 通知别人，别人改了 shared/ 用 sync-main 知道
- **pre-push** — sync-main 拉最新，pre-push 推最新
