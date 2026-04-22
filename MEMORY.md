# User Memory

## About User
- [user_role.md](user_role.md) — Non-coder game farmer; AI viet code, user direct vision

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
- [m4vn_server_ids.md](m4vn_server_ids.md) — M4VN server ID: Sub-1=0, Sub-2=1, Sub-19=4 (khong phai so UI)
- [vttt_project.md](vttt_project.md) — VTTT (MU Vu Tru Than Thoai): bypass CMPlay AC OK (kill function patch), opcode capture in progress
- [thanhlong_project.md](thanhlong_project.md) — ThanhLong (Unity IL2CPP MMO VN): scouted 9/10 green, HOLD — game testing phase
- [rejected_games.md](rejected_games.md) — Games scouted & rejected (CABAL Mobile XIGNCODE3, JX1 blocked)

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
- [feedback_ui_design.md](feedback_ui_design.md) — UI Tkinter: fixed size, grid 4 cot aligned, tab Notebook, auto-save

## Techniques & Reference
- [re_workflow_optimized.md](re_workflow_optimized.md) — Quy trinh: danh gia → hack → bot, tool mapping
- [frida_gadget_approach.md](frida_gadget_approach.md) — Frida Gadget: hook ARM tren x86 emulator
- [bepinex_approach.md](bepinex_approach.md) — BepInEx: Unity IL2CPP hook C# bang ten
- [re_workflow_levels.md](re_workflow_levels.md) — IDA 8.3 FLIRT + RTTI + MCP setup
- [feedback_adobe_air_re_workflow.md](feedback_adobe_air_re_workflow.md) — Adobe AIR mobile RE: libair.so custom check, SWF crack, JPEXS AS3 decompile
- [window_monitor_tool.md](window_monitor_tool.md) — WindowMonitor: DWM preview
- [frida-mcp-setup](../../.claude/skills/frida-mcp-setup/SKILL.md) — Frida MCP: install, config, device connection, troubleshooting
- [frida-mcp-usage](../../.claude/skills/frida-mcp-usage/SKILL.md) — Frida MCP: tool selection (eval_js vs load_script vs run_recipe), recipe library
- [reference_github_backup.md](reference_github_backup.md) — Memory backup: github (PRIVATE)
- [reference_frida_mcp_backup.md](reference_frida_mcp_backup.md) — frida-mcp backup: github mjnhhjen9x-source/frida-mcp (PRIVATE)
- [technique_upx_bypass.md](technique_upx_bypass.md) — UPX packed + server hash: doi giai nen xong moi patch
- [technique_scene_state_bot.md](technique_scene_state_bot.md) — Scene-based game: set state vars thay vi call function truc tiep
- [technique_packet_capture_caller.md](technique_packet_capture_caller.md) — Tim function tu packet send + caller RVA
- [technique_multi_instance_bot.md](technique_multi_instance_bot.md) — Multi-account: worker-per-account, spawn stagger, per-session Frida
- [feedback_frida_hook_upx_timing.md](feedback_frida_hook_upx_timing.md) — Interceptor attach SAU UPX decompress (truoc bi overwrite silent fail)
- [technique_opengl_vram_optimize.md](technique_opengl_vram_optimize.md) — VRAM toi uu game OpenGL: noop glTexImage2D + glDeleteTextures tu render thread (wglSwapBuffers)
- [technique_threading_local_crypto.md](technique_threading_local_crypto.md) — Crypto state (counter/nonce) phai threading.local() khi multi-account 1 process
- [technique_pyinstaller_frozen_path.md](technique_pyinstaller_frozen_path.md) — PyInstaller --onefile: dung sys.executable khong dung __file__ cho config persist
- [technique_semaphore_rate_limit.md](technique_semaphore_rate_limit.md) — Server chan multi-conn → Semaphore rate-limit, khong bypass
- [technique_drain_vs_retry.md](technique_drain_vs_retry.md) — Verify action success bang total response bytes trong time window, khong retry spam
- [technique_bitpack_byteorder.md](technique_bitpack_byteorder.md) — Custom cipher bit-packed: tach 8-bit bytes roi ghep LE, khong doc full value
- [technique_mu_template_reuse.md](technique_mu_template_reuse.md) — MU login: reverse KEY32 chain 1 lan, extract session_enc36, reuse N account
- [technique_game_update_offset_patch.md](technique_game_update_offset_patch.md) — Game update: regions dời khác nhau (game code/anti-cheat/data globals), scan bằng wildcard pattern cho imm32
- [technique_upx_memory_dump_ida.md](technique_upx_memory_dump_ida.md) — Dump UPX binary qua Frida + fix PE header (UPX0 RAW_SZ=VSZ) cho IDA analyze
- [technique_socks5_antiproxy_bypass.md](technique_socks5_antiproxy_bypass.md) — Multi-layer IP ban + client anti-proxy: SOCKS5 tunnel + getpeername spoof

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
- "ThanhLong", "Thanh Long" → doc thanhlong_project.md (currently HOLD)
- Game UPX packed + server hash check → doc technique_upx_bypass.md
- "frida-mcp", "frida mcp" → doc frida-mcp-usage skill, check frida-mcp-setup if issues

## User Preferences
- Giao tiep: Tieng Viet | Code: Python + Frida JS
- Tools: `D:\Code Tools\TOOLS\` | PC Games: `D:\Code Tools\PC_GAMES\` | Mobile: `D:\Code Tools\MOBILE_GAMES\`
- ADB: `D:\Code Tools\TOOLS\Adb\adb.exe` | LDPlayer: Android 7 x86
- Frida tren houdini: chi doc memory, KHONG hook ARM
