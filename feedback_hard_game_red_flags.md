---
name: feedback_hard_game_red_flags
description: Red flags cho game kho RE - check truoc khi bat dau du an de tranh lang phi effort
type: feedback
---

# Red Flags Cho Game Kho RE

Check nhung dau hieu nay TRUOC khi dau tu vao 1 game RE project. Neu gap 2+ red flag, nen chon game khac hoac chuan bi rat ky.

## Red Flag 1: NetEase NeAC (libanogs.so + libanort.so)
- Integrated vao engine, khong phai lib rieng
- **Delayed-kill sau ~5 phut** du khong hook gi (chi attach Frida = game chet)
- libc integrity watchlist 33 ham: open, read, send, recv, ptrace, mmap, syscall...
- String encryption + control flow flattening + opaque predicates
- inotify watch /data/local/tmp cho frida-server
- Direct syscall bypass libc hook
- **Chi co ADB+OCR bot moi stable 24/7, Frida khong work**

## Red Flag 2: Cocos Creator 3.x + V8
- .jsc khong phai XXTEA/AES, la V8 ScriptCompiler::CachedData bytecode
- KHONG decompile ve JS source duoc (V8 bytecode version-specific)
- Chi hook duoc qua `ScriptEngine::evalString` (nhung thuong bi NeAC monitor)
- So sanh: Cocos Creator 2.x dung XXTEA (de crack), 3.x dung V8 bytecode (kho)

## Red Flag 3: ARM64-only split APK tren LDPlayer x86
- libs load qua houdini translation layer
- **Frida KHONG hook duoc ARM64 native code** (bytes la data, khong phai exec x86)
- libcocos.so / libanogs.so mapped `r--p` only (khong `r-xp`)
- Houdini JIT code o vung `[anon:Mem_0x20000000]` (rwxp), addresses dynamic
- Chi doc duoc memory data, khong Interceptor.attach duoc
- **Giai phap: dung dien thoai ARM that co root**

## Red Flag 4: libcocos.so/engine bi pack
- String `stub_decrypt_elf` va `"Can't decrypt code for %s"`
- `ELF_HOOK_BRIDGE_cocos2d_JNI_OnLoad` - packer hook JNI_OnLoad goc
- Code bi encrypt trong file, decrypt runtime
- IDA static view khong day du
- Phai runtime dump `/proc/pid/mem` de thay code that

## Red Flag 5: Game online idle RPG TQ
- Server validate hau het (HP, damage, gold)
- Value hack classic KHONG work
- Chi con con duong: BOT + packet replay exploit
- Neu co NeAC -> ca 2 deu kho

## Quy Trinh Danh Gia (do 5 phut)

```
1. Unzip APK, kiem tra lib/arm64-v8a/
   - libanogs.so / libanort.so -> NeAC (red flag 1) STOP
   - libcocos.so -> check kich thuoc, cocos version
2. Check base.apk manifest: co tencent/netease SDK?
3. assets/ co .jsc files? Check header:
   - Bytes random -> V8 bytecode (red flag 2)
4. APK co chi arm64-v8a split, khong co x86? (red flag 3 neu chay LDPlayer)
5. Strings trong libcocos.so: co "stub_decrypt_elf"? (red flag 4)
```

## Khi Nao Nen Tiep Tuc

- 0 red flag: GO (game de)
- 1 red flag: GO but chuan bi ky
- 2 red flag: CAN NHAC ky, co the tiec thoi gian
- 3+ red flag: STOP, chon game khac

## Anh Hung Bat Diet (Apr 2026) - case study
Gap ca 5 red flag -> 4min43s attach thi game bi NeAC delayed-kill -> abandoned.
Bai hoc: Giai doan 0 "DANH GIA GAME" trong re_workflow_optimized.md phai
them buoc check NeAC + houdini limitation TRUOC khi dau tu effort.
