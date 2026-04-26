---
name: qq-file-delivery-wsl
description: QQ文件无法传递到WSL运行Hermes的解决方案
---

# QQ File Delivery to WSL Hermes Agent

## Problem
When user sends files via QQ to this Hermes agent (running in WSL), the files are downloaded to Windows QQ cache directories, NOT accessible from the WSL filesystem. The agent cannot read them.

## Root Cause
- Hermes agent runs inside WSL (Windows Subsystem for Linux)
- QQ downloads files to Windows paths like `C:\Users\<username>\Documents\Tencent Files\...`
- WSL cannot see these files unless explicitly accessed via `/mnt/c/...` AND the user manually moved the file there

## Solution: Instruct User to Move Files
When user wants to send a file to Hermes:
1. Tell them to locate the file in QQ's download cache (or have QQ auto-save to a folder)
2. **Move or copy the file** to a WSL-accessible location (e.g., Desktop, Downloads, or any `/mnt/c/Users/<username>/...` path)
3. Provide the new path or ask them to drag-and-drop again

## Quick Check
```bash
# Common QQ download locations in Windows
/mnt/c/Users/shumi/Documents/Tencent\ Files/
/mnt/c/Users/shumi/Downloads/
/mnt/c/Users/shumi/Desktop/
```

## Key Insight
QQ attachment sending works (agent receives the message with attachment metadata), but the **file content lands in Windows user profile**, not WSL. User must manually bridge this gap by moving the file to a path WSL can read.

## Tags
- platform: qq + wsl
- file delivery
- windows-wsl interop
