---
name: rejected_games
description: Games scouted and rejected - avoid re-scouting same targets
type: reference
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# Rejected Games (Scouted but not worth effort)

Danh sach cac game da scout va tu choi, **de khong re-scout lai** khi user de cap.

## CABAL Online Mobile SEA (2026-04-22)

- **APK:** `CMSEA-32.apk`
- **Package:** `com.estgames.cm.sa`
- **Engine:** Custom "gabriel" native engine (khong phai Unity/Cocos)
- **Anti-cheat:** **XIGNCODE3** commercial (nProtect/Wellbia)
- **Reason:** XIGNCODE3 + custom engine = 3-5 tuan effort, quy mo lon, khong worth ROI
- **Publisher:** EST Games (Korean, CCR Inc related)
- Workspace: `D:\Code Tools\MOBILE_GAMES\CMSEA\` (minimal, chi con recon_summary.json + REJECTED.md)

### Neu user nhac lai CABAL
- Remind: da scout, reject vi XIGNCODE3
- Neu user commit 3-5 tuan + can phone root → reconsider
- Xem `D:\Code Tools\MOBILE_GAMES\CMSEA\REJECTED.md` de refresh context

## JX1 Mobile (trước đây, chi tiet trong jx1_mobi_project.md)
- Blocked: can phone root, LDPlayer Houdini chan hook ARM
- Unity IL2CPP 202K methods, co potential nhung chua enable

## Red flag checklist (de quyet reject nhanh)

Tu `feedback_hard_game_red_flags.md`, neu co **2+ red flag → skip**:
- [ ] Commercial anti-cheat (XIGNCODE3, EAC, BattlEye, NEAC)
- [ ] Kernel-mode protection
- [ ] VMProtect / Themida / packed native lib
- [ ] Cocos 3.x V8 engine (logic in V8 sandbox)
- [ ] Houdini chan (ARM-only + emulator target, khong co phone root)

## Green flag checklist (de quyet go)

Neu ≥3 green flag → start build:
- [ ] Unity IL2CPP dumpable (global-metadata.dat co san)
- [ ] Protocol naming ro (G2C_/C2G_ hoac similar)
- [ ] Co built-in auto-feature (auto-fight, auto-path) → leverage
- [ ] Anti-cheat yeu (user-mode, game-level checks)
- [ ] Server chap nhan multi-account same IP (hoac co proxy duoc)
