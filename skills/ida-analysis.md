---
name: ida-analysis
description: Use when analyzing native libraries (.so/.dll) for game RE - decompile functions, find xrefs, map structs, rename symbols. Requires IDA Pro running with MCP plugin loaded.
---

# IDA Pro Analysis Skill

## Prerequisites
- IDA Pro 8.3+ with MCP plugin (Ctrl+Alt+M to start)
- Verify: `mcp__ida-pro-mcp__check_connection`

## Workflow

### 1. Find targets
```
list_strings_filter / list_globals_filter — keyword search
get_function_by_name — get address from name
list_imports / get_entry_points — find key exports
```

### 2. Decompile & trace
```
decompile_function — C pseudocode (MUCH better than assembly)
get_callees / get_callers — trace call chain
get_xrefs_to — find all references to address/field
```

### 3. Extract info
From C pseudocode, extract:
- Struct field offsets (this->field_XX)
- Function signatures (params, return type)
- State checks, coordinate systems, call chains

### 4. Apply in Frida
- Use discovered offsets for memory reads
- Use function addresses for hooks
- Use struct layouts for pointer chains

## Rules
- Let IDA finish auto-analysis before querying
- Rename functions/vars as you discover them
- Trace xrefs: string -> function -> caller -> caller
- Prefer exported globals over computed offsets
- Read struct from memory > hook function (more stable)
- When Frida hook fails/wrong result -> decompile in IDA first (see memory: feedback_ida_frida_combo.md)
- Online games: IDA shows client code but server may reject (see memory: feedback_server_side_validation.md)

## Integration
```
ida-analysis -> frida-memory-scan / pointer-chain-finder / frida-packet-reverse / game-bot-builder
```
