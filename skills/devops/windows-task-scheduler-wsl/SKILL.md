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

## Pitfalls
- `-UserId 'shumi'` fails with "No mapping between account names and security IDs" — use SID `S-1-5-21-...-500` or actual Windows username `001`
- `-d Ubuntu-22.04` causes error 267011 on some WSL versions — **never use `-d`, always use `wsl.exe -e` directly**
- `wsl.exe -l -v` output has encoding issues in PowerShell (UTF-16 with null bytes) — don't try to parse it programmatically
- `schtasks` is not a Linux command, use `powershell.exe -Command "schtasks ..."`
- Task appears in "Ready" state but LastRunTime shows "1999/11/30" with result 267011 if the command failed at trigger time
