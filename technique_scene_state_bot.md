---
name: Technique - Scene state manipulation thay vi goi function truc tiep
description: Khi bot cho game scene-based (MU Online, Unity scenes...), set scene state vars thay vi goi function truc tiep
type: reference
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Van de
Bot muon chuyen scene (vd char select → in-game). Goi thang `EnterGame()` → packet gui nhung UI khong chuyen scene day du → game "nua trong nua ngoai".

## Nguyen nhan
Game dung scene state machine. Button click thuc su set:
1. Scene ID var (vi du dword_11BA320 = 5)
2. Scene state var (dword_63C72C8 = 61)
3. Init flag (byte_A395D73 = 0 de force re-init)

Sau do game **main thread** tu chay scene update → goi EnterGame tu dung context. Neu Frida thread goi truc tiep tu ngoai, vi pham thread affinity (graphics/audio), scene khong init.

## Cach tim scene state vars
1. Hook function dich (vd EnterGame via sub_8B9990) va capture call stack
2. Caller gan nhat trong user code = scene update function
3. Decompile scene update, tim `if (dword_XXX == N)` branches
4. Cac dword_XXX la scene ID / state / init flag
5. Xref cac bien do de tim cho set chung (scene manager, button handlers)

## Cach ap dung
```js
// THAY VI: EnterGame(name)
// DUNG:
SLOT_ADDR.writeU32(slot);
INIT_FLAG.writeU8(0);
SCENE_STATE.writeU32(61);
SCENE_ID.writeU32(5);
// Game next frame tu run scene → goi EnterGame tu dung context
```

## Khi nao dung ky thuat nay
- Game scene-based (MU Online, Unity, Unreal games with scene manager)
- Goi function truc tiep gay crash/nua trang thai
- Goi function tu Frida thread (khong phai game thread)

## Khi nao KHONG can
- Function idempotent, khong phu thuoc scene (vd SendPacket, SendSell)
- Function chay tren bat ky thread
