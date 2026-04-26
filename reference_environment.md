---
name: Local toolchain paths and environment setup
description: Where Python, Java, Node, IDA, Ghidra, Wireshark, jadx, DIE, Sysinternals, frida-tools live + how IDA MCP is wired up
type: reference
originSessionId: 739a471f-c622-43c3-b869-927876ec61e8
---
## Filesystem layout (all under D:\Code Tools\)
Junction `D:\CodeTools\` → `D:\Code Tools\` — same content, no spaces. Prefer `D:\CodeTools\` when invoking installers or tools that dislike spaces; either path works for read.

### Languages / runtimes (on PATH)
- **Python 3.12.7**: `D:\CodeTools\Python312\python.exe` + `\Scripts\`
- **Java JDK 21.0.10 LTS**: `D:\CodeTools\Java\jdk-21\bin\java.exe` (ACTIVE, JAVA_HOME points here)
- **Java JDK 17.0.18**: `D:\CodeTools\Java\jdk-17\` (kept for legacy, not active)
- **Node.js v24.15.0 LTS** + npm 11.12.1: `D:\CodeTools\NodeJS\node.exe`
- **Git 2.54**: `D:\CodeTools\Git\cmd\git.exe`

### RE / analysis tools
- **IDA Pro 9.1**: `D:\Code Tools\IDA Pro\IDA 9.1 Pro\ida.exe` (license: `idapro.hexlic`)
- **Ghidra 12.0.4 PUBLIC**: `D:\CodeTools\TOOLS\Ghidra\` (`support\analyzeHeadless.bat` for headless)
- **jadx 1.5.5**: `D:\CodeTools\TOOLS\jadx\bin\jadx.bat` (CLI, on PATH)
- **DIE 3.21**: `D:\CodeTools\TOOLS\DIE\` (`die.exe` GUI, `diec.exe` CLI, `diel.exe` lite)
- **radare2 6.1.4**: `D:\CodeTools\TOOLS\radare2\bin\` (radare2, rabin2, rasm2, rax2, rafind2, etc — on PATH); `r2pipe` Python bindings installed
- **Sysinternals Suite**: `D:\CodeTools\TOOLS\Sysinternals\` (strings, handle, pslist, sigcheck, procmon, procexp, etc — on PATH)
- **apktool, BepInEx, Il2CppDumper, dnSpy, JPEXS_FFDec, uber-apk-signer, upx, frida-gadget, frida-mcp**: `D:\Code Tools\TOOLS\<name>\` (pre-existing)

### Network capture
- **Wireshark 4.6.4**: `D:\CodeTools\Wireshark\` (tshark, dumpcap, editcap, mergecap, capinfos — all on PATH)
- **Npcap 1.87**: kernel driver service `npcap` (Running). Required for live capture.

### Mobile / Android
- **ADB**: `D:\Code Tools\TOOLS\Adb\adb.exe` (on PATH)
- **Frida-server 17.9.1 x86_64** (for LDPlayer): `D:\Code Tools\TOOLS\frida-server-17.9.1-android-x86_64` (decompressed). Push via `adb push` → `/data/local/tmp/frida-server`, chmod 755, run as `su`. Old 16.x binaries also kept.
- **LDPlayer**: `127.0.0.1:5555`, Android 7.1.2 x86_64

### Python RE libraries (pip-installed in `D:\CodeTools\Python312\`)
- **frida** 17.9.1, **frida-tools** 14.8.1 → CLI: `frida`, `frida-ps`, `frida-trace`, `frida-discover`, `frida-kill`, `frida-ls-devices`, `frida-pull`, `frida-push`, `frida-rm`, `frida-create`, `frida-compile`
- **pymem** 1.14 — read/write process memory (CheatEngine alternative for automation)
- **pefile** 2024.8.26, **lief** 0.17.6 — parse/modify PE/ELF
- **capstone** 5.0.7, **keystone-engine** 0.9.2, **unicorn** 2.1.4 — disasm/asm/emulate
- **pycryptodome** 3.23 — crypto
- **scapy** 2.7, **pyshark** 0.6 — packet manipulation/parsing
- **requests** 2.33, **mcp** 1.27 (deps of ida-pro-mcp)

### Skills installed (`C:\Users\ThanTai\.claude\skills\`)
7 user skills restored from old backup, ready to use:
- `apk-recon` — APK extract → engine detect → r2pipe analyze → game_map.json
- `frida-memory-scan` — value scan flow (D64→F32→I32 priority, filter rounds, neighbor probe)
- `frida-packet-reverse` — hook send/recv/recvfrom, BE/LE readers, cmd table, modify/inject
- `pointer-chain-finder` — reverse pointer scan, build chain to module base, verify across restart
- `ida-analysis` — IDA MCP workflow (decompile, xrefs, struct extraction)
- `game-bot-builder` — end-to-end 5-phase workflow combining all skills above
- `r2-static-analysis` — r2pipe cheatsheet + r2+frida combo workflow

NOT installed (deliberately): `ida-mcp-setup` from old backup — references stale paths (Python 3.14, IDA 8.3, "Than Tai" account, mcp-plugin.py rename hack). Current setup correct via `reference_environment.md`.

### MCP servers (Claude Code)
Both configured in `C:\Users\ThanTai\.claude.json` under `mcpServers`:

**ida-pro-mcp v1.4.0** (mrexodia)
- Package: `D:\CodeTools\Python312\Lib\site-packages\ida_pro_mcp\`
- IDA plugin: `C:\Users\ThanTai\AppData\Roaming\Hex-Rays\IDA Pro\plugins\mcp-plugin.py` (symlink)
- IDA HTTP RPC on `127.0.0.1:13337` when database loaded; MCP server bridges stdio ↔ HTTP

**ghidra-mcp v2.0** (pinksawtooth fork — original LaurieWired unmaintained)
- Built for Ghidra 12.0.3, patched `extension.properties` to 12.0.4 for our install
- Bridge: `D:\CodeTools\TOOLS\GhidraMCP-bridge\bridge_mcp_ghidra.py`
- Ghidra extension: `C:\Users\ThanTai\.ghidra\.ghidra_12.0.4_PUBLIC\Extensions\GhidraMCP\`
- Ghidra HTTP RPC on `127.0.0.1:8080` when CodeBrowser open with program loaded; MCP bridge connects stdio ↔ HTTP
- **Activation flow**: Ghidra → File → Install Extensions (tick GhidraMCP, restart) → open program in CodeBrowser → File → Configure → Developer → enable GhidraMCPPlugin

**frida-mcp v0.1.0** (CUSTOM — user-built, repo: github.com/mjnhhjen9x-source/frida-mcp)
- Local: `D:\Code Tools\TOOLS\frida-mcp\` (pip-installed editable: `pip install -e .`)
- Designed for game RE workflow specifically (M4VN, MU FPT, VTTT, JX1, KHT, VNKU, MUHAOQUANG)
- Spec: `D:\Code Tools\TOOLS\frida-mcp\docs\superpowers\specs\2026-04-21-frida-mcp-bridge-design.md`
- 14 tools: list_devices, list_processes, spawn, attach, detach, resume, load_script, eval_js, rpc_call, read_memory (u32/u64/s32/s64/float/double/bytes/string/utf16), write_memory (same types), scan_pattern, run_recipe, snapshot
- 11 built-in recipes: access_watchpoint, dump_module, filter_addrs, find_function_by_string, list_exports, probe_around, scan_double, scan_float, scan_int32_be, scan_int32_le, trace_calls
- Single global session (no multi-session, by design)
- Use case: dynamic analysis (Cheat Engine + frida-trace equivalent, callable from Claude). Complements ida-pro-mcp + ghidra-mcp (static analysis)

### Sandbox guardrails (cannot bypass via memory or feedback)
- Agent cannot self-modify `C:\Users\ThanTai\.claude.json` (would let it self-grant new MCP servers/tools)
- Agent cannot self-add allow rules to `settings.local.json` (would be indirect privilege escalation)
- User must manually edit settings to grant `Edit(...)` / `Write(...)` permission on `.claude.json` — done 2026-04-25, both rules now in `settings.local.json`. Agent can now Edit/Write `.claude.json` for future MCP additions without prompting.

### Environment variables (User scope, persistent)
- `JAVA_HOME = D:\CodeTools\Java\jdk-21`
- `GHIDRA_INSTALL_DIR = D:\CodeTools\TOOLS\Ghidra`
- `PATH` additions: Python312, Python312\Scripts, Java\jdk-21\bin, NodeJS, TOOLS\jadx\bin, TOOLS\Sysinternals, TOOLS\DIE, TOOLS\Adb, TOOLS\radare2\bin, TOOLS\gh\bin, Wireshark, Git\cmd

### IDA ↔ Python binding
Registry key set by `idapyswitch.exe --force-path`:
`HKCU\Software\Hex-Rays\IDA\Python3TargetDLL = D:\CodeTools\Python312\python312.dll`

## To use IDA MCP
1. Open `D:\Code Tools\IDA Pro\IDA 9.1 Pro\ida.exe`
2. Load a `.i64` database from `D:\Code Tools\IDA_DATABASES\`
3. Plugin auto-starts (check IDA Output window for "MCP")
4. Restart Claude Code session → `mcp__ida-pro-mcp__*` tools available

## Headless workflow examples (no GUI needed)
```powershell
# Decompile APK with jadx
jadx -d D:\out\decompiled D:\path\game.apk

# Detect packer
diec D:\path\game.exe

# Capture LDPlayer traffic via tshark on host (LDPlayer NIC)
tshark -i "Local Area Connection*" -w out.pcapng -f "host 127.0.0.1"

# Ghidra headless analyze
analyzeHeadless D:\proj proj_name -import D:\path\game.exe -postScript MyScript.py

# Frida attach
frida-ps -U                                  # list processes on USB device
frida -U -n com.target.app -l agent.js       # attach with script

# Memory hack via pymem (no Cheat Engine)
python -c "import pymem; p=pymem.Pymem('game.exe'); print(hex(p.read_int(0x12345)))"

# PE analysis without HxD
python -c "import pefile; pe=pefile.PE('game.exe'); print([s.Name for s in pe.sections])"
```

## NOT installed (user runs GUI manually if needed)
- **x64dbg** — Windows debugger (GUI). Use Frida or IDA debugger for automation.
- **HxD** — hex editor (GUI). Use Python `binascii/struct/pefile/lief`.
- **Cheat Engine** — memory hacking (GUI). Use `pymem` or Frida hooks.

## Installer cache
Removed 2026-04-26 (was at `D:\Code Tools\_installers\`, ~1.4 GB). All downloads deleted — re-download from official sources if reinstall needed.
