---
name: Per-thread crypto state cho multi-instance bot
description: Encryption counter/nonce phai threading.local() khi chay multi-account chung process, khong duoc de global
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Bot multi-account trong cung 1 process → mọi crypto state (counter, nonce, IV, sequence number) PHAI dung `threading.local()`, khong duoc dung global module-level.

**Why**: Project M4VN dinh truc tiep. Bien `_counter` global cho MU packet counter → 3 account chay song song thi counter cua account A bi thread B tang len, server reject packet vi counter sai → hang loat "Login fail" hoac "WinError 10053". Fix bang threading.local cong thi het.

**How to apply**:
- Khi thay state thay doi moi packet (counter, nonce, IV chain), hoi ngay: "neu 2 thread goi cung luc thi co hong khong?"
- Pattern:
```python
_tls = threading.local()
def _get_counter():
    if not hasattr(_tls, 'counter'): _tls.counter = 0
    return _tls.counter
def _inc_counter():
    c = _get_counter()
    _tls.counter = (c + 1) & 0xFF
    return c
```
- Moi socket/worker phai co thread rieng, khong share socket giua cac thread.
- Dau hieu lo: multi-account chay 1 acc thi OK, nhieu acc thi random fail → 99% la race condition crypto state.
