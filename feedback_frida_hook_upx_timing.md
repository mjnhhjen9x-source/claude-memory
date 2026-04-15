---
name: Feedback - Frida Interceptor phai attach SAU khi UPX decompress
description: Hook UPX-packed binary truoc khi decompress → hook bi ghi de, silent fail
type: feedback
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Rule
Khi target la UPX-packed binary (vd M4VN Main.exe), **KHONG** attach Interceptor truc tiep sau spawn+resume. Phai doi UPX unpack xong.

## Why
UPX decompress in-memory SAU khi process start. Entry point la unpacker, no doc compressed section, giai ma, ghi ra memory, nhay toi unpacked code.
Neu Interceptor.attach tai RVA X truoc khi UPX write tai RVA X → Frida inline breakpoint bi overwrite bai UPX → hook silent fail, KHONG co error.

## How to apply
Pattern 1: check mark byte sau UPX done (vd byte cu the tai RVA biet truoc)
```js
var poll = setInterval(function() {
    if (moduleBase.add(UPX_CHECK_RVA).readU8() === KNOWN_BYTE_AFTER_UPX) {
        clearInterval(poll);
        setTimeout(attachHooks, 500);  // them 500ms buffer
    }
}, 50);
```
Pattern 2: attach SAU khi bypass.js da finish patch (thu tu: bypass load → bypass waits UPX → patches → signal → hook_send load)

Pattern 3: Attach Frida SAU khi game da o login screen (process chay duoc vai giay). Simple, luon work.

## Red flag
- Hook khong fire nhung khong co error
- `Interceptor.attach` return OK nhung onEnter khong bao gio chay
- Game bypass chay xong, game chay binh thuong → nghia la UPX done, chuyen sang pattern 3 la an toan nhat

## Related
- technique_upx_bypass.md (giai nen UPX + server hash check)
- M4VN bypass_v5.js co `waitForUPXDecompress()` pattern
