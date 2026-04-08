---
name: bepinex_approach
description: BepInEx/MelonLoader approach cho Unity IL2CPP games - hook C# method bang ten, khong can offset IDA
type: reference
---

# BepInEx / MelonLoader Approach

## Khi nao dung
- Game Unity IL2CPP (nhu JX1 Mobi)
- Muon hook C# method bang TEN (khong can tim offset IDA)
- Build mod/bot stable, de maintain qua game updates

## So sanh tools

| Game type | Best tool |
|-----------|-----------|
| Unity IL2CPP | BepInEx > Frida Gadget > Frida Server |
| Unity Mono | BepInEx / MelonLoader |
| Native C++ / Cocos2d | Frida |
| Packet hack | Frida |

## Uu diem
- Hook bang C# method name (khong can offset)
- Khong can IDA phan tich
- Stable qua game updates (method name it thay doi)
- Chay trong process (work tren houdini/LDPlayer)
- IL2CppInterop: truy cap type system, fields, properties bang ten

## Nhuoc diem
- BepInEx 6 Android chua stable (dang development)
- Debug kho: log qua file, khong interactive nhu Frida REPL
- Moi thay doi code → rebuild C# plugin → restart game (cham)
- Packet hook yeu (khong hook native send/recv de nhu Frida)
- It tai lieu Android, da so cho PC
- Can .NET/Mono runtime → nang hon Frida Gadget
- Mot so anti-cheat detect BepInEx de hon Frida

## Workflow khuyen nghi
1. **Frida Gadget TRUOC** - kham pha game, tim hieu logic (nhanh, flexible, REPL)
2. **BepInEx SAU** - build mod/bot hoan chinh (stable, C# method name)

## MelonLoader
- Chi ho tro PC (khong co Android)
- Tuong tu BepInEx nhung ecosystem nho hon
- Dung cho game Unity PC

## Tu dong hoa
Co the tu dong: patch APK + inject BepInEx + viet plugin C# + build + install LDPlayer
