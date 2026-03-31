---
name: MU Xua 2003 RE Project
description: MU Online private server RE - auto-reset bot with Frida, multi-account, auto-login (fixed 5min disconnect), render throttle
type: project
---

MU Xua 2003 private server client RE project at `D:\Code Tools\MU_XUA_2003\`.

**Game client:** `C:\MU_XUA_2003\Main.exe` (PE32 x86, base 0x400000)

## Architecture
- **bot_ui.py** - Tkinter dark UI, multi-account management
- **bot_worker.py** - Per-account bot logic (train, reset, stats, warp)
- **game_session.py** - FridaAPI wrapper + GameSession (connect, login, walk, warp)
- **auto_reset.js** - Frida script: RPC exports, game thread dispatcher, stealth
- **config.py** - Train spots, stat allocation, accounts
- **account_manager.py** - Process detection via frida
- **memory_map.py** - All memory addresses

## Login Flow (CRITICAL - fixed 2026-03-29)

**Root cause of 5min disconnect:** `connectToConnectServer()` creates double TCP connection to CS when game already at gs=2 -> server kicks after 5min.

**Rule:** When gs=2, NEVER call connectToConnectServer(). Game handles CS connection on startup.

**Correct flow:**
1. Game starts at gs=2 (CS already connected)
2. Send F4/03 (server select) - NO connectToConnectServer()
3. Wait cf=1, send F1 login via _sendLoginPacket (on game thread)
4. Wait gs=4, send F3/E8 (enter game)
5. Wait ls=60 (F3/E5 response), send F3/0E (char select)
6. Wait gs=5 (game sets automatically) or force if needed

**Security code:** Stored at byte_104DBD8[0..4] (transform: char+index+1). Read by _sendLoginPacket. Writing via setSecurityCode before login causes version check error. Currently uses value from last manual login.

**All packet-sending functions** must run via runOnGameThread() (queue processed in render hook).

## Anti-hack (APICB.dll)
- Work + Attack: patched to ret (safe)
- Init + Recv: kept alive (challenge-response)
- Stealth: hides Frida from Module32/EnumProcessModules

## Key Addresses (Main.exe base 0x400000)
- gameStatePtr: 0x104E478, loginStatePtr: 0x614E274
- loginFlagPtr: 0x614E0A0, connFlagPtr: 0x614E0B4
- charBasePtr: 0x9704AA4, moveStructPtr: 0x7E2CD0C
- _sendLoginPacket: 0x535660, _sendPacketRaw: 0x43AEA0
- _connectToServer: 0x6F3EA0, sendChat: 0x44D550
- mainRender: 0x8B2F80, pathFind: 0x7491C0, _sendMovePkt: 0x849BE0
- securityCode: 0x104DBD8 (5 bytes)

## Movement
- pathFind: max ~5-15 steps, sendMovePkt target must be within 1 tile of path endpoint
- Far target clamp to avoid rubber-banding (direction table -1,0,1)

## Map Names
- 6 = Stadium (warp command: `/warp Stadium`)
- Train progression: Devias -> Atlans -> Tarkan -> Stadium

## Stats
- Add all free points via /addagi (or configured stat)
- Overflow: if primary stat maxed, dump into STR/AGI/VIT/ENE
- 1s delay between commands for server processing

## Bot Features
- Train, reset, add stats, warp, pickup zen, respawn
- Speed hack: Python thread writes PhysiSpeed/MagicSpeed
- Render throttle: SwapBuffers sleep (10fps OFF, 60fps ON)
- Scene toggle: code cave replaces full 3D render with minimal UI
- Multi-account: 5 simultaneous, SOCKS5 proxy support
- Auto-login: full flow without manual intervention (except first-time security code)
