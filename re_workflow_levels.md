---
name: RE Workflow Levels & Tool Setup
description: Game RE skill levels, tool combo, IDA 8.3 setup with FLIRT (347 sigs) + RTTI (built-in) + MCP
type: reference
---

## RE Skill Levels

**Level 1 — ADB + OpenCV** (screen bot)
- Tap toa do, nhan dien hinh anh

**Level 2 — Frida hook + Memory R/W**
- Doc/ghi memory, hook function, modify packet

**Level 3 — Frida + IDA combo** ← CURRENT LEVEL
- Decompile logic game, tim function, struct layout, state machine

**Level 4 — Full packet emulation** (bot khong can game client)
- Reverse toan bo protocol, viet bot Python thuan

**Level 5 — Server emulation** (private server)

## IDA Pro 8.3 Setup (Optimized 2026-03-30)

**Plugins:**
- MCP plugin: mcp_plugin.py (Ctrl+Alt+M) - Claude doc/query IDA
- FLIRT: 347 signatures (155 built-in + 192 FLIRTDB community)
- RTTI: rtti.dll built-in - auto identify C++ class names
- Keypatch: ignore "No module named keystone" error

**FLIRT auto-identifies:** malloc, free, sprintf, strncpy, rand, memmove...
**RTTI auto-identifies:** CGameObject, CCharacter, CAntiHack, CNewUI*...
**Ket qua:** ~3x faster analysis - skip library functions, focus game code

**IMPORTANT:**
- FLIRT + RTTI = passive, cai 1 lan tu dong chay mai
- Khong can ClassInformer plugin - IDA 8.3 rtti.dll du dung
- FLIRTDB sigs: github.com/Maktm/FLIRTDB → copy vao IDA/sig/pc/
- MCP plugin: chi giu 1 file mcp_plugin.py, xoa mcp-plugin.py trung

## Tool Combo

1. **IDA + MCP** → static analysis (decompile, strings, xrefs, FLIRT, RTTI)
2. **Frida** → runtime (hook, memory scan, packet capture)
3. Combo: scan tim value → IDA hieu logic → Frida hook/modify

## Phan Cong

- **User**: len kich ban, tim game, test ket qua
- **Claude**: scan memory, decompile (IDA), hook/modify (Frida), viet code
