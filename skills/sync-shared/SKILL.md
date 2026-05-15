# Sync Shared

## Overview

When changing shared/common code (database models, auth, API contracts), notify the team and check downstream impact.

**团队架构参考:** https://lucian-why.github.io/team-skills/team-architecture.html

**Core principle:** Change shared code → auto-generate changelog → check what's affected → notify.

**Announce at start:** "I'm using the sync-shared skill to sync shared code changes."

## When to Use

- After modifying files in `shared/`, `common/`, `lib/`, or any directory everyone depends on
- After changing API contracts, database models, or auth logic
- User says "改了公共代码" / "sync shared" / "通知大家"

## The Process

### Step 1: Detect What Changed

```bash
# Check if shared code was modified
git diff --name-only HEAD~1 | grep -E "^(shared|common|lib)/"
```

**If no shared files changed:**
```
✅ No shared code changes detected. No sync needed.
```
→ Done.

**If shared files changed:** → Continue.

### Step 2: Analyze Impact

```bash
# Find all files that import from the changed shared modules
git diff --name-only HEAD~1 | grep -E "^(shared|common|lib)/" | while read file; do
  module=$(basename "$file" | sed 's/\.[^.]*$//')
  echo "Files importing $module:"
  grep -rl "from.*$module\|import.*$module\|require.*$module" --include="*.ts" --include="*.js" --include="*.py" . 2>/dev/null | grep -v node_modules | grep -v ".git"
done
```

### Step 3: Generate Change Summary

Create a structured changelog of what changed in shared code:

```markdown
## 🔗 Shared Code Changes

### Changed Files
- `shared/database/models.ts` — Added `profile` field to User model
- `shared/auth/middleware.ts` — Updated JWT validation logic

### Impact
- `features/user-profile/` — Uses User model, needs to update types
- `features/chat/` — Uses auth middleware, verify behavior unchanged

### Action Required
- [ ] @member-a: Update user-profile types for new `profile` field
- [ ] @member-b: Verify chat auth still works after middleware change
```

### Step 4: Present to User

```
⚠️ 公共代码有变更，可能影响其他模块：

变更内容：
- shared/database/models.ts — User 模型新增 profile 字段
- shared/auth/middleware.ts — JWT 校验逻辑更新

影响范围：
- features/user-profile/ — 需要更新类型定义
- features/chat/ — 需要验证认证逻辑

建议操作：
1. 把上面的变更摘要发到团队群
2. 或者直接在 PR 描述里注明影响范围

Options:
1. Copy summary to clipboard
2. Add to current PR description
3. Skip notification
```

### Step 5: Save Notification Record

```bash
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)
mkdir -p .claude
echo "sync-shared|${BRANCH}|$(date -Iseconds)|${COMMIT}" >> .claude/.skill-checkpoints
```

## Quick Reference

| What | How |
|------|-----|
| Shared directories | `shared/`, `common/`, `lib/` (configurable) |
| Impact check | grep for imports of changed modules |
| Output | Change summary + affected files + action items |
| Notification | Copy to clipboard or add to PR description |

## Tips

- **Don't over-notify**: Only trigger when actual shared code changes, not docs
- **Be specific**: "User model新增profile字段" not "改了数据库"
- **Include action items**: Tell people exactly what they need to check
- **PR description is enough**: For small changes, noting it in the PR is sufficient
- **Group chat for big changes**: Breaking changes warrant a direct message
