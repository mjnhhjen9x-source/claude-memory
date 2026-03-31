---
name: pointer-chain-finder
description: Use when a memory address changes every restart and needs a stable pointer chain from a module base - reverse scanning pointers, building chains, verifying across game restarts
---

# Pointer Chain Finder

## Overview
Turn an unstable heap address into a stable chain: `module_base + offset -> ptr + offset -> ... -> value`. The chain survives game restarts because it starts from a static base in a loaded module (.so/.dll).

## When to Use
- Found address via memory scan but it changes on restart
- Need persistent read/write access to a game value
- Building automation that must survive game restarts

## Core Pattern

### Step 1: Confirm target address
Use frida-memory-scan skill to find and verify the address holds your target value.

```javascript
// Confirmed: 0x7fff82144418 holds playerY as Double
// This address is on HEAP - changes every restart
```

### Step 2: Scan reverse - find what points to this region

```javascript
function findPointersTo(targetAddr, tolerance) {
    // Pointers usually point to start of struct, not exact field
    // Search for values in range [targetAddr - tolerance, targetAddr]
    var target = targetAddr;
    var results = [];
    var ranges = Process.enumerateRanges('rw-');

    for (var i = 0; i < ranges.length; i++) {
        var r = ranges[i];
        if (r.size > 80 * 1024 * 1024 || r.size < 8) continue;
        try {
            // Read as pointer-sized values (4 or 8 bytes)
            var step = Process.pointerSize;
            for (var off = 0; off < r.size - step; off += step) {
                var val = r.base.add(off).readPointer();
                var diff = target.sub(val).toInt32();
                if (diff >= 0 && diff <= tolerance) {
                    results.push({
                        addr: r.base.add(off),
                        pointsTo: val,
                        offset: diff
                    });
                }
            }
        } catch(e) {}
    }
    return results;
}

// Usage: find pointers within 0x600 of our target
var ptrs = findPointersTo(ptr('0x7fff82144418'), 0x600);
// Result: addr=0xAAAA points to 0x7fff82144000, offset=0x418
// So: *0xAAAA + 0x418 = our value
```

### Step 3: Check if pointer is in a module

```javascript
function classifyAddr(addr) {
    var modules = Process.enumerateModules();
    for (var i = 0; i < modules.length; i++) {
        var m = modules[i];
        var mEnd = m.base.add(m.size);
        if (addr.compare(m.base) >= 0 && addr.compare(mEnd) < 0) {
            return {
                module: m.name,
                offset: addr.sub(m.base).toInt32(),
                type: 'static'  // STABLE!
            };
        }
    }
    return { type: 'heap' };  // Need more levels
}
```

### Step 4: Repeat until base is static

```
Level 0: value at 0x7fff82144418 (heap)
Level 1: ptr at 0x7fff90001000 -> +0x418 (heap) -> continue
Level 2: ptr at 0x7fff80002000 -> +0x100 (heap) -> continue
Level 3: ptr at libgame.so+0x5A3C00 -> +0x50 (STATIC!) -> DONE

Chain: libgame.so+0x5A3C00 -> +0x50 -> +0x100 -> +0x418 -> value
```

### Step 5: Verify across restart

```javascript
function readChain(moduleName, baseOffset, offsets) {
    var base = Process.getModuleByName(moduleName).base;
    var ptr = base.add(baseOffset).readPointer();

    for (var i = 0; i < offsets.length - 1; i++) {
        ptr = ptr.add(offsets[i]).readPointer();
    }
    // Last offset reads the value, not a pointer
    return ptr.add(offsets[offsets.length - 1]).readDouble();
}

// After restart:
var value = readChain('libgame.so', 0x5A3C00, [0x50, 0x100, 0x418]);
// If matches current game value -> chain is STABLE
```

## Chain Depth Guide

| Game engine | Typical depth | Notes |
|-------------|--------------|-------|
| Cocos2d-x | 3-5 levels | Globals -> Scene -> Layer -> Node -> value |
| Unity IL2CPP | 2-4 levels | Static fields shorter |
| Flash/AIR | 3-6 levels | ActionScript VM adds layers |
| Native C++ | 2-3 levels | Usually shorter |

## Optimization: Narrow search space

```javascript
// Don't scan ALL memory - focus on likely regions
function smartReverseScan(target, tolerance) {
    var results = [];
    var ranges = Process.enumerateRanges('rw-');

    // Prioritize: same region as target, then module data sections
    var targetRange = null;
    for (var i = 0; i < ranges.length; i++) {
        var r = ranges[i];
        if (target.compare(r.base) >= 0 &&
            target.compare(r.base.add(r.size)) < 0) {
            targetRange = r;
            break;
        }
    }

    // Scan target's own region first (struct self-pointers)
    if (targetRange) scanRange(targetRange, target, tolerance, results);

    // Then scan module data/bss sections
    var modules = Process.enumerateModules();
    for (var i = 0; i < modules.length; i++) {
        var m = modules[i];
        // Module's writable sections contain global/static pointers
        for (var j = 0; j < ranges.length; j++) {
            var r = ranges[j];
            if (r.base.compare(m.base) >= 0 &&
                r.base.compare(m.base.add(m.size)) < 0) {
                scanRange(r, target, tolerance, results);
            }
        }
    }
    return results;
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Tolerance too small | Struct can be 0x600+ bytes. Use tolerance 0x800-0x1000 |
| Skip module data sections | Static pointers live in .data/.bss of modules |
| Wrong pointer size | 32-bit game = 4 bytes, 64-bit = 8 bytes. Check `Process.pointerSize` |
| Chain works once, fails later | Verify across 3+ restarts. Some chains only work per-session |
| Only search exact match | Pointer may point to struct base, value is at +offset within struct |
| Too many levels | If >6 levels, probably wrong path. Backtrack and try different branch |

## Verification Checklist

1. Restart game
2. Read chain from module base
3. Compare with actual value (from packet or UI)
4. Repeat 3 times minimum
5. Test after map change / scene change (some chains are scene-specific)
