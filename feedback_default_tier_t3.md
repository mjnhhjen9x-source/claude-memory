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
