---
name: dependency-tracker
description: |
  Find and update all references after renames, moves, config changes.
  Triggers: "check dependencies", "update references", "what references X", "sync dependencies", after any rename/move/restructure.
  NOT FOR: deploying changes (use your deploy process), code review (use review).
  Produces: all stale references found and updated across codebase.
---

# Dependency Tracker

## The Problem

Every change has downstream references scattered across many files. Missing even one creates silent breakage:
- Renamed module -> stale references in config, monitoring, admin tools, cron, docs
- Changed thread/channel ID -> messages routed to wrong destination
- Moved file -> imports break, cron paths break, startup scripts break
- Changed env var -> .env, shell profiles, systemd units, configs all need updating

## When to Use

**ALWAYS** after any of these actions:
- Rename/move/delete a file
- Change an identifier or display name
- Modify IDs or routing keys
- Change config values
- Update cron jobs
- Rename env variables
- Change model names or API endpoints
- Restructure directories

## The Dependency Map

### For ANY change, scan ALL of these locations:

#### Code Files
- Entry points and main application files
- Config files (config.py, settings.py, etc.)
- Admin/management scripts
- Monitoring and health check scripts
- Scheduled task scripts
- All business logic files

#### Config Files
- JSON/YAML/TOML configuration files
- `.env` and environment files
- Domain/routing configuration

#### Infrastructure
- Crontab entries (`crontab -l`)
- Systemd units
- Docker files
- CI/CD configuration

#### Documentation
- README and project docs
- Memory/knowledge base files
- Architecture docs
- Active plans/roadmaps

## Execution Process

### Step 1: Identify the Change
What was changed? Extract the OLD value and NEW value.

### Step 2: Full Grep Scan
Search for ALL occurrences of the old value across the entire project:

```bash
# Search code
grep -rn "OLD_VALUE" ~/project/ --include="*.py" --include="*.sh" --include="*.json" --include="*.md" --include="*.env"

# Search cron
crontab -l | grep "OLD_VALUE"

# Search systemd
grep -rn "OLD_VALUE" /etc/systemd/system/ 2>/dev/null
```

### Step 3: Categorize Results

Sort findings into:
| Category | Action |
|----------|--------|
| **Must update** | Direct references that will break if not changed |
| **Should update** | Display names, comments, docs — won't break but misleading |
| **Skip** | Git history, logs, cache files — read-only/ephemeral |

### Step 4: Propose Changes (DO NOT EDIT YET)
For each "must update" and "should update" reference, prepare the exact edit:
```
Proposed changes (N total):

MUST UPDATE:
1. config.py:45 — "old_name" -> "new_name"
2. monitor.py:89 — "old_name" -> "new_name"

SHOULD UPDATE:
3. docs/architecture.md:13 — "Old Name" -> "New Name"

NO ACTION:
4. git history — read-only
5. log files — ephemeral
```

### Step 5: User Confirmation
**STOP and show the proposed changes. Ask: "Approve all? Or select which to apply?"**

Options:
- "yes" / "approve" -> execute all proposed edits
- "skip 3,5" -> execute all except items 3 and 5
- "no" / "cancel" -> abort, no changes made

### Step 6: Execute + Verify
After approval:
- Apply all approved edits
- `python3 -c "import py_compile; py_compile.compile('file.py', doraise=True)"` for each changed .py
- Verify JSON validity for any changed .json files
- Check cron syntax if cron was modified
- If ANY verification fails -> revert that specific file and warn

### Step 7: Integration Test
After all edits pass syntax check, verify nothing broke:
- Import each changed .py module: `python3 -c "import module_name"`
- For config changes: verify config loads and has required fields
- For routing changes: send a test message to the new destination
- For cron changes: `crontab -l | grep new_value` to confirm cron updated
- For .env changes: `source .env && echo $NEW_VAR` to confirm env loads
- Report: "X/Y integration checks passed" or "FAILED: [which check]"
- If ANY integration test fails -> warn user before committing

### Step 8: Report
Output a change report:

```
## Dependency Update Report

### Change: [description]

### Updated (X files):
- file.py:123 — old -> new
- config.json — old -> new

### Verified:
- [x] Python syntax check passed
- [x] JSON valid
- [x] Cron syntax valid

### No action needed:
- git history (read-only)
- log files (ephemeral)
```

## Reference Files
- [references/dependency-chains.md](references/dependency-chains.md) — common cascade patterns per change type
- [references/anti-patterns.md](references/anti-patterns.md) — what NOT to do
