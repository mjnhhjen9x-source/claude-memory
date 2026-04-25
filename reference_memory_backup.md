---
name: Memory backup workflow
description: How to push memory dir to GitHub backup repo and restore on new machine
type: reference
originSessionId: 739a471f-c622-43c3-b869-927876ec61e8
---
# Memory Backup

## GitHub repo
https://github.com/mjnhhjen9x-source/claude-memory (PUBLIC)

## Backup (push) — single command
```bash
cd "C:/Users/ThanTai/.claude/projects/C--Users-ThanTai/memory" && git add -A && git commit -m "update" && git push
```

## When to backup
- After adding new project memory
- After major changes (multi-file edits, new techniques)
- User says "backup memory" or equivalent
- Before significant local change that might need rollback

## Restore on new machine
```bash
# 1. Clone to memory dir (account name may differ)
git clone https://github.com/mjnhhjen9x-source/claude-memory.git \
  "C:/Users/<account>/.claude/projects/C--Users-<account>/memory"

# 2. If memory dir already exists with files, clone to temp and selectively restore (skip stale entries)
```

## Repo state
- Memory dir is itself a git repo (`.git/` inside)
- Remote `origin` → mjnhhjen9x-source/claude-memory
- Branch `main`
- `.gitignore` excludes `_pre_restore_backup/` (local-only safety dir)
- Auth: cached in Windows Credential Manager (gh CLI installed at D:\CodeTools\TOOLS\gh\bin\)

## Companion repo
- frida-mcp source: https://github.com/mjnhhjen9x-source/frida-mcp (custom MCP server, installed locally at D:\Code Tools\TOOLS\frida-mcp\)
