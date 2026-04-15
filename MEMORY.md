# User Memory

## Projects
- [mu_xua_project.md](mu_xua_project.md) — MU Xua 2003: Frida bot, auto-login, multi-account
- [mu_login_lessons.md](mu_login_lessons.md) — MU PC login: packet order, gs=2 rule
- [mu_fpt_project.md](mu_fpt_project.md) — MU FPT Xua RE: Frida OK, key addresses found
- [kht_project.md](kht_project.md) — KHT Mobile (MemoryScanner + AutoDaTau 2.0)
- [autodatau2_project.md](autodatau2_project.md) — AutoDaTau 2.0 (login, quest, UseItem)
- [vnku_project.md](vnku_project.md) — VNKU: encryption cracked, protocol mapped
- [jx1_mobi_project.md](jx1_mobi_project.md) — JX1 Mobile: Unity IL2CPP, 202K methods, blocked (can dien thoai root)
- [m4vn_project.md](m4vn_project.md) — M4VN MU Online: PhoenixCheat bypass OK, UPX timing, san sang bot
- [m4vn_progress.md](m4vn_progress.md) — M4VN: full auto bot multi-instance (login + tanthu + sell + delete + UI + exe)

## Feedback & Rules
- [bot_code_standards.md](bot_code_standards.md) — Kien truc 5 tang, state machine, logging, error handling
- [feedback_tool_priority.md](feedback_tool_priority.md) — Uu tien tool manh nhat, chuyen nhanh khi bi chan
- [feedback_server_side_validation.md](feedback_server_side_validation.md) — Game online validate server-side → lam bot
- [feedback_game_thread_calls.md](feedback_game_thread_calls.md) — Goi function phai tu game thread
- [feedback_ida_frida_combo.md](feedback_ida_frida_combo.md) — Hook sai → IDA decompile → doc memory
- [feedback_render_toggle.md](feedback_render_toggle.md) — Render: Interceptor.attach + Sleep
- [feedback_dialog_close.md](feedback_dialog_close.md) — Dong dialog bang ADB back key
- [feedback_bot_design.md](feedback_bot_design.md) — Thiet ke 24/7, multi-account, giam do hoa
- [feedback_hard_game_red_flags.md](feedback_hard_game_red_flags.md) — 5 red flag check truoc khi RE game (NeAC, Cocos 3.x V8, houdini, packed .so)
- [feedback_default_tier_t3.md](feedback_default_tier_t3.md) — MAC DINH tier T3 (game UI + memory + packet inject), chi T4/T5 khi user explicit
- [feedback_language_style.md](feedback_language_style.md) — Viet thuan tieng Viet, khong chen tu tieng Anh giua cau (tru thuat ngu ky thuat)

## Techniques & Reference
- [re_workflow_optimized.md](re_workflow_optimized.md) — Quy trinh: danh gia → hack → bot, tool mapping
- [frida_gadget_approach.md](frida_gadget_approach.md) — Frida Gadget: hook ARM tren x86 emulator
- [bepinex_approach.md](bepinex_approach.md) — BepInEx: Unity IL2CPP hook C# bang ten
- [re_workflow_levels.md](re_workflow_levels.md) — IDA 8.3 FLIRT + RTTI + MCP setup
- [feedback_adobe_air_re_workflow.md](feedback_adobe_air_re_workflow.md) — Adobe AIR mobile RE: libair.so custom check, SWF crack, JPEXS AS3 decompile
- [window_monitor_tool.md](window_monitor_tool.md) — WindowMonitor: DWM preview
- [reference_github_backup.md](reference_github_backup.md) — Memory backup: github (PRIVATE)
- [technique_upx_bypass.md](technique_upx_bypass.md) — UPX packed + server hash: doi giai nen xong moi patch
- [technique_scene_state_bot.md](technique_scene_state_bot.md) — Scene-based game: set state vars thay vi call function truc tiep
- [technique_packet_capture_caller.md](technique_packet_capture_caller.md) — Tim function tu packet send + caller RVA
- [technique_multi_instance_bot.md](technique_multi_instance_bot.md) — Multi-account: worker-per-account, spawn stagger, per-session Frida
- [feedback_frida_hook_upx_timing.md](feedback_frida_hook_upx_timing.md) — Interceptor attach SAU UPX decompress (truoc bi overwrite silent fail)

## Trigger Rules
- "MU", "MU Xua" → doc mu_xua_project.md
- "MU FPT" → doc mu_fpt_project.md
- "KHT", "AutoDaTau" → doc kht_project.md
- "VNKU" → doc vnku_project.md
- "JX1", "jx1_mobi" → doc jx1_mobi_project.md
- Game Adobe AIR (libair.so + .swf) → doc feedback_adobe_air_re_workflow.md
- Game ARM + LDPlayer → doc frida_gadget_approach.md
- Game Unity IL2CPP → doc bepinex_approach.md
- Bat dau du an moi → doc re_workflow_optimized.md + bot_code_standards.md
- "M4VN" → doc m4vn_project.md
- Game UPX packed + server hash check → doc technique_upx_bypass.md

## User Preferences
- Giao tiep: Tieng Viet | Code: Python + Frida JS
- Tools: `D:\Code Tools\TOOLS\` | PC Games: `D:\Code Tools\PC_GAMES\` | Mobile: `D:\Code Tools\MOBILE_GAMES\`
- ADB: `D:\Code Tools\TOOLS\Adb\adb.exe` | LDPlayer: Android 7 x86
- Frida tren houdini: chi doc memory, KHONG hook ARM
