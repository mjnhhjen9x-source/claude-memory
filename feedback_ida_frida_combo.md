---
name: IDA + Frida combo pattern
description: When Frida alone fails (wrong offsets, timing-dependent hooks), use IDA to decompile and understand struct layout first, then read memory directly
type: feedback
---

Khi Frida hook/computed formula cho ket qua sai hoac khong on dinh -> dung IDA decompile function lien quan de hieu struct layout, roi doc memory truc tiep thay vi hook.

**Why:** KHTD UseItem can grid position (col,row). Chi dung Frida: hook KItemList::Add phai inject truoc khi vao game, computed formula sai gay dung nham item. Sau khi dung IDA decompile KInventory::PlaceItem va KInventory::GetItem, biet duoc grid 5x36, doc truc tiep tu memory -> luon dung, bat ky luc nao.

**How to apply:** Khi gap van de Frida hook/scan khong du:
1. Mo IDA (Ctrl+Alt+M start MCP) -> decompile function lien quan
2. Tim exported globals (Player, Item, etc.) thay vi tinh offset tu NpcArray
3. Doc struct truc tiep tu memory thay vi hook function
4. Pattern: IDA hieu cau truc -> Frida doc runtime

## IDA workflow hieu qua (da verified):
1. `list_strings_filter` / `list_globals_filter` — tim function/global theo ten
2. `get_function_by_name` — lay address
3. `decompile_function` — doc C pseudocode, hieu logic + params
4. Tu C code -> biet coordinate system, state checks, call chain
5. Frida goi function voi dung params da hieu tu IDA

## Case study: Movement system (KHT)
- IDA decompile `SendRunCommand` → biet no goi `ClientGotoPos(npc, 32*tileX, 32*tileY, 0)`
- IDA decompile `ClientGotoPos` → biet state checks (field54, field831, Player+19152 cooldown)
- IDA tim `KAutoFindPathManager` → biet game co auto-pathfinding built-in
- IDA doc `SendClientCmds` strings → biet tat ca packet types game ho tro
- Ket qua: hieu toan bo movement system tu static analysis, khong can guess
