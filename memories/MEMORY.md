D: 盘有审计项目 audit-platform-new2（Vue3 + Electron 桌面应用，接入 Hermes Gateway ws://127.0.0.1:18789）

electron main.js 安全修复要点：
- nodeIntegration: false, contextIsolation: true 时要配 preload: path.join(__dirname, 'preload.js')
- preload.js 需用 contextBridge.exposeInMainWorld('electronAPI', {...}) 暴露 API
- 硬编码 token 危险，必须清空

WSL Windows 用户名问题：Windows 用户名是 "001"（SID S-1-5-21-...-500），WSL 用户名是 shumi。schtasks 用 Windows 用户名。
§
华为云盘(D:\华为云盘)里有AI新架构研究目录，含人脑启发AI架构蓝图、路线A(Mamba/SSM)、路线B(符号神经混合/TTT)、路线C(自主进化)、路线D(轻量MoE)四套方案。2026-04-05曾与老板讨论模型架构探索。

当前思路：两条腿走路 — (1)TTT变体小模型研究验证新架构，(2)认知架构层外挂现有LLM。

硬件确认待做：问清用户是A100还是3090，以定TTT训练方案。