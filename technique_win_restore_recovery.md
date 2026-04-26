---
name: Recover toolchain after Windows System Restore
description: What System Restore rolls back vs preserves, and the steps to re-bring toolchain online
type: technique
originSessionId: 739a471f-c622-43c3-b869-927876ec61e8
---
# Win System Restore — Recovery Procedure

## What System Restore touches vs preserves

**Rolls back (gone after restore):**
- `HKCU\Software\*` registry keys set after the restore point (env vars from `[Environment]::SetEnvironmentVariable`, IDA Python binding `HKCU\Software\Hex-Rays\IDA\Python3TargetDLL`)
- `HKLM\Software\*` (Visual C++ Redistributable installation, anything in `Uninstall\*`)
- Windows services/drivers (Npcap kernel driver, anything via `sc create` or installer-registered services)
- Some Program Files components

**Preserved:**
- D:\ drive entirely (or any non-system drive) — `D:\Code Tools\` workspace untouched
- Files in `C:\Users\<account>\` profile (.claude/, AppData)
- VSCode + extensions
- Git credentials in Credential Manager (usually)

## Recovery steps (run in order)

### 1. Restore PATH (User scope)
```powershell
$paths = @(
  'D:\CodeTools\Python312', 'D:\CodeTools\Python312\Scripts',
  'D:\CodeTools\Java\jdk-21\bin', 'D:\CodeTools\NodeJS',
  'D:\CodeTools\TOOLS\jadx\bin', 'D:\CodeTools\TOOLS\Sysinternals',
  'D:\CodeTools\TOOLS\DIE', 'D:\CodeTools\TOOLS\Adb',
  'D:\CodeTools\TOOLS\radare2\bin', 'D:\CodeTools\TOOLS\gh\bin',
  'D:\CodeTools\Wireshark', 'D:\CodeTools\Git\cmd'
)
$cur = [Environment]::GetEnvironmentVariable('Path','User')
$existing = if ($cur) { $cur -split ';' | Where-Object { $_ } } else { @() }
$merged = (@($existing) + ($paths | Where-Object { $existing -notcontains $_ })) -join ';'
[Environment]::SetEnvironmentVariable('Path', $merged.TrimEnd(';'), 'User')
```

### 2. Restore env vars
```powershell
[Environment]::SetEnvironmentVariable('JAVA_HOME', 'D:\CodeTools\Java\jdk-21', 'User')
[Environment]::SetEnvironmentVariable('GHIDRA_INSTALL_DIR', 'D:\CodeTools\TOOLS\Ghidra', 'User')
```

### 3. Reinstall VC++ Redistributable (CRITICAL — IDA/r2/Wireshark depend on it)
Without this, exit code `-1073741515` (`STATUS_DLL_NOT_FOUND`) on any tool needing MSVCP140/VCRUNTIME140.
```powershell
$url = 'https://aka.ms/vs/17/release/vc_redist.x64.exe'
$out = 'D:\Code Tools\_installers\vc_redist.x64.exe'
Invoke-WebRequest -Uri $url -OutFile $out -UseBasicParsing
Start-Process -FilePath $out -ArgumentList '/install','/quiet','/norestart' -Wait -Verb RunAs
```

### 4. Re-bind IDA to Python
```powershell
& 'D:\CodeTools\IDA Pro\IDA 9.1 Pro\idapyswitch.exe' --force-path 'D:\CodeTools\Python312\python3.dll' -v
```

### 5. Reinstall Npcap (kernel driver, needs UAC interactive)
Silent install (`/S`) returns exit 2 — driver signing dialog needs user click. Use interactive:
```powershell
$url = 'https://npcap.com/dist/npcap-1.87.exe'
$out = 'D:\Code Tools\_installers\npcap-1.87.exe'
Invoke-WebRequest -Uri $url -OutFile $out -UseBasicParsing
Start-Process -FilePath $out -Verb RunAs   # user clicks through
```

### 6. Verify
```powershell
Get-Service 'npcap' | Select-Object Name, Status   # should be Running
& 'D:\CodeTools\Wireshark\dumpcap.exe' -D | Select-Object -First 3   # interfaces listed
& 'D:\CodeTools\TOOLS\radare2\bin\radare2.exe' -v                    # version prints
```

## Tools NOT affected by restore (work without re-config)
Anything with bundled runtime: Python, Java, Node, Git (all portable from D:\). Run directly even if PATH not restored — just use full path.

## Diagnosis: how to tell what System Restore broke
```powershell
# 1. Check PATH — most common loss
[Environment]::GetEnvironmentVariable('Path','User')

# 2. Check VC++ Redist
Get-ItemProperty 'HKLM:\Software\WOW6432Node\Microsoft\VisualStudio\14.0\VC\Runtimes\X64' -ErrorAction SilentlyContinue

# 3. Check Npcap service
Get-Service 'npcap' -ErrorAction SilentlyContinue

# 4. Test each tool — exit code -1073741515 = missing VC++ runtime
& 'D:\CodeTools\TOOLS\radare2\bin\radare2.exe' -v
```

## Time to recover
~5 phút (download + install): VC++ Redist 24MB, Npcap 1.3MB. Plus 1-2 phút config.
