---
name: reference_github_backup
description: Memory backup on GitHub private repo - push when memory changes
type: reference
---

# Memory Backup

## GitHub Repo
https://github.com/mjnhhjen9x-source/claude-memory (PRIVATE)

## Backup Command
```bash
cd "C:\Users\Than Tai\.claude\projects\C--Users-Than-Tai\memory"
git add -A && git commit -m "backup" && git push
```

## Restore (new machine)
```bash
git clone https://github.com/mjnhhjen9x-source/claude-memory.git
# Copy files to C:\Users\Than Tai\.claude\projects\C--Users-Than-Tai\memory\
```

## When to backup
- After adding new project
- After major changes to memory files
- User says "backup memory"
