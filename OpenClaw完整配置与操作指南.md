# OpenClaw 完整配置与操作指南

> **项目地址**：https://github.com/openclaw/openclaw
> 
> **官方文档**：https://docs.openclaw.ai/
> 
> **更新日期**：2026年4月10日
> 
> **Star数量**：353k+ | **Fork数量**：71.3k+

---

## 📋 目录

- [OpenClaw 简介](#openclaw-简介)
- [核心概念](#核心概念)
- [安装部署](#安装部署)
- [配置文件详解](#配置文件详解)
- [常用命令](#常用命令)
- [消息平台配置](#消息平台配置)
- [工具与技能](#工具与技能)
- [高级功能](#高级功能)
- [故障排查](#故障排查)

---

## OpenClaw 简介

### 什么是 OpenClaw？

**OpenClaw** 是一个运行在个人设备上的 **AI 个人助手**，具有以下特点：

- 🏠 **本地优先**：在你自己的设备上运行
- 💬 **多平台支持**：连接 WhatsApp、Telegram、Slack、Discord 等 20+ 消息平台
- 🎙️ **语音交互**：支持语音唤醒和持续对话（macOS/iOS/Android）
- 🎨 **实时画布**：agent 驱动的可视化工作空间
- 🔧 **丰富工具**：浏览器控制、Canvas、节点、定时任务等

### 核心架构

```
WhatsApp/Telegram/Slack/Discord/...
               │
               ▼
┌───────────────────────────────┐
│         Gateway               │
│      (控制平面)                │
│    ws://127.0.0.1:18789       │
└──────────────┬────────────────┘
               │
               ├─ Pi Agent (RPC)
               ├─ CLI (openclaw …)
               ├─ WebChat UI
               ├─ macOS App
               └─ iOS/Android Nodes
```

---

## 核心概念

### Gateway（网关）

**功能**：
- WebSocket 控制平面
- 管理会话、频道、工具和事件
- 默认端口：18789
- 绑定地址：loopback (127.0.0.1)

**启动方式**：
```bash
openclaw gateway --port 18789 --verbose
```

### Agent（代理）

**功能**：
- Pi agent 运行时（RPC 模式）
- 处理工具流和块流
- 管理会话和上下文

### Session（会话）

**类型**：
- `main`：直接聊天
- `group`：群组隔离
- 激活模式、队列模式、回复模式

### Channels（频道）

**支持的平台**：
- WhatsApp（Baileys）
- Telegram（grammY）
- Slack（Bolt）
- Discord（discord.js）
- Google Chat
- Signal
- iMessage/BlueBubbles
- IRC
- Microsoft Teams
- Matrix
- 还有 10+ 其他平台...

---

## 安装部署

### 系统要求

| 组件 | 要求 |
|------|------|
| **Node.js** | v24（推荐）或 v22.16+ |
| **操作系统** | macOS / Linux / Windows (WSL2) |
| **包管理器** | npm / pnpm / bun |

---

### Node.js 安装配置

**使用 NVM（推荐，支持多版本管理）**

**macOS/Linux 安装 NVM**：

1. **安装 NVM**：
```bash
# 使用 curl 安装
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 或使用 wget
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

2. **配置环境变量**：

安装完成后，添加到 shell 配置文件：

```bash
# 对于 zsh (macOS 默认)
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.zshrc
echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.zshrc

# 重新加载配置
source ~/.zshrc
```

```bash
# 对于 bash
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.bashrc

# 重新加载配置
source ~/.bashrc
```

3. **验证 NVM 安装**：
```bash
nvm --version
# 输出示例：0.39.7
```

4. **安装 Node.js v24**：
```bash
# 安装 Node.js v24 (推荐)
nvm install 24

# 或安装最新 LTS 版本
nvm install --lts

# 设置默认版本
nvm alias default 24

# 验证安装
node --version  # 输出：v24.x.x
npm --version   # 输出：10.x.x
```

5. **NVM 常用命令**：
```bash
# 列出已安装版本
nvm ls

# 列出所有可用版本
nvm ls-remote

# 切换版本
nvm use 22

# 卸载版本
nvm uninstall 22

# 查看当前使用版本
nvm current
```

---

### 推荐安装方式

#### 1. 全局安装（推荐）

```bash
# npm
npm install -g openclaw@latest

# pnpm（推荐）
pnpm add -g openclaw@latest

# bun
bun add -g openclaw@latest
```

#### 2. 运行引导安装

```bash
openclaw onboard --install-daemon
```

**引导安装会自动**：
- 配置 Gateway
- 设置工作空间
- 配置频道
- 安装技能
- 安装守护进程（launchd/systemd）

#### 3. 从源码安装（开发者）

```bash
# 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 安装依赖（推荐使用 pnpm）
pnpm install

# 构建 UI
pnpm ui:build

# 构建项目
pnpm build

# 运行引导
pnpm openclaw onboard --install-daemon
```

### 验证安装

```bash
# 检查版本
openclaw --version

# 健康检查
openclaw doctor

# 设置agents超时
openclaw config set agents.defaults.llm.idleTimeoutSeconds 600

# 日志
tail -f ～/.openclaw/logs/*.log
tail -f ~/.ollama/logs/*.log
```

---

## 配置文件详解

### 配置文件位置

**默认路径**：`~/.openclaw/openclaw.json`

### 最小配置示例

```json
{
  "agent": {
    "model": "<provider>/<model-id>"
  }
}
```

### 完整配置结构

```json
{
  "agent": {
    "model": "openai/gpt-4",
    "workspace": "~/.openclaw/workspace"
  },
  
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your-secure-token"
    },
    "tailscale": {
      "mode": "off"
    }
  },
  
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+1234567890"]
    },
    "telegram": {
      "enabled": true,
      "botToken": "123456:ABCDEF"
    },
    "discord": {
      "enabled": true,
      "token": "your-discord-bot-token"
    }
  },
  
  "browser": {
    "enabled": true,
    "color": "#FF4500"
  },
  
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  }
}
```

### 关键配置项

#### 1. Agent 配置

| 配置项 | 说明 | 示例 |
|--------|------|------|
| `agent.model` | LLM 模型 | `"openai/gpt-4"` |
| `agent.workspace` | 工作空间路径 | `"~/.openclaw/workspace"` |

#### 2. Gateway 配置

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `gateway.port` | WebSocket 端口 | `18789` |
| `gateway.bind` | 绑定地址 | `"loopback"` |
| `gateway.auth.mode` | 认证模式 | `"token"` / `"password"` |
| `gateway.auth.token` | 访问令牌 | - |

#### 3. Channels 配置

**WhatsApp**：
```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+1234567890", "+9876543210"],
      "groups": {
        "*": {
          "requireMention": true
        }
      }
    }
  }
}
```

**Telegram**：
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456:ABCDEF",
      "allowFrom": ["@username"],
      "groups": {
        "*": {
          "requireMention": true
        }
      }
    }
  }
}
```

**Discord**：
```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "your-bot-token",
      "allowFrom": ["user-id-1", "user-id-2"],
      "guilds": {
        "server-id": {
          "channels": ["channel-id-1", "channel-id-2"]
        }
      }
    }
  }
}
```

#### 4. 安全配置

**DM 策略**（Direct Message）：
```json
{
  "channels": {
    "telegram": {
      "dmPolicy": "pairing"
    },
    "discord": {
      "dmPolicy": "pairing"
    }
  }
}
```

**DM 策略选项**：
- `"pairing"`：需要配对码（默认，安全）
- `"open"`：开放（需显式配置 `allowFrom: ["*"]`）

**沙箱模式**：
```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "allowlist": ["bash", "process", "read", "write"],
        "denylist": ["browser", "canvas", "nodes"]
      }
    }
  }
}
```

---

## 常用命令

### Gateway 管理

```bash
# 启动 Gateway
openclaw gateway --port 18789 --verbose

# 后台运行（安装守护进程后）
# 自动通过 systemd/launchd 管理

# 检查 Gateway 状态
openclaw doctor
```

### Onboarding（引导配置）

```bash
# 完整引导
openclaw onboard

# 安装守护进程
openclaw onboard --install-daemon

# 重新配置
openclaw onboard --reconfigure
```

### 消息发送

```bash
# 发送文本消息
openclaw message send --to +1234567890 --message "Hello"

# 发送到 Telegram
openclaw message send --channel telegram --to @username --message "Hi"

# 发送到 Discord
openclaw message send --channel discord --to user-id --message "Test"
```

### Agent 交互

```bash
# 直接与 Agent 对话
openclaw agent --message "今天天气怎么样？"

# 指定思考级别
openclaw agent --message "分析这个问题" --thinking high

# 回复到指定频道
openclaw agent --message "生成报告" --reply-to telegram:@username
```

### 频道管理

```bash
# 登录频道（如 WhatsApp）
openclaw channels login --channel whatsapp

# 列出已连接频道
openclaw channels list

# 登出
openclaw channels logout --channel whatsapp
```

### 配对管理

```bash
# 批准配对请求
openclaw pairing approve telegram 123456

# 列出待配对
openclaw pairing list

# 拒绝配对
openclaw pairing reject telegram 123456
```

### 更新管理

```bash
# 更新到最新版本
openclaw update

# 切换发布频道
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

### 诊断工具

```bash
# 健康检查
openclaw doctor

# 检查特定频道
openclaw doctor --channel telegram

# 查看日志
openclaw logs --tail 100

# 查看配置
openclaw config show
```

### 技能管理

```bash
# 搜索技能
openclaw skills search "keyword"

# 安装技能
openclaw skills install skill-name

# 列出已安装技能
openclaw skills list

# 移除技能
openclaw skills remove skill-name
```

### 节点管理

```bash
# 列出节点（iOS/Android/macOS）
openclaw nodes list

# 查看节点详情
openclaw nodes describe node-id

# 调用节点功能
openclaw nodes invoke node-id camera.snap
```

---

## 消息平台配置

### 1. WhatsApp 配置

**步骤**：

1. **登录设备**：
```bash
openclaw channels login --channel whatsapp
```

2. **扫描二维码**：
   - 命令会显示二维码
   - 使用 WhatsApp 扫描

3. **配置白名单**：
```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": [
        "+1234567890",
        "+9876543210"
      ],
      "groups": {
        "*": {
          "requireMention": true
        }
      }
    }
  }
}
```

4. **凭证存储**：
   - 存储在 `~/.openclaw/credentials/`

---

### 2. Telegram 配置

**步骤**：

1. **创建机器人**：
   - 打开 Telegram，搜索 `@BotFather`
   - 发送 `/newbot`
   - 按提示创建机器人
   - 获取 Bot Token

2. **配置**：
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456:ABCDEF-your-token",
      "allowFrom": ["@username1", "@username2"],
      "groups": {
        "*": {
          "requireMention": true
        }
      },
      "webhookUrl": "https://your-domain.com/webhook",
      "webhookSecret": "your-secret"
    }
  }
}
```

3. **环境变量方式**（优先级更高）：
```bash
export TELEGRAM_BOT_TOKEN="123456:ABCDEF"
```

---

### 3. Slack 配置

**步骤**：

1. **创建 Slack App**：
   - 访问 https://api.slack.com/apps
   - 创建新应用
   - 获取 Bot Token 和 App Token

2. **配置**：
```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "botToken": "xoxb-your-bot-token",
      "appToken": "xapp-your-app-token",
      "allowFrom": ["U12345", "U67890"]
    }
  }
}
```

3. **环境变量方式**：
```bash
export SLACK_BOT_TOKEN="xoxb-..."
export SLACK_APP_TOKEN="xapp-..."
```

---

### 4. Discord 配置

**步骤**：

1. **创建 Discord Bot**：
   - 访问 https://discord.com/developers/applications
   - 创建应用并添加 Bot
   - 获取 Bot Token
   - 启用必要的 Intents（Message Content Intent）

2. **配置**：
```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "your-discord-bot-token",
      "allowFrom": ["user-id-1", "user-id-2"],
      "guilds": {
        "server-id-1": {
          "channels": ["channel-id-1", "channel-id-2"]
        }
      },
      "dmPolicy": "pairing",
      "commands": {
        "native": true,
        "text": true
      }
    }
  }
}
```

3. **邀请 Bot 到服务器**：
   - 在开发者门户生成 OAuth2 URL
   - 选择权限：Send Messages, Read Message History, Use Slash Commands
   - 访问 URL 邀请 Bot

---

### 5. Signal 配置

**要求**：
- 需要安装 `signal-cli`

**配置**：
```json
{
  "channels": {
    "signal": {
      "enabled": true,
      "accountId": "+1234567890",
      "allowFrom": ["+9876543210"]
    }
  }
}
```

---

### 6. iMessage 配置

**选项 1：BlueBubbles（推荐）**

```json
{
  "channels": {
    "bluebubbles": {
      "enabled": true,
      "serverUrl": "http://your-mac:1234",
      "password": "your-password",
      "webhookPath": "/webhook/bluebubbles"
    }
  }
}
```

**选项 2：Legacy iMessage（仅 macOS）**

```json
{
  "channels": {
    "imessage": {
      "enabled": true,
      "allowFrom": ["+1234567890"],
      "groups": {
        "*": {}
      }
    }
  }
}
```

---

## 工具与技能

### 内置工具

#### 1. Browser（浏览器控制）

**配置**：
```json
{
  "browser": {
    "enabled": true,
    "color": "#FF4500",
    "profiles": {
      "default": {}
    }
  }
}
```

**功能**：
- 自动化浏览器操作
- 网页截图
- 表单填写
- 文件上传

#### 2. Canvas（画布）

**功能**：
- Agent 驱动的可视化工作空间
- A2UI 支持
- 实时协作

**使用**：
```bash
# 通过 agent 控制
openclaw agent --message "在 canvas 上画一个流程图"
```

#### 3. Nodes（节点）

**支持的节点类型**：
- **macOS**：`system.run`, `system.notify`, `camera.snap`, `screen.record`
- **iOS**：`camera.snap`, `location.get`, `screen.record`
- **Android**：`camera.snap`, `location.get`, `notifications`, `sms`, `contacts`

**使用**：
```bash
# 列出节点
openclaw nodes list

# 调用节点
openclaw nodes invoke macos-node camera.snap
```

#### 4. Cron（定时任务）

**配置**：
```json
{
  "cron": {
    "enabled": true,
    "jobs": [
      {
        "name": "daily-report",
        "schedule": "0 9 * * *",
        "action": "agent",
        "message": "生成每日报告",
        "replyTo": "telegram:@me"
      }
    ]
  }
}
```

#### 5. Sessions（会话工具）

**Agent-to-Agent 通信**：
- `sessions_list`：列出活动会话
- `sessions_history`：获取会话历史
- `sessions_send`：向其他会话发送消息

### 技能系统

#### 技能类型

1. **Bundled Skills**：内置技能
2. **Managed Skills**：从 ClawHub 安装
3. **Workspace Skills**：本地自定义技能

#### 技能目录结构

```
~/.openclaw/workspace/skills/
├── skill-name-1/
│   └── SKILL.md
├── skill-name-2/
│   └── SKILL.md
└── ...
```

#### 创建自定义技能

**SKILL.md 示例**：
```markdown
# 技能名称

## 描述
这是一个示例技能

## 工具
- bash
- read
- write

## 使用方法
...
```

#### ClawHub 技能注册表

**官网**：https://clawhub.com/

**搜索技能**：
```bash
openclaw skills search "weather"
```

**安装技能**：
```bash
openclaw skills install weather-info
```

---

## 高级功能

### 1. Tailscale 远程访问

**配置 Serve（仅 Tailnet）**：
```json
{
  "gateway": {
    "tailscale": {
      "mode": "serve"
    },
    "bind": "loopback"
  }
}
```

**配置 Funnel（公开访问）**：
```json
{
  "gateway": {
    "tailscale": {
      "mode": "funnel",
      "resetOnExit": true
    },
    "bind": "loopback",
    "auth": {
      "mode": "password",
      "password": "your-secure-password"
    }
  }
}
```

### 2. SSH 隧道远程访问

```bash
# 本地转发
ssh -L 18789:localhost:18789 user@remote-host

# 反向转发
ssh -R 18789:localhost:18789 user@remote-host
```

### 3. Docker 部署

```bash
# 拉取镜像
docker pull ghcr.io/openclaw/openclaw:latest

# 运行
docker run -d \
  --name openclaw \
  -v ~/.openclaw:/root/.openclaw \
  -p 18789:18789 \
  ghcr.io/openclaw/openclaw:latest
```

### 4. 沙箱模式

**启用沙箱**：
```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "allowlist": [
          "bash",
          "process",
          "read",
          "write",
          "edit"
        ],
        "denylist": [
          "browser",
          "canvas",
          "nodes",
          "cron"
        ]
      }
    }
  }
}
```

**说明**：
- `mode: "non-main"`：仅对非主会话启用沙箱
- 主会话（直接聊天）：完全访问权限
- 群组/频道：在 Docker 沙箱中运行

### 5. Gmail Pub/Sub 触发器

**配置**：
```json
{
  "automation": {
    "gmail": {
      "enabled": true,
      "projectId": "your-gcp-project",
      "topicName": "gmail-notifications",
      "filters": {
        "from": ["important@example.com"],
        "subject": ["紧急"]
      },
      "action": "agent",
      "replyTo": "telegram:@me"
    }
  }
}
```

### 6. Webhook 触发器

**配置**：
```json
{
  "gateway": {
    "webhook": {
      "enabled": true,
      "path": "/webhook",
      "secret": "your-webhook-secret"
    }
  }
}
```

**触发**：
```bash
curl -X POST http://localhost:18789/webhook \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your-webhook-secret" \
  -d '{
    "message": "执行任务",
    "replyTo": "telegram:@me"
  }'
```

---

## 聊天命令

### 基本命令

| 命令 | 功能 | 示例 |
|------|------|------|
| `/status` | 会话状态（模型、tokens、成本） | `/status` |
| `/new` 或 `/reset` | 重置会话 | `/reset` |
| `/compact` | 压缩会话上下文 | `/compact` |
| `/think <level>` | 设置思考级别 | `/think high` |
| `/verbose on\|off` | 详细模式 | `/verbose on` |
| `/usage <mode>` | 使用统计显示 | `/usage tokens` |

### 群组专用命令

| 命令 | 功能 | 仅所有者 |
|------|------|----------|
| `/restart` | 重启 Gateway | ✅ |
| `/activation mention\|always` | 激活模式 | ✅ |

### 思考级别

- `off`：无思考过程
- `minimal`：最小
- `low`：低
- `medium`：中
- `high`：高
- `xhigh`：极高（仅 GPT-5.2+ 和 Codex）

### 使用统计模式

- `off`：不显示
- `tokens`：仅显示 token 数
- `full`：完整信息（tokens + 成本）

---

## 故障排查

### 通用诊断

```bash
# 健康检查
openclaw doctor

# 查看日志
openclaw logs --tail 100

# 查看 Gateway 日志
openclaw logs --gateway

# 查看 Agent 日志
openclaw logs --agent
```

### 常见问题

#### 1. Gateway 无法启动

**症状**：
```
Error: Address already in use
```

**解决方案**：
```bash
# 查找占用端口的进程
lsof -i :18789

# 杀死进程
kill -9 <PID>

# 或更换端口
openclaw gateway --port 18790
```

#### 2. 频道连接失败

**WhatsApp**：
```bash
# 清除凭证
rm -rf ~/.openclaw/credentials/whatsapp*

# 重新登录
openclaw channels login --channel whatsapp
```

**Telegram**：
```bash
# 检查 Bot Token
openclaw config show | grep telegram

# 测试 Bot
curl https://api.telegram.org/bot<YOUR_TOKEN>/getMe
```

**Discord**：
```bash
# 验证 Token
openclaw doctor --channel discord

# 检查 Intents
# 确保在 Discord 开发者门户启用了 Message Content Intent
```

#### 3. 权限问题（macOS）

**系统权限**：
- 系统偏好设置 > 安全性与隐私 > 隐私
- 授予 OpenClaw 以下权限：
  - 完全磁盘访问
  - 辅助功能
  - 屏幕录制（如需要）
  - 相机/麦克风（如需要）

**签名问题**：
```bash
# 重新签名（开发版本）
codesign --force --deep --sign - /Applications/OpenClaw.app
```

#### 4. 配对失败

**症状**：
```
Pairing code not found
```

**解决方案**：
```bash
# 列出待配对
openclaw pairing list

# 批准配对
openclaw pairing approve <channel> <code>

# 或切换为开放模式（不推荐）
# 在配置中设置 dmPolicy: "open"
```

#### 5. 模型错误

**症状**：
```
Model not found: openai/gpt-4
```

**解决方案**：
```json
{
  "agent": {
    "model": "openai/gpt-4-turbo-preview"
  }
}
```

**检查可用模型**：
```bash
openclaw models list
```

#### 6. 沙箱问题

**症状**：
```
Tool execution failed: permission denied
```

**解决方案**：
```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "off"
      }
    }
  }
}
```

**或添加工具到白名单**：
```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "allowlist": ["bash", "browser", "your-tool"]
      }
    }
  }
}
```

### 日志位置

| 平台 | 日志路径 |
|------|----------|
| **macOS** | `~/Library/Logs/OpenClaw/` |
| **Linux** | `~/.local/share/openclaw/logs/` |
| **Windows** | `%APPDATA%\OpenClaw\logs\` |

### 调试模式

```bash
# 启用详细日志
openclaw gateway --verbose --log-level debug

# 或设置环境变量
export OPENCLAW_LOG_LEVEL=debug
export OPENCLAW_VERBOSE=true
```

---

## 工作空间结构

### 默认路径

```
~/.openclaw/
├── openclaw.json          # 主配置文件
├── credentials/           # 频道凭证
├── workspace/             # Agent 工作空间
│   ├── AGENTS.md         # Agent 指令
│   ├── SOUL.md           # Agent 个性
│   ├── TOOLS.md          # 工具说明
│   └── skills/           # 技能目录
│       ├── skill-1/
│       │   └── SKILL.md
│       └── skill-2/
│           └── SKILL.md
├── sessions/              # 会话数据
└── logs/                  # 日志文件
```

### 关键文件说明

**AGENTS.md**：
- Agent 的主配置文件
- 定义 Wiki 结构、约定、工作流

**SOUL.md**：
- Agent 的个性和行为
- 语气、风格、价值观

**TOOLS.md**：
- 可用工具说明
- 使用指南

---

## 环境变量

### 常用环境变量

```bash
# Telegram
export TELEGRAM_BOT_TOKEN="123456:ABCDEF"

# Slack
export SLACK_BOT_TOKEN="xoxb-..."
export SLACK_APP_TOKEN="xapp-..."

# Discord
export DISCORD_BOT_TOKEN="your-token"

# OpenAI
export OPENAI_API_KEY="sk-..."

# Claude
export ANTHROPIC_API_KEY="sk-ant-..."

# OpenClaw 日志
export OPENCLAW_LOG_LEVEL="debug"
export OPENCLAW_VERBOSE="true"

# 工作空间
export OPENCLAW_WORKSPACE="~/custom-workspace"
```

---

## 开发频道

### 切换频道

```bash
# stable（稳定版）
openclaw update --channel stable

# beta（测试版）
openclaw update --channel beta

# dev（开发版）
openclaw update --channel dev
```

### 频道说明

| 频道 | 说明 | npm tag |
|------|------|---------|
| **stable** | 正式发布版本 | `latest` |
| **beta** | 预发布版本 | `beta` |
| **dev** | 开发版（main 分支） | `dev` |

---

## 安全最佳实践

### 1. DM 访问控制

**推荐配置**：
```json
{
  "channels": {
    "telegram": {
      "dmPolicy": "pairing",
      "allowFrom": ["@trusted-user"]
    }
  }
}
```

### 2. 群组安全

```json
{
  "channels": {
    "discord": {
      "guilds": {
        "allowed-server-id": {
          "channels": ["allowed-channel-id"]
        }
      }
    }
  }
}
```

### 3. Gateway 认证

```json
{
  "gateway": {
    "auth": {
      "mode": "password",
      "password": "strong-secure-password"
    }
  }
}
```

### 4. 工具访问控制

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "allowlist": ["bash", "read", "write"],
        "denylist": ["system.run", "nodes"]
      }
    }
  }
}
```

### 5. Tailscale 安全

**仅启用 Serve**（Tailnet 内部）：
```json
{
  "gateway": {
    "tailscale": {
      "mode": "serve"
    }
  }
}
```

**Funnel 必须配置密码**：
```json
{
  "gateway": {
    "tailscale": {
      "mode": "funnel"
    },
    "auth": {
      "mode": "password",
      "password": "required-for-funnel"
    }
  }
}
```

---

## 更新与维护

### 更新 OpenClaw

```bash
# 更新到最新版本
openclaw update

# 更新到特定版本
openclaw update --version 2026.4.9

# 更新后运行健康检查
openclaw doctor
```

### 备份配置

```bash
# 备份配置和凭证
cp -r ~/.openclaw ~/.openclaw.backup

# 仅备份配置文件
cp ~/.openclaw/openclaw.json ~/openclaw.json.backup
```

### 重置配置

```bash
# 重新运行引导
openclaw onboard --reconfigure

# 或手动删除配置
rm ~/.openclaw/openclaw.json
openclaw onboard
```

---

## 社区资源

### 官方资源

- **官网**：https://openclaw.ai/
- **文档**：https://docs.openclaw.ai/
- **GitHub**：https://github.com/openclaw/openclaw
- **Discord**：https://discord.gg/clawd

### 相关项目

- **Pi-mono**：https://github.com/badlogic/pi-mono
- **ClawHub**：https://clawhub.com/
- **Nix-openclaw**：https://github.com/openclaw/nix-openclaw

---

## 常见使用场景

### 1. 个人助手

**配置**：
- 连接 Telegram/WhatsApp
- 启用语音唤醒（macOS/iOS）
- 配置日常技能

### 2. 团队协作

**配置**：
- 连接 Slack/Discord
- 启用群组支持
- 配置沙箱模式
- 设置权限控制

### 3. 自动化任务

**配置**：
- 启用 Cron 定时任务
- 配置 Webhook 触发器
- 设置 Gmail Pub/Sub

### 4. 开发助手

**配置**：
- 启用浏览器控制
- 配置 Canvas
- 安装开发相关技能

---

## 附录

### A. 完整配置模板

见官方文档：https://docs.openclaw.ai/gateway/configuration

### B. 支持的模型提供商

- OpenAI
- Anthropic (Claude)
- Google (Gemini)
- Mistral
- Groq
- OpenRouter
- 本地模型（Ollama等）

### C. 快捷键（macOS App）

| 功能 | 快捷键 |
|------|--------|
| 唤醒 | `Cmd+Shift+Space` |
| 推送说话 | `Space`（按住） |
| 停止 | `Esc` |
| WebChat | `Cmd+W` |

---

**文档完成**

> 💡 **提示**：本文档基于 OpenClaw v2026.4.9。更多详细信息请参考官方文档：https://docs.openclaw.ai/
