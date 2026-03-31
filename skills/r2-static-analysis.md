---
name: r2-static-analysis
description: Use when needing to analyze a game's native library (.so/.dll) to find functions, cross-references, strings, or understand code structure - complements Frida runtime hooks with static disassembly via r2pipe
---

# r2pipe Static Analysis

## Overview
Use radare2 via Python r2pipe to statically analyze game binaries (.so/.dll). Find functions, strings, xrefs, and understand code flow WITHOUT running the game. Complements Frida (runtime) with offline analysis.

## When to Use
- Found a memory address via Frida, need to understand surrounding code
- Need to find function that accesses a specific address/string
- Reverse-engineering a large .so (10MB+) to find entry points for Frida hooks
- Want to understand game logic before hooking

## Setup

```bash
# Install radare2
# Windows: download from https://github.com/radareorg/radare2/releases
# Or via chocolatey: choco install radare2

# Install r2pipe
pip install r2pipe
```

## Quick Reference

```python
import r2pipe

# Open binary
r2 = r2pipe.open("libCore.so")
r2.cmd("aaa")  # Full analysis (slow for large files, use "aa" for quick)

# List functions
r2.cmd("afl")             # All functions
r2.cmd("afl~send")        # Functions containing "send"

# Disassemble
r2.cmd("pd 20 @ 0x1234")  # 20 instructions at address
r2.cmd("pdf @ sym.func")  # Full function disassembly

# Strings
r2.cmd("iz")              # All strings in data section
r2.cmd("iz~password")     # Strings containing "password"
r2.cmd("izz")             # All strings (including code section)

# Cross-references
r2.cmd("axt 0x1234")      # Who references this address? (xrefs TO)
r2.cmd("axf 0x1234")      # What does this address reference? (xrefs FROM)

# Imports/Exports
r2.cmd("ii")              # Imports
r2.cmd("iE")              # Exports

# Sections
r2.cmd("iS")              # Sections (.text, .data, .bss, etc.)

# JSON output (for parsing in Python)
import json
funcs = json.loads(r2.cmd("aflj"))  # Function list as JSON
```

## Core Workflows

### Find function by string reference

```python
# Game shows "Login Success" - find the function that uses this string
r2.cmd("iz~Login")         # Find string address
# e.g., 0x123456 "Login Success"
r2.cmd("axt 0x123456")     # Find code that references this string
# e.g., sym.LoginHandler+0x48
r2.cmd("pdf @ sym.LoginHandler")  # Disassemble the function
```

### Find function that accesses a known offset

```python
# Frida found value at base+0x5AC offset in a struct
# Find what code reads/writes at +0x5AC
r2.cmd("'/x ac050000'")    # Search for 0x5AC as immediate value (LE)
# Or search for the instruction pattern
r2.cmd("'/a ldr r0, [r1, #0x5ac]'")  # ARM assembly search
```

### Map game architecture

```python
# Find all network-related functions
r2.cmd("afl~send")
r2.cmd("afl~recv")
r2.cmd("afl~socket")
r2.cmd("afl~connect")

# Find all Lua/script-related functions (Cocos2d-x games)
r2.cmd("afl~lua")
r2.cmd("afl~script")
r2.cmd("afl~tolua")

# Find game-specific functions
r2.cmd("iz~Quest")
r2.cmd("iz~Battle")
r2.cmd("iz~Player")
```

### Analyze function call graph

```python
# What does a function call?
r2.cmd("axf @ sym.GameLoop")  # Functions called by GameLoop

# Who calls a function?
r2.cmd("axt @ sym.SendPacket")  # All callers of SendPacket

# Visual call graph (text mode)
r2.cmd("agc @ sym.GameLoop")  # Call graph
```

## Combo: r2pipe + Frida

```
r2pipe (offline)                    Frida (runtime)
-----------------                   ----------------
Find function name/address    ->    Hook that function
Find string references        ->    Hook function that uses string
Find struct offsets            ->    Read struct fields at runtime
Map call graph                ->    Know which functions to trace
Identify encryption/encoding  ->    Hook to get plaintext

Workflow:
1. r2: afl~Send -> find CGameClient::SendMsg at 0x12345
2. Frida: Interceptor.attach(base.add(0x12345), ...)
3. r2: axt 0x12345 -> find all callers
4. Frida: hook callers to understand when SendMsg is triggered
```

## Performance Tips

| Binary size | Analysis command | Time |
|-------------|-----------------|------|
| < 5MB | `aaa` (full) | seconds |
| 5-20MB | `aa` (basic) | 30s-2min |
| 20MB+ | `aa` then targeted `af @ addr` | varies |

For large binaries (libCore.so 23MB), use `aa` first, then `af` on specific functions.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Run `aaa` on 20MB+ binary | Use `aa` then `af` on specific functions |
| Forget architecture | Set with `e asm.arch=arm; e asm.bits=32` |
| Search wrong endianness | ARM LE vs network BE - check both |
| Ignore symbols/exports | `iE` often reveals function names directly |
| Only search .text | Strings in .rodata, data in .data - use `izz` for all |
