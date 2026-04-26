---
name: wechat-cli-wsl-install
description: 在 WSL 环境中安装 wechat-cli（微信本地数据库 CLI）的方法，包含源码下载、pip 安装和命令包装
triggers:
  - wechat-cli
  - 微信 cli
  - 微信 本地 数据库
---

# wechat-cli 在 WSL 的安装方法

## 背景
wechat-cli 是 `huohuoer/wechat-cli`，支持从本地微信数据库查询聊天记录、联系人等。npm 包只有 macOS 二进制，GitHub clone 需要认证，因此需要变通方案。

## 安装步骤

### 1. 确定源码下载方式
GitHub API + raw.githubusercontent.com 可以逐文件下载，不依赖 git clone 认证。

### 2. 获取文件列表
```bash
curl -sL "https://api.github.com/repos/huohuoer/wechat-cli/git/trees/main?recursive=1" | python3 -c "import sys,json; data=json.load(sys.stdin); [print(f['path']) for f in data.get('tree',[]) if f['type']=='blob']"
```

### 3. 下载源码
```python
import subprocess, json, os
result = subprocess.run(['curl', '-sL', 'https://api.github.com/repos/huohuoer/wechat-cli/git/trees/main?recursive=1'], capture_output=True, text=True)
data = json.loads(result.stdout)
files = [f for f in data['tree'] if f['type'] == 'blob' and f['path'].startswith(('wechat_cli/', 'entry.py', 'pyproject.toml'))]
dest = '/tmp/wechat-cli-src'
os.makedirs(dest, exist_ok=True)
for f in files:
    url = f"https://raw.githubusercontent.com/huohuoer/wechat-cli/main/{f['path']}"
    local = os.path.join(dest, f['path'])
    os.makedirs(os.path.dirname(local), exist_ok=True)
    subprocess.run(['curl', '-sL', url, '-o', local], timeout=15)
```

### 4. 安装 Python 依赖（本 WSL 无 pip，需先安装）
```bash
curl -sS https://bootstrap.pypa.io/get-pip.py -o /tmp/get-pip.py
python3 /tmp/get-pip.py --user
python3 -m pip install click pycryptodome zstandard --user
```

### 5. 创建命令包装脚本
```bash
cat > ~/.local/bin/wechat-cli << 'EOF'
#!/bin/bash
cd /tmp/wechat-cli-src && python3 entry.py "$@"
EOF
chmod +x ~/.local/bin/wechat-cli
```

### 6. 验证
```bash
wechat-cli --version  # → wechat-cli, version 0.2.4
```

## 关键坑点

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| `npm install -g wechat-cli` 空包 | 包名错误 + 无 Linux 二进制 | 用 `@canghe_ai/wechat-cli` |
| npm 包装完只有 macOS binary | platform-specific optionalDependencies | 不用 npm 包，直接拉源码 |
| `git clone` 需要认证 | GitHub 要求登录 | 用 raw.githubusercontent.com 单文件下载 |
| pip 不存在 | WSL 最小化安装 | 用 get-pip.py 安装 |
| pip install -e 超时 | 网络或依赖解析慢 | 直接用 entry.py + 包装脚本 |

## 微信数据库定位方法（WSL）

微信桌面版数据目录不在常规位置，需通过注册表定位：

### 方法：通过 config .ini 定位（Windows）
```powershell
# 微信配置目录
$iniDir = "$env:APPDATA\Tencent\xwechat\config"
# 读 .ini 文件内容（微信数据根目录）
Get-Content "$iniDir\*.ini" -Raw
# 结果如：F:\
```

### 典型数据目录结构
```
F:\xwechat_files\
  all_users\
  Backup\
  wxid_xxxxxxxxx\        ← 账号1
    db_storage\
      message\           ← 聊天记录
      contact\            ← 联系人
      session\            ← 会话
      ...
  wxid_yyyyyyyyy\        ← 账号2
    db_storage\
      ...
```

### WSL 路径映射
- `F:\` → `/mnt/f/`
- 完整 db_storage：`/mnt/f/xwechat_files/wxid_xxxxxxxxx/db_storage`

## 初始化（必须先在 Windows 启动微信桌面版）

```bash
# Linux/WSL（需要 root 权限扫描进程内存）
sudo wechat-cli init --db-dir /mnt/f/xwechat_files/wxid_xxxxxxxxx/db_storage
```

init 需要从微信进程（Weixin.exe）内存中扫描提取密钥，所以：
- 微信桌面版必须在 Windows 上**保持运行状态**
- 需要 sudo/管理员权限

### 初始化后的数据位置
- 配置：`~/.wechat-cli/config.json`
- 密钥：`~/.wechat-cli/all_keys.json`
