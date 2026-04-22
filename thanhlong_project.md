---
name: thanhlong_project
description: ThanhLong Unity IL2CPP MMO scouted but not active (game in testing phase, wait for market signals)
type: project
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# ThanhLong — SCOUT ONLY (2026-04-22)

## Status: 🟡 SCOUT COMPLETE, NOT ACTIVE

Game dang giai doan thu nghiem (user noi). Chi scout so qua, **khong build bot ngay**.
Cho cac tin hieu thi truong truoc khi invest build effort.

## Game info
- **Package:** `vn.slgplay.thanhlong`
- **Version:** 1.1.1442
- **Publisher:** SLG Play (VN) — Chinese MMO port VN
- **Engine:** Unity IL2CPP v31 (Unity 2022+)
- **Server:** `http://18.142.240.143` (AWS Singapore)
- **Scripting:** MoonSharp Lua
- **Arch:** arm64-v8a + armeabi-v7a (no x86 → Houdini issue on LDPlayer)

## Workspace
`D:\Code Tools\MOBILE_GAMES\ThanhLong\` — 609 MB
- `README.md` — human-readable scout report
- `recon_summary.json` — machine-readable
- `extracted/` — APK contents (libil2cpp.so, libunity.so, assets)
- `il2cpp_out/dump.cs` — 1M+ lines C# equivalent (MAIN RE asset)
- `il2cpp_out/il2cpp.h` — struct layout
- `il2cpp_out/script.json` — IDA rename script

## RE-ability: 9/10 GREEN

### Green (de)
- IL2CPP metadata dumped → 4032 classes ten ro
- **Built-in auto-fight:** `KTAutoFightManager` + `KTAutoPickUpItemAround` → game CO SAN auto-fight, bot chi cần toggle
- Protocol clean: 48 G2C + 17 C2G packets (Protobuf-style)
- No commercial packer (bangcle/ijiami khong co)
- MoonSharp Lua → business logic modifiable

### Yellow (chu y khi build)
- ARM only → Houdini chan hook tren LDPlayer
- `G2C_Captcha` packet → server co the challenge captcha
- `DoubleClickDetector` → anti-spam (stagger action)
- `ChannelingSkillDetection`, `SameMapTeleportDetection` → server validate hard
- `ClientVerifySIDData` + `ServerVerifySIDData` → session verify
- `DebuggerAction` → anti-debug

### Red: None critical

## Key packets (bot can send — 17 C2G)

Movement: `C2G_AutoPathChangeMap`
Combat: `C2G_UseSkill`, `C2G_ClientRevive`, `C2G_SpriteChangeAction`
Quest/NPC: `C2G_LuaNPCDialog`, `C2G_LuaItemDialog`
Guild: `C2G_AcceptGuildTask`, `C2G_GuildTaskProgress`, `C2G_GuildLevelUpRequest`, `C2G_GuildSkillUpgrade`
Inventory: `C2G_InputEquipAndMaterials`, `C2G_InputItems`
Pet: `C2G_PetAssignPotential`, `C2G_PetChangeName`
Progress: `C2G_RequestLevelUp`, `C2G_RequestBookLevelUp`, `C2G_DistributeSkillPoint`

## Tin hieu cho de activate project

Trước khi invest build bot, cho cac dau hieu:
1. **Server on duoc > 2 tuan** (khong maintenance lon)
2. **Player count tang** (rank board trong game)
3. **Item/gem/gold co gia on dinh** (co market, lien thi thi ne)
4. **Khong co patch rebalance lon** (dau hieu game da settle)
5. **Publisher SLG Play co track record** (games truoc cua ho live bao lau?)

Neu 4/5 signal tich cuc → activate build bot.

## Neu activate: de xuat phase

### Phase 1: Decide deployment
- Phone Android root → full Frida hook (tot nhat)
- LDPlayer → memory-read + packet-inject only (compromise, Houdini limit)

### Phase 2: Deep analysis (1-2 ngay)
1. `grep KTAutoFightManager` trong dump.cs → read state machine
2. `grep KTCrypto` → understand packet encryption (can IDA xem byte-level)
3. `grep G2C_Captcha` → captcha flow
4. Install game + sniff traffic → verify wire protocol

### Phase 3: Build bot (1-2 tuan)
- Frida script toggle `KTAutoFightManager`
- Packet inject via reversed `KTCrypto`
- Captcha solver (OCR) neu can
- UI pattern: bot_ui.py M4VN-style

## Uoc tinh cost build bot (neu activate)

- Scout + deep analysis: 2 ngay
- Reverse KTCrypto: 2-3 ngay
- Build bot core (auto-fight toggle + packet inject): 3-5 ngay
- UI + multi-instance + exe: 2-3 ngay
- Captcha solver: 1-2 ngay (optional)

**Total: 10-15 ngay** neu game proven worth it.

## So sanh voi M4VN (reference)
- M4VN: T3 MU native C++, IDA crucial, 2-3 tuan tu scout toi production bot
- ThanhLong: T3 Unity IL2CPP, Il2CppDumper enough for scout, 1.5-2 tuan neu activate

## Related memories
- `feedback_hard_game_red_flags.md` — red flag criteria (Thanh Long pass)
- `bepinex_approach.md` — Unity IL2CPP hook pattern
- `technique_socks5_antiproxy_bypass.md` — neu server ban IP
- `apk-recon` skill — pipeline da dung de scout game nay

## Decision log
- 2026-04-22: Scout complete. User noi "thu nghiem, xem so qua". Hold, khong build.
