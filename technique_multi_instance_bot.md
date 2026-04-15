---
name: Technique - Multi-instance game bot (N game song song)
description: Kien truc worker-per-account, moi account 1 game process + 1 Frida session rieng
type: reference
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Pattern
Moi account = 1 AccountWorker class = 1 game process + 1 Frida session + 1 thread. Khong share state ngoai UI log.

## Code skeleton
```python
class AccountWorker:
    def __init__(self, user, ...):
        self.pid = None
        self.session = None
        self.script = None
        self.thread = None
    
    def start(self):
        self.thread = threading.Thread(target=self._run, daemon=True)
        self.thread.start()
    
    def _run(self):
        while self.running:
            if not self._is_alive():
                self._spawn_game()     # spawn moi
                self._attach_script()  # attach Frida rieng
            # login + loop
```

## Diem can luu y
1. **Stagger spawn** (3s giua moi game) — tranh race trong Windows (mutex creation, single-instance check).
2. **Tung session rieng** — khong share. Frida script load vao tung session.
3. **Watchdog per-worker**: moi worker tu check game cua minh con song khong (scene_id RPC). Neu chet, respawn chi worker do.
4. **UI log tu nhieu thread**: dung `root.after(0, lambda: ...)` de thread-safe.
5. **Multi-instance bypass**: game thuong co single-instance check (mutex/window). Patch function do trong bypass script (vd M4VN sub_7E66E0 → return 1).

## Scaling limits (empirical)
- 5 account: OK, khong van de
- 10+ account: phai check Frida session limit, RAM, CPU, server rate limit
- **Test incremental**: 2 → 5 → 10. Neu 10 fail, do luong:
  - Status cua tung worker (dung o buoc nao)
  - RAM total
  - Game windows con mo khong
  - Frida RPC con response khong (script.exports_sync.ping)

## Server-side gotchas
- Nhieu account cung 1 IP: server co the rate-limit login
- Login too fast: server refuse connection
- Fix: stagger login + retry with backoff
