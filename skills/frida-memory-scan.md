---
name: frida-memory-scan
description: Use when needing to find a game value's memory address via Frida - scanning for coordinates, HP, gold, force, angle, or any runtime value in a mobile game process on Android emulator
---

# Frida Memory Scan

## Overview
Find where a game stores a known value in memory by scanning all writable regions, then narrow down to the real address by filtering when the value changes.

## When to Use
- You know a value (position, HP, gold) and need its memory address
- You need to read/write game state directly without packets
- Packet-based approach is too slow or unavailable (e.g., PVP)
- Starting point for pointer chain discovery

## Quick Reference

| Data type | Priority | Game engine |
|-----------|----------|-------------|
| Double (64-bit) | Try FIRST | Flash/AIR, Unity IL2CPP |
| Float (32-bit) | Second | Cocos2d-x, native C++ |
| Int32 LE | Third | Most native games |
| Int32 BE | Last | Network packet values cached in memory |

## Core Pattern

### Step 1: Get known value
Source the target value from packets, UI, or manual observation.

```javascript
// Example: get player position from packet
if (cmd === 0x09 && len === 35) {
    var px = readBE32(buf, 23);  // known value from packet
    scanFor('pX', px);
}
```

### Step 2: Scan all rw- memory

```javascript
function scanFor(label, value) {
    var results = [];
    var ranges = Process.enumerateRanges('rw-');

    // 1. Double (64-bit) - try first
    var dblBuf = Memory.alloc(8);
    dblBuf.writeDouble(value);
    var dblBytes = new Uint8Array(dblBuf.readByteArray(8));
    var pattern = bytesToHex(dblBytes);

    for (var i = 0; i < ranges.length; i++) {
        var r = ranges[i];
        if (r.size > 80 * 1024 * 1024 || r.size < 8) continue;
        try {
            Memory.scan(r.base, r.size, pattern, {
                onMatch: function(addr) {
                    results.push({ addr: addr, type: 'D64' });
                },
                onError: function() {},
                onComplete: function() {}
            });
        } catch(e) {}
    }
    // Repeat for F32, LE32, BE32...
    return results;
}
```

### Step 3: Filter when value changes

```javascript
function filterAddrs(label, newValue) {
    var old = foundAddrs[label];
    var kept = [];
    for (var i = 0; i < old.length; i++) {
        try {
            var current;
            if (old[i].type === 'D64') current = Math.round(old[i].addr.readDouble());
            else if (old[i].type === 'F32') current = Math.round(old[i].addr.readFloat());
            else current = old[i].addr.readS32();

            if (current === newValue) kept.push(old[i]);
        } catch(e) {}
    }
    foundAddrs[label] = kept;
    // Typically: 5000+ -> 50 -> 5 -> 1 after 3 changes
}
```

### Step 4: Probe neighbors
Once you find one value (e.g., Y), related values (X, Z) are often nearby:

```javascript
function probeAround(addr) {
    for (var off = -64; off <= 64; off += 8) {
        try {
            var val = addr.add(off).readDouble();
            if (Math.abs(val) < 10000 && val !== 0) {
                console.log('[' + off + '] = ' + Math.round(val));
            }
        } catch(e) {}
    }
}
```

### Step 5 (Advanced): Access watchpoint
Instead of waiting for value changes, monitor which code accesses the address:

```javascript
// MemoryAccessMonitor - confirms real address
MemoryAccessMonitor.enable([{
    base: candidateAddr, size: 8
}], {
    onAccess: function(details) {
        console.log(details.operation + ' from ' +
            details.from + ' (freq confirms real addr)');
    }
});
```

Real game addresses are accessed every frame (~60/sec). Coincidental matches are rarely accessed.

## Filter Optimization

| Technique | When to use |
|-----------|-------------|
| Changed/Unchanged scan | Value type unknown, just know "it changed" |
| Range scan | Float/double with rounding issues |
| Struct scan | Know multiple values are adjacent (x,y pair) |
| Access watchpoint | Value rarely changes (max HP, level) |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Only scan Int32 | Flash/AIR uses Double. Always try D64 first |
| Skip huge ranges | Keep limit reasonable (80MB) but don't skip all large ranges |
| Exact float compare | Round to integer before comparing |
| Scan once, give up | Need 2-3 filter rounds minimum |
| Ignore neighbors | Found Y? Check offset -8 and +8 for X and Z |
