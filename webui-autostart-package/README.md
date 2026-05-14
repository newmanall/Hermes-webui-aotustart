# Hermes WebUI 中文版的 WSL 自启动方案

## 项目概述

本方案实现了 **Hermes WebUI 中文版** 在 WSL2 环境下的自动启动功能，支持两种启动方式：
1. **WSL 会话启动** — 打开 WSL 终端时自动启动
2. **Windows 任务计划程序** — Windows 登录时自动启动

---

## 核心文件清单

| 文件 | 说明 |
|------|------|
| `docs/wsl-autostart.md` | 完整的自启动配置文档 |
| `scripts/wsl/hermes_webui_autostart.sh` | WSL 自启动脚本（核心） |
| `scripts/windows/setup_webui_autostart.ps1` | Windows 任务计划程序配置脚本 |
| `ctl.sh` | WebUI 控制脚本（start/stop/status/logs） |
| `start.sh` | WebUI 启动入口脚本 |
| `bootstrap.py` | WebUI 引导程序 |

---

## 环境配置

### 默认配置
- **监听地址**: `127.0.0.1:8787`
- **日志目录**: `~/.hermes/webui/logs/`
- **PID 文件**: `~/.hermes/webui/logs/hermes-webui.pid`

### 环境变量
```bash
HERMES_WEBUI_REPO      # WebUI 仓库路径（默认：脚本所在仓库根目录）
HERMES_WEBUI_LOG_DIR   # 日志目录（默认：~/.hermes/webui/logs）
HERMES_WEBUI_HOST      # 监听主机（默认：127.0.0.1）
HERMES_WEBUI_PORT      # 监听端口（默认：8787）
HERMES_WEBUI_PID_FILE  # PID 文件路径
```

---

## 使用方法

### 方式一：WSL 会话启动

1. 在 WSL 中使脚本可执行：
```bash
cd ~/projects/hermes-webui-cn
chmod +x scripts/wsl/hermes_webui_autostart.sh
```

2. 添加到 `~/.bashrc` 或 `~/.profile`：
```bash
if [ -x "$HOME/hermes-webui/scripts/wsl/hermes_webui_autostart.sh" ]; then
  HERMES_WEBUI_REPO="$HOME/hermes-webui" \
    "$HOME/hermes-webui/scripts/wsl/hermes_webui_autostart.sh" >/dev/null 2>&1 &
fi
```

3. 验证：
```bash
curl -fsS http://127.0.0.1:8787/health
```

### 方式二：Windows 任务计划程序启动

在 Windows PowerShell 中运行：
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\scripts\windows\setup_webui_autostart.ps1 `
  -WslScriptPath "/home/your-user/hermes-webui/scripts/wsl/hermes_webui_autostart.sh" `
  -Distro "Ubuntu"
```

参数说明：
- `-Distro`: WSL 发行版名称（可选，默认使用默认发行版）
- `-TaskName`: 任务名称（可选，默认 `HermesWebUIAutoStart`）
- `-RunNow`: 注册后立即运行
- `-WhatIf`: 预览操作
- `-SkipValidation`: 跳过路径验证

---

## 控制命令

```bash
# 启动
./ctl.sh start

# 停止
./ctl.sh stop

# 重启
./ctl.sh restart

# 查看状态
./ctl.sh status

# 查看日志
./ctl.sh logs
./ctl.sh logs --lines 200 --follow
```

---

## 故障排查

### 查看日志
```bash
tail -n 80 ~/.hermes/webui/logs/webui_autostart.log
tail -n 80 ~/.hermes/webui/logs/hermes_webui.log
```

### 常见问题

| 症状 | 可能原因 | 解决方案 |
|------|----------|----------|
| 任务存在但 WebUI 无法访问 | WSL 脚本路径错误 | 重新运行 PowerShell 脚本，指定正确的 `-WslScriptPath` |
| 仅打开 WSL 后才启动 | 使用了 WSL 会话启动而非任务计划程序 | 安装 Windows 计划任务 |
| 多次登录触发多次启动 | 正常 Windows 启动行为 | 脚本有锁机制，应记录 `already running` |
| 健康检查失败但 PID 存在 | WebUI 仍在启动或端口不同 | 检查 `HERMES_WEBUI_PORT` 和日志 |

---

## 安全特性

1. **防重复启动** — 使用 `flock` 文件锁 + PID 文件 + 健康检查三重机制
2. **权限最小化** — Windows 任务以当前用户最低权限运行
3. **幂等性** — 多次运行不会产生重复进程或重复任务

---

## 项目信息

- **项目名称**: hermes-webui-cn
- **描述**: Hermes WebUI 的中文本地化分支
- **上游**: https://github.com/nesquena/hermes-webui
- **默认端口**: 8787
- **Python 版本**: 3.10+

---

## 快速启动脚本（用户自定义）

用户可在 WSL 中创建 `~/start-webui.sh`：
```bash
#!/bin/bash
cd ~/projects/hermes-webui-cn && python3 bootstrap.py --no-browser
```

---

## 许可证

遵循 Hermes WebUI 上游许可证。
