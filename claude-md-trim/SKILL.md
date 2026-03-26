---
name: claudemd-trim
description: |
  Audit CLAUDE.md rules — classify as internalized/custom/historical/redundant, trim bloat, report savings.
  Triggers: "clean claude.md", "trim rules", "claudemd maintenance", "consolidate rules", "audit claude.md".
  NOT FOR: editing specific rules (just edit directly), memory maintenance (use memorykeeper).
  Produces: classification report with line/token savings, then applies trims on approval.
---

# CLAUDE.md Maintenance

Audit and consolidate CLAUDE.md to reduce context token burn.

## Process

### Phase 1: Audit
Read the target CLAUDE.md file. For each section (identified by ### or ## headers), classify:

| Classification | Definition | Action |
|---|---|---|
| INTERNALIZED | Claude already does this by default (safety, reasoning, coding best practices) | REMOVE |
| REINFORCED | Claude generally does this but has FAILED at it in this project | TRIM to 1-2 lines |
| CUSTOM | Project-specific rules Claude would never know without being told | KEEP |
| HISTORICAL | Specific past incidents, not actionable rules | MOVE to memory/ |
| REDUNDANT | Says the same thing as another rule | MERGE with the other |

### Phase 2: Report
Present a summary table:

```
| Action | Count | Lines Saved |
|--------|-------|-------------|
| REMOVE | X | Y |
| TRIM | X | Y |
| MOVE | X | Y |
| MERGE | X | Y |
| KEEP | X | 0 |
| TOTAL | | ~Z lines |
```

Show current line count, projected line count, and estimated token savings (~2.5 tokens per line).

### Phase 3: Apply (on user approval)
- REMOVE: delete the section
- TRIM: shorten to 1-2 lines keeping the action rule, cutting explanation/examples
- MOVE: create memory file with frontmatter, move content there
- MERGE: combine into the stronger of the two rules
- After all changes, report: "CLAUDE.md: X lines (was Y), saves ~Z tokens/message"

## Rules for classification

### INTERNALIZED indicators (safe to remove):
- Generic engineering advice ("verify your work", "don't guess", "be thorough")
- Built-in Claude safety behavior ("don't expose API keys", "don't inject SQL")
- Aspirational platitudes ("continuously improve", "learn from mistakes")
- Rules with NO evidence of failure in self_review.md

### REINFORCED indicators (trim, don't remove):
- Has a corresponding entry in self_review.md (Claude failed at this)
- Project-specific nuance that makes the generic rule non-obvious
- Keep the ACTION ("do X when Y"), cut the STORY ("last time Z happened because...")

### CUSTOM indicators (must keep):
- File paths, directory structure, tool names specific to the project
- Workflow patterns (agent spawning rules, hook patterns)
- Integration details (API endpoints, service configurations)
- Team conventions (commit style, deploy flow, architecture)

### HISTORICAL indicators (move to memory):
- Contains dates
- References specific incidents
- Starts with "Lesson:" or "Example:" followed by a story

### REDUNDANT indicators:
- Two sections that both say "verify before doing X" in different words
- A rule that's a subset of a more comprehensive rule
- A checklist item that's covered by a higher-level rule

## References
- Target file: the CLAUDE.md in the current project root
- Self-review: memory/self_review.md (if it exists)
- Historical lessons go to: memory/reference_historical_lessons.md
