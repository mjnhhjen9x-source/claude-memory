---
name: Semaphore rate-limit thay vi bypass server chan multi-conn
description: Server chan nhieu connection dong thoi → dieu tiet bang threading.Semaphore, khong patch bypass
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Server co anti-multi-connection (rate-limit, IP throttle, account-per-PC limit) → dieu tiet concurrent bang `threading.Semaphore`, KHONG tim cach patch/bypass.

**Why**: M4VN server chan >N conn dong thoi. Ban dau thu start 20 account cung luc → 50% fail. Thay vi di reverse rate-limit logic de bypass, them Semaphore(8) → 80% success, Semaphore(10) → 65%. Bot on dinh quan trong hon toc do. Bypass → dau tranh voi detection, Semaphore → song hoa binh voi server.

**How to apply**:
```python
_MAX_CONCURRENT = 8
_SESSION_SEM = threading.Semaphore(_MAX_CONCURRENT)

def run_session(...):
    with _SESSION_SEM:
        # connect + login + work
        ...

def set_max_concurrent(n):
    global _SESSION_SEM
    _SESSION_SEM = threading.Semaphore(n)
```

Expose max-concurrent ra UI de user tune theo server. Bat dau conservative (4-8), tang dan khi thay success rate OK. Ket hop voi stagger delay (3s giua moi account start) de khong burst.
