---
name: icarus_project
description: Ky Nguyen Icarus (Legend of Icarus) mobile game RE - MU Online engine port, SDL3, ARM32, SSL encrypted
type: project
---

# Ky Nguyen Icarus (Legend of Icarus)

## Project Location
`D:\Code Tools\MOBILE_GAMES\KyNguyenIcarus\`
IDA database: `D:\Code Tools\IDA_DATABASES\KyNguyenIcarus\libmain.so`

## Game Info
- Package: com.hnxgames.legend
- Developer: DEV ICARUS / HNXGames
- Engine: **SDL3 + MU Online engine port**
- Platform: Android 5.0+ (armeabi-v7a ARM32)
- Size: 1.5GB (main APK 100MB + data 1.4GB)
- Network: **SSL/TLS + custom NET_* layer + libcurl**

## Key Finding: MU Online Engine!
Game uses MU Online .bmd format, same strings ("Version dismatch", "Player.bmd").
Experience from MU Xua/MU FPT directly applicable!

## IDA Analysis (libmain.so, 15MB)
42127 strings found. Key strings:
- SPEEDCHECK (anti-cheat speed check)
- AttackSpeed, WalkSpeed, ActionSpeed (speed values)
- AutoSelectAttackPlayer (auto target)
- LOGIN %s %s (login format)
- NET_WriteToStreamSocket, NET_ReadFromStreamSocket (network)
- Java_com_hnxgames_legend_MainActivity_nativeOnLogin* (JNI login)

## Key Functions (IDA)
- sub_87F80C: AttackSpeed handler (references AttackSpeed string 6 times)
- JNI exports: nativeOnLoginWithAccount, nativeOnLoginSuccessful, nativeOnLoginFail

## Architecture
- libmain.so: ALL game logic (15MB, C++ native, stripped)
- libSDL3.so: SDL3 engine (1.5MB)
- 3 DEX files: Java layer (SDK, billing, ads only)
- SSL/TLS for all network communication
- libcurl embedded for HTTP requests

## Speed Hack - FAILED
- Memory edit speed display value → no gameplay effect (server-side)
- sub_87F80C = XML item parser, not runtime speed handler
- AttackSpeed/WalkSpeed are item attributes, server validates

## Guest Mode Reset - FAILED
- Game lưu device fingerprint sâu (không phải Android ID, Advertising ID, hay SharedPreferences)
- pm clear + reset all IDs → vẫn không tạo guest mới
- Cần LDPlayer instance mới (IMEI/serial/MAC khác)
- JWT login: device_{hash}@sdk.local format

## Kết luận
- Server validate mọi thứ (giống MU FPT)
- Guest mode lock device fingerprint hardware-level
- Bot khả thi nhưng cần đăng ký account thật
