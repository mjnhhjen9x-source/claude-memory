---
name: mu_login_lessons
description: MU Online PC login flow - critical lessons learned from MU Xua 2003 RE. NEVER call connectToConnectServer when game already at server select screen (gs=2). Applicable to all MU-based games.
type: project
---

# MU Online Login Flow - Critical Lessons

## Root Cause of 5-min Disconnect
`connectToConnectServer()` resets loginFlag=0, loginState=0 and calls `_connectToServer()` again when game is ALREADY connected to CS (gs=2). Server detects double/invalid connection -> kicks after 5 minutes.

**Why:** Game starts at gs=2 = already at server select screen with CS connected. Calling connectToConnectServer() creates a second TCP connection to CS, corrupting the session state on server side.

**How to apply:** When gs=2, SKIP connectToConnectServer entirely. Only send F4/03 (server select) + F1 (login). Game handles CS connection on its own during startup.

## Correct Login Flow (match manual login exactly)
1. Game starts at gs=2 (CS already connected by game itself)
2. Send F4/03 (server code) - NO connectToConnectServer()
3. Wait cf=1 (game server connected)
4. Send F1 login (via _sendLoginPacket native function)
5. Wait gs=4 (character select screen)
6. Send F3/E8 (enter game)
7. Wait for F3/E5 response (ls=60)
8. Send F3/0E (select char confirm) - sent AFTER F3/E5, not before
9. Wait gs=5 (game sets it after receiving F3/E1 from server)

## Key Packet Order (from manual login capture)
```
OUT: F4/03 (select server)
IN:  F4/03 response + F1/00
OUT: F1/0E enc=1 sz=59 (login credentials)
IN:  F1/01 -> gs=4, ls=19
OUT: 0x0E enc=1 (auto packet)
OUT: F3/0D (auto packet)
IN:  F3/F2, F3/00 -> ls=50,51
OUT: F3/E8 (enter game)
IN:  F3/E5 -> ls=60
OUT: F3/0E (select char - sent AFTER F3/E5!)
IN:  F3/E1 -> gs=5 (game sets this automatically)
```

## Anti-hack (APICB.dll) - Safe to Patch
- CBAnihack_Work (scanner): safe to patch ret - confirmed no disconnect
- CBAnihack_Attack (reporter): safe to patch ret
- CBAnihack_Init: keep alive (runs once at startup before Frida attach)
- CBAnihack_Recv (challenge-response): keep alive

## Security Code
- Server custom may require secondary password
- sendSecurityCode native: sub_5B90F0(a1, a2, a3)
- Two calling conventions found in IDA - needs per-server testing
- Some servers allow entering game even with wrong code (just shows dialog)

## Network Functions Must Run on Game Thread
All packet-sending functions (sendLogin, sendServerConnect, enterGame, sendPacketRaw) must be wrapped in runOnGameThread() to avoid race conditions with game's message loop.

## Key Addresses (MU Xua 2003, Main.exe base 0x400000)
- gameStatePtr: 0x104E478
- loginStatePtr: 0x614E274
- loginFlagPtr: 0x614E0A0
- connFlagPtr: 0x614E0B4
- _connectToServer: 0x6F3EA0
- _sendLoginPacket: 0x535660
- _sendPacketRaw: 0x43AEA0
- _sendSecurityCode: 0x5B90F0
- connectServerIP: 0x614DA78
- connectServerPort: 0x614DA98
