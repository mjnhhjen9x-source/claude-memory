---
name: frida-packet-reverse
description: Use when needing to reverse-engineer a mobile game's network protocol - hooking send/recv to capture packets, identify command bytes, parse binary structures, build packet injectors
---

# Frida Packet Reverse Engineering

## Overview
Hook libc network functions to intercept game packets, identify protocol structure (magic, header, cmd, data), and build parsers/injectors.

## When to Use
- New game, unknown protocol
- Need to automate game actions via packets (move, attack, use item)
- Need to detect game events (turn, dialog, battle start)
- Building a bot that reacts to server messages

## Core Pattern

### Step 1: Hook all network functions

Games may alternate between `send/recv` and `sendto/recvfrom`. Hook ALL:

```javascript
var libc = Process.getModuleByName('libc.so');
var sendPtr = libc.getExportByName('send');
var recvPtr = libc.getExportByName('recv');
var recvfromPtr = libc.getExportByName('recvfrom');

function hexdump(buf, len) {
    var hex = [];
    for (var i = 0; i < Math.min(len, 64); i++) {
        hex.push(buf.add(i).readU8().toString(16).padStart(2, '0'));
    }
    return hex.join(' ');
}

// SEND hook
Interceptor.attach(sendPtr, {
    onEnter: function(args) {
        var fd = args[0].toInt32();
        var buf = args[1];
        var len = args[2].toInt32();
        if (len < 10 || len > 500) return;
        console.log('S[' + len + '] ' + hexdump(buf, len));
    }
});

// RECV hook (shared handler)
function onRecvData(fd, buf, got) {
    if (got < 10) return;
    console.log('R[' + got + '] ' + hexdump(buf, got));
}

Interceptor.attach(recvPtr, {
    onEnter: function(a) { this.fd = a[0].toInt32(); this.buf = a[1]; },
    onLeave: function(r) { onRecvData(this.fd, this.buf, r.toInt32()); }
});
Interceptor.attach(recvfromPtr, {
    onEnter: function(a) { this.fd = a[0].toInt32(); this.buf = a[1]; },
    onLeave: function(r) { onRecvData(this.fd, this.buf, r.toInt32()); }
});
```

### Step 2: Identify protocol structure

Correlate UI actions with packets. Tap button -> observe sent packet.

```
Typical binary protocol:
magic(2) + length(2) + checksum(2) + header(N) + cmd(1) + data(...)

Look for:
- Magic bytes: constant first 1-2 bytes (0x71, 0x71AB, etc.)
- Length field: matches packet size
- Cmd byte: changes per action type
- Data: varies per cmd
```

### Step 3: Parse with BE/LE readers

```javascript
function readBE32(ptr, off) {
    return ((ptr.add(off).readU8() << 24) >>> 0) +
           (ptr.add(off+1).readU8() << 16) +
           (ptr.add(off+2).readU8() << 8) +
           ptr.add(off+3).readU8();
}
function readBES32(ptr, off) {
    var v = readBE32(ptr, off);
    return v > 0x7FFFFFFF ? v - 0x100000000 : v;
}
function readLE32(ptr, off) { return ptr.add(off).readU32(); }

// Determine endianness: if known value (e.g., x=414) appears as
// 00 00 01 9E -> Big Endian
// 9E 01 00 00 -> Little Endian
```

### Step 4: Build cmd table

```javascript
var CMD_NAMES = {
    0x02: 'SHOOT', 0x09: 'TURN_ACK', 0x0c: 'TURN',
    0x29: 'WIND',  0x37: 'WALK',     0x60: 'PRE_SHOOT'
};

function onRecvData(fd, buf, got) {
    if (got < 21) return;
    try { if (buf.readU8() !== MAGIC) return; } catch(e) { return; }
    var cmd = buf.add(CMD_OFFSET).readU8();
    var name = CMD_NAMES[cmd] || ('0x' + cmd.toString(16));
    console.log('R ' + name + '[' + got + ']');
}
```

### Step 5: Modify or inject packets

```javascript
// MODIFY existing packet (safer - game handles state)
Interceptor.attach(sendPtr, {
    onEnter: function(args) {
        var buf = args[1], len = args[2].toInt32();
        var cmd = buf.add(CMD_OFFSET).readU8();
        if (cmd === 0x02) {  // SHOOT
            writeBE32(buf, 29, newForce);
            writeBE32(buf, 33, newAngle);
            recalcChecksum(buf, len);
        }
    }
});

// INJECT new packet (risky - server may reject if state wrong)
var nativeSend = new NativeFunction(sendPtr, 'int', ['int', 'pointer', 'int', 'int']);
function sendPacket(fd, bytes) {
    var buf = Memory.alloc(bytes.length);
    for (var i = 0; i < bytes.length; i++) buf.add(i).writeU8(bytes[i]);
    return nativeSend(fd, buf, bytes.length, 0);
}
```

## Checksum Patterns

| Type | Implementation |
|------|---------------|
| Sum of bytes | `sum = 0; for (i=offset; i<len; i++) sum += buf[i]; sum & 0xFFFF` |
| CRC16 | Lookup table based |
| XOR | `xor = 0; for (i=offset; i<len; i++) xor ^= buf[i]` |
| None | Some games don't checksum |

Always recalculate after modifying packet data.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Hook only send() | Hook send + sendto + recv + recvfrom |
| Inject packets directly | Modify existing packets instead - server validates state |
| Ignore concatenated packets | Multiple cmds in one recv buffer - parse by length field |
| Wrong endianness | Check both BE and LE against known values |
| Filter by exact size | Packets may be concatenated, check cmd byte not total size |
| Duplicate processing | send + sendto both fire for same packet - use counter or hook one |

## Reverse Engineering Flow

```
1. Passive capture (log all S/R hex)
2. Correlate (tap UI -> which packet?)
3. Identify header (magic, length, checksum, cmd offset)
4. Map cmds (repeat actions, note cmd values)
5. Parse data fields (compare known values with byte offsets)
6. Build parser (structured logging per cmd)
7. Test modify (change one field, observe result)
8. Automate (trigger actions via packet modify or inject)
```
