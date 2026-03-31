# User Memory

## Projects
- **mu_xua_project.md** — MU Xua 2003: Frida bot, auto-login (5min disconnect FIXED), multi-account, render throttle
- **mu_login_lessons.md** — MU PC login: NEVER connectToConnectServer at gs=2, packet order F4/03→F1→F3/E8→F3/0E→gs=5
- Khi user nhac den "MU", "MU Xua" → doc mu_xua_project.md truoc
- Khi lam game MU PC khac → doc mu_login_lessons.md truoc
- **mu_fpt_project.md** — MU FPT Xua RE: Frida OK, key addresses found, charBase pending
- Khi user nhac den "MU FPT" → doc mu_fpt_project.md truoc
- **vlcm_project.md** — VLCM (Vo Lam Chi Mong): Flash game FULLY CRACKED - 4334 AS files, protocol+encryption cracked, bot Python thuan kha thi
- Khi user nhac den "VLCM", "Vo Lam Chi Mong", "Mong Chi Ton" → doc vlcm_project.md truoc
- **kht_project.md** — Kiem Hiep Tinh Duyen Mobile (MemoryScanner + AutoDaTau 2.0)
- **autodatau2_project.md** — AutoDaTau 2.0 chi tiet (login flow, quest flow, UseItem packet)
- **vnku_project.md** — VNKU game RE: encryption cracked, protocol mapping (move=0x4c, attack=0x29)
- Khi user nhac den "KHT", "AutoDaTau", "MemoryScanner", "auto quest" → doc kht_project.md truoc
- Khi user nhac den "VNKU" → doc vnku_project.md truoc
- **icarus_project.md** — Ky Nguyen Icarus: MU Online engine port, SDL3, ARM32, IDA analysis started
- Khi user nhac den "Icarus", "Ky Nguyen" → doc icarus_project.md truoc
- **gunbound_project.md** — Gunbound Legend PC: Cocos2d, XIGNCODE anti-cheat, AI FireSimulation for aimbot
- Khi user nhac den "Gunbound", "GunboundM" → doc gunbound_project.md truoc
- **vl_multi_project.md** — VL Multi-Instance: deep RE of VietGuards auth/HWID/kick
- Khi user nhac den "VL", "VietGuards", "multi instance" → doc vl_multi_project.md truoc

## Feedback
- **feedback_dialog_close.md** — Dong dialog game bang ADB back key, KHONG dung vtable hack
- **feedback_ida_frida_combo.md** — Khi Frida hook/formula sai, dung IDA decompile struct roi doc memory truc tiep
- **feedback_server_side_validation.md** — Game online validate speed+position server-side: speed hack/teleport fail
- **feedback_game_thread_calls.md** — Goi game function phai tu game thread (hook callback), khong tu Frida thread
- **feedback_bot_design.md** — Moi feature phai thiet ke cho 24/7 + multi-account, giam tai do hoa qua Frida hook
- **feedback_render_toggle.md** — Render toggle: dung Interceptor.attach + Sleep, KHONG dung replace/byte-patch
- **feedback_warp_raw_packet.md** — MU Xua: sendChat("/warp") KHONG hoat dong, phai dung _sendWarpPkt raw packet

## Reference
- **re_workflow_levels.md** — RE tool setup: IDA 8.3 (FLIRT 347 sigs + RTTI built-in + MCP) + Frida
- **window_monitor_tool.md** — Window Monitor: DWM live preview nhieu cua so game PC
- **reference_github_backup.md** — Memory backup: github.com/mjnhhjen9x-source/claude-memory (PRIVATE)

## User Preferences
- Ngon ngu giao tiep: Tieng Viet
- **Ngon ngu lap trinh: Python (cho Frida automation), Frida JS (game interaction)**
- **Folder structure**:
  - `D:\Code Tools\PC_GAMES\` - MU_XUA_2003, MU_FPT, VLCM_PC
  - `D:\Code Tools\MOBILE_GAMES\` - VNKU, AutoDaTau2.0, jx1shxt, VL_MULTI
  - `D:\Code Tools\IDA_DATABASES\` - .i64/.idb files theo ten du an
  - `D:\Code Tools\TOOLS\` - Adb, JPEXS_FFDec, WindowMonitor, ABD Cap
  - `D:\Code Tools\IDA Pro\` - IDA Pro 8.3
- ADB path: `D:\Code Tools\TOOLS\Adb\adb.exe`
- Frida-server: `/data/local/tmp/frida-server` (v17.7.3 x86)
- **Frida tren houdini**: hook ARM code KHONG hoat dong, chi doc memory duoc
