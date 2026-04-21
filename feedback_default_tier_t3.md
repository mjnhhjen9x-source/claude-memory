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

## Tier definitions (da thong nhat voi user)

```
T1 - Ga:        Pixel read + PostMessage/ADB tap (khong game data)
T2 - Kha:       Read-only memory / packet capture
T3 - Manh:      R/W state qua memory write hoac packet inject
                (GAME UI VAN HIEN - user thay duoc bot lam gi)
T4 - Cao thu:   Frida Interceptor.attach ARM function hook
                (thuong can ARM phone root vi emulator houdini chan)
T5 - Pro:       Fully custom Python client (headless, khong can game UI)
                (scale 100+ account tren PC)
T6 - God:       Internal cheat, DLL inject, bypass all AC
```

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
   - LDPlayer/emulator chay game
   - Frida attach (read memory + inject packet qua libc send)
   - Python scripts control bot
   - Multi-script pattern (1 script per task)
2. KHONG auto de xuat T5 / custom client
3. KHONG auto de xuat phone that + Frida hook ARM

### Khi user noi "T4" hoac "T5" explicit
- Chi khi do moi chuyen tier
- Neu ho hoi ve option khac, giai thich nhung KHONG chuyen mac dinh

### Pattern nhan biet khi can len T4/T5
User se noi tu khoa nhu:
- "multi-account 10+", "scale", "24/7 farm production"
- "headless", "khong can UI"
- "function hook", "intercept game function"
- "phone that", "root phone"
- "T4" hoac "T5" truc tiep

Neu KHONG co keyword tren, MAC DINH T3.

## Example response templates

### User: "Lam bot auto farm cho game X"
✅ Dung: "OK, dung T3 pattern: LDPlayer + Frida memory read + packet inject,
         script auto_train kieu VNKU. Ban van xem duoc game UI."

❌ Sai: "Co 2 option: T3 voi emulator, hoac T5 custom client Python..."

### User: "Muon scale 50 bot cho game X"
✅ Dung: "Scale 50 bot can T5. Chuyen sang T5 plan: custom Python client..."

### User: "Hook function login cua game"
✅ Dung: "Hook function = T4. Can ARM phone that root vi LDPlayer houdini chan..."

## Related memories

- `feedback_adobe_air_re_workflow.md` - Adobe AIR RE technique
- `feedback_hard_game_red_flags.md` - red flag check truoc khi RE
- `re_workflow_optimized.md` - quy trinh RE tong quat
- `bot_code_standards.md` - code standards cho bot

## CRITICAL CLARIFICATION (2026-04-21)

**Tier = "cong nghe gi DUNG TRONG tool", KHONG phai "muc do automation"**

Tool tier nao cung **van hanh 100% tu dong**. Tier chi tra loi:
**ben trong tool dung ky thuat gi de can thiep game?**

Common confusion (AI session truoc da nham):
- ❌ "T3.5 = nhieu human-in-loop hon T4" — SAI
- ❌ "T4 = scale 100 bot" — SAI (scale la T5)
- ❌ "T3 -> T4 chu yeu la infrastructure" — SAI (la deeper hook tech)

Phan biet KEY giua T3 / T4 / T5:
- **T3 vs T4** = packet inject qua libc (replicate logic) VS Frida hook function thuc
  - T3: doc memory + tu encrypt + goi libc send() (VNKU/M4VN pattern)
  - T4: `Interceptor.attach()` lay control khi game goi function
- **T3 vs T5** = co/khong game UI
  - T3: game UI van chay, bot can thiep song song
  - T5: khong game, Python tu handshake + login + farm (headless_v9 pattern)
- **T4 vs T6** = user-mode vs kernel-mode bypass
  - T4: user-mode, can phone root vi Houdini chan ARM hook tren emulator
  - T6: kernel driver / DSE bypass / zero-day kernel exploit

## Toolset hien tai (2026-04-21 status)

| Tier | Status | Note |
|------|--------|------|
| T1-T2 | Available, not used | T3 du roi, khong xai T1-T2 |
| **T3** | ✅ MATURE | M4VN bot_ui.py + bypass_v5.js + frida-mcp. Default tier. |
| T4 | ❌ NOT YET | Can phone Android root + Frida hook ARM tu do |
| T5 | 🟡 PATTERN ONLY | M4VN headless_v9 da xoa, mu_crypto/mu_decrypt phai rebuild khi can. Pattern lưu tai m4vn_progress.md |
| T6 | ❌ NOT WORTH | Skip — rui ro cao, ROI thap, can budget triệu USD cho zero-day |
