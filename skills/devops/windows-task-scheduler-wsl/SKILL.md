---
name: windows-task-scheduler-wsl
description: Configure Windows Task Scheduler via PowerShell from WSL bash to auto-start WSL commands at Windows boot
---

# Windows Task Scheduler via WSL

## Problem
Create a Windows Scheduled Task to auto-start a WSL command at Windows boot, from within WSL bash.

## Key Learnings

### 1. schtasks not available in WSL
`schtasks` is a Windows binary, not a Linux command. Must use `powershell.exe` from WSL to invoke it.

### 2. Windows username ≠ WSL username
Windows user is `001` (SID S-1-5-21-...-500), NOT `shumi`. Use SID or username `001` in `-UserId`. Query with:
```powershell
Get-LocalUser | Select-Object Name,SID,Enabled
```

### 3. WSL distro name discovery
WSL distro names aren't always obvious from bash. Options:
- `wsl.exe -l -v` (but output encoding issues in PowerShell — returns UTF-16 with null bytes, cannot parse reliably)
- Check `/etc/wsl.conf` for systemd=true
- Default is often `Ubuntu-22.04`
- Check `cat /etc/os-release` from inside WSL

### 4. Command execution via wsl.exe — CRITICAL
```powershell
# ❌ FAILS with error 267011 (file not found)
wsl.exe -d Ubuntu-22.04 -e /home/shumi/.local/bin/hermes gateway run

# ✅ WORKS — use -e WITHOUT -d distro flag
wsl.exe -e /home/shumi/.local/bin/hermes gateway run
```
Error 267011 means the WSL distro name passed to `-d` is not recognized by the installed WSL version.
Always use `wsl.exe -e /path/to/command` (no `-d`) to invoke the default distro's entry point.

## Full Working Script
```powershell
$action = New-ScheduledTaskAction -Execute 'wsl.exe' -Argument '-e /home/shumi/.local/bin/hermes gateway run' -WorkingDirectory 'C:\'
$trigger = New-ScheduledTaskTrigger -AtStartup
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable
$principal = New-ScheduledTaskPrincipal -UserId '001' -LogonType Interactive -RunLevel Limited
Register-ScheduledTask -TaskName 'HermesAgent' -Action $action -Trigger $trigger -Settings $settings -Principal $principal -Description 'Hermes AI Agent Gateway Auto-Start' -Force
```

## Verification
```powershell
# Via PowerShell
Get-ScheduledTask -TaskName 'HermesAgent' | Select-Object State,TaskName
Get-ScheduledTaskInfo -TaskName 'HermesAgent'

# Via schtasks
schtasks /query /tn 'HermesAgent' /fo LIST /v
```

## Alternative: Using .bat File (avoids PowerShell quote nesting)

When command strings get complex, PowerShell's quote handling breaks down. Solution: write a `.bat` wrapper, then schedule that.

```bat
@echo off
wsl.exe -e bash -c "cd /home/shumi && your_command_here"
```

Then schedule the `.bat` file:
```powershell
schtasks /create /tn 'TaskName' /tr 'D:\path\to\wrapper.bat' /sc onlogon /rl HIGHEST /f
```

**Key advantages:**
- No quote nesting hell
- Easy to debug (just double-click the .bat)
- schtasks via `-Command` approach works reliably when the target command is simple

## Hermes-AutoStart Working Config

```bat
@echo off
cd /d D:\AI文档\01_摸鱼工作区\MIND
wsl.exe -e bash -c "cd /home/shumi && echo HERMES STARTED && sleep 999"
```

```powershell
schtasks /create /tn 'Hermes-AutoStart' /tr 'D:\\AI文档\\01_摸鱼工作区\\MIND\\hermes_start.bat' /sc onlogon /rl HIGHEST /f
schtasks /run /tn 'Hermes-AutoStart'  // 测试运行
```

**Pitfalls discovered:**
- Task can disappear after running if the script runs indefinitely (sleep 999) — system may mark it as hung
- Use `tail -f /dev/null` instead of `sleep 999` to keep WSL alive without appearing hung
- `/rl HIGHEST` requires the user to have admin privileges (user `001` does)
- Task disappears = check if the command inside .bat actually works by double-clicking the .bat first

## Desktop Shortcut via WScript.Shell (PowerShell)

To create a desktop shortcut to Windows Terminal (for Chinese IME support):
```powershell
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("C:\Users\001\Desktop\Hermes-Terminal.lnk")
$Shortcut.TargetPath = "C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_8wekyb3d8bbwe\wt.exe"
$Shortcut.Description = "Hermes Terminal with Chinese IME support"
$Shortcut.Save()
```

**Note:** Variables ($) in -Command strings get stripped by WSL bash — always write multi-line scripts to .ps1 files and run with `powershell.exe -ExecutionPolicy Bypass -File path\to\script.ps1`

## Pitfalls
- `-UserId 'shumi'` fails with "No mapping between account names and security IDs" — use SID `S-1-5-21-...-500` or actual Windows username `001`
- `-d Ubuntu-22.04` causes error 267011 on some WSL versions — **never use `-d`, always use `wsl.exe -e` directly**
- `wsl.exe -l -v` output has encoding issues in PowerShell (UTF-16 with null bytes) — don't try to parse it programmatically
- `schtasks` is not a Linux command, use `powershell.exe -Command "schtasks ..."`
- Task appears in "Ready" state but LastRunTime shows "1999/11/30" with result 267011 if the command failed at trigger time
- Complex `schtasks /create` commands via `-Command` can timeout — use a `.bat` wrapper file instead
