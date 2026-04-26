---
name: mcporter-windows-mcp
description: Configure Windows-MCP (mcporter) from WSL — write config file directly, restart from Windows side
tags: [mcp, mcporter, windows, wsl, cross-env]
---

# mcporter + Windows-MCP Setup

Configure Windows-MCP (or any MCP server) on Windows from WSL, bypassing cross-environment command timeouts.

## The Problem

mcporter is installed on **Windows** (`C:\Users\Administrator\AppData\Roaming\npm\mcporter`). When you run `mcporter` commands from **WSL/Linux**, they timeout (60s+) because each call must redirect to the Windows-side process.

## The Solution

Use the config file directly instead of the CLI:

### 1. Find the correct config path

mcporter's config lives in the **Windows user profile**, NOT in the WSL path:

```
C:\Users\<WindowsUsername>\.mcporter\mcporter.json
```

In WSL, that translates to:
```
/mnt/c/Users/<WindowsUsername>/.mcporter/mcporter.json
```

For the Administrator account: `/mnt/c/Users/Administrator/.mcporter/mcporter.json`

To confirm: `find /mnt/c/Users/Administrator -maxdepth 3 -name "mcporter.json" -path "*mcporter*" 2>/dev/null`

### 2. Edit the JSON directly

Open the file and add your MCP server entry under `"mcpServers"`:
```json
{
  "mcpServers": {
    "windows-mcp": {
      "command": "uvx",
      "args": ["windows-mcp"]
    }
  }
}
```

**Note:** mcporter may already have existing servers configured (e.g. `tencent-docs`). Preserve them — do NOT overwrite the entire file, just add the new server entry.

### 3. Restart from Windows side

**Open PowerShell on Windows** (not WSL), then:
```powershell
mcporter daemon restart
mcporter list
```

## Key Insight

- mcporter lives on **Windows**, not WSL
- The correct config path is `C:\Users\<username>\.mcporter\mcporter.json`, NOT the WSL `.hermes` directory
- Commands from WSL to Windows npm executables are slow/unreliable (timeout even at 60s)
- Direct file editing + Windows-side restart is the reliable workflow
- Existing servers (e.g. `tencent-docs`, `windows`) are preserved — add only new entries

## Verification

```powershell
mcporter list
# Expected: windows-mcp (18 tools, healthy)
```

## First-start Gotcha

First launch installs dependencies — may take 1-2 minutes. If `mcporter list` shows unhealthy/empty, just run `mcporter daemon restart` again.

## Daemon Auto-Start

After `mcporter daemon restart`, the daemon may exit with "No MCP servers are configured for keep-alive". To start it persistently, run:

```powershell
mcporter daemon start
```

Then verify:
```powershell
mcporter list
# Expected: windows-mcp (18 tools, healthy)
```

## Cross-Environment Calling Problem

**WSL cannot execute Windows binaries directly.** Running `mcporter` from WSL fails with:
```
/usr/bin/bash: /mnt/c/WINDOWS/system32/cmd.exe: cannot execute binary file: Exec format error
```

This means:
- `mcporter call`, `mcporter tools`, `mcporter list` all **timeout or fail** when called from WSL/bash
- Windows PowerShell/CMD also cannot be invoked from WSL (same exec error)
- **mcporter has no HTTP server mode in v0.8.1** — the `mcporter http serve` command does not exist

## Cross-Environment Calling Solution: Python HTTP Proxy Bridge

The most practical solution is a Python HTTP server running on Windows that wraps mcporter calls for WSL clients.

### Server Setup (Windows PowerShell)

Save this to `C:\Users\Administrator\mcporter_proxy.py`:

```python
#!/usr/bin/env python3
"""MCP Proxy: expose mcporter tools via HTTP for WSL/Linux clients"""
import subprocess, json
from http.server import HTTPServer, BaseHTTPRequestHandler

PORT = 8099
MCPORTER = r'C:\Users\Administrator\AppData\Roaming\npm\mcporter.cmd'

class MCPHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/health':
            self.send_response(200)
            self.send_header('Content-Type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps({'status': 'ok'}).encode())
        elif self.path == '/tools':
            try:
                result = subprocess.run([MCPORTER, 'list'], capture_output=True, text=True, timeout=60)
                self.send_response(200)
                self.send_header('Content-Type', 'application/json')
                self.end_headers()
                self.wfile.write(json.dumps({'stdout': result.stdout, 'stderr': result.stderr, 'returncode': result.returncode}).encode())
            except Exception as e:
                self.send_response(500); self.send_header('Content-Type', 'application/json'); self.end_headers()
                self.wfile.write(json.dumps({'error': str(e)}).encode())
        else:
            self.send_response(404); self.end_headers()

    def do_POST(self):
        if self.path == '/call':
            try:
                content_length = int(self.headers['Content-Length'])
                body = json.loads(self.rfile.read(content_length))
                server = body.get('server', 'windows-mcp')
                tool = body.get('tool', '')
                args = body.get('args', {})
                cmd = [MCPORTER, 'call', server, tool] + [f'{k}={v}' for k, v in args.items()]
                result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
                self.send_response(200); self.send_header('Content-Type', 'application/json'); self.end_headers()
                self.wfile.write(json.dumps({'stdout': result.stdout, 'stderr': result.stderr, 'returncode': result.returncode}).encode())
            except Exception as e:
                self.send_response(500); self.send_header('Content-Type', 'application/json'); self.end_headers()
                self.wfile.write(json.dumps({'error': str(e)}).encode())
        else:
            self.send_response(404); self.end_headers()

    def log_message(self, format, *args): print(f"[HTTP] {format % args}")

if __name__ == '__main__':
    print(f"Starting MCP Proxy on port {PORT}...")
    HTTPServer(('0.0.0.0', PORT), MCPHandler).serve_forever()
```

Run it:
```powershell
cd C:\Users\Administrator
python mcporter_proxy.py
```

### Client Usage (WSL/Linux)

Find Windows IP from WSL: `ip route show | grep default` (gateway IP, e.g. `172.19.128.1`)

Call from WSL:
```bash
# Health check
curl http://172.19.128.1:8099/health

# List tools
curl http://172.19.128.1:8099/tools

# Call a tool (POST JSON)
curl -X POST -H "Content-Type: application/json" \
  -d '{"server":"windows-mcp","tool":"create_process","args":{"command":"notepad.exe"}}' \
  http://172.19.128.1:8099/call
```

### Hermes Integration

Add to `~/.hermes/config.yaml` to use Hermes native MCP with this proxy:
```yaml
mcp_servers:
  windows-mcp:
    url: "http://172.19.128.1:8099/call"
```

### Known Issues

- **WinError 2 from subprocess**: mcporter.cmd may not resolve `mcporter` internally when called via `subprocess.run([...])`. Use list form `subprocess.run([MCPORTER, 'call', ...])` without `shell=True`.
- **mcporter.cmd path**: Full path `C:\Users\Administrator\AppData\Roaming\npm\mcporter.cmd` is more reliable than relying on PATH.
- **daemon must be running first**: `mcporter daemon start` must be running on Windows before the proxy can successfully call mcporter tools.

### Verified: WSL → Windows binary execution

| Command | Result |
|---------|--------|
| `mcporter` from WSL | Timeout (30s+) |
| `cmd.exe /c mcporter` from WSL | `Exec format error` |
| `powershell.exe -Command mcporter` from WSL | `Exec format error` |
| `mcporter` from Windows PowerShell | Works correctly |
