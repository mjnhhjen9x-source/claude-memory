---
name: feedback-dialog-close
description: Fix for game UI stuck after closing dialogs via Frida vtable hack
type: feedback
---

Khi dong dialog game qua Frida, KHONG dung vtable hack (dialogInstance.readPointer().add(0xf8).readPointer() -> call close). Cach nay dong visual nhung de lai touch-blocking layer, lam ket UI.

**Why:** Da gap loi nhieu lan — sau khi brute-force vi tri item trong tui do, dong dialog sai bang vtable hack lam game khong tuong tac duoc (khong mo tui, khong tap duoc). User phai restart game.

**How to apply:** Khi can dong dialog game (KuiDialog) trong Frida scripts:
- Dung ADB back key: `adb shell input keyevent 4` — game xu ly tu nhien, clean up day du
- Sau khi select option (selectAndClose), cung nen gui ADB back key de dam bao dialog dong sach
- Cho 0.8s sau moi lan back de UI co thoi gian clean up
