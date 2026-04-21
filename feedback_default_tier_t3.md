---
name: feedback_default_tier_t3
description: Default bot tier for user is T3 (game UI + memory read + packet inject) unless explicitly requested to go higher
type: feedback
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
# Default Bot Tier: T3

**Rule:** Khi user yeu cau lam tool/bot cho game, MAC DINH dung tier T3.
Chi nang len T4 hoac T5 khi user EXPLICITLY goi ten.

**Why:** User da quyet dinh sau nhieu thao luan (Apr 2026). User co concerns
cu the voi cac tier khac va T3 fit nhu cau.

## Tier definitions (CHUAN, da thong nhat voi user 2026-04-21)

**Framework chi co T1-T4. T5/T6 KHONG dung (xoa khoi vocabulary).**

```
T1 - Ga:        Pixel read + PostMessage/ADB tap (khong game data)
                Game UI: CO. Khong dung game internal data.

T2 - Kha:       Read-only memory + packet capture (sniff thu dong)
                Game UI: CO. Doc data nhung khong can thiep.

T3 - Manh:      R/W memory + packet inject qua Frida + libc send()
                Game UI: CO (debug duoc, thay bot lam gi)
                Vi du: M4VN bot_ui.py, VNKU auto_train

T4 - Cao thu:   HEADLESS CLIENT (custom Python TCP client)
                Game UI: KHONG (chay 100% bang code)
                Phai RE protocol + crypto + replicate handshake
                Vi du: M4VN headless_v9.py
```

Framework **ap dung chung cho mobile va PC**:
- Mobile T4 = Python TCP + crypto giả lập client (không cần emulator/phone)
- PC T4 = Python TCP + crypto giả lập client (không cần Main.exe)
- Giong nhau ve tech, khac nhau ve protocol game cu the

## User preferences for T3 (why it fits)

- **Van thay game UI** - LDPlayer/emulator chay binh thuong, bot can thiep nen
- **Debug de** - nhin game biet bot dang lam gi
- **Da quen workflow** - packet_sniffer, auto_train, filter_sell...
- **Du cho 1-3 account** - scale nho, chat luong cao
- **Khong can phone that** - dung emulator san co (LDPlayer/MuMu)

## User concerns ve cac tier khac

- **T5 headless**: "khong thay UI game de biet bot lam gi" -> tao apprehension
- **T4 hook function**: cần phone that root -> tien + thoi gian setup
- **T6**: overkill, khong can

## How to apply

### Khi user noi "lam bot cho game X"
1. Mac dinh plan T3 workflow:
   - Game (PC) hoac emulator (mobile) chay
   - Frida attach (read memory + inject packet qua libc send)
   - Python scripts control bot
   - User thay duoc game UI, debug de
2. KHONG auto de xuat T4 / headless client (rebuild cost cao)

### Khi user noi "T4" hoac "headless" explicit
- Chi khi do moi chuyen tier
- Headless = phai RE full protocol + crypto + handshake (cost cao gap 3-5 lan T3)
- Loi the: scale lon (100+ acc), khong can game UI, deploy VPS dễ

### Pattern nhan biet khi can len T4 (headless)
User se noi tu khoa nhu:
- "headless", "khong can UI", "khong mo game"
- "scale 50+ bot", "100 acc"
- "VPS deploy", "chay nen tren server"
- "T4" truc tiep

Neu KHONG co keyword tren, MAC DINH T3.

## Example response templates

### User: "Lam bot auto farm cho game X"
✅ Dung: "OK, dung T3 pattern: Frida hook + packet inject, van thay game UI.
         Pattern bot_ui.py M4VN."

❌ Sai: "Co 2 option: T3 voi emulator, hoac T4 headless..."

### User: "Muon scale 50 bot khong can mo game"
✅ Dung: "Scale + headless = T4. Phai RE full protocol + crypto. Cost 1-2 tuan
         build. Pattern m4vn_progress.md headless_v9 (da xoa, can rebuild)."

### User: "Khong muon thay game UI, chay nen"
✅ Dung: "T4 headless. Phai RE full protocol truoc..."

## Related memories

- `feedback_adobe_air_re_workflow.md` - Adobe AIR RE technique
- `feedback_hard_game_red_flags.md` - red flag check truoc khi RE
- `re_workflow_optimized.md` - quy trinh RE tong quat
- `bot_code_standards.md` - code standards cho bot

## CRITICAL CLARIFICATION (2026-04-21, FINAL)

**Tier = "cong nghe gi DUNG TRONG tool", KHONG phai "muc do automation"**

Tool tier nao cung **van hanh 100% tu dong**. Tier chi tra loi:
**ben trong tool dung ky thuat gi de can thiep game?**

**Framework chi co T1-T4. KHONG dung T5/T6.**
Cac concept "kernel cheat", "zero-day", "internal cheat" = **out of scope**,
khong thuoc framework cua user. Khi noi den toi user, treat as "skip".

Phan biet KEY:
- **T1 vs T2** = co dung game internal data hay khong (T1 chi pixel/click ngoai)
- **T2 vs T3** = read-only vs read+write (T3 inject packet, T2 chi sniff)
- **T3 vs T4** = co/khong game UI
  - T3: game UI van chay, bot can thiep song song qua Frida + libc send()
  - T4: KHONG game, Python tu handshake + login + farm (headless client)

## Toolset hien tai (2026-04-21 status)

| Tier | Status | Note |
|------|--------|------|
| T1-T2 | Available, not used | T3 du roi |
| **T3** | ✅ MATURE | M4VN bot_ui.py + bypass_v5.js + frida-mcp. Default tier. |
| T4 | 🟡 PATTERN ONLY | M4VN headless_v9 da xoa, mu_crypto/mu_decrypt phai rebuild khi can. Pattern lưu tai m4vn_progress.md. Cost: 1-2 tuan rebuild voi game co protocol da known. |

## Headless client = ten technical cua T4

T4 trong framework user = **headless client** (custom TCP client, replicate
game protocol). Cac ten goi khac (community):
- "Custom client", "protocol emulation bot"
- "Network bot / TCP bot"
- "Self-bot" (1 acc) hoac "botnet" (nhieu acc)
- "WPE bot" (cu, ~2000s)
- "Reverse-engineered client"

Workflow build T4:
1. RE network protocol (sniff + decrypt)
2. Implement crypto layer (mu_crypto / aes / xor / custom)
3. Replicate handshake flow (CS -> GS -> login -> in-game)
4. Build action senders (sell/buy/move/attack/quest)
5. Optional: parse server packets (HP/wcoin/state)
