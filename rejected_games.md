---
name: rejected_games
description: Games scouted and rejected - avoid re-scouting same targets
type: reference
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# Rejected Games (Scouted but not worth effort)

Danh sach cac game da scout va tu choi, **de khong re-scout lai** khi user de cap.

## ~~CABAL Online Mobile SEA~~ — REACTIVATED 2026-04-22

Reject bi dao: user confirm commit 3-5 tuan. Xem `cabal_mobile_project.md` de biet status hien tai.

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
