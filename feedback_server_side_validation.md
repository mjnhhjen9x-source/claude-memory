---
name: Server-side validation in online games
description: KHT game validates movement speed + position server-side - speed hack/teleport approaches that fail and move boost that works
type: feedback
---

Game online KHT validate server-side rat chat. Cac approach DA THU va ket qua:

## Speed hack - THAT BAI
- `gettimeofday` hook (libc level): 2x = disconnect, 1.5x = rubberband (giut ve vi tri cu), 1.3x = on dinh nhung cham
- `IR_UpdateTime` (game timer): Khong anh huong vi chi control game logic, khong phai animation
- `CCScheduler::update(float dt)`: x86 float qua FPU ST0, Frida khong intercept duoc
- `KSkillList::CanCast` bypass: Chi bo cooldown check, khong tang toc animation

## Teleport - THAT BAI
- Fake packet 0x4C (src=0,0 dest=far): Server chi cho di toc do binh thuong
- Fake packet 0x4C (src=dest=target): Server biet vi tri that, chi cho di toc do binh thuong
- `ClientGotoPos` tu code: Chi work khi nhan vat DANG DI, khong work khi idle (can state transition)
- `SendCommand(3, destX, destY, 0)`: Return 1 nhung NPC khong move (missing state machine transition)
- Khong co portal/teleport packet trong game protocol

## Move boost (packet modification) - THANH CONG
- Hook `SendPackToServer`, khi thay packet 0x4C (17 bytes), nhan dest offset * multiplier
- 5x boost cho ~4x toc do thuc te
- Server chap nhan vi moi frame chi tang nhe, tich luy theo thoi gian

**Why:** Game online thuong validate position + speed server-side. Khong the "nhay" tuc thi.

**How to apply:**
1. Khong nen thu teleport/speed hack tren game online co server validation
2. Move boost (modify packet dest incrementally) la approach kha thi nhat
3. Truoc khi hack movement, hook packet 0x4C de hieu format va server response
4. Kiem tra: gui packet fake -> server co kick/rubberband khong?
