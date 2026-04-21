---
name: reference_frida_mcp_backup
description: frida-mcp project backup on GitHub private repo
type: reference
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# frida-mcp Backup

## GitHub Repo
https://github.com/mjnhhjen9x-source/frida-mcp (PRIVATE)

## Local path
`D:\Code Tools\TOOLS\frida-mcp\`

## Backup command
```bash
cd "D:/Code Tools/TOOLS/frida-mcp"
git add -A && git commit -m "..." && git push
```

## Restore (new machine)
```bash
git clone https://github.com/mjnhhjen9x-source/frida-mcp.git "D:/Code Tools/TOOLS/frida-mcp"
cd "D:/Code Tools/TOOLS/frida-mcp"
pip install -e ".[dev]"
```

Sau restore cần:
1. Update `.mcp.json` entry trỏ đúng path
2. Tạo lại skill files nếu machine mới chưa có (xem `frida-mcp-setup` SKILL.md)

## When to push
- Sau khi thêm recipe mới vào `recipes/`
- Sau khi fix bug trong `frida_mcp/*.py`
- Sau khi bump version trong `pyproject.toml`

## Current version
v0.1.0 (2026-04-21) — first release, 14 tools + 11 recipes, 44 tests pass
