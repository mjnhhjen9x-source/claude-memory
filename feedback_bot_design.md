---
name: Bot design principles
description: Moi feature/hack phai thiet ke cho 24/7 va multi-account - giam tai do hoa, toi uu resource
type: feedback
---

Moi thu lam ra phai tinh toi bot chay 24/24 va multi-account.

**Why:** User chay nhieu account cung luc, can toi uu resource (CPU/GPU/RAM) de 1 may chay duoc nhieu instance.

**How to apply:**
- Khi lam bot, luon hoi/tinh: "chay 24h co stable khong? chay 10 account co nang khong?"
- Giam tai do hoa game qua Frida hook (cocos2d-x):
  + Tat particle effects (lua, khoi, skill)
  + Tat animation (force sprite dung yen)
  + Giam FPS (Director::setAnimationInterval → 5-10 FPS)
  + Tat texture loading (sprite thanh hinh trang)
  + Tat am thanh (hook audio engine skip)
- Emulator: giam resolution xuong thap nhat, headless mode neu co
- Code bot: handle reconnect, error recovery, memory leak prevention
- Level 4 (tuong lai): bot Python thuan khong can emulator/render → nhe nhat
