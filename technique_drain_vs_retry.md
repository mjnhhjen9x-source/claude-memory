---
name: Drain-based response detection thay vi retry spam
description: Verify action success bang total bytes trong time window, khong retry mu nhieu lan
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Khi can verify 1 action server-side success (vd /tanthu teleport, login complete, item used) → do total response bytes trong time window, KHONG retry spam action.

**Why**: M4VN `/tanthu` ban dau code 3-retry vi socket timeout → user complain "vi sao ban lai nhap nhieu lan vay". Retry khong giai quyet duoc gi vi server xu ly async, chi lam spam packet va co the trigger anti-flood. Thuc te: teleport that response >14KB, fake/fail response <5KB → dung size lam heuristic.

**How to apply**:
```python
send_plain(gs, build_action(...))
total = 0
deadline = time.time() + 10.0  # time window
while time.time() < deadline:
    chunk = recv_nonblock(gs, 0.5)
    if chunk:
        total += len(chunk)
        if total > THRESHOLD: break  # success
if total < THRESHOLD:
    log("Action failed (only Xb response)")
    return  # khong retry, cycle sau tu reset
```

Rule of thumb:
- Real success response usually 10KB+ (full state update)
- Fake/partial response <5KB (just ACK or error code)
- Pick threshold tu observation (capture 5-10 successful + fail responses)
- Threshold low = false positive (accept fail), threshold high = false negative (reject real). Ban dau nen set conservative (cao).
