# User Memory

## About User
- [user_role.md](user_role.md) — Game farmer 5-6 năm, YouTube "Thanh Cay Thue", IT hardware background, không code, dùng AI ~3 tháng — User=scout/judgment, AI=technical execution

## Post-reinstall infrastructure (2026-04-25)
- [feedback_autonomy.md](feedback_autonomy.md) — Auto install/download/delete in personal RE workspace; still confirm for out-of-scope or hard-to-reverse
- [reference_environment.md](reference_environment.md) — Local toolchain paths: Python 3.12 D:\CodeTools\Python312, Java 21, Node 24, IDA 9.1, Ghidra 12, Frida 17, 3 MCP servers wired (ida-pro-mcp + ghidra-mcp + frida-mcp)
- [reference_workspace_map.md](reference_workspace_map.md) — Surviving context after Win reinstall: CLAUDE.md + plan file + IDA databases + PC_GAMES reports

## Projects
- [mu_xua_project.md](mu_xua_project.md) — MU Xua 2003: Frida bot, auto-login, multi-account
- [mu_login_lessons.md](mu_login_lessons.md) — MU PC login: packet order, gs=2 rule
- [mu_fpt_project.md](mu_fpt_project.md) — MU FPT Xua RE: Frida OK, key addresses found
- [kht_project.md](kht_project.md) — KHT Mobile (MemoryScanner + AutoDaTau 2.0)
- [autodatau2_project.md](autodatau2_project.md) — AutoDaTau 2.0 (login, quest, UseItem)
- [vnku_project.md](vnku_project.md) — VNKU: encryption cracked, protocol mapped
- [jx1_mobi_project.md](jx1_mobi_project.md) — JX1 Mobile: Unity IL2CPP, 202K methods, blocked (cần điện thoại root)
- [m4vn_project.md](m4vn_project.md) — M4VN MU Online: PhoenixCheat bypass OK, UPX timing, sẵn sàng bot
- [m4vn_progress.md](m4vn_progress.md) — M4VN: full auto bot multi-instance (login + tanthu + sell + delete + UI + exe)
- [m4vn_server_ids.md](m4vn_server_ids.md) — M4VN server ID: Sub-1=0, Sub-2=1, Sub-19=4 (không phải số UI)
- [vttt_project.md](vttt_project.md) — VTTT (MU Vũ Trụ Thần Thoại): bypass CMPlay AC OK (kill function patch), opcode capture in progress
- [thanhlong_project.md](thanhlong_project.md) — ThanhLong (Unity IL2CPP MMO VN): scouted 9/10 green, HOLD — game testing phase
- [cabal_mobile_project.md](cabal_mobile_project.md) — CABAL Online Mobile SEA: ACTIVE, XIGNCODE3 bypass + custom Lua engine (3-5 weeks)
- [rejected_games.md](rejected_games.md) — Games scouted & rejected (JX1 blocked; CABAL reactivated)

## Feedback & Rules
- [bot_code_standards.md](bot_code_standards.md) — Kiến trúc 5 tầng, state machine, logging, error handling
- [feedback_tool_priority.md](feedback_tool_priority.md) — Ưu tiên tool mạnh nhất, chuyển nhanh khi bị chặn
- [feedback_server_side_validation.md](feedback_server_side_validation.md) — Game online validate server-side → làm bot
- [feedback_game_thread_calls.md](feedback_game_thread_calls.md) — Gọi function phải từ game thread
- [feedback_ida_frida_combo.md](feedback_ida_frida_combo.md) — Hook sai → IDA decompile → đọc memory
- [feedback_render_toggle.md](feedback_render_toggle.md) — Render: Interceptor.attach + Sleep
- [feedback_dialog_close.md](feedback_dialog_close.md) — Đóng dialog bằng ADB back key
- [feedback_bot_design.md](feedback_bot_design.md) — Thiết kế 24/7, multi-account, giảm đồ họa
- [feedback_hard_game_red_flags.md](feedback_hard_game_red_flags.md) — 5 red flag check trước khi RE game (NeAC, Cocos 3.x V8, houdini, packed .so)
- [feedback_default_tier_t3.md](feedback_default_tier_t3.md) — MẶC ĐỊNH tier T3 (game UI + memory + packet inject), chỉ T4/T5 khi user explicit
- [feedback_language_style.md](feedback_language_style.md) — Viết thuần tiếng Việt, không chèn từ tiếng Anh giữa câu (trừ thuật ngữ kỹ thuật)
- [feedback_ui_design.md](feedback_ui_design.md) — UI Tkinter: fixed size, grid 4 cột aligned, tab Notebook, auto-save
- [feedback_adobe_air_re_workflow.md](feedback_adobe_air_re_workflow.md) — Adobe AIR mobile RE: libair.so custom check, SWF crack, JPEXS AS3 decompile
- [feedback_frida_hook_upx_timing.md](feedback_frida_hook_upx_timing.md) — Interceptor attach SAU UPX decompress (trước bị overwrite silent fail)

