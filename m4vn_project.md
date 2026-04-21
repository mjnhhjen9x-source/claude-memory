---
name: M4VN SEASON UPGRADE Project
description: MU Online private server - PhoenixCheat bypass OK, packet sniffer OK, auto sell NPC OK
type: project
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Trang thai: Bypass THANH CONG, san sang phat trien bot

## Thong tin game
- Game: M4VN SEASON UPGRADE (MU Online private server)
- Duong dan: `C:\M4VN SEASON UPGRADE\Main.exe`
- Code: `D:\Code Tools\PC_GAMES\M4VN\`
- Engine: PE32 x86 (Windows native)
- Anti-cheat: PhoenixCheat PREMIUM (server-side)
- Main.exe: UPX packed (4.4MB -> 16MB khi giai nen)

## PhoenixCheat PREMIUM
- File integrity check (SHA256 hash 533 files)
- MemoryGuard (227/429 addresses)
- ChecksumList (11 checksums)
- Hardware ID tracking
- Server-side hash validation (kiem tra hash Main.exe)
- Mutex "#32770" check (game kiem tra Launcher.exe dang chay)

## Bypass v5 (THANH CONG)
- File: `bypass_v5.js` + `run_bypass_v2.py`
- Runner: `python -u run_bypass_v2.py` (can quyen Administrator)

### Ky thuat chinh:
1. **Giu Main.exe packed goc** - server kiem tra hash file tren disk, phai giu nguyen
2. **Python tao mutex "#32770"** bang ctypes (gia lap Launcher.exe)
3. **Frida spawn** Main.exe (suspended) -> attach -> load script -> resume
4. **Doi UPX giai nen xong** bang cach poll byte tai RVA 0x1FF00:
   - `0x23` = chua giai nen (du lieu UPX)
   - `0x53` = da giai nen (push ebx - code that)
   - Poll moi 50ms
5. **Patch PhoenixCheat** sau khi UPX xong:
   - FileIntegrityCheck (0x1FF00) -> `mov eax, 1; ret` (return 1 = pass)
   - SafeExitProcess (0x21E50) -> `xor eax, eax; ret` (return 0)
   - sub_41F2A0 (0x1F2A0) -> `xor eax, eax; ret`
   - DisconnectSplash (0x22B60) -> `xor eax, eax; ret`
6. **Chan tat ca API exit**:
   - TerminateProcess -> args[0] = NULL (fail am tham)
   - ExitProcess -> ret (khong exit)
   - NtTerminateProcess -> invalid handle
   - RtlExitUserProcess -> ret
   - IsDebuggerPresent -> return 0

### Bai hoc quan trong:
- **UPX timing**: Frida patch truoc UPX = bi ghi de. PHAI doi UPX xong
- **Server hash check**: Thay file Main.exe = server tu choi. PHAI giu file goc
- **Mutex**: Game check mutex "#32770", khong co = game goi WinExec(Launcher) roi exit

## Dia chi ham quan trong (RVA tu base 0x400000)
- SafeExitProcess: 0x421E50 (RVA 0x21E50)
- PhoenixCheat_FileIntegrityCheck: 0x41FF00 (RVA 0x1FF00, 3220 bytes)
- sub_41F2A0: 0x41F2A0 (RVA 0x1F2A0, caller SafeExitProcess)
- DisconnectSplash: 0x422B60 (RVA 0x22B60)
- FileCheck1: 0x42D1D0, FileCheck2: 0x42D250
- ExtraChecks: 0x43C370, 0x43C430, 0x43BCA0, 0x43C4E0
- InitCheck: 0x430830
- PhoenixCheat logger: 0x43B910

## Hack/Bot da lam duoc

### Auto Sell NPC (THANH CONG - khong UI)
- File: `auto_npc_sell.js`
- Goi truc tiep game functions, khong can mo UI
- Quy trinh: NpcClick → Interact(122,0) → StoreOpen → Interact(122,4) → SendSell(0-127, 1) → Interact(122,3)
- Cac ham game:
  - NpcClick: 0x8C1C60 (void, no args)
  - Interact: 0x8ADCF0 (cdecl, a1=122, a2=0/3/4)
  - StoreOpen: 0x7FCEC0 (thiscall)
  - SendSell: 0x7FD5A0 (stdcall, item_code, tab_id)
- Packet dung CStreamPacketEngine XOR encryption (khong phai C3/C4)

### Packet Sniffer (THANH CONG)
- File: `packet_sniffer_v2.js` + `sniff_attach.py`
- Hook sub_8AD120 (send) va sub_8A1E30 (recv) de bat plaintext
- Opcode map day du tai `opcode_map.md`

### Doc Memory (THANH CONG)
- HP, MP, Shield, vi tri, entity list
- Entity array: 0x12FAC90, player ID: 0x63C7264
- Entity struct: size=1712, +188=ID, +256=tileX, +260=tileY, +992=active, +1012=type

### Khong kha thi (server validate)
- Attack speed hack: server tinh 1 packet/giay
- Teleport: path encoding phuc tap, server validate
- Damage hack: server-side

## Buoc tiep theo
- Auto farm (danh quai + nhat do + sell NPC loop)
- Auto potion (uong thuoc khi HP thap)
- Multi-account support

## 2026-04-21: IP BAN + ANTI-PROXY BYPASS
Server ban IP + client check getpeername. Fix 2 tang trong bypass_v5.js:
1. SOCKS5 hook on `ws2_32!connect` -> redirect to proxy `42.96.3.54:50793`
2. `getpeername` spoof -> return real GAME_SERVER_IP (14.225.209.79)

PROXY_CFG o dau bypass_v5.js, enabled=true. Khi can change proxy hoac off,
edit truc tiep. `bot_ui.py` + `bot_loop.js` dung chung bypass_v5 -> auto route qua proxy.

Xem chi tiet technique: `technique_socks5_antiproxy_bypass.md`

## 2026-04-21: Proxy Tab trong bot_ui.py
bot_ui.py da co tab Proxy (giua Bot va Log) cho phep:
- Paste list SOCKS5 proxy (format `host:port:user:pass:SOCKS5`, `:SOCKS5` optional)
- Chon 1 proxy active hoac Direct (combobox readonly)
- Save button: parse + persist vao bot_config.json + rebuild dropdown
- Test button: CS handshake 8s qua proxy duoc chon, status `OK (NB list)` / `FAIL: <reason>`
- Comment lines (#) + empty lines duoc skip; warning duoc log

Khi `_spawn_game` chay, `self.ui.apply_proxy_config(src)` se regex-substitute
`var PROXY_CFG = {...};` trong bypass_v5.js source bang config user chon.
Direct = `enabled: false` (no SOCKS5 redirect).

Spec: `D:/Code Tools/PC_GAMES/M4VN/docs/specs/2026-04-21-proxy-tab-design.md`
Plan: `D:/Code Tools/PC_GAMES/M4VN/docs/plans/2026-04-21-proxy-tab.md`
Tests: `tests/test_proxy_parse.py` (14 tests, run `pytest tests/ -v`)

Module-level functions in bot_ui.py:
- `parse_proxy_list(raw)` -> (entries, warnings)
- `apply_proxy_config_to_source(source, active_key)` -> modified source
- `_PROXY_CFG_RE` for regex substitution

Config schema (bot_config.json) them 2 keys:
- `proxies`: raw textarea string (preserve formatting + comments)
- `active_proxy`: full key `host:port:user:pass` or empty for Direct
