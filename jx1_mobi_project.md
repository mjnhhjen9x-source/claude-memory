---
name: jx1_mobi_project
description: JX1 Mobile (Kiem Hiep 1) - Unity IL2CPP + xLua, APK recon done, IDA analyzing libil2cpp.so
type: project
---

# JX1 Mobile

## Project Location
`D:\Code Tools\MOBILE_GAMES\jx1_mobi\`
APK: `C:\Users\Than Tai\Downloads\jx1.mobi_04.04.02_full.apk` (2.2GB)

## Engine
- Unity IL2CPP + xLua
- libil2cpp.so: 74MB (arm32), 88MB (arm64)
- global-metadata.dat: 11.8MB
- Il2CppDumper: 202,704 methods, 9,998 classes, dump.cs 908K lines

## Key Addresses (armeabi-v7a libil2cpp.so)

### Network
- NetWorkClient$$WriteDataClientEncrypt: 0x124CCCC
- NetWorkClient$$WriteDataClient: 0x124C624 / 0x124CCC4
- NetWorkClient$$OnReceivedData: 0x124BF58
- NetWorkClient$$ReceiveDataCallback: 0x124C1C0
- NetWorkClient$$SendDataCallback: 0x124D160
- ServerMessageSender$$TrySendDataToServer: 0x17B22AC
- ServerMessageSender$$EnqueueRequestToSend: 0x17B2548
- ServerMessageSender$$SetClientKey: 0x17B2AE4

### Combat
- Npc$$DoBlurAttack: 0x13198E4
- Npc$$DoRunAttack: 0x13199C8
- Npc$$DoManyAttack: 0x1319CC0
- SkillEffectServiceImpl$$TryCastSkill: 0x16025BC / 0x1602614
- 282 Attack methods total

### Auto System
- AutoServiceImpl$$ListenerTeleportAndCloseAutoTuDongDanh: 0x1207F0C

### Login
- ReconnectServiceImpl$$StartConnectToLoginServerPhase: 0x14714B4

### Teleport
- MissionServiceImpl$$TelePortToMapByMission: 0x14708D8
- Npc$$ForceTeleportPosition: 0x17BED7C

## Files
- dump.cs: IL2CPP dump (908K lines)
- script.json: 202K methods with addresses
- stringliteral.json: string literals
- extracted/: libil2cpp.so, libxlua.so, metadata

## IDA Database
- libil2cpp.so loaded in IDA Pro
- Il2CppDumper ida_py3.py script needs re-run (rename didnt apply)

## Key Findings
- Frida trên LDPlayer: đọc/ghi memory OK, KHÔNG hook được (houdini ARM translation)
- Damage/Stats hack: server-side validation, ghi memory chỉ đổi hiển thị
- TimeScale hack: bị overwrite
- Position offset: tìm được nhưng GC relocate, không stable
- Android Studio AVD ARM64: Google đã bỏ support ARM emulation trên x86 host

## Offsets (ARM32 libil2cpp.so)
- Npc$$GetMpsPos: +0x24=X, +0x28=Y
- Npc$$GetMapPos: +0x310=X, +0x314=Y (Script.Core variant)
- Npc$$SetPhysicsDamage: +0xF8→dmg_array, +0x10=min, +0x18=max
- MainCoreShell: +0x78 = MainPlayer
- MainPlayerServiceImpl: +0x60 = Npc (get_MainPlayerNpc)
- MainCoreShell$$Awake: TypeInfo->static_fields[0] = instance

## Blocked: Cần điện thoại Android root ARM
1. Frida hook trên điện thoại → tìm pointer chain ổn định
2. Dùng pointer chain trên LDPlayer (chỉ đọc memory, không hook)
3. Build bot: memory read + ADB tap

## Tools
- Il2CppDumper: D:\Code Tools\TOOLS\Il2CppDumper\
- frida-server x86: D:\Code Tools\TOOLS\frida-server-x86
- frida-server x86_64: D:\Code Tools\TOOLS\frida-server-x86_64
