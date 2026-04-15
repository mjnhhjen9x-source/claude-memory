---
name: M4VN Progress - Full Auto Bot
description: M4VN auto bot full pipeline: login + tanthu + sell + delete + multi-instance
type: project
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Trang thai: HOAN THANH - Full auto bot multi-instance

## Binary version
Dung `Main_unpacked_v2.exe` (IDA db). ImageBase 0x400000.
Cac dia chi o memory cu (0x8B2FC0 CreateChar etc) la **cua version cu, khong dung nua**.

## Dia chi function (v2 unpacked binary)
| Function | VA | RVA | Args | Packet |
|---|---|---|---|---|
| CreateChar | 0x8B18D0 | 0x4B18D0 | (name, class_hi, class_lo) | F3 0C |
| DeleteChar | 0x8B3000 | 0x4B3000 | (name, data20_zeros_ptr) | F3 0F |
| Logout | 0x8BA710 | 0x4BA710 | (byte) | F1 0D |
| RequestCharList | 0x8B0FD0 | 0x4B0FD0 | () | F3 0D |
| EnterGame (ham)| 0x8B9990 | 0x4B9990 | (char* name) | F3 0E |
| SelectServer | 0x8C0840 | 0x4C0840 | (uint16 server_id) | F4 03 |
| Login | 0x8B9F60 | 0x4B9F60 | (user, pass, flag_byte) | F1 0E |
| SendSell | 0x7FD5E0 | 0x3FD5E0 | (item_code, tab_id) stdcall | |
| NpcClick | 0x8C1CA0 | 0x4C1CA0 | () | |
| Interact | 0x8ADD30 | 0x4ADD30 | (a1=122, a2=0/3/4) | |
| chatFunc | 0x8ABA30 | 0x4ABA30 | (6 uint32 = wstring) | 0D 00 |
| GetCharBySlot | 0x50D0D0 | 0x10D0D0 | (slot) → char struct ptr; name @ +118 | |
| SendPacket (low) | 0x8AD160 | 0x4AD160 | (src, size, enc, a4) | |

## State vars (global)
- dword_11BA330 = slot index (selected char)
- dword_11BA320 = scene ID (2=login, 4=char select, 5=in-game)
- dword_63C72C8 = scene state (20=list request, 61=in-game enter, 56=delete, ...)
- byte_A395D73 = scene init flag (reset = 0 to force re-init)

## Flow chinh (da giai quyet van de scene transition)
**EnterGame KHONG goi sub_8B9990 truc tiep**. Chi set state:
```
dword_11BA330 = slot
byte_A395D73 = 0      // force init
dword_63C72C8 = 61    // in-game state
dword_11BA320 = 5     // in-game scene
```
Game thread next frame tu chay sub_A577C0 → goi sub_8B9990. Goi truc tiep bi conflict thread.

**Multi-instance bypass**: bypass_v5.js patch sub_7E66E0 (RVA 0x3E66E0) → return 1. Cho phep spawn nhieu game cung luc.

## Cac buoc test da qua
1. ✅ CreateChar - F3 0C packet (15 byte)
2. ✅ EnterGame passive scene switch 
3. ✅ /tanthu + sell (chatFunc + NpcClick + Interact + SendSell)
4. ✅ Logout - F1 0D packet
5. ✅ DeleteChar - F3 0F packet (34 byte, data20 = zeros, khong can password)
6. ✅ Login - SelectServer + Login with user/pass
7. ✅ Multi-instance: N account chay song song, stagger 3s

## File chinh (HIEN TAI)
- `D:\Code Tools\PC_GAMES\M4VN\bot_ui.py` - UI multi-instance (PRIMARY)
- `D:\Code Tools\PC_GAMES\M4VN\bot_loop.js` - Frida RPC (all functions)
- `D:\Code Tools\PC_GAMES\M4VN\bypass_v5.js` - bypass PhoenixCheat (da them multi-instance patch)
- `D:\Code Tools\PC_GAMES\M4VN\dist\M4VN_Bot.exe` - PyInstaller bundle

## Tool features
- Load accounts tu file .txt (hoac nhap vao textarea)
- Password chung tat ca account
- Server ID configurable (default 4)
- Class selector (DW/DK/FE/MG/DL)
- Prefix ten bot (default "Bot") - safety: KHONG xoa char khong bat dau bang prefix
- Vong/login = 0 = vo han
- Stagger 3s giua moi spawn
- Auto-recovery: game crash → detect → respawn + relogin + resume
- Slot-full: tu xoa cac char co prefix → create lai

## Luu y khi dev tiep
- Prefix MUST NOT match user's main char name (else se bi xoa)
- Server ID = 4 la "Sub-19" cua 1 user test, co the khac voi server khac
- Data20 trong DeleteChar = all zeros duoc (server nay khong require password)
