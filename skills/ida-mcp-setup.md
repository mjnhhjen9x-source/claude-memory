---
name: ida-mcp-setup
description: Use when setting up or starting IDA Pro MCP connection - covers installation, plugin loading, server start, and troubleshooting common errors
---

# IDA Pro MCP Setup & Start

## When to Use
- First time setting up IDA MCP on a new machine
- IDA MCP plugin not loading or not showing in menu
- MCP server connection fails
- User asks to connect Claude Code to IDA Pro

## Installation (One-time)

### 1. Install ida-pro-mcp package
```bash
pip install ida-pro-mcp
ida-pro-mcp --install
```
This creates plugin file at `%APPDATA%\Hex-Rays\IDA Pro\plugins\mcp-plugin.py`

### 2. Fix plugin filename (CRITICAL)
IDA cannot load Python plugins with dashes in filename. Rename:
```bash
mv "%APPDATA%/Hex-Rays/IDA Pro/plugins/mcp-plugin.py" "%APPDATA%/Hex-Rays/IDA Pro/plugins/mcp_plugin.py"
```

### 3. Copy to IDA plugins directory
IDA loads plugins from TWO locations. To avoid class name conflicts, keep plugin in ONLY ONE location:
- **Keep:** `<IDA_INSTALL>/plugins/mcp_plugin.py`
- **Delete:** `%APPDATA%/Hex-Rays/IDA Pro/plugins/mcp_plugin.py` (or mcp-plugin.py)

Current IDA path: `D:\Code Tools\IDA Pro\8.3\IDA Pro 8.3 (x86_x64_ARM_ARM64_PPC_PPC64_MIPS)_P.Y.G_Team\`

### 4. Configure Python for IDA
IDA 8.3 needs Python 3.11 (NOT 3.14). Run idapyswitch:
```bash
"<IDA_INSTALL>/idapyswitch.exe"
```
Select: `C:\Users\Than Tai\AppData\Local\Programs\Python\Python311\python311.dll`

### 5. MCP config in Claude Code
File: `C:\Users\Than Tai\.claude\.mcp.json`
```json
{
  "mcpServers": {
    "ida-pro-mcp": {
      "command": "C:\\Python314\\python.exe",
      "args": ["C:\\Python314\\Lib\\site-packages\\ida_pro_mcp\\server.py"],
      "timeout": 1800,
      "disabled": false
    }
  }
}
```

## Starting MCP (Every Session)

### Quy trinh khoi dong:
1. Mo IDA Pro
2. Mo file binary (.so/.dll) - **plugin chi hien khi co file dang mo**
3. Doi auto-analysis xong
4. **Edit -> Plugins -> MCP** (hoac **Ctrl+Alt+M**)
5. Output window se hien: `[MCP] Server started at http://localhost:13337`
6. Trong Claude Code, dung tool `mcp__ida-pro-mcp__check_connection` de kiem tra

### IMPORTANT: Plugin chi hien trong menu khi da mo binary file!

## Troubleshooting

### Plugin khong hien trong Edit -> Plugins
| Van de | Nguyen nhan | Cach fix |
|--------|-------------|----------|
| Khong co MCP trong menu | Chua mo binary file | Mo 1 file .so/.dll truoc |
| `PLUGIN_ENTRY was not defined or class name 'MCP' was already used` | 2 file plugin ton tai (mcp-plugin.py VA mcp_plugin.py) | Xoa 1 trong 2, giu lai mcp_plugin.py |
| `No module named 'keystone'` | Plugin keypatch.py thieu dependency | Bo qua, khong anh huong MCP |
| Python version mismatch | IDA dung sai Python version | Chay idapyswitch chon Python 3.11 |

### Kiem tra plugin bang Python console
Trong IDA Python console (goc duoi), go:
```python
import mcp_plugin
```
Neu khong bao loi = plugin load thanh cong.

### Khong co file binary de test
Pull libc.so tu emulator:
```bash
MSYS_NO_PATHCONV=1 "D:/Code Tools/Adb/adb.exe" -P 5038 -s <device> pull /system/lib/libc.so "C:/Users/Than Tai/Downloads/libc.so"
```

## Plugin Directories (IDA 8.3)
IDA loads plugins from BOTH:
1. `<IDA_INSTALL>/plugins/` (installation dir)
2. `%APPDATA%/Hex-Rays/IDA Pro/plugins/` (user dir)

If same class name exists in both -> conflict error. Keep plugin in only ONE location.
