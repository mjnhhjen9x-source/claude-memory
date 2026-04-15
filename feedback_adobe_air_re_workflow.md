---
name: feedback_adobe_air_re_workflow
description: Adobe AIR mobile game RE workflow - identify, crack SWF, decompile AS3, map protocol
type: feedback
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
# Adobe AIR Mobile Game RE Workflow

**Why:** Adobe AIR games (VLCM Mobile) are GREEN TARGETS - AS3 decompiles to near-source code.
DWARF-leaked symbols in custom libair.so make crypto trivial to reverse.

**How to apply:** Follow this checklist when APK has libair.so + libCore.so + assets/*.swf

## Identify Adobe AIR APK

**Positive signals:**
- `lib/*/libair.so` + `lib/*/libCore.so` in APK
- `assets/META-INF/AIR/application.xml` present
- AIR SDK version in manifest `xmlns="http://ns.adobe.com/air/application/51.x"`
- `assets/APPNAME.swf` (main game SWF)
- ANE extensions under `assets/META-INF/AIR/extensions/`
- `com.adobe.air.AndroidActivityWrapper` in Java exports

## Step 1: Check If libair.so Is Custom (CRITICAL)

Vanilla Adobe AIR libair.so has only `Java_com_adobe_air_*` exports. Custom builds
(like tpl-libair) add their own namespace.

```bash
python -c "
import re
d = open('libair.so','rb').read()
strs = re.findall(rb'[\x20-\x7e]{6,}', d)
for kw in [b'Secure', b'Crypto', b'Cipher', b'decrypt', b'mastertoan', b'GrowX']:
    if any(kw in s for s in strs):
        print(f'{kw.decode()}: FOUND')
"
```

**If ANY custom keyword found** → libair.so is modified → contains SWF decrypt routine.

## Step 2: Check Main SWF For Encryption

```bash
xxd assets/APPNAME.swf | head -1
```

**Valid SWF magic:**
- `FWS` (uncompressed)
- `CWS` (zlib compressed)
- `ZWS` (LZMA compressed)

**Non-standard magic** (like VLCM's `GRXTPL`) = custom container format. Need to reverse.

## Step 3: Find Decrypt Function in libair.so

Target function names in modern custom libair:
- `decryptAndLoadAsset`, `loadSecureAsset`, `loadEncryptedSwf`
- Namespace: `SecureApp::`, `AppSecurity::`, `NativeLoader::`

Entry points to check:
- `Java_com_adobe_air_AndroidActivityWrapper_initAppInstance` (runs at app start)
- `Java_com_adobe_air_AndroidActivityWrapper_startAppInstance`

Strategy: Load libair.so in IDA 9.1 with DWARF → search `SecureApp::` in list_globals_filter.

## Step 4: Extract AES Key/IV (common pattern)

Modern custom libair uses AES-256-CBC. Find:
1. `SecureApp::SecureAppManager::initialize(SecurityConfig&)` - config struct copier
2. `initAppInstance` JNI function - constructs SecurityConfig with hardcoded bytes
3. Look for `xmmword_XXXXX` constants loaded into encryptionKey/IV fields

Key pattern in IDA pseudocode:
```c
*(_OWORD *)assetData = xmmword_4B8D4;    // first half of AES key
*(_OWORD *)&assetData[16] = xmmword_4B8E4;  // second half
std::vector<unsigned char>::assign(&v8.encryptionKey, assetData, ...);

*(_OWORD *)assetData = xmmword_4BAE0;    // IV (16 bytes)
std::vector<unsigned char>::assign(&v8.encryptionIV, assetData, ...);
```

Extract bytes with `read_memory_bytes(xmmword_addr, 32)` for key, 16 for IV.

## Step 5: Understand Obfuscation Layers

Common layers before AES:
1. **Byte rotation** (`std::rotate` / `unrotateBytes`) - right rotate by N
2. **XOR with simple key** (uncommon with AES present)
3. **Custom magic header** (e.g. "GRXTPL") - strip first N bytes

Check `decryptAndLoadAsset` body for pre-AES transforms.

## Step 6: Write Python Decrypter

Standalone, no game needed:
```python
# Typical template
import struct
from Crypto.Cipher import AES

KEY = bytes.fromhex("...")  # 32 bytes AES-256
IV  = bytes.fromhex("...")  # 16 bytes

def unrotate(data, n):
    n %= len(data)
    return data[-n:] + data[:-n] if n else data

def pkcs7_unpad(d):
    p = d[-1]
    return d[:-p] if 1 <= p <= 16 else d

raw = open(in_path, 'rb').read()
assert raw[:8] == b"MAGIC"
rotate_amount = struct.unpack("<I", raw[8:12])[0]
payload = raw[32:]
data = unrotate(payload, rotate_amount)
plaintext = pkcs7_unpad(AES.new(KEY, AES.MODE_CBC, IV).decrypt(data))
# plaintext starts with FWS/CWS/ZWS = valid SWF
```

## Step 7: Decompile SWF with JPEXS FFDec

```bash
ffdec-cli.exe -export script out_dir decrypted.swf
```

Output: folder tree of .as files, readable AS3 (source-like quality).

Expected structure:
```
scripts/
├── com/
│   ├── tgame/           game code (grep for this namespace)
│   ├── adobe/           AIR runtime
│   └── greensock/ gs/   tween library
├── init/                startup + HTTP API
├── org/                 PureMVC framework
└── br/com/stimuli/      BulkLoader (popular AS3 loader)
```

## Step 8: Map Protocol

Find classes with:
- `flash.net.Socket` import → TSocket-like classes
- `sendMsg` / `registerMsg` methods → opcode table
- `doConnect` / `host` / `port` → connection setup

Common AS3 game protocol patterns:
- **MSG_HEAD_MARK** constant (magic byte) 
- **seq** field (sequence number)
- **KEY** const array (stream cipher)
- **encryptBytes** / **decryptBytes** static methods

## Step 9: Find API Endpoints

Search for `http://` or `https://` string literals:
```bash
grep -r "https://" scripts/init/ scripts/com/tgame/
```

Look for:
- `USER_LOGIN_URL`, `USER_LOGOUT_URL`
- `GET_SERVER_URL`, `LOAD_CONFIG_URL`
- `DEV_BUILD` flag → separate dev endpoints
- Debugger endpoints (`Debugger`, `Test`, `Dev`, `Internal` in URL)

## Common Pitfalls

- **AIR encryption target wrong lib:** Always check libair.so FIRST - it's the modified one.
  libCore.so is usually vanilla.
- **DWARF stripped in release:** Pre-release/dev builds often have symbols. Release may not.
  If release stripped: use IDA FLIRT or compare with vanilla Adobe AIR 51.x.
- **Multiple SWF in APK:** Only main SWF is encrypted usually. ANE library.swf files
  are standard CWS.
- **XMMword in IDA:** Decompile may show as `xmmword_XXXXX` - use `read_memory_bytes` to dump.

## Tool Chain

| Tool | Purpose |
|---|---|
| IDA 9.1 Pro | libair.so analysis with DWARF |
| Python + pycryptodome | AES decrypter |
| JPEXS FFDec | SWF decompiler (CLI or GUI) |
| apktool | APK unpack (alternative to unzip) |
| Wireshark / mitmproxy | Runtime packet capture |

## Why Adobe AIR is EASY compared to Unity/Cocos

| Aspect | Adobe AIR | Unity IL2CPP | Cocos 3.x + V8 |
|---|---|---|---|
| Bytecode format | AS3 (ABC) | x86 native after IL2CPP | V8 cached data |
| Decompiler | JPEXS (source-quality) | Il2CppDumper + IDA | NONE reliable |
| Modify & repack | Easy (JPEXS → resave) | Very hard (need recompile) | Near impossible |
| Source readability | Near-source | Obfuscated native | Unreadable bytecode |
| RE time estimate | Hours-days | Weeks | Weeks-months |

**Adobe AIR is typically 5-10x faster to RE than other engines.**
