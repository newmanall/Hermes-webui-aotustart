# 使用指南

## 快速开始

### 1. 解压到 WSL 环境

```bash
cd ~
# 将压缩包解压到合适位置
tar -xzf webui-autostart-package.tar.gz
cd webui-autostart-package
```

### 2. 配置环境变量（可选）

编辑 `~/.bashrc` 添加：
```bash
export HERMES_WEBUI_REPO="$HOME/projects/hermes-webui-cn"
export HERMES_WEBUI_PORT=8787
```

### 3. 选择启动方式

#### 方式 A：WSL 会话启动（推荐）

```bash
# 添加到 ~/.bashrc
cat >> ~/.bashrc << 'EOF'
if [ -x "$HOME/projects/hermes-webui-cn/scripts/wsl/hermes_webui_autostart.sh" ]; then
  HERMES_WEBUI_REPO="$HOME/projects/hermes-webui-cn" \
    "$HOME/projects/hermes-webui-cn/scripts/wsl/hermes_webui_autostart.sh" >/dev/null 2>&1 &
fi
EOF

# 生效
source ~/.bashrc

# 验证
curl http://127.0.0.1:8787/health
```

#### 方式 B：Windows 任务计划程序（开机自启）

在 Windows PowerShell 中运行：
```powershell
cd path\to\webui-autostart-package
.\setup_webui_autostart.ps1 `
  -WslScriptPath "/home/你的用户名/projects/hermes-webui-cn/scripts/wsl/hermes_webui_autostart.sh" `
  -Distro "Ubuntu" `
  -RunNow
```

### 4. 管理 WebUI

```bash
# 查看状态
./ctl.sh status

# 手动启动/停止
./ctl.sh start
./ctl.sh stop

# 查看日志
./ctl.sh logs --follow
```

## 文件说明

| 文件 | 用途 |
|------|------|
| `hermes_webui_autostart.sh` | WSL 自启动脚本（核心） |
| `setup_webui_autostart.ps1` | Windows 任务计划配置脚本 |
| `ctl.sh` | WebUI 控制脚本 |
| `start.sh` | WebUI 启动入口 |
| `wsl-autostart.md` | 详细配置文档 |
| `start-webui.sh` | 用户自定义快速启动脚本 |

## 故障排查

```bash
# 查看自启动日志
tail -f ~/.hermes/webui/logs/webui_autostart.log

# 查看 WebUI 日志
tail -f ~/.hermes/webui/logs/hermes_webui.log

# 检查进程
ps aux | grep bootstrap.py

# 手动测试健康检查
curl -v http://127.0.0.1:8787/health
```

## 卸载

### 移除 WSL 会话启动
从 `~/.bashrc` 中删除相关代码块。

### 移除 Windows 任务计划
```powershell
Unregister-ScheduledTask -TaskName HermesWebUIAutoStart -Confirm:$false
```
