D: 盘有审计项目 audit-platform-new2（Vue3 + Electron 桌面应用，接入 Hermes Gateway ws://127.0.0.1:18789）

electron main.js 安全修复要点：
- nodeIntegration: false, contextIsolation: true 时要配 preload: path.join(__dirname, 'preload.js')
- preload.js 需用 contextBridge.exposeInMainWorld('electronAPI', {...}) 暴露 API
- 硬编码 token 危险，必须清空

WSL Windows 用户名问题：Windows 用户名是 "001"（SID S-1-5-21-...-500），WSL 用户名是 shumi。schtasks 用 Windows 用户名。