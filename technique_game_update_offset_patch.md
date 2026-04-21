---
name: Game update - tìm offset mới bằng signature scan với wildcard imm
description: Game update làm dời offset, không phải uniform shift toàn bộ binary - các region khác nhau dời khác nhau
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Khi game update làm dời code, **KHÔNG giả định uniform shift** toàn bộ binary. Scan từng function/region riêng — các region code khác nhau có thể dời khác nhau.

**Why**: M4VN update v2→v3 có 4 shift khác nhau:
- Game main code: -0x4F0 (10 functions)
- GetCharBySlot helper: -0x550
- Anti-cheat (PhoenixCheat, SafeExit, DisconnectSplash): -0xE0
- 1 anti-cheat function dời xa: +0x14A30
- Data globals: -0x1000

Nếu assume uniform shift -0x4F0 và patch anti-cheat, sẽ patch nhầm vào data hoặc giữa function khác → crash.

**How to apply**:

1. **Scan game functions** (SHIFT thường uniform cho bulk code):
```python
def make_pattern(code_bytes):
    """Regex pattern, replace imm32 after push/mov/call/jmp with wildcards."""
    import re
    pattern = b''
    i = 0
    while i < len(code_bytes):
        b = code_bytes[i]
        if b == 0x68 or 0xB8 <= b <= 0xBF or b == 0xE8 or b == 0xE9:
            pattern += re.escape(bytes([b])) + b'....'  # 4 wildcards cho imm32
            i += 5
        elif b == 0xFF and i+1 < len(code_bytes) and code_bytes[i+1] in (0x15, 0x25):
            pattern += re.escape(code_bytes[i:i+2]) + b'....'
            i += 6
        else:
            pattern += re.escape(bytes([b]))
            i += 1
    return pattern

sig = v2_code[func_offset+32:func_offset+32+48]  # 48 bytes after prologue
pattern = make_pattern(sig)
matches = list(re.finditer(pattern, v3_binary, flags=re.DOTALL))
# Unique match = function found in V3
```

2. **Scan anti-cheat riêng** - thường recompile khác với game code.

3. **Scan globals qua instruction context**:
```python
# Find V2 instruction writing to global: C7 05 <addr> <value>
# Match V3 with prefix/suffix context + wildcard for addr
prefix = v2[ref-16:ref+2]  # up to C7 05
suffix = v2[ref+6:ref+6+16]  # value + bytes after
pattern = re.escape(prefix) + b'....' + re.escape(suffix)
# Unique match → extract new addr from 4 wildcard bytes
```

4. **Globals data section**: thường shift đồng đều trong section. Nếu tìm được 1-2 globals → assume các globals khác cùng shift.

**Key insight**: Khác biệt shift giữa regions là do game thêm/xóa code ở các vị trí khác nhau:
- Thêm code vào anti-cheat (+N bytes) → code sau anti-cheat dời +N
- Thay data struct layout → data globals dời khác
- Không phải luôn forward; có region có thể shift ngược chiều
