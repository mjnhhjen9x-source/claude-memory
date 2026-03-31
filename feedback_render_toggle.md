---
name: Render toggle approach for PC games
description: How to reduce CPU by throttling render in OpenGL/DX games - use Interceptor.attach+Sleep on SwapBuffers, NOT replace or byte-patch
type: feedback
---

Giam CPU game PC bang render throttle: hook SwapBuffers + Sleep.

**Why:** Da thu 3 cach va fail:
1. Interceptor.replace + NativeFunction cung address = infinite recursion, crash Frida
2. Byte patch (xor eax,eax; ret) render function = skip vsync wait, CPU TANG thay vi giam
3. sub_8B69F0 (gameplay render MU) co game logic mixed = khong the skip toan bo

**How to apply:**
- Dung `Interceptor.attach` tren `SwapBuffers` (GDI32) hoac `eglSwapBuffers` (mobile)
- Khi render OFF: `Sleep(200)` trong onEnter = ~5fps, giam CPU ~80%
- Khi render ON: khong Sleep, render binh thuong
- KHONG dung Interceptor.replace (conflict NativeFunction)
- KHONG dung byte patch ma khong them Sleep (mat vsync = CPU spin)
- Render function thuong co game logic mixed -> chi throttle, khong skip
