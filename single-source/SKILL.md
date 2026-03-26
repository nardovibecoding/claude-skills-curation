---
name: single-source
description: |
  Dev machine to server sync via GitHub — deploy, pull, push code/config/state.
  Triggers: "deploy", "push to server", "sync", "single source of truth".
  NOT FOR: git commits (just use git), CI/CD pipelines, Docker deployments.
  Produces: synced state between local dev machine and remote server via git.
user-invocable: true
---

# Single Source of Truth

All state flows through GitHub. No rsync. No scp. Git is the bus.

**Why this exists:** Every developer with a local machine and a remote server has the
"is this the latest code?" problem. This skill makes sync a one-command operation.

## Architecture

```
Local dev  ──push──>  GitHub  <──pull──  Remote server
Local dev  <──pull──  GitHub  ──push──>  Remote server (when server commits)
```

### What syncs (template)

| What | Repo | Local path | Remote path | Auto-sync |
|------|------|-----------|-------------|-----------|
| App code | `$PROJECT_REPO` | `~/$PROJECT_REPO/` | `~/$PROJECT_REPO/` | Server pulls every 1 min |
| Config | `$PROJECT_REPO` | synced via git | synced via git | With code |
| Skills/plugins | `$SKILLS_REPO` | `~/.claude/skills/` | `~/.claude/skills/` | Every 10 min |
| Secrets (.env) | **NEVER in git** | local only | local only | Manual only |

### What does NOT sync (stays local)

| What | Why |
|------|-----|
| `.env` | Secrets — never in git |
| `*.db` | Database files — platform-specific |
| `venv/` | Python virtual env — platform-specific |
| `node_modules/` | Dependencies — reinstall per machine |
| Browser profiles | Session data — machine-specific |

## Setup

### 1. Configure your environment

```bash
# Add to your .env or .zshrc:
export REMOTE_HOST="your-server-ip"
export REMOTE_USER="your-username"
export PROJECT_REPO="your-project"
export SERVICE_NAME="your-service"  # systemd service name
```

### 2. Set up server auto-pull

On your server, add a cron job:

```bash
# Pull latest code every minute
* * * * * cd ~/$PROJECT_REPO && git pull --ff-only origin main >> /tmp/git-pull.log 2>&1
```

### 3. Set up local sync (optional)

On your dev machine, set up periodic sync (launchd on macOS, cron on Linux):
```bash
# Sync script — runs every 10 minutes
cd ~/$PROJECT_REPO && git pull --ff-only origin main
cd ~/$SKILLS_REPO && git pull --ff-only origin main
```

## Commands

### Deploy local changes to server

```bash
# 1. Commit and push
cd ~/$PROJECT_REPO && git add -A && git commit -m "description" && git push origin main

# 2. Server pulls automatically within 1 min, or force immediate:
ssh $REMOTE_USER@$REMOTE_HOST "cd ~/$PROJECT_REPO && git pull --ff-only origin main"

# 3. Restart service
ssh $REMOTE_USER@$REMOTE_HOST "sudo systemctl restart $SERVICE_NAME"

# 4. Verify
ssh $REMOTE_USER@$REMOTE_HOST "sleep 3 && sudo systemctl status $SERVICE_NAME --no-pager | head -5"
```

### Pull server changes to local

```bash
cd ~/$PROJECT_REPO && git pull --ff-only origin main
```

### Switch to local development (stop server)

```bash
# 1. Stop server
ssh $REMOTE_USER@$REMOTE_HOST "sudo systemctl stop $SERVICE_NAME"

# 2. Pull latest
cd ~/$PROJECT_REPO && git pull --ff-only origin main

# 3. Start locally
cd ~/$PROJECT_REPO && ./start.sh

# 4. When done: push changes, restart server
cd ~/$PROJECT_REPO && git add -A && git commit -m "Local changes" && git push origin main
ssh $REMOTE_USER@$REMOTE_HOST "sudo systemctl start $SERVICE_NAME"
```

## Conflict resolution

```bash
# Check what diverged
git log --oneline HEAD..origin/main
git diff HEAD origin/main

# Usually: different files edited → safe to merge
git merge origin/main

# Same file edited on both sides → manual resolve
# Fix the conflict markers, then:
git add <file> && git commit -m "Resolve merge conflict"
```

## Common setups

### Mac + Cloud Server (DigitalOcean/Linode/Vultr)
- Local: macOS dev machine
- Remote: Linux server with systemd service
- Sync: cron (server) + launchd (Mac)

### Mac + AWS EC2
- Local: macOS dev machine
- Remote: EC2 instance
- Sync: cron (EC2) + launchd (Mac)
- Note: use `~/.ssh/config` for key management

### Laptop + Desktop (same network)
- Both machines pull from GitHub
- No SSH needed — both push/pull independently
- Use branches if working on different features simultaneously

### Local + Docker host
- Local: dev machine
- Remote: Docker host
- Sync: git pull + `docker-compose up -d --build`

## Safety rules

1. **NEVER use scp to deploy code** — it bypasses git and overwrites commits
2. **NEVER run the same service on both machines simultaneously** — causes conflicts
3. **Always pull before push** — `git pull --ff-only` first
4. **Secrets stay local** — `.env` files are NEVER in git
5. **One writer at a time** — if editing on server, don't edit locally (or use branches)