## Workflow & Approaches
- [re_workflow_optimized.md](re_workflow_optimized.md) — Quy trình: đánh giá → hack → bot, tool mapping
- [re_workflow_levels.md](re_workflow_levels.md) — IDA 8.3 FLIRT + RTTI + MCP setup
- [frida_gadget_approach.md](frida_gadget_approach.md) — Frida Gadget: hook ARM trên x86 emulator
- [bepinex_approach.md](bepinex_approach.md) — BepInEx: Unity IL2CPP hook C# bằng tên
- [window_monitor_tool.md](window_monitor_tool.md) — WindowMonitor: DWM preview

## Techniques
- [technique_upx_bypass.md](technique_upx_bypass.md) — UPX packed + server hash: đợi giải nén xong mới patch
- [technique_upx_memory_dump_ida.md](technique_upx_memory_dump_ida.md) — Dump UPX binary qua Frida + fix PE header (UPX0 RAW_SZ=VSZ) cho IDA analyze
- [technique_scene_state_bot.md](technique_scene_state_bot.md) — Scene-based game: set state vars thay vì call function trực tiếp
- [technique_packet_capture_caller.md](technique_packet_capture_caller.md) — Tìm function từ packet send + caller RVA
- [technique_multi_instance_bot.md](technique_multi_instance_bot.md) — Multi-account: worker-per-account, spawn stagger, per-session Frida
- [technique_threading_local_crypto.md](technique_threading_local_crypto.md) — Crypto state (counter/nonce) phải threading.local() khi multi-account 1 process
- [technique_pyinstaller_frozen_path.md](technique_pyinstaller_frozen_path.md) — PyInstaller --onefile: dùng sys.executable không dùng __file__ cho config persist
- [technique_semaphore_rate_limit.md](technique_semaphore_rate_limit.md) — Server chặn multi-conn → Semaphore rate-limit, không bypass
- [technique_drain_vs_retry.md](technique_drain_vs_retry.md) — Verify action success bằng total response bytes trong time window, không retry spam
- [technique_bitpack_byteorder.md](technique_bitpack_byteorder.md) — Custom cipher bit-packed: tách 8-bit bytes rồi ghép LE, không đọc full value
- [technique_mu_template_reuse.md](technique_mu_template_reuse.md) — MU login: reverse KEY32 chain 1 lần, extract session_enc36, reuse N account
- [technique_game_update_offset_patch.md](technique_game_update_offset_patch.md) — Game update: regions dời khác nhau (game code/anti-cheat/data globals), scan bằng wildcard pattern cho imm32
- [technique_socks5_antiproxy_bypass.md](technique_socks5_antiproxy_bypass.md) — Multi-layer IP ban + client anti-proxy: SOCKS5 tunnel + getpeername spoof
- [technique_opengl_vram_optimize.md](technique_opengl_vram_optimize.md) — VRAM tối ưu game OpenGL: noop glTexImage2D + glDeleteTextures từ render thread (wglSwapBuffers)
- [technique_win_restore_recovery.md](technique_win_restore_recovery.md) — Win System Restore rollback: PATH/env/VC++Redist/Npcap mất, files D:\ giữ. Recovery 5 phút

## Trigger Rules
- "MU", "MU Xua" → đọc mu_xua_project.md
- "MU FPT" → đọc mu_fpt_project.md
- "KHT", "AutoDaTau" → đọc kht_project.md
- "VNKU" → đọc vnku_project.md
- "JX1", "jx1_mobi" → đọc jx1_mobi_project.md
- "M4VN" → đọc m4vn_project.md + m4vn_progress.md
- "ThanhLong", "Thanh Long" → đọc thanhlong_project.md (HOLD)
- "CABAL", "CMSEA" → đọc cabal_mobile_project.md (ACTIVE)
- "VTTT", "Vũ Trụ Thần Thoại" → đọc vttt_project.md
- "MUHAOQUANG" → folder PC_GAMES/MUHAOQUANG_bypass/ (RECON.md + README.md)
- Game Adobe AIR (libair.so + .swf) → đọc feedback_adobe_air_re_workflow.md
- Game ARM + LDPlayer → đọc frida_gadget_approach.md
- Game Unity IL2CPP → đọc bepinex_approach.md
- Bắt đầu dự án mới → đọc re_workflow_optimized.md + bot_code_standards.md
- Game UPX packed + server hash check → đọc technique_upx_bypass.md
- Frida hook không hit → đọc feedback_frida_hook_upx_timing.md (UPX timing)

## User Preferences
- Giao tiếp: Tiếng Việt | Code: Python + Frida JS | Code comments: English
- Workspace root: `D:\Code Tools\` (junction `D:\CodeTools\` không space)
- ADB: `D:\Code Tools\TOOLS\Adb\adb.exe` | LDPlayer: Android 7 x86_64
- Frida trên houdini: chỉ đọc memory, KHÔNG hook ARM
- MẶC ĐỊNH: tier T3 (UI + memory + packet inject)
- "Không ổn là dừng" — pivot/stop nhanh khi feature/game fail
