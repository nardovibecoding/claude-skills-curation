## Anti-Patterns (What NOT to do)

1. **Update only the primary file** — "I changed the config, done!" No. Many files reference it.
2. **Search only .py files** — Shell scripts, JSON configs, markdown docs, cron jobs all have references.
3. **Skip memory/docs** — Stale docs cause future sessions to make wrong assumptions.
4. **Trust your memory of the codebase** — ALWAYS grep. Files you forgot about will bite you.
5. **Update one-by-one without a list** — Make the full list FIRST, then update. Partial updates = partial breakage.
