---
name: hermes-gateway-fault-tolerance
description: Configure Hermes Gateway to survive platform connection failures by enabling the api_server fallback platform and other fault-tolerance settings
category: devops
tags: [hermes, gateway, resilience, fault-tolerance]
created: 2026-04-27
---

# Hermes Gateway 容错配置

## 问题

Gateway 只配置了 QQ Bot 一个消息平台时，如果 QQ 连接失败（超时/网络问题），整个 Gateway 进程会直接退出：

```
ERROR gateway.run: Gateway failed to connect any configured messaging platform: qqbot: QQ startup failed: Failed to get QQ Bot gateway URL: httpx.ConnectTimeout
```

日志位置：`~/.hermes/logs/errors.log`、`~/.hermes/logs/agent.log`

## 根因

`gateway/run.py` 第 2153-2161 行：所有启用的平台连接失败时，如果没有任何一个平台在 `notify_exclude_platforms` 列表中，Gateway 直接 `return False`（退出）。

`api_server` 和 `webhook` 在该列表中，所以它们的连接失败不会导致致命退出。

## 解法：启用 api_server 平台

在 `~/.hermes/.env` 末尾添加：

```bash
# API SERVER (local OpenAI-compatible HTTP API — gateway WebSocket fallback)
API_SERVER_ENABLED=true
API_SERVER_KEY=hermes-local
```

重启 Gateway：`hermes gateway run --replace`

效果：Gateway 同时运行 api_server（127.0.0.1:8642）和 qqbot 两个平台，即使 QQ Bot 再次断线，Gateway 不会退出。

## 验证

```bash
curl http://127.0.0.1:8642/health/detailed
```

正常输出：
```json
{
  "gateway_state": "running",
  "platforms": {
    "qqbot": {"state": "connected"},
    "api_server": {"state": "connected"}
  }
}
```

## 相关环境变量（gateway/config.py 第 1050-1075 行）

| 变量 | 作用 |
|------|------|
| `API_SERVER_ENABLED` | 启用 api_server 平台 |
| `API_SERVER_KEY` | API 密钥（本地可设简单值）|
| `API_SERVER_PORT` | 端口（默认 8642）|
| `API_SERVER_HOST` | 监听地址（默认 127.0.0.1）|
| `API_SERVER_CORS_ORIGINS` | 允许的跨域来源（逗号分隔）|

## 其他平台容错方式

如果希望 QQ Bot 本身更稳定，检查网络连通性：
```bash
curl -v https://api.sgroup.qq.com/websocket
```
国内可能需要代理（设置 `http_proxy`/`https_proxy` 环境变量）。
