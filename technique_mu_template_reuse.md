---
name: MU login template reverse 1 lan, reuse N account
description: Extract session UUID tu login template hardcode, reuse cho mot account khac chung HWID
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Game MU (hoac tuong tu) co login packet chua session key / HWID XOR-chained → reverse KEY32 chain 1 lan tu template biet truoc, extract session_enc36, build lai template cho account bat ky voi cung session.

**Why**: M4VN login template 70 bytes:
- `[0..3]`: header C1 60 F1 0E
- `[4..13]`: XOR-encoded username (10B pad)
- `[14..33]`: XOR-encoded password (20B pad)
- `[34..69]`: XOR-encoded session UUID (36B, = HWID)
- Apply KEY32 chain bytes 4..69: `enc[i] = plain[i] XOR enc[i-1] XOR KEY32[i%32]`

Chi can 1 template captured tu game client that (vd batmanlk) → reverse chain → lay session_enc36 (machine-specific, khong phu thuoc user/pass) → build template cho batman1..20 cung session. Khong can reverse tung account, khong can hook game client cho moi lan login.

**How to apply**:
```python
def reverse_key32_chain(buf, start=4, end=70):
    out = bytearray(buf)
    for i in range(end-1, start-1, -1):
        out[i] ^= out[i-1] ^ KEY32[i % 32]
    return bytes(out)

def build_template(user, pw, session_enc36):
    buf = bytearray(70)
    buf[0:4] = b'\xC1\x60\xF1\x0E'
    buf[4:14] = xor_encode(user.encode().ljust(10, b'\x00'))
    buf[14:34] = xor_encode(pw.encode().ljust(20, b'\x00'))
    buf[34:70] = session_enc36  # reuse tu template goc
    return apply_key32_chain(bytes(buf), 4, 70)
```

**Important**: Session UUID thuong gan voi PC (HWID limit). Neu server check account-per-HWID, phai random session moi va hook F4/04 CS packet de send cung UUID. Xem them `mu_login_lessons.md`.

**Reference**: `D:\Code Tools\PC_GAMES\M4VN\template_gen.py` — full implementation
