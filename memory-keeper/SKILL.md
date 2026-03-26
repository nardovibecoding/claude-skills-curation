---
name: memorykeeper
description: |
  Memory maintenance, stale cleanup, session mining, pattern promotion.
  Triggers: "memory maintenance", "clean memory", "stale memory", "mine sessions", "promote patterns".
  NOT FOR: editing specific memory files (just edit directly), skill management (use skillcleaning).
  Produces: updated memory, stale entries removed, patterns promoted to rules.
---

# Memory Keeper — Extract, validate, promote, and evolve memory

Five jobs: mine expiring sessions for unsaved knowledge, detect stale memory, clean up, promote proven patterns to enforced rules, and extract reusable skills from recurring solutions.

## When to run
- On a regular schedule (e.g., daily or weekly)
- Manually via /memory-maintenance
- After any major debugging session or architecture change

---

## Part 1: Extract from expiring sessions

1. Find session files older than 7 days
2. Read existing memory index
3. For each session: extract head 100 + tail 300 lines, grep for text content
4. Look for: architecture decisions, new features, bug root causes, user corrections, config changes
5. Cross-check against existing memory — skip duplicates
6. Save genuinely new findings to memory files

## Part 2: Detect stale memory

1. Read all memory files in the project memory directory
2. For each project/reference memory, verify claims against current state:
   - Cron schedules: compare against actual crontab
   - File paths: do referenced files still exist?
   - Process names: do referenced services still run?
   - API endpoints: are referenced URLs still valid?
   - Feature descriptions: does the code still work that way?
   - Version numbers: are referenced library versions still current?
   - Dependencies: are referenced packages still installed?
3. For each file path reference in all memory files:
   ```bash
   grep -roE '[a-zA-Z0-9_/.-]+\.(ts|js|py|md|json|yaml|yml|sh)' "$MEMORY_DIR/"*.md | while read f; do
     [ ! -f "$f" ] && echo "STALE: $f"
   done
   ```
4. Flag mismatches: "memory says X but reality is Y"
5. Auto-fix simple ones (wrong schedule times, renamed files)
6. Report complex ones for user decision

## Part 3: Clean up

1. Delete scanned session files (>7 days)
2. Remove memory files that are completely obsolete (only if confirmed by user)
3. Consolidate topic files that overlap (merge into the broader file)
4. Git commit and push changes

---

## Part 4: Auto-promotion pipeline (memory to rules)

Identify patterns in memory that should graduate to enforced rules in CLAUDE.md or `.claude/rules/`. Auto-memory captures well but has no judgment about what is temporary vs permanent.

### Step 1: Score each memory entry

Rate every actionable entry on three dimensions (0-3 each):

| Dimension | 0 | 1 | 2 | 3 |
|-----------|---|---|---|---|
| **Durability** | One-time fix | Session-specific | Multi-week relevant | Permanent convention |
| **Impact** | Nice to know | Saves minor time | Prevents bugs | Prevents outages |
| **Scope** | Single file | Single project area | Whole project | Cross-project |

**Promotion threshold: total score >= 6**

### Step 2: Detect recurring patterns

An entry qualifies as "recurring" when ANY of these signals are present:
- Same concept appears in 2+ memory files
- User has corrected the agent about this same thing in multiple sessions
- The pattern matches a feedback file (already an explicit user preference)
- Entry contains imperative language ("always", "never", "must", "don't")
- Entry has survived 3+ memory maintenance cycles without being flagged stale

### Step 3: Choose promotion target

| Pattern type | Target | Example |
|---|---|---|
| Project-wide convention | CLAUDE.md | "Package manager: pnpm" |
| File-type specific rule | `.claude/rules/<topic>.md` | "Test files must use factories" |
| Personal preference | `~/.claude/CLAUDE.md` | "Use consistent UI styling" |
| Already a feedback file | `.claude/rules/` with path scope | Move from soft memory to hard rule |

### Step 4: Transform the entry

Distill from description to prescription:
- **Before** (memory note): "Sometimes the dev server doesn't pick up .env changes"
- **After** (enforced rule): "Restart dev server after any `.env` change"

Rules for transformation:
- Write as imperative ("Do X", "Never Y", "Always Z")
- Include specific commands or values, not abstract concepts
- Keep to one line when possible; two lines max
- If scoped to file types, add YAML frontmatter with paths

### Step 5: Execute promotion

1. Append the distilled rule to the target file
2. Remove the original entry from memory
3. If a feedback file was promoted, note that it's now an enforced rule
4. Verify CLAUDE.md stays under 200 lines; overflow to `.claude/rules/`
5. Report: what was promoted, where it lives, lines freed in memory

### Capacity monitoring

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Memory index lines | < 120 | 120-180 | > 180 |
| CLAUDE.md lines | < 150 | 150-200 | > 200 |
| Topic files count | 0-15 | 16-30 | > 30 |
| Stale entries | 0 | 1-3 | > 3 |
| Feedback files not yet promoted | 0 | 1-3 | > 3 |

Report capacity status on every run using this table.

---

## Part 5: Skill extraction from proven patterns

When a solution is recurring, non-obvious, and useful across projects, extract it into a standalone reusable skill under `~/.claude/skills/`.

### When to extract

Extract when ALL of these are true:
- The pattern has been used in 2+ projects or could be
- It required real debugging (not obvious from docs)
- It involves a multi-step solution or has edge cases
- It would save significant time if encountered again

### Extraction process

1. **Identify** the pattern from memory entries or recent debugging sessions
2. **Name** using lowercase-hyphen format, 2-4 words describing the problem
3. **Generate SKILL.md** with:
   - Description of the problem (including exact error messages when applicable)
   - Step-by-step solution with copy-pasteable commands
   - Edge cases and gotchas
   - No hardcoded paths, credentials, or project-specific values
4. **Validate** the skill:
   - Self-contained (readable without original context)
   - Portable (works in any project with the same stack)
   - Under 200 lines
   - All code examples are complete and runnable
5. **Write** to `~/.claude/skills/<skill-name>/SKILL.md`
6. **Remove** the source entries from memory to free space
7. **Report** what was extracted and where it lives

### What NOT to extract

- Project-specific configurations (those belong in CLAUDE.md or rules/)
- Temporary workarounds for bugs that will be fixed upstream
- Patterns that are well-documented in official docs
- Anything containing credentials or secrets

---

## Important

- Never delete memory without strong evidence it's wrong
- When in doubt, flag for review rather than auto-fix
- Always cross-check with code/cron/processes, not assumptions
- Promotion removes from memory but adds to rules — net knowledge is preserved
- Skills are project-independent; rules are project-specific
- Run capacity check (Part 4 table) on every maintenance cycle
- Check for contradictions between memory entries and existing CLAUDE.md rules
