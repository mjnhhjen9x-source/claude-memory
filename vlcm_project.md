---
name: vlcm_project
description: Vo Lam Chi Mong PC - Flash game RE project, encrypted TCP packets, CDN resource loading
type: project
---

# Vo Lam Chi Mong PC (VLCM)

## Project Location
`D:\Code Tools\VLCM_PC\`
Game client: `D:\MongChiTonTPL\`

## Game Architecture
- **Launcher**: MongChiTonLauncher.exe (.NET WPF, PE64)
- **Game**: engine/Flash.exe (Adobe Flash Player projector, PE32 x86, 8MB)
- **Type**: Flash/SWF game running inside Flash Player
- **CDN**: cdn-mct.tepaylink.vn (HTTPS)
- **Resource path**: /eflashdir0918/
- **Localhost API**: 127.0.0.1:31866 (launcher<->game communication)

## Network Protocol
- TCP socket (via WS2_32 send/recv)
- Packet format: `00 7F 00 [size] [encrypted_data]`
- Header 3 bytes fixed: 00 7F 00
- Byte 3: packet size (0x0E=14, 0x10=16, 0x12=18)
- All data ENCRYPTED - no plaintext visible
- Periodic packets every ~500ms (keepalive/movement)

## Frida
- Attach to Flash.exe OK (PID varies)
- Modules: WS2_32, d3d9, wininet, DSOUND
- No anti-cheat DLL detected

## SWF Analysis
- Flash.exe contains 2 embedded SWFs (40KB + 4KB) = Flash Player internal only
- Game SWF loads from CDN at startup (exact URL unknown - 404 on common paths)
- Need to capture SWF URL during fresh game launch

## HTTP Resources (captured during startup)
- Config: config.tsze, configdata.tsze, lan.tsze, npc.tsze, rankaward.tsze
- Skin/UI: skin.tse, ui.tse, loginui.tse, sceneRes.tse (custom encrypted format)
- SWFs: ~100+ resource SWFs (UI effects, animations) - NOT game logic
- Game logic SWF NOT found via HTTP - likely embedded or loaded via encrypted TCP

## Encryption CRACKED (2026-03-30)
**CodeMixer.decodeBytes** algorithm:
- XOR key: a=121, block size: b=39, skip: c=297
- For each block: XOR 39 bytes with 121, skip 297 bytes, repeat
- .tse files = encrypted SWF → decrypt → valid CWS/FWS

## SWFs in Memory (during gameplay)
- FWS v46 15.2MB @ 0xF160000 (possibly all game resources)
- **FWS v15 6.8MB @ 0xD620000** → DECOMPILED! 1733 AS files (mostly UI)
- FWS v10 3.7MB @ 0x175F0000 (module)
- FWS v9 2.3MB @ 0x18450000 (module)
- Total: 139 SWFs > 100KB in memory

## Decompiled Successfully
- game_logic_v15.swf: 6.8MB → 1733 ActionScript files (UI components)
- skin.tse: decrypted → 63 scripts (UI skin)
- TGameLoader: found game loading flow + encryption

## PROTOCOL CRACKED (2026-03-30)
From TSocket.as decompile:
- Packet: [00 7F] [size:u16] [msgId:u32] [seq:u16] [data...]
- Encryption key: [174, 191, 86, 120, 171, 205, 239, 241]
- Rolling key: mutates with each packet (chain cipher)
- encrypt/decrypt_bytes: XOR + chain with key rotation
- Python implementation: D:\Code Tools\VLCM_PC\protocol.py

## Game Core Decompiled (4334 AS files!)
- game_core_v46.swf: 15MB → 4334 ActionScript files
- TSocket.as: network protocol + encryption
- NetWorkManager.as: connection management
- *_MsgSenderProxy.as: packet builders for each feature
- *_MsgReceivedProxy.as: packet parsers for each feature

## Key Packet IDs (from MsgSenderProxy decompile)
- 10083: Attack player (PK) - params: targetId, type, skillId, timestamp
- 10283: Attack ground (AOE) - params: skillId, x, y, timestamp
- 20075: Respawn - params: type, timestamp
- 10901: Toggle PK mode
- 50581: Save AFK settings (auto fight config)
- 50583: Start/stop AFK (param: 1=start, 0=stop)
- 50587: Toggle x2 exp timer
- 52011: Speed up hidden weapon

## Hack Potential
- Spam attack (10083) with modified timing
- Auto AFK via packet (50583)
- Auto respawn (20075)
- Auto quest (Task_MsgSenderProxy)
- Python bot via direct socket (Level 4 RE!)

## Hack Attempts
- Replay packet: ❌ Rolling key - replayed packet rejected
- Inject via Frida send(): ❌ Need correct key state
- Flash VM injection: ❌ Too complex (no ExternalInterface access)
- **Conclusion**: Need Python bot with own socket connection to test hacks

## Next Steps (Priority)
1. **Build Python socket bot** - connect to server, implement encryption, login
2. Test hack packets: 53203 (free income), 11193 (negative money)
3. Auto AFK via packet 50583
4. Auto quest via Task_MsgSenderProxy

## Tools
- JPEXS FFDec: D:\Code Tools\JPEXS_FFDec\ffdec-cli.exe
- Frida: attach to Flash.exe
