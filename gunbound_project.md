---
name: gunbound_project
description: Gunbound Legend PC - Cocos2d engine, XIGNCODE anti-cheat, aimbot potential via AI behavior classes
type: project
---

# Gunbound Legend PC

## Files
- Game: `D:\VTCZone\Games\Gunbound_Legend\Win32\GunboundM.exe` (5.4MB, PE32 x86)
- Engine: libcocos2d.dll (5.1MB)
- Anti-cheat: XIGNCODE (folder `XIGNCODE/`)
- Network: websockets.dll + libssl + libcurl
- IDA database: loaded GunboundM.exe

## Mobile version
- `D:\Code Tools\IDA_DATABASES\Gunbound\libMyGame.so` (13.7MB, ARM32)
- `D:\Code Tools\MOBILE_GAMES\KyNguyenIcarus\` (shared analysis)

## Key Classes (from RTTI)
- **ShotManager** - manages shooting (singleton)
- **Behavior_Fire** - AI fire behavior
- **Behavior_FireSimulation** - AI calculates shot trajectory
- **Behavior_SearchEnemy** - AI finds enemy target
- **Behavior_Move** - AI movement
- **Behavior_Pattern** - AI pattern logic
- **FireAngleHUD** - angle display UI
- **DontClimbingAngleHUD** - climbing angle limit

## Key Strings
- `/eangle`, `/rangle`, `/cangle` - debug commands? (angle manipulation)
- `FireSimulation:` - fire simulation log
- `Observer_ShotManager` - shot event observer
- `ShotJudgmentBestDamage` - best shot damage tracker
- `Node_FireAngle` - fire angle UI node
- `angle_based_velocity` - velocity from angle (mobile only)

## Anti-cheat: XIGNCODE
- Folder: Win32/XIGNCODE/ (with xigncode.log)
- NOT referenced in GunboundM.exe strings (loaded dynamically)
- Known to detect: Frida, debuggers, memory editors, process scanners
- Need to test if Frida attach works

## Aimbot Approach
Game has built-in AI that calculates shots (Behavior_FireSimulation).
Aimbot = hook/read AI calculation results and apply to player's shot.
Need to:
1. Find Behavior_FireSimulation vtable and functions
2. Find where angle/power values are stored after calculation
3. Read enemy position + wind
4. Either: call AI simulation for player, or calculate externally

## XIGNCODE Bypass - ALL FAILED
- ❌ Frida attach → VirtualAllocEx denied (error 0x05)
- ❌ Frida spawn → process killed during injection
- ❌ Remove XIGNCODE folder → game refuses to start
- XIGNCODE = kernel-level anti-cheat, beyond current tools
- Need ring0 driver to bypass (out of scope)

## Mobile Version (no XIGNCODE)
- Mobile has NO XIGNCODE anti-cheat
- But houdini translation on LD emulator blocks Frida hook
- Need real ARM device for full Frida hook
- Memory read/write still works on emulator

## ALL Bypass Attempts - FAILED
- ❌ Frida attach PC → XIGNCODE VirtualAllocEx denied
- ❌ Frida spawn PC → XIGNCODE kills process
- ❌ Remove XIGNCODE folder → game won't start
- ❌ Fake x3.xem (our compile) → game won't start
- ❌ Fake x3.xem (XignCodeBypass project) → login fails (no heartbeat)
- ❌ Mobile emulator (houdini) → can't hook ARM code
- ❌ No real ARM device available
- Need: heartbeat simulation OR kernel driver OR real ARM phone

## Status: BLOCKED - no viable hack path with current tools
