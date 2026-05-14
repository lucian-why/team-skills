# Pre-Push

## Overview

One skill for everything before pushing: review code → generate PR description → confirm → push → save checkpoint.

**Core principle:** Check checkpoint → AI reviews code → human reviews architecture → generate description → push → save checkpoint.

**Announce at start:** "I'm using the pre-push skill to review and prepare PR."

## When to Use

- Before `git push` on a feature/fix branch
- When you finish a feature or fix and want to create a PR
- Triggers automatically if you say "推送" / "提 PR" / "push"

## Checkpoint System

**Purpose:** Avoid running the same skill twice on the same state.

**Checkpoint file:** `.claude/.skill-checkpoints` (gitignored)

**Format:** `skill_name|branch_name|timestamp|commit_hash`

### Step 0: Check Checkpoint

```bash
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)

if [ -f .claude/.skill-checkpoints ]; then
  grep "pre-push|${BRANCH}||" .claude/.skill-checkpoints | tail -1
fi
```

**If checkpoint exists and commit matches:**
```
✅ Pre-push already completed for branch 'feature/login' (commit abc1234)
   Created at: 2026-05-15 10:30

   Options:
   1. Skip (PR already exists)
   2. Re-run (new commits since last run)
   3. Edit existing PR description
```

- "Skip" → done
- "Re-run" → continue to Step 1
- "Edit" → jump to Step 4 with existing description

**If no checkpoint or commit changed:** → Continue to Step 1

---

## The Process

### Step 1: Gather Context

```bash
git branch --show-current
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
git diff <base>..HEAD
```

For large diffs (>500 lines), focus on `--stat` and `--name-status`, read key files directly.

### Step 2: AI Code Review

AI runs a thorough code review and generates a structured report.

**AI checks (代码细节):**
- Syntax errors, typos
- Edge cases, null handling
- Security vulnerabilities (SQL injection, XSS, hardcoded secrets)
- Code style consistency
- Hallucinated APIs (functions that don't exist)
- Test coverage
- Dependency sanity

**Output format:**
```
## 🔍 AI Code Review Report

### ✅ 通过
- 安全：无 SQL 注入、XSS 风险
- 边界：空值处理完整
- 风格：与项目一致

### ⚠️ 需要人确认
- `auth.service.ts:42` — 这里的重试逻辑，3 次够吗？会不会打爆下游？
- `api/users.ts:18` — 返回了全量用户数据，需要分页吗？

### ❌ 必须修复
- `config/database.ts:7` — 密码硬编码在代码里
```

**Rules:**
- ✅ 通过的只列一行总结，不展开
- ⚠️ 需要确认的，指出具体文件:行号 + 问题 + 建议
- ❌ 必须修复的，给出修复建议
- 不要啰嗦，每个问题最多两句话

### Step 3: Human Reviews Architecture

Present the review report to the human, then ask **only** about architecture:

```
AI 审完了代码细节，上面是报告。

现在需要你确认架构层面：
1. 这个功能的设计模式对不对？
2. 模块间耦合合理吗？
3. 有没有性能瓶颈风险？

（不懂代码没关系，你审的是"这样做对不对"，不是"写得对不对"）
```

**Rules:**
- Human doesn't need to read code line by line
- Focus on: design decisions, business logic, maintainability
- If human says "looks good" → proceed
- If human has concerns → address them before proceeding

### Step 4: Fill Information Gaps

Check if PR description needs more context:

| Missing Info | Ask |
|-------------|-----|
| **Motivation unclear** | "commit 里看不出为什么改这个，能说下背景吗？" |
| **No test evidence** | "diff 里没看到测试文件，这个是怎么验证的？" |
| **Breaking changes ambiguous** | "这个接口改了，会影响现有调用方吗？" |
| **Commit messages useless** | (全是 "fix" / "update" / "wip") "能简单说下这次做了什么吗？" |

Max 2-3 questions. If diff is self-explanatory, skip.

### Step 5: Generate PR Description

Template:

```markdown
## 变更概述
<1-2 sentences: what this PR does and why>

## 具体改动
- <change 1: what and where>
- <change 2: what and where>

## 影响范围
- <affected module/feature>

## 测试验证
- [ ] <how this was tested>

## 注意事项
- <breaking changes, migration steps, things reviewers should pay attention to>
```

**Rules:**
- Title: conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `perf:`, `test:`)
- Title: Chinese or English, under 72 chars
- Be specific: "修复登录页密码输入框的白屏问题" not "修复 bug"
- Omit 注意事项 if nothing notable
- Write "待补充" in 测试验证 if no tests were run

### Step 6: Present for Confirmation

```
---
**Title:** feat: 用户登录模块重构

## 变更概述
重构用户登录模块，将认证逻辑从 Controller 抽取到独立的 AuthService。

## 具体改动
- 新增 `src/services/auth.service.ts`，封装登录/注册/token 刷新逻辑
- 修改 `src/controllers/user.controller.ts`，调用 AuthService 替代直接数据库操作

## 影响范围
- 用户登录/注册功能
- Token 刷新机制

## 测试验证
- [x] AuthService 单元测试通过
- [x] 手动测试登录/注册流程正常
---

Options:
1. Accept and push + create PR
2. Edit description (tell me what to change)
3. I'll write it myself
```

### Step 7: Push and Create PR

```bash
git push -u origin $(git branch --show-current)
gh pr create --title "<title>" --body "$(cat <<'EOF'
<confirmed PR description>
EOF
)"
```

### Step 8: Save Checkpoint

```bash
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)
mkdir -p .claude
echo "pre-push|${BRANCH}|$(date -Iseconds)|${COMMIT}" >> .claude/.skill-checkpoints
```

```
✅ PR created: https://github.com/xxx/xxx/pull/123
📌 Checkpoint saved — same branch + same commit won't re-run. New commits will auto re-run.
```

---

## Quick Reference

| What | How |
|------|-----|
| AI 审什么 | 语法、边界、安全、风格、幻觉 API、测试、依赖 |
| 人审什么 | 设计模式、业务逻辑、耦合度、可维护性 |
| Title format | `type: 简短描述` (conventional commits) |
| Checkpoint | `.claude/.skill-checkpoints` — 分支+commit，新 commit 自动重跑 |
| 推送 | AI 在 feature 分支推代码 + 创建 PR |

## Tips

- **Review 报告要精炼**: ✅ 一行总结，⚠️ 和 ❌ 指出文件:行号
- **人不需要逐行看代码**: 审架构决策，不审语法细节
- **Checkpoint = 幂等**: 同分支+同 commit = 跳过，新 commit = 重跑
- **Branch name helps**: `fix/login-white-screen` → fix, `feat/user-auth` → feat

## Common Mistakes

**AI review 太啰嗦**
- Bad: 每个文件写三段分析
- Good: ✅ 通过列一行，⚠️ 和 ❌ 直接指出问题

**人去逐行看代码**
- Bad: 人花 30 分钟看每一行
- Good: 人只看 AI 报告里的 ⚠️ 部分，确认架构决策

**PR 描述太笼统**
- Bad: "修改了一些文件"
- Good: "重构 AuthService，将 token 刷新逻辑从 Controller 抽取为独立方法"
