---
name: UPX memory dump từ Frida + PE fix cho IDA analysis
description: Dump binary unpacked qua Frida + fix PE header để IDA load được
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Game UPX packed → dump memory qua Frida khi đã decompress. Dump raw không dùng được với IDA vì UPX section RAW_SZ=0. Phải **fix PE header** (set UPX0 RAW_SZ = VSZ) trước khi load IDA.

**Why**: Main.exe M4VN là UPX packed. 
- On disk: file ~4MB (compressed)
- In memory sau decompress: ~160MB (UPX0 section mở rộng)
- Frida dump memory contiguous từ base 0x400000 → binary ~160MB
- IDA load file này: parse PE header thấy `UPX0 VA=0x1000 RAW_OFF=0 RAW_SZ=0` → **không load code**, chỉ thấy 0xFF
- List_functions trả về <10 functions ở region cao (nullsubs) → useless

**How to apply**:

1. **Dump memory qua Frida** (chia chunk 1MB tránh OOM):
```python
DUMP_JS = '''
rpc.exports = {
    dump: function() {
        var mod = Process.getModuleByName("Main.exe");
        var base = mod.base, size = mod.size;
        var f = new File(OUTPUT_PATH, "wb");
        var CHUNK = 0x100000;
        for (var off = 0; off < size; off += CHUNK) {
            var readSz = Math.min(CHUNK, size - off);
            var addr = base.add(off);
            try { Memory.protect(addr, readSz, 'rwx'); } catch(e) {}
            try {
                var chunk = addr.readByteArray(readSz);
                if (chunk) f.write(chunk);
                else f.write(new ArrayBuffer(readSz));  // zero-fill
            } catch(e) { f.write(new ArrayBuffer(readSz)); }
        }
        f.close();
    }
};
'''
```

2. **Fix PE header** cho dump (Python):
```python
import struct
data = bytearray(open(dump_path, 'rb').read())
pe_off = struct.unpack_from('<I', data, 0x3C)[0]
coff = pe_off + 4
opt_sz = struct.unpack_from('<H', data, coff + 16)[0]
sec_off = coff + 20 + opt_sz  # first section header

# Section 0 = UPX0, set RAW_SZ=VSZ, RAW_OFF=0x1000, flags=CODE|EXEC|READ
vsize = struct.unpack_from('<I', data, sec_off + 8)[0]
struct.pack_into('<I', data, sec_off + 16, vsize)          # SizeOfRawData
struct.pack_into('<I', data, sec_off + 20, 0x1000)          # PointerToRawData
struct.pack_into('<I', data, sec_off + 36, 0x60000020)      # Characteristics

open(dump_fixed_path, 'wb').write(data)
```

3. **Load fixed file vào IDA** → auto-analysis sẽ tìm functions.

**Alternative khi IDA không load được**: dùng **Python pattern matching** trên raw V3 dump thay vì IDA. Memory dump có `file_off = VA - base`, nên:
- V2 (PE file): map VA qua section table
- V3 (raw dump): direct offset VA - base = file offset

**Reference**: `D:\Code Tools\PC_GAMES\M4VN\dump_main_v3.py`
