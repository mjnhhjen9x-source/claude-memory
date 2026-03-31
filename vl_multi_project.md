---
name: VL Multi-Instance Project
description: Bypass VietGuards anti-cheat 4-instance limit - deep RE of auth/HWID/kick system, current best approach is spawn-mode hardware spoof
type: project
---

## Muc tieu
Chay nhieu hon 4 game instances cua VL (Vo Lam) tren cung 1 PC. VietGuards gioi han 4 instances per HWID - instance thu 5 bi kick tu server.

## Game Setup
- Game path: `D:\VL 1\`, process: game.exe -> vggame.exe
- VietGuards: VGengine.dll (12.9MB, DirectDraw proxy, 42 DC31_* exports, Enigma packed)
- Server: 103.206.216.82 ports 5622 (auth), 6662 (game), 58899 (connection check only)
- All game data encrypted boi Rainbow.dll

## HWID System (DA XAC DINH)
- HWID: `268757AB94D44F8D8E0C82FEFE5B1615` (32 ASCII hex, MD5-like)
- Hardware sources (qua kernel32 API, KHONG qua direct syscalls):
  - PhysicalDrive0: STORAGE_QUERY (0x2d1400) + SMART (0x7c088) serial
  - Wi-Fi MAC: 04cf4b226c66 (qua GetAdaptersInfo)
  - C:\ volume serial: 0x564dff83 (qua GetVolumeInformationW)
- HWID computed on-the-fly, KHONG cached trong memory (scan 0 matches)
- HWID xuat hien tren stack chi khi Login function chay

## Auth Flow (DA XAC DINH)
1. Port 58899: chi connect+close (jx.dll, connection check, KHONG gui data)
2. Port 5622: RECV 44-byte challenge -> Login function -> Rainbow encrypt -> SEND 129 bytes
3. Port 6662: game server (sau auth thanh cong)

## Login Function (DA XAC DINH)
- Address: 0x5297E0 (self-modifying code, region 0x401000 size=0x2bf000, prot=rwx)
- Calling convention: thiscall, ECX=0x6dec30 (game object)
- Args: arg0=username_buffer, arg1=buffer(HWID@offset0 + password@offset64), arg2=1
- Rainbow.dll+0x1150: encrypt+send (2 calls: 2-byte header + 127-byte auth body)
- Auth packet fully encrypted, HWID khong visible trong output

## VGengine.dll Internals (DA XAC DINH)
- 42 exports: tat ca DC31_DirectDraw* (DDraw proxy wrapper)
- 21 direct syscall stubs (syscall instruction): 0x30,0x46,0x54,0x5b,0x73,0x78,0x7c,0x7d,0x81,0x9b,0xb1,0xd2,0xfc
- Syscall stubs la cho anti-debug/anti-tamper, KHONG co NtDeviceIoControlFile(0x7), NtCreateFile(0x55), NtQueryVolumeInformationFile(0x49)
- VGengine hook ntdll stubs (overwrite B8->E9 jmp) de monitor syscalls
- Hardware queries di qua kernel32 APIs (hookable)

## Kick System (DA XAC DINH)
- Server-side enforced: server ngung gui data sau khi kick
- Kick packets: RECV 127 bytes (0x7f00) + RECV 73 bytes (0x4900) tren port 6662
- Client-side kick blocking (replace/drop/NOP) KHONG hieu qua vi server da ngung gui
- Rainbow.dll disconnect chain: +0x2ce1 -> +0x26cc -> +0x15f2 -> +0x1fef -> +0x16ba -> vggame.exe

## HWID-Password Binding (DA XAC DINH)
- Thay doi HWID tai Login function (v25) -> server tra "wrong password"
- HWID duoc dung trong tinh toan password hash/MAC
- Server verify bang HWID + password hash -> doi HWID = hash sai = "wrong password"
- KET LUAN: KHONG the dung account cu voi HWID moi (phai dang ky account moi voi HWID moi)

## Current Best Approach: Spawn-Mode Hardware Spoof (v33)
- Frida child gating: spawn game.exe -> bat vggame.exe suspended -> inject hooks -> resume
- Hook kernel32!DeviceIoControl, iphlpapi!GetAdaptersInfo, kernel32!GetVolumeInformationW
- Spoof hardware responses TRUOC KHI VGengine tinh HWID
- STORAGE_DEVICE_DESCRIPTOR: SerialNumberOffset tai offset 24 (KHONG phai 12)
- Script: `spawn_spoof.py` + `hwid_spawn_spoof.js`
- Status: hooks fire thanh cong, can fix serial offset va test login

## Failed Approaches (tat ca versions)
1. v4-v7: IOCTL/SMBIOS/Volume/MAC/Registry/Pipe spoof - FAILED
2. v12: engine.dll+0xa0a0 hook - game crash
3. v13: Rainbow.dll+0x1150 HWID patch - MAC/signature mismatch
4. v14: Rainbow monitor/patch - auth rejected (MAC inside Login)
5. v16-v19: Kick packet blocking (zero/WSAEWOULDBLOCK/closesocket/XOR) - server-side
6. v21: kernel32 API hooks (late attach) - hooks fire nhung HWID khong doi (timing)
7. v25: Login function HWID overwrite - "wrong password" (HWID in hash)
8. v28-v29: NOP disconnect + replace kick with dummy - server ngung gui data
9. v30: Port 58899 monitor - chi la connection check, khong co data
10. v32: VGengine syscall stub patch - khong co target stubs (hardware khong qua direct syscall)
11. System-level spoof (MachineGuid + Ethernet MAC) - VGengine doc Wi-Fi MAC, khong doc Ethernet

## Dumped Files
- `vietguards_decrypted.bin` - Enigma module 25.4MB (IDA base 0x400000)
- `vggame_dumped.bin` - vggame.exe memory dump (IDA, Login function analysis)

## Scripts (D:\Code Tools\VL_MULTI\)
- `spawn_spoof.py` + `hwid_spawn_spoof.js` - **CURRENT: spawn-mode hardware spoof**
- `test_spawn_spoof.py` - Original polling launcher
- `login_dump_ida.js` (v31) - Login function dumper + auth tracer
- `vgengine_syscall_patch.js` (v32c) - Syscall stub finder/patcher
- `port58899_monitor.js` (v30) - Port 58899 + NtDeviceIoControlFile monitor
- `kick_nop_disconnect.js` (v29) - Replace kick + block disconnect
- `kick_trace.js` (v27) - Kick handler backtrace
- `hwid_login_spoof.js` (v25) - Login HWID overwrite
- `hwid_source_finder.js` (v20) - Hardware source discovery

## Next Steps (neu tiep tuc)
1. Fix STORAGE_DEVICE_DESCRIPTOR serial offset (24 thay vi 12) trong hwid_spawn_spoof.js
2. Chay spawn_spoof.py va login -> kiem tra [LOGIN] HWID co doi khong
3. Neu HWID doi: dang ky account moi voi HWID moi, login -> bypass thanh cong
4. Neu HWID khong doi: debug them xem VGengine dung hardware source nao khac
5. Alternative: chay instance 5+ trong VM (da confirm hoat dong nhung nang)

**Why:** Server gioi han 4 connections/HWID. HWID bound vao password hash nen khong the doi HWID cho account cu. Can tao HWID moi + account moi.

**How to apply:** Chay spawn_spoof.py voi fix serial offset. Neu HWID doi -> tao account moi tu may voi HWID moi -> dung account do cho instance 5+.
