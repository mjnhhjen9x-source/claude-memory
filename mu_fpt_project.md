---
name: mu_fpt_project
description: MU FPT Xua RE project - Season 3B MU Online, KillerCheatPro anti-hack, Frida attach OK, key addresses found
type: project
---

# MU FPT Xua - RE Project

## Project Location
`D:\Code Tools\MU_FPT\`
Game client: `D:\MUFPTXUA.COM\Main.exe` (PE32 x86, 11.6MB, Season 3B)

## Server
- IP: 14.225.207.36
- Same engine as MU Xua 2003 but different addresses and XOR key

## Anti-hack
- KillerCheatPro/Antihack.dll (2.3MB) - loaded dynamically via LoadLibrary
- NOT loaded in module list when Frida attaches (may load later or on demand)
- "Antihack CRC error!" string at 0xb9575c
- CAntiHack class found in RTTI

## Frida
- Attach OK (PID found, no crash)
- No anti-hack DLL detected in module list during gameplay
- Stealth may not be needed (test further)

## Found Addresses (IDA base 0x400000)
- **gameState**: 0xEC2B2C (confirmed: reads 5 when in game)
- **loginState**: 0x5F85B88 (confirmed: reads 61 when in game)
- **mapNumber**: 0xEA1A48 (confirmed: reads correct map)
- **sendPacketRaw**: 0x44F660 (C1 packets, confirmed via send() hook)
- **recvHandler**: 0x81C9D0 (23KB function)
- **WinMain**: 0x7EC280
- **loginHandler**: 0x5AFCF0
- **loginSceneInit**: 0x9CB4E0
- **moveStructPtr**: 0x7DF0A9C (confirmed pointer, but NOT charBase)

## NOT Found Yet
- charBasePtr (stats STR/AGI/VIT/ENE not found via memory scan - may use different storage format)
- pathFind, sendMovePkt, sendChat
- mainRender function
- Exact anti-hack mechanism

## XOR Key (different from MU Xua)
`[-1992659481, 1939845820, -1224824797, 1564040521, -1922839670, 1596685802, -2081836605, 1458065705]`

## Key Differences from MU Xua
| Feature | MU Xua | MU FPT |
|---------|--------|--------|
| gameState | 0x104E478 | 0xEC2B2C |
| loginState | 0x614E274 | 0x5F85B88 |
| sendPacketRaw | 0x43AEA0 | 0x44F660 |
| recvHandler | 0x726400 | 0x81C9D0 |
| Anti-hack | APICB.dll (small) | KillerCheatPro (2.3MB) |
| XOR key | different | different |

## Speed Hack - FAILED
- charBase+0x7C = speed display only (U16)
- Memory edit has NO EFFECT on actual game speed
- Server controls speed entirely (server-side validation)
- Time hack (GetTickCount/timeGetTime) → detected by anti-cheat
- Only way to increase speed = add AGI points legitimately
- AGI +300 → speed 131→141 (+10)

## CharBase Full Struct (confirmed via AGI diff)
- +0x0E = Level (U16)
- +0x10 = Reset (U32)
- +0x28 = STR (U16)
- +0x2A = AGI (U16)
- +0x2C = VIT (U16)
- +0x2E = ENE (U16)
- +0x34 = HP (U16)
- +0x38 = Mana (U16)
- +0x3C = MaxHP (U16)
- +0x40 = MaxMana (U16)
- +0x44/0x48 = unknown combat stat
- +0x7C = Speed display (U16)
- +0x7E = Attack power (U16)
- +0x80 = Attack min (U16)
- +0x84 = Attack max (U16)
- +0x88 = Defense combo (U16)
- +0x8C = Defense base (U16)
- +0x98 = Speed copy (U16)
- +0xB0 = Defense ability (U16)
- +0xB4 = Defense total (U16)
- +0xC0 = Free points (U16)

## Anti-Cheat: KTeam AntiCheat Pro v2.0
- Global flag: dword_5F85A1C (1=active)
- 3 layers: File CRC, Runtime packets, Plugin DLLs
- Speed report: 3 functions send speed to server
- All bypassed by hooking onEnter/onLeave

## Hack Results - ALL FAILED
- ❌ Speed hack (memory edit) - server validates position change
- ❌ Damage hack (memory edit) - server calculates damage
- ❌ Time hack (GetTickCount) - anti-cheat detects
- ❌ Packet replay (/addagi) - encrypted + server validates
- ❌ Multi-hit (duplicate attack packets) - server uses packet hash, ignores duplicates
- ❌ Skill cooldown bypass - game has no skill cooldown

**Conclusion**: Server validates everything server-side. KTeam AntiCheat Pro v2.0 catches client-side modifications. No viable hacks found.

## Remaining Options
1. Build bot (auto train, auto reset, auto login) - automation only, no cheats
2. Move to different game
