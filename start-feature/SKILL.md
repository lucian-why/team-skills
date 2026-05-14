# Start Feature

## Overview

Automate the setup of a new feature branch: create branch, scaffold directory, pull latest, generate API contract template.

**Core principle:** One command to go from "I need to start X" to "ready to code".

**Announce at start:** "I'm using the start-feature skill to set up a new feature."

## When to Use

- Starting a new feature or fix
- User says "开始新功能" / "新建分支" / "start feature"

## The Process

### Step 0: Check for Uncommitted Changes

```bash
git status --porcelain
```

**If uncommitted changes exist:**
```
⚠️ You have uncommitted changes:
  M src/services/auth.service.ts
  ?? src/temp.ts

Options:
1. Stash and continue (git stash)
2. Commit first, then start new feature
3. Abort
```

Don't proceed until working tree is clean.

### Step 1: Confirm Feature Details

Ask the user (2-3 questions max):

```
新功能信息：
1. 功能名称？（用于分支名和目录名）
2. 简单描述这个功能做什么？（用于接口契约模板）
```

**Naming rules:**
- Branch: `feature/<name>` or `fix/<name>`, kebab-case
- Directory: `features/<name>/`, same as branch name
- Example: `feature/user-profile` → `features/user-profile/`

### Step 2: Pull Latest Main

```bash
git checkout main 2>/dev/null || git checkout master 2>/dev/null
git pull origin main 2>/dev/null || git pull origin master 2>/dev/null
```

### Step 3: Create Branch

```bash
git checkout -b feature/<name>
```

### Step 4: Scaffold Feature Directory

Create the standard feature directory structure:

```
features/<name>/
├── components/     # UI components
├── api/            # API routes
├── service/        # Business logic
└── tests/          # Tests
```

```bash
mkdir -p features/<name>/{components,api,service,tests}
```

If the project uses a different structure, match the existing pattern.

### Step 5: Generate API Contract Template

Create a template file for the feature's API contract:

```markdown
# <Feature Name> API Contract

## Endpoints

### GET /api/<name>
- Description: <what it does>
- Request: <params>
- Response: <shape>

### POST /api/<name>
- Description: <what it does>
- Request Body: <shape>
- Response: <shape>

## Data Models

### <ModelName>
- field1: type — description
- field2: type — description

## Notes
- <any constraints, auth requirements, etc.>
```

Save as `features/<name>/CONTRACT.md`.

### Step 6: Confirm Setup

```
✅ Feature setup complete:

  Branch:   feature/user-profile
  Directory: features/user-profile/
  Contract:  features/user-profile/CONTRACT.md

  Next steps:
  1. Edit CONTRACT.md to define your API
  2. Start coding in features/user-profile/
  3. When done, say "推送" to run pre-push
```

## Quick Reference

| What | How |
|------|-----|
| Branch naming | `feature/<kebab-name>` or `fix/<kebab-name>` |
| Directory | `features/<name>/` with components/api/service/tests |
| Contract | `features/<name>/CONTRACT.md` — fill before coding |
| Pre-requisite | Clean working tree (stash or commit first) |

## Tips

- **Keep it fast**: This should take < 30 seconds
- **Match existing patterns**: If the project has a different directory structure, adapt
- **Contract is optional**: For tiny fixes, skip the contract template
- **One feature, one branch**: Don't mix unrelated changes
