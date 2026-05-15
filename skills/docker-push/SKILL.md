# Docker-Push

## Overview

本地构建 Docker 镜像 → 推送到阿里云 ACR → SSH 到云端拉取并重启。

**Core principle:** 用 git commit hash 做 tag，每个版本可追溯、可回滚。

**Announce at start:** "I'm using the docker-push skill to build and deploy."

**团队架构参考:** https://lucian-why.github.io/team-skills/team-architecture.html

## When to Use

- pre-push 完成后，代码已推到 git，准备部署上线
- 修复了线上 bug，需要紧急部署
- 触发词："部署" / "上线" / "deploy" / "推镜像"

## Prerequisites

- 本地已登录阿里云 ACR：`docker login crpi-28o7hgig011x8444.cn-beijing.personal.cr.aliyuncs.com`
- 云端服务器 SSH 免密登录已配置
- 项目根目录有 Dockerfile 和 docker-compose.yml

## The Process

### Step 0: Pre-flight Check

```bash
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)
COMMIT_MSG=$(git log -1 --pretty=%s)
TAG=$(git rev-parse --short HEAD)

# 确认工作区干净
git status --porcelain
```

**If not on main and has uncommitted changes:**
```
⚠️ 当前在 feature/chat 分支，且有未提交的改动。
   建议先跑 pre-push 提 PR，合并到 main 后再部署。

   强制部署当前分支？输入 "deploy anyway" 确认。
```

**If on main or clean:** Continue.

### Step 1: Build

```bash
REGISTRY=crpi-28o7hgig011x8444.cn-beijing.personal.cr.aliyuncs.com/agent-project/agent-project
TAG=$(git rev-parse --short HEAD)

docker build -t ${REGISTRY}:${TAG} .
```

输出：
```
🔨 构建镜像...
   镜像: crpi-xxx/agent-project/agent-project:abc1234
   commit: abc1234 — fix: 修复登录页样式
```

**If build fails:**
```
❌ 构建失败，检查 Dockerfile 和构建日志。
   常见原因：依赖下载超时、前端 build 报错
```
Stop.

### Step 2: Tag (latest + commit hash)

```bash
# 同时打 latest tag（docker-compose.yml 默认拉 latest）
docker tag ${REGISTRY}:${TAG} ${REGISTRY}:latest
```

输出：
```
🏷️ 打标签:
   ${REGISTRY}:abc1234  ← 具体版本（可追溯、可回滚）
   ${REGISTRY}:latest   ← 默认标签（docker-compose.yml 用这个）
```

### Step 3: Push

```bash
docker push ${REGISTRY}:${TAG}
docker push ${REGISTRY}:latest
```

输出：
```
📤 推送到阿里云 ACR...
   ✅ abc1234 推送完成
   ✅ latest 推送完成
```

**If push fails (auth):**
```
❌ 推送失败，未登录阿里云 ACR。
   运行: docker login crpi-28o7hgig011x8444.cn-beijing.personal.cr.aliyuncs.com
```
Stop.

### Step 4: Deploy to Server

```bash
SERVER=180.76.227.159

ssh root@${SERVER} << 'DEPLOY'
  cd /opt/agent-project

  # 拉取最新镜像（docker-compose.yml 里是 :latest）
  docker compose pull
  docker compose up -d

  # 确认容器状态
  docker compose ps
DEPLOY
```

输出：
```
🚀 部署到 180.76.227.159...
   ✅ 镜像拉取完成
   ✅ 容器重启完成
   ✅ agent-project running (abc1234)
   🌐 http://180.76.227.159:8502
```

**If deploy fails:**
```
❌ 部署失败，SSH 到服务器检查:
   ssh root@180.76.227.159
   cd /opt/agent-project && docker compose logs --tail=50
```

### Step 5: Verify

```
🌐 打开 http://180.76.227.159:8502 确认服务正常。
   当前版本: abc1234
```

## Configuration

| 变量 | 值 | 说明 |
|------|-----|------|
| `REGISTRY` | `crpi-28o7hgig011x8444.cn-beijing.personal.cr.aliyuncs.com/agent-project/agent-project` | 阿里云 ACR 镜像仓库 |
| `SERVER` | `180.76.227.159` | 生产服务器 IP |
| `SSH_USER` | `root` | SSH 用户名 |
| `COMPOSE_DIR` | `/opt/agent-project` | 服务器上 docker-compose.yml 所在目录 |
| `IMAGE_PORT` | `8502` | 服务对外端口 |

## Rollback

新版本有问题，回滚到上一个版本：

```bash
# 查看镜像历史
ssh root@180.76.227.159
docker images crpi-28o7hgig011x8444.cn-beijing.personal.cr.aliyuncs.com/agent-project/agent-project

# 回滚：修改 docker-compose.yml 中的 tag
cd /opt/agent-project
# 把 image 行的 :latest 改为 :旧hash
# 例如: image: crpi-xxx/agent-project/agent-project:abc1234
docker compose up -d
```

## Quick Reference

| 场景 | 操作 |
|------|------|
| 正常部署 | 说"部署"，skill 自动完成全部流程 |
| 紧急修复 | 在 main 上改 → 说"部署" |
| 回滚 | SSH → 改 tag → docker compose up -d |
| 构建失败 | 检查 Dockerfile → 本地 `docker build` 测试 |
| 推送失败 | `docker login` 重新登录 ACR |

## Common Mistakes

**只推 latest 不推 commit hash**
- **Problem:** 无法追溯版本，无法回滚到具体 commit
- **Fix:** 每次构建必须同时打两个 tag

**构建前不确认分支**
- **Problem:** 在 feature 分支上部署了未完成的代码
- **Fix:** 确认在 main 分支，或明确知道在做什么

**docker-compose.yml 里的 image 和实际推的不一致**
- **Problem:** 服务器拉的镜像和本地构建的不是同一个
- **Fix:** docker-compose.yml 里的 image 必须指向 ACR 地址，tag 用 latest

## Red Flags

**Never:**
- 只推 latest 不推 commit hash（无法回滚）
- 在未合并的 feature 分支上部署
- 跳过构建直接改服务器上的镜像 tag

**Always:**
- 用 commit hash 做主 tag，latest 做辅助 tag
- 部署后打开 http://180.76.227.159:8502 验证
- 保留至少最近 5 个版本的镜像

## Integration

**Called by:**
- **pre-push** — 可以在 pre-push 最后问"要不要部署？"然后调用 docker-push

**Pairs with:**
- **pre-push** — 先审代码推 git，再推镜像部署
- **sync-main** — 同步最新代码后，确认没问题再部署
