---
name: Game function calls require game thread
description: Calling game functions from Frida thread often fails silently - must call from game thread via hook callbacks (Active, SendPackToServer)
type: feedback
---

Goi game function tu Frida thread thuong KHONG hoat dong, mac du function return thanh cong. Phai goi tu game thread.

## Van de
- `SendClientCmdRun(0, 0, mpsX, mpsY)` tu Frida thread: packet gui di nhung client khong animate
- `ClientGotoPos(knpc, destX, destY, mode)` tu Frida thread: return stack cookie, khong move
- `SendCommand(knpc, 3, x, y, 0)` tu Frida thread: return 1 nhung NPC khong move

## Giai phap: Goi tu game thread
Dung `_pendingAction` global + execute trong hook callback:
```javascript
// Set action
_pendingAction = {fn: myFunc, args: [...]};
// Execute in game thread hook
Interceptor.attach(gameThreadFunc, {
    onEnter: function() {
        if (_pendingAction) {
            var a = _pendingAction;
            _pendingAction = null;
            a.fn.apply(null, a.args);
        }
    }
});
```

## Game thread hooks kha dung (KHT)
- `KPlayerAI::Active` — goi moi frame, tot nhat cho game logic
- `SendPackToServer` onEnter — goi khi game gui packet (heartbeat moi ~2s)
- TRANH goi function ben trong `SendPackToServer` onEnter neu function do cung goi `SendPackToServer` (re-entrancy!)

**Why:** Game functions su dung thread-local state, global flags, va state machine can dung thread. Frida thread khong co context nay.

**How to apply:** Moi khi can goi game function tu Frida, LUON queue va execute tu game thread callback. Khong bao gio goi truc tiep tu RPC export.
