---
name: electron-vue3-security-hardening
description: Electron + Vue3 项目安全审计与修复 — nodeIntegration/contextIsolation/preload 修复流程
---

# Electron + Vue3 项目安全审计与修复

## 触发条件
- Electron 项目中 `nodeIntegration: true` + `contextIsolation: false`
- preload.js 为空或未配置
- 发现硬编码 token/API Key

## 修复步骤

### 1. 分析 main.js webPreferences
```js
// 错误示例（不安全）
webPreferences: {
  contextIsolation: false,
  nodeIntegration: true,
  sandbox: false,
}

// 正确示例（安全）
webPreferences: {
  contextIsolation: true,
  nodeIntegration: false,
  sandbox: false,
  preload: path.join(__dirname, 'preload.js'),
}
```

### 2. 重写 preload.js（使用 contextBridge）
```js
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('electronAPI', {
  // 文件系统
  selectRootDir: () => ipcRenderer.invoke('fs:selectRootDir'),
  getRootDir:    () => ipcRenderer.invoke('fs:getRootDir'),
  setRootDir:    (p) => ipcRenderer.invoke('fs:setRootDir', p),
  readDir:       (dirPath) => ipcRenderer.invoke('fs:readDir', dirPath),
  readFile:      (filePath) => ipcRenderer.invoke('fs:readFile', filePath),
  writeFile:     (fp, content) => ipcRenderer.invoke('fs:writeFile', fp, content),
  deleteFile:    (fp) => ipcRenderer.invoke('fs:deleteFile', fp),
  renameFile:    (oldPath, newPath) => ipcRenderer.invoke('fs:renameFile', oldPath, newPath),
  copyFile:      (src, dest) => ipcRenderer.invoke('fs:copyFile', src, dest),
  selectFile:    (opts) => ipcRenderer.invoke('fs:selectFile', opts),
  openPath:      (p) => ipcRenderer.invoke('fs:openPath', p),

  // API Key
  getApiKey:     () => ipcRenderer.invoke('fs:getApiKey'),
  setApiKey:     (key) => ipcRenderer.invoke('fs:setApiKey', key),

  // Skill 文件
  skillUploadFile:   (id, src) => ipcRenderer.invoke('skill:uploadFile', id, src),
  skillListFiles:    (id) => ipcRenderer.invoke('skill:listFiles', id),
  skillDownloadFile: (sp) => ipcRenderer.invoke('skill:downloadFile', sp),
  skillDeleteFile:   (sp) => ipcRenderer.invoke('skill:deleteFile', sp),

  // AI 窗口
  openAiWindow:   () => ipcRenderer.invoke('ai:openWindow'),

  // 系统
  systemOpenPath: (p) => ipcRenderer.invoke('system:openPath', p),
})
```

### 3. 清除硬编码 token
```js
// 错误
const HARDCODED_TOKEN = '9da0b5...'

// 正确
const HARDCODED_TOKEN = ''
```

### 4. 验证步骤
- 打包后运行 exe，确认 electronAPI 调用正常
- 确认 AI 窗口能打开并连接 Gateway
- 检查 DevTools Console 无 "window.electronAPI is not defined" 报错

## 坑点
- `contextIsolation: true` 后 Vue 组件里所有 `window.electronAPI` 调用必须通过 preload 暴露，否则全报错
- preload 路径用 `path.join(__dirname, 'preload.js')`，不要用相对路径
- 打包后路径会变，`__dirname` 在 packaged 环境下指向 `app.asar` 内部，仍有效
