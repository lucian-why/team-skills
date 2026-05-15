# Docker-Push

## Overview

本地构建 Docker 镜像 → 推送到镜像仓库 → SSH 到云端拉取并重启。

**Core principle:** 用 git commit hash 做 tag，每个版本可追溯、可回滚。

**Announce at start:** "I'm using the docker-push skill to build and deploy."

## When to Use

- pre-push 完成后，代码已推到 git，准备部署上线
- 修复了线上 bug，需要紧急部署
- 触发词："部署" / "上线" / "deploy" / "推镜像"

## Prerequisites

- 本地已登录镜像仓库（`docker login <registry>`）
- 云端服务器 SSH 免密登录已配置
- 项目根目录有 Dockerfile 和 docker-compose.yml

## The Process

### Step 0: Pre-flight Check

```bash
# 确认在 main 分支（或已合并到 main）
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)

# 确认工作区干净（没有未提交的改动）
git status --porcelain
```

**If not on main and not clean:**
```
⚠️ 当前在 feature/chat 分支，且有未提交的改动。
   建议先跑 pre-push 提 PR，合并到 main 后再部署。

   强制部署当前分支？输入 "deploy anyway" 确认。
```

**If on main or clean:** Continue.

### Step 1: Build

```bash
REGISTRY=${REGISTRY:-crpi-28o7hgig011x8444.cn-beijing.personal.cr.aliyuncs.com/agent-project/agent-project}
IMAGE_NAME=${IMAGE_NAME:-agent-project}
TAG=$(git rev-parse --short HEAD)
FULL_IMAGE="${REGISTRY}/${IMAGE_NAME}:${TAG}"

docker build -t "${FULL_IMAGE}" .
```

输出：
```
🔨 构建镜像...
   镜像: crpi-xxx/agent-project/agent-project:abc1234
   commit: abc1234 (fix: 修复登录页样式)
   耗时: 2m 30s
```

**If build fails:**
```
❌ 构建失败，检查 Dockerfile 和构建日志。
```
Stop.

### Step 2: Tag (latest + commit hash)

```bash
# 同时打 latest tag（方便默认拉取）
LATEST="${REGISTRY}/${IMAGE_NAME}:latest"
docker tag "${FULL_IMAGE}" "${LATEST}"
```

输出：
```
🏷️ 打标签:
   abc1234  ← 具体版本（可追溯）
   latest   ← 默认标签（方便拉取）
```

### Step 3: Push

```bash
docker push "${FULL_IMAGE}"
docker push "${LATEST}"
```

输出：
```
📤 推送到镜像仓库...
   ✅ abc1234 推送完成
   ✅ latest 推送完成
```

**If push fails (auth):**
```
❌ 推送失败，未登录镜像仓库。
   运行: docker login ${REGISTRY}
```
Stop.

### Step 4: Deploy to Server

```bash
SERVER=${SERVER:-180.76.227.159}
SSH_USER=${SSH_USER:-root}
COMPOSE_DIR=${COMPOSE_DIR:-/opt/agent-project}

ssh ${SSH_USER}@${SERVER} << DEPLOY
  cd ${COMPOSE_DIR}

  # 更新 docker-compose.yml 中的镜像 tag
  # （如果用 :latest 则不需要改，直接 pull）
  docker compose pull
  docker compose up -d

  # 确认容器状态
  docker compose ps
DEPLOY
```

输出：
```
🚀 部署到服务器 180.76.227.159...
   ✅ 镜像拉取完成
   ✅ 容器重启完成
   ✅ 状态: agent-project running (abc1234)
```

**If deploy fails:**
```
❌ 部署失败，SSH 到服务器检查:
   ssh root@180.76.227.159
   cd /opt/agent-project && docker compose logs
```

### Step 5: Verify

```
🌐 打开 http://180.76.227.159:8502 确认服务正常。
   当前版本: abc1234
```

## Configuration

环境变量覆盖默认值：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `REGISTRY` | `crpi-28o7hgig011x8444.cn-beijing.personal.cr.aliyuncs.com/agent-project/agent-project` | 镜像仓库地址 |
| `IMAGE_NAME` | `agent-project` | 镜像名称 |
| `SERVER` | `180.76.227.159` | 云端服务器 IP |
| `SSH_USER` | `root` | SSH 用户名 |
| `COMPOSE_DIR` | `/opt/agent-project` | 服务器上 docker-compose.yml 所在目录 |

## Rollback

如果新版本有问题，回滚到上一个版本：

```bash
# 查看镜像历史（在服务器上）
ssh root@180.76.227.159
docker images | grep agent-project

# 回滚到指定 tag
cd /opt/agent-project
# 修改 docker-compose.yml 中的 tag 为旧版本
# 或直接: IMAGE_TAG=旧hash docker compose up -d
docker compose pull
docker compose up -d
```

## Quick Reference

| 场景 | 操作 |
|------|------|
| 正常部署 | docker-push skill，全程自动 |
| 紧急修复 | 直接在 main 上改 → docker-push |
| 回滚 | SSH 到服务器 → 改 tag → docker compose up -d |
| 构建失败 | 检查 Dockerfile → 本地 docker build 测试 |
| 推送失败 | docker login 重新登录 |

## Common Mistakes

**推代码和推镜像搞混**
- **Problem:** 代码推了但镜像没推，服务器还是旧版本
- **Fix:** pre-push 管代码，docker-push 管镜像，两个都要跑

**不打 commit hash tag**
- **Problem:** 只用 latest，无法追溯版本，无法回滚
- **Fix:** 每次构建必须同时打 commit hash tag 和 latest tag

**构建前不确认分支**
- **Problem:** 在 feature 分支上构建部署，推了未完成的代码
- **Fix:** 确认在 main 分支，或明确知道在做什么

## Red Flags

**Never:**
- 只推 latest 不推 commit hash（无法回滚）
- 在未合并的 feature 分支上部署
- 跳过构建直接改服务器上的镜像 tag

**Always:**
- 用 commit hash 做主 tag，latest 做辅助 tag
- 部署后验证服务是否正常
- 保留至少最近 5 个版本的镜像

## Integration

**Called by:**
- **pre-push** — 可以在 pre-push 最后问"要不要部署？"然后调用 docker-push

**Pairs with:**
- **pre-push** — 先审代码推 git，再推镜像部署
- **sync-main** — 同步最新代码后，确认没问题再部署
