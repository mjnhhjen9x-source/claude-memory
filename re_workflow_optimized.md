---
name: re_workflow_optimized
description: Quy trinh toi uu RE + hack + bot game online - 5 buoc, tool mapping, multi-instance, danh gia hack nhanh
type: reference
---

# Quy Trinh RE & Hack/Bot Game

## Buoc 0: DANH GIA GAME
```
Engine?
├── Unity IL2CPP → Il2CppDumper + IDA + Frida Gadget/BepInEx
├── Unity Mono → dnSpy + Frida
├── Cocos2d/Native → IDA + r2pipe + Frida
├── Flash/SWF → JPEXS FFDec + Python bot
└── .NET (launcher) → dnSpy/ILSpy

Platform + Hook tool?
├── PC x86 → Frida Server
├── Mobile ARM + root phone → Frida Server ARM
├── Mobile ARM + LDPlayer x86 → Frida Gadget (patch APK)
└── Unity IL2CPP (build bot) → BepInEx

Anti-cheat? → Danh gia truoc khi attach
```

## Buoc 1: STATIC ANALYSIS
- IDA Pro: decompile function, struct, offset
- Il2CppDumper: dump class/method names (Unity IL2CPP)
- JPEXS: decompile ActionScript (Flash)
- r2pipe: string search, xref nhanh
- apktool: extract/repack APK
- dnSpy: decompile .NET/Unity Mono

## Buoc 2: THU HACK (5 phut moi cai)
| Test | Cach | OK | Fail |
|------|------|-----|------|
| Speed | Ghi speed memory | Nhanh hon | Khong doi |
| Damage | Ghi damage/stats | Quai chet nhanh | Khong doi |
| Teleport | Ghi X/Y | Nhay duoc | Bi reset |
| Gold | Ghi gold value | Mua duoc | Reset |
| Cooldown | Ghi = 0 | Spam skill | Van cho |

→ Hack duoc → XONG
→ Server reject → biet validate gi → BUOC 3

## Buoc 3: LAM BOT
- L2.5: Frida doc memory + ADB tap
- L3: Frida hook game function (goi truc tiep)
- L4: Python socket bot (standalone)
- BepInEx: C# plugin hook method name (Unity)

## Buoc 4: MULTI-INSTANCE
- Game PC: WindowMonitor + render throttle + SOCKS5 proxy
- Game Mobile: LDPlayer Multi-Instance + ADB per instance
- Bot: Python multiprocessing + Tkinter UI + auto-restart + log

## Tools
| Tool | Buoc | Status |
|------|------|--------|
| IDA Pro 8.3/9.1 | 1 | OK |
| Il2CppDumper 6.7.46 | 1 | OK |
| JPEXS FFDec | 1 | OK |
| r2pipe/radare2 | 1 | OK |
| apktool 3.0.1 | 1 | OK |
| dnSpy | 1 | OK |
| Frida 17.9.1 | 2,3 | OK |
| Frida Server x86/x64 17.7.3 | 2,3 | OK |
| Frida Gadget ARM/ARM64 17.7.3 | 2,3 | OK |
| BepInEx Android | 3 | OK |
| ADB 1.0.41 | 2,3,4 | OK |
| Python 3.14 | 3,4 | OK |
| LDPlayer | 4 | OK |
| WindowMonitor | 4 | OK |

## Danh gia hack game online
```
Client tinh toan → Speed/Damage/Teleport/Gold ✅
Server validate co ban → Chi Zoom/ESP/Aim ✅, BOT ✅✅✅
Server validate manh → Chi BOT ✅
```
