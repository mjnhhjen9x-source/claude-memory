---
name: frida_gadget_approach
description: Frida Gadget approach - patch APK de hook ARM code tren x86 emulator (LDPlayer), giai quyet van de houdini
type: reference
---

# Frida Gadget Approach

## Van de
- LDPlayer x86 + houdini: Frida Server KHONG hook duoc ARM code
- Frida Server x86 ptrace vao ARM process → FAIL
- Chi doc/ghi memory, khong Interceptor.attach

## Giai phap: Frida Gadget
- Gadget la .so duoc load TRONG game process
- Chay cung context ARM voi game → hook duoc
- Khong can root, khong can frida-server

## Flow tu dong hoa
1. Extract APK (zipfile)
2. Download frida-gadget ARM (github releases)
3. Copy libfrida-gadget.so vao lib/armeabi-v7a/
4. Patch smali: inject System.loadLibrary("frida-gadget") vao MainActivity
5. Them config file (gadget-config.json) de listen mode
6. Repack APK (apktool)
7. Sign APK (apksigner/jarsigner)
8. ADB install
9. ADB launch game
10. Python frida.get_usb_device().attach("gadget") 
11. Run hook scripts → tim offset, pointer chain
12. Luu ket qua

## Tools can
- apktool: decompile/recompile APK
- keytool + jarsigner: sign APK
- frida-gadget .so: ARM version matching frida client version
- Python frida client

## Config gadget (libfrida-gadget.config.so)
```json
{
  "interaction": {
    "type": "listen",
    "address": "0.0.0.0",
    "port": 27042,
    "on_load": "wait"
  }
}
```

## Uu diem
- Hook ARM functions tren x86 emulator ✅
- Interceptor.attach WORK ✅
- NativeFunction call WORK ✅
- Khong can root ✅
- Tu dong hoa toan bo bang Python ✅

## Ap dung cho
- JX1 Mobi (Unity IL2CPP ARM) tren LDPlayer
- Bat ky game ARM-only nao tren x86 emulator
