---
name: VTTT (MU Vu Tru Than Thoai)
description: MU Online private server "Vu Tru Than Thoai" - bypass CMPlay anti-cheat thanh cong, hook send packet OK
type: project
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Game info
- **Folder**: `D:\MU_VU_TRU_THAN_THOAI\`
- **Binary**: `Main.exe` (launcher) → spawns `vttt.main` (10.8 MB game PE32, KHONG packed)
- **Anti-cheat**: CMPlay (custom multi-layer in vttt.main itself)
- **IDA db**: `D:\MU_VU_TRU_THAN_THOAI\vttt.main.i64` (file → "Open as PE")
- **ImageBase**: 0x400000, runtime ASLR-shifted (typically 0x340000)

## Anti-cheat findings
- Process hide: NtQuerySystemInformation hook (Frida enum khong thay - dung tasklist /FI)
- Detour API hooks: OpenProcess/NtOpenProcess/ZwOpenProcess/CreateToolhelp32Snapshot
- Scan threads: 3 threads (sub_4FBBD0/4FB650/4FB5A0) tao boi sub_4FB730
- Handle scan: sub_500AE0 (HandleProcessScan)
- CRC verifier: sub_507800 (walk tree, compare CRC32 of code regions)
- **KILL function: sub_746520 (RVA 0x346520)** - shows splash + kills game

## CHIA KHOA BYPASS
**Patch sub_746520 → ret immediately** = AC van detect nhung KHONG the kill duoc.
Chi can patch 1 byte (`C3`) la game khong the bi tat.

Combined voi 4 patch khac de minimize logging + scan:

```javascript
// File: D:\Code Tools\PC_GAMES\VTTT\bypass.js
patchBytes(0x346520, [0xC3], "kill_splash -> ret");                          // KEY!
patchBytes(0x107800, [0xB0, 0x01, 0xC3], "CRC verifier -> 1");
patchBytes(0x2B7D60, [0xB0, 0x01, 0xC2, 0x04, 0x00], "whitelist -> 1");      // __thiscall ret 4
patchBytes(0x2B86B0, [0x33, 0xC0, 0xC2, 0x04, 0x00], "scan thread -> ret");
patchBytes(0x1098D0, [0xB0, 0x01, 0xC3], "detour install -> 1");
```

Test 5 phut: game alive 100%.

## Files
- `D:\Code Tools\PC_GAMES\VTTT\bypass.js` - 5-patch bypass
- `D:\Code Tools\PC_GAMES\VTTT\hook_send.js` - hook ws2_32 send
- `D:\Code Tools\PC_GAMES\VTTT\run_bypass.py` - apply bypass + survival test
- `D:\Code Tools\PC_GAMES\VTTT\run_hook.py` - bypass + hook send

## Frida attach quirks
- `device.enumerate_processes()` KHONG thay vttt.main (process hide)
- Phai dung `tasklist /FI "IMAGENAME eq vttt.main"` de tim PID
- Attach by PID truc tiep: works

## Opcodes captured (chua complete - du an pause)
| Opcode | Size | Note |
|---|---|---|
| F3 79 | 14 | EnterGame (10b name padded) - caller 0x56a8ce |
| F3 4A | 34 | Auth/heartbeat dinh ky |
| F3 7A | 5 | heartbeat |
| F3 6D | 5 | heartbeat |
| F3 58 | 4 | no-arg request |
| C3 18 ... | 24 | encrypted small packet (movement) |

## TODO neu lam tiep
- Capture login flow (logout + login lai) → tim Login function
- Capture CreateChar / DeleteChar packets → opcodes
- Build bot tu duong M4VN (bot_loop.js + bot_ui.py pattern)
- Dia chi function: SelectServer, Login, CreateChar, DeleteChar, RequestCharList, Logout
