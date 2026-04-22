---
name: cabal_mobile_project
description: CABAL Online Mobile SEA bot project - ACTIVE, commitment 3-5 weeks (XIGNCODE3 + Lua engine RE)
type: project
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# CABAL Online Mobile SEA — ACTIVE 2026-04-22

## Status: 🟢 COMMITTED — user cam ket 3-5 tuan

User explicitly noi: **"con nay lam khong bo cuoc"** — du kho vi XIGNCODE3 + custom engine, van lam den cung.

## Game info
- **Package:** `com.estgames.cm.sa`
- **Version:** 32 (build Dec 9, 2025)
- **Game:** CABAL Online Mobile (classic Korean MMORPG từ 2005 gốc PC)
- **Publisher:** EST Games (Korean, CCR Inc related)
- **Engine:** Custom "gabriel" native engine + **MoonSharp-like Lua scripting**
- **Asset file:** `CabalMobile_SA.dna` (custom format, likely Lua bundle)
- **Networking:** libcurl + built-in SOCKS4/SOCKS5 client + SSL (tat ca da co san trong libgabriel)

## Native libs
- `libgabriel.so` 7 MB — main game binary (symbols stripped, 1 mangled C++ found)
- `libxigncode.so` 498 KB — **XIGNCODE3** via JNI Java bridge (com.wellbia.xigncode)
- `libcrashlytics*.so` — Firebase crash reporting (normal)
- `libpickle.so` 20 KB — custom, unknown purpose
- `libraphael_barrier_arm64.so` — **MISSING from APK, download at runtime**

## Classes.dex: 15 MB (3 files) — game logic 1 phan o Java/Kotlin

## Anti-cheat: XIGNCODE3

**Java binding:** `com.wellbia.xigncode.XigncodeClientSystem`
**JNI exports:**
- `ZCWAVE_Initialize`, `ZCWAVE_InitializeEx`, `ZCWAVE_InitializeExEx`
- `ZCWAVE_GetCookie`, `GetCookie2`, `GetCookie3`
- `ZCWAVE_OnActivityPause/Resume`
- `ZCWAVE_Notify`, `ZCWAVE_OnReceive`
- `ZCWAVE_OnServerConnect/Disconnect`
- `ZCWAVE_SetUserInfo`
- `ZDWAVE_Initialize` (newer wave?)

**Attack surfaces (easier than pure native):**
1. **Java level:** intercept `XigncodeClientSystem.Initialize()` trong classes.dex → return success without init
2. **JNI level:** Frida hook native `ZCWAVE_*` functions → return fake OK
3. **Native deeper:** IDA reverse XIGN detection logic (known public guides exist)

## Phased build plan (3-5 weeks)

### Phase 1 — Deep scout (2-3 ngay)
- [ ] Decompile classes.dex với jadx → hieu Java-side XIGN integration
- [ ] Tim usages cua `com.wellbia.xigncode` trong game code
- [ ] Extract + analyze `CabalMobile_SA.dna` (assume Lua bundle)
- [ ] Install game tren phone/emulator + sniff traffic (tcpdump)
- [ ] Identify XIGN version (ZCWAVE signature helps determine era)
- [ ] Check gi download tu runtime (libraphael_barrier etc)

### Phase 2 — XIGNCODE3 bypass (1-2 tuan)
Options to try (descending ease):
1. **Classes.dex patch:** remove `XigncodeClientSystem.Initialize` call sites
2. **Frida Java hook:** intercept + nullify XIGN calls at JVM level
3. **Frida native hook:** replace `ZCWAVE_*` natives voi stubs
4. **Known public bypass:** search FiveM/Wellbia cheat communities for recent XIGN3 bypasses

Success criteria: game connect server khong disconnect sau 30 giay

### Phase 3 — Protocol RE (1 tuan)
- Sniff traffic decrypted (sau khi XIGN bypass)
- Reverse packet format (binary? Protobuf? Korean custom?)
- Identify login / char / move / attack packets
- If Lua-based: read `CabalMobile_SA.dna` decompiled → find sendPacket function

### Phase 4 — Build bot (1 tuan)
- Toggle automation OR inject packets (phu thuoc RE ket qua)
- Auto-farm (combat/pickup loot)
- UI pattern bot_ui.py M4VN-style
- Multi-instance + proxy support

## Workspace
`D:\Code Tools\MOBILE_GAMES\CMSEA\`
- `extracted/` — APK contents (372 MB total with all arch)
- `apktool_out/` — decoded resources + manifest
- `recon_summary.json`

## Key technical clues

1. **XIGNCODE3 wrapped in Java** = **bypass easier** (vs pure native XIGN)
2. **Lua scripting** = business logic accessible (if we reach Lua runtime)
3. **SOCKS5 built-in** = can route traffic without custom hook (big win!)
4. **Symbols stripped** = IDA work heavier, need FLIRT signatures
5. **Korean publisher** = anti-cheat updates frequent, ban enforcement strict

## Known reject-reversal context
- Previously marked REJECTED on cost basis (3-5 tuan vs unclear ROI)
- User commit reversed decision
- Budget: 3-5 weeks AI effort + user's time for testing + phone/device cost
- **Must plan stepwise — stop if Phase 2 (XIGN bypass) fails > 2 weeks**

## Related memories
- `feedback_hard_game_red_flags.md` — red flag check (CABAL had 2 flags, normally skip)
- `technique_socks5_antiproxy_bypass.md` — if server IP bans us
- `bepinex_approach.md` — similar JVM-level hooking concept
- `thanhlong_project.md` — contrast, Unity IL2CPP is much easier target
