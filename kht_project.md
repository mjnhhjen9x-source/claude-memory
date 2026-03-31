---
name: KHT Project
description: Kiem Hiep Tinh Duyen Mobile (com.growx.kht1) - MemoryScanner + AutoDaTau 2.0, Cocos2d-x + Lua, full RE project
type: project
---

## Game Info
- Game: Kiem Hiep Tinh Duyen Mobile (`com.growx.kht1`), engine Cocos2d-x + Lua
- ADB serial: `emulator-5554` (fallback, TCP `127.0.0.1:5555` offline tren LDPlayer)

## Sub-projects
- **MemoryScanner**: `D:\Code Tools\MemoryScanner\` — context chi tiet: `PROJECT_CONTEXT.md`
- **AutoDaTau 2.0**: `D:\Code Tools\AutoDaTau2.0\` — chi tiet: xem `autodatau2_project.md`

## MemoryScanner - DA HOAN THANH
- Pointer chain toa do, map ID (verified qua restart)
- Doc NPC names + danh sach 160+ NPC/player
- Auto-dialog NPC (goi OnDialogNpc khong can tap)
- Doc tui do + trang bi (item names, category, durability, sprite, qty +0x354)
- Giai ma legacy Vietnamese encoding (30+ bytes confirmed)
- Doc noi dung Gio ca (fish names + qty qua SyncScriptAction protocol 0x71)
- Game networking: KProtocolProcess, CGameClient::SendMsg/ProcessDataStream
- Lua VM idle trong runtime (Lua 4.x, chi dung luc init)
- Auto UseItem bat ky item (packet build tu item ID tai +0x4E0)
- Auto Move (StartAutoPath, pathfinding tu dong)
- Mining nodes = KObj (kind=10, stride 0x5AC)
- Auto Mining (packet 0x4D interact + 0x6D mine action)
- Auto chon dialog option (selectMethod qua handler global)

## AutoDaTau 2.0 - CAN LAM TIEP
- Nhan/huy quest packet, nop quest, cau ca, mua shop, hoi sinh
- Chi tiet: xem `autodatau2_project.md`

## Coordinate Systems
- **Tile coords**: (216, 193) — dung trong game UI va pathfinding
- **Internal coords**: tile * 32 — dung trong ClientGotoPos, SendCommand
- **MPS coords**: tile * 256 (X), tile * 512 (Y) — dung trong 0x4C packet va GetMpsPos
- Convert: `KSubWorld::NewMap2Mps` (chinh xac), hoac tile*256/512 (xap xi)

## Key Functions (da xac nhan qua IDA)
- `ClientGotoPos` (0x6754b0): Move NPC + send packet, nhung can dung state
- `SendClientCmdRun` (0x6b6230): Build+send packet 0x4C, 4 args (srcX,srcY,destX,destY)
- `SendClientCmdWalk` (0xf6e6c area): Tuong tu nhung walk
- `KAutoFindPathManager::StartNewFinding` (0x802780): A* pathfinding, can obstacle data loaded
- `KAutoFindPathManager::SendRunCommand` (0x802aa0): Clear cooldown + ClientGotoPos
- `g_cAutoFindPathManager` (0x14b2334): Global pathfinding manager
- `g_SubWorldSet` (0x145f32c): Frame counter (~53000+), dung cho cooldown check
- `Player+19152`: Movement cooldown (set 0 truoc khi goi ClientGotoPos)
- `Player+19148`: Auto-run flag (0=walk path, non-0=run path trong ClientGotoPos)
- `KNpc field54`: NPC action state (0=idle, 2=walk, 3=run, 5=dead, 9-19=skill)
- `KNpc field831`: NPC movement mode (2=walk, 3=run)

## Hack Results
- **Speed hack**: THAT BAI — server validate (xem feedback_server_side_validation.md)
- **Teleport**: THAT BAI — server validate position
- **Move boost (packet mod)**: THANH CONG — 5x boost = ~4x actual speed
- **Shop items reading**: THANH CONG — g_cStoreItem, 30 slots, full OPT

## Feedback
- Dong dialog game bang ADB back key, KHONG dung vtable hack (gay ket UI)
- Chi tiet: xem `feedback_dialog_close.md`
