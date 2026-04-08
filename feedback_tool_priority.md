---
name: feedback_tool_priority
description: Uu tien cong cu manh nhat truoc, linh hoat chuyen khi bi chan - khong bam 1 tool
type: feedback
---

# Uu tien cong cu

## Nguyen tac: Dung tool MANH NHAT truoc, chuyen khi bi chan

**Why:** Mat nhieu gio voi Frida Server tren houdini trong khi co the dung Frida Gadget/BepInEx tu dau.

**How to apply:** Check kha nang truoc khi bat dau, khong bam vao 1 tool.

## Thu tu uu tien theo tung viec

### Hook game function
```
1. Frida Server (neu hook duoc)        → nhanh, flexible, REPL
2. Frida Gadget (neu Server fail)      → patch APK, chay trong process
3. BepInEx (Unity IL2CPP)              → hook C# method bang ten
4. Memory read + ADB input (cuoi cung) → khi khong hook duoc gi
```

### Static analysis
```
1. Il2CppDumper (Unity IL2CPP)  → 202K method names mien phi
2. dnSpy (Unity Mono / .NET)    → full C# source
3. JPEXS (Flash/SWF)            → full ActionScript source
4. IDA Pro (native code)        → khi khong co tool chuyen biet
5. r2pipe                       → scan nhanh string/xref
```

### Bot level
```
1. L3 Frida hook (goi function game)   → manh nhat, thay ket qua
2. BepInEx plugin (Unity)              → stable, C# method name
3. L4 Python socket (standalone)       → doc lap, khong can game
4. L2.5 Memory + ADB                   → don gian, moi game deu work
```

### Debug
```
1. Frida REPL (realtime test)   → nhanh nhat
2. Log file (per account)       → debug 24/7 bot
3. IDA decompile (xem lai code) → khi hook/formula sai
4. Packet capture (Frida hook send/recv) → debug network
```

## Quy tac chuyen tool

| Tinh huong | Chuyen tu | Sang |
|-----------|-----------|------|
| Frida Server fail (houdini) | Frida Server | Frida Gadget |
| Hook ARM fail tren x86 | Frida hook | Memory read + ADB |
| Offset thay doi moi restart | Memory scan | Pointer chain hoac BepInEx |
| Hack fail (server validate) | Hack | Bot |
| IDA offset sai runtime | IDA static | Frida REPL verify |

## KHONG lam

- KHONG mat nhieu gio fix 1 tool khi co alternative
- KHONG bam Frida Server khi biet houdini block
- KHONG tim offset bang memory scan khi co the hook function
- KHONG hack stats khi biet server validate → chuyen bot ngay
