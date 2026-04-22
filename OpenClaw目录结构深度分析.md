# OpenClaw ~/.openclaw 目录结构深度分析

> **分析日期**: 2026年4月13日  
> **OpenClaw 版本**: 2026.4.9  
> **系统**: macOS 14.0+ (Apple Silicon)

---

## 📋 目录

- [总览](#总览)
- [核心配置文件](#核心配置文件)
- [工作空间目录](#工作空间目录)
- [插件系统](#插件系统)
- [缓存与日志](#缓存与日志)
- [守护进程配置](#守护进程配置)
- [最佳实践](#最佳实践)

---

## 📂 总览

### 完整目录树

```
~/.openclaw/
├── openclaw.json                    # 主配置文件
├── daemon/                          # 守护进程目录
│   ├── openclaw.plist              # macOS LaunchAgent 配置
│   └── openclaw.log                # 守护进程日志
├── workspace/                       # 工作空间根目录
│   ├── .git/                       # Git 版本控制（可选）
│   ├── skills/                     # Skills 技能目录
│   │   ├── mcporter/               # MCP 桥接 Skill
│   │   │   └── SKILL.md            # Skill 定义文件
│   │   ├── taobao-native/          # 淘宝原生 Skill
│   │   │   └── SKILL.md
│   │   └── [custom-skills]/        # 自定义 Skills
│   ├── config/                     # 配置文件目录
│   │   ├── mcporter.json           # mcporter 配置
│   │   └── [other-configs].json
│   ├── agents/                     # Agent 定义目录
│   │   └── AGENTS.md               # Agent 配置文件
│   ├── prompts/                    # 提示词模板目录
│   │   └── [prompt-files].md
│   ├── tools/                      # 自定义工具目录
│   │   └── [custom-tools]/
│   ├── data/                       # 数据存储目录
│   │   ├── sessions/               # 会话数据
│   │   ├── cache/                  # 缓存数据
│   │   └── logs/                   # 应用日志
│   └── .instructions.md            # 工作空间指令（可选）
├── plugins/                        # 插件目录
│   ├── clawbot-wechat/             # 微信机器人插件
│   │   ├── config.json             # 插件配置
│   │   ├── plugin.js               # 插件主程序
│   │   └── node_modules/           # 插件依赖
│   └── [other-plugins]/
├── cache/                          # 全局缓存目录
│   ├── models/                     # 模型缓存
│   ├── embeddings/                 # 向量嵌入缓存
│   └── temp/                       # 临时文件
├── logs/                           # 全局日志目录
│   ├── gateway.log                 # Gateway 日志
│   ├── agent.log                   # Agent 日志
│   ├── error.log                   # 错误日志
│   └── access.log                  # 访问日志
└── backups/                        # 备份目录
    └── [timestamp]/                # 按时间戳的备份
```

---

## 🔧 核心配置文件

### 1. openclaw.json

**路径**: `~/.openclaw/openclaw.json`

**作用**: OpenClaw 的主配置文件，定义系统级行为

**完整配置示例**:

```json
{
  "version": "2026.4.9",
  
  "agent": {
    "model": "ollama/qwen3.5:14b",
    "workspace": "~/.openclaw/workspace",
    "temperature": 0.7,
    "max_tokens": 2048,
    "top_p": 0.9,
    "frequency_penalty": 0.0,
    "presence_penalty": 0.0
  },
  
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "host": "127.0.0.1",
    "protocol": "http",
    "cors": {
      "enabled": true,
      "origins": ["http://localhost:3000"]
    },
    "auth": {
      "mode": "token",
      "token": "your-secure-token-here",
      "jwt_secret": "your-jwt-secret"
    },
    "ratelimit": {
      "enabled": true,
      "max_requests": 100,
      "window_ms": 60000
    },
    "tailscale": {
      "mode": "off",
      "hostname": "openclaw-gateway",
      "advertise_routes": []
    }
  },
  
  "browser": {
    "enabled": true,
    "color": "#FF4500",
    "headless": false,
    "profiles": {
      "default": {
        "user_agent": "Mozilla/5.0..."
      }
    }
  },
  
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main",
        "allowlist": [
          "bash",
          "read",
          "write",
          "edit",
          "browser",
          "mcporter",
          "taobao-native"
        ],
        "denylist": [],
        "timeout": 30000
      },
      "memory": {
        "enabled": true,
        "max_tokens": 4096,
        "persistence": true
      }
    }
  },
  
  "ollama": {
    "baseUrl": "http://localhost:11434",
    "timeout": 120000,
    "keepAlive": true,
    "numCtx": 4096,
    "numGpu": 1
  },
  
  "logging": {
    "level": "info",
    "file": "~/.openclaw/logs/gateway.log",
    "maxSize": "10mb",
    "maxFiles": 5,
    "console": true
  },
  
  "cache": {
    "enabled": true,
    "ttl": 3600,
    "maxSize": "1gb",
    "path": "~/.openclaw/cache"
  },
  
  "plugins": {
    "enabled": true,
    "path": "~/.openclaw/plugins",
    "autoload": ["clawbot-wechat"]
  },
  
  "security": {
    "sandbox_mode": true,
    "allow_shell": false,
    "allow_network": true,
    "max_file_size": "100mb"
  }
}
```

**配置项详解**:

| 配置节 | 作用 | 关键参数 |
|--------|------|----------|
| `agent` | AI 代理配置 | model, temperature, max_tokens |
| `gateway` | 网关服务配置 | port, auth, ratelimit |
| `browser` | 浏览器自动化 | enabled, headless, profiles |
| `agents.defaults` | 默认代理行为 | sandbox.allowlist, memory |
| `ollama` | Ollama 连接 | baseUrl, timeout, numCtx |
| `logging` | 日志配置 | level, maxSize, maxFiles |
| `cache` | 缓存配置 | ttl, maxSize |
| `plugins` | 插件系统 | enabled, autoload |
| `security` | 安全策略 | sandbox_mode, allow_shell |

---

## 📁 工作空间目录

### 2. workspace/

**路径**: `~/.openclaw/workspace/`

**作用**: OpenClaw 的工作目录，存放 Skills、配置、数据

#### 2.1 skills/ - Skills 技能目录

**路径**: `~/.openclaw/workspace/skills/`

**作用**: 存放可调用的 Skills，每个 Skill 一个子目录

**Skill 结构**:

```
skills/
├── mcporter/
│   ├── SKILL.md              # Skill 定义（必需）
│   ├── README.md             # 说明文档（推荐）
│   ├── package.json          # Node.js 依赖（可选）
│   ├── index.js              # 主程序（可选）
│   └── lib/                  # 库文件（可选）
└── taobao-native/
    └── SKILL.md
```

**SKILL.md 格式**:

```markdown
---
name: taobao-native
description: 淘宝原生 MCP 服务接口
version: 1.0.0
author: Taobao Team
applyTo:
  - pattern: "淘宝|购物|商品"
    match: "semantic"
tools:
  - taobao_search
  - taobao_detail
  - taobao_cart
---

# Taobao Native Skill

## 工具列表

### taobao_search
- 描述: 搜索淘宝商品
- 参数:
  - keyword: 搜索关键词
  - price_min: 最低价格
  - price_max: 最高价格
  - sort: 排序方式

### taobao_detail
- 描述: 获取商品详情
- 参数:
  - item_id: 商品ID

...
```

**Skill 发现机制**:

1. **启动时扫描** - Gateway 启动时自动扫描 skills/ 目录
2. **YAML 解析** - 解析 SKILL.md 中的 YAML frontmatter
3. **工具注册** - 将工具注册到 Agent 的调用列表
4. **模式匹配** - 根据 applyTo 规则决定何时激活

#### 2.2 config/ - 配置文件目录

**路径**: `~/.openclaw/workspace/config/`

**作用**: 存放各类配置文件

**典型配置文件**:

```
config/
├── mcporter.json             # mcporter MCP 桥接配置
├── browser.json              # 浏览器配置
├── taobao.json               # 淘宝特定配置
└── custom.json               # 自定义配置
```

**mcporter.json 示例**:

```json
{
  "mcpServers": {
    "taobao-native": {
      "type": "streamableHttp",
      "url": "http://localhost:3654/mcp",
      "timeout": 30000,
      "retry": {
        "max_attempts": 3,
        "delay": 1000
      },
      "headers": {
        "User-Agent": "OpenClaw/2026.4.9"
      }
    },
    "custom-mcp": {
      "type": "stdio",
      "command": "node",
      "args": ["./custom-mcp-server.js"],
      "env": {
        "NODE_ENV": "production"
      }
    }
  },
  "logging": {
    "level": "debug",
    "file": "~/.openclaw/logs/mcporter.log"
  }
}
```

#### 2.3 agents/ - Agent 定义目录

**路径**: `~/.openclaw/workspace/agents/`

**作用**: 定义自定义 Agent 行为

**AGENTS.md 示例**:

```markdown
---
version: 1.0.0
---

# 淘宝购物助手

你是一个专业的淘宝购物助手，帮助用户搜索商品、比价和下单。

## 核心能力

- 商品搜索与筛选
- 价格比较与分析
- 订单管理
- 价格监控

## 工作流程

1. **理解需求**: 准确识别用户购物意图
2. **搜索商品**: 使用 taobao-native 工具
3. **信息展示**: 结构化展示商品信息
4. **智能推荐**: 基于性价比推荐
5. **确认操作**: 敏感操作前征求确认

## 行为准则

- 使用官方工作流：navigate → read_page_content → scan_page_elements → click_element
- 准确提取信息，不编造数据
- 敏感操作（下单、支付）必须人工确认
- 主动提供建议，尊重用户选择

## 响应格式

### 商品列表
\```
1. 商品名称 - ¥价格 - 月销XXX+ ⭐评分
   店铺：店铺名称 (平台)
\```

### 商品详情
\```
━━━━━━━━━━━━━━━━━━━━
📦 商品名称
💰 价格：¥XXX
📊 月销：XXX+
⭐ 评分：X.X/5
🏪 店铺：XXX
━━━━━━━━━━━━━━━━━━━━
\```
```

#### 2.4 prompts/ - 提示词模板目录

**路径**: `~/.openclaw/workspace/prompts/`

**作用**: 存放可复用的提示词模板

**示例结构**:

```
prompts/
├── system.md                 # 系统提示词
├── shopping-assistant.md     # 购物助手提示词
├── price-compare.md          # 比价提示词
└── templates/
    ├── product-listing.hbs   # 商品列表模板
    └── order-summary.hbs     # 订单摘要模板
```

#### 2.5 data/ - 数据存储目录

**路径**: `~/.openclaw/workspace/data/`

**作用**: 持久化数据存储

**子目录**:

```
data/
├── sessions/                 # 会话数据
│   ├── 2026-04-13/
│   │   ├── session-001.json
│   │   └── session-002.json
│   └── archive/              # 归档会话
├── cache/                    # 应用缓存
│   ├── embeddings/           # 向量嵌入
│   └── search-results/       # 搜索结果缓存
├── logs/                     # 应用日志
│   ├── agent-2026-04-13.log
│   └── error-2026-04-13.log
└── user-data/                # 用户数据
    ├── preferences.json      # 用户偏好
    ├── watchlist.json        # 价格监控列表
    └── history.json          # 操作历史
```

**会话数据格式**:

```json
{
  "session_id": "session-001",
  "created_at": "2026-04-13T10:00:00Z",
  "user_id": "user-001",
  "messages": [
    {
      "role": "user",
      "content": "帮我找个机械键盘",
      "timestamp": "2026-04-13T10:00:01Z"
    },
    {
      "role": "assistant",
      "content": "好的，请问您的预算范围是多少？",
      "timestamp": "2026-04-13T10:00:03Z",
      "tools_used": []
    }
  ],
  "metadata": {
    "platform": "web",
    "model": "qwen3.5:14b",
    "total_tokens": 150
  }
}
```

---

## 🔌 插件系统

### 3. plugins/

**路径**: `~/.openclaw/plugins/`

**作用**: 扩展 OpenClaw 功能的插件

#### 3.1 插件结构

**标准插件目录**:

```
plugins/
└── clawbot-wechat/
    ├── package.json          # 插件元数据
    ├── plugin.json           # 插件配置
    ├── config.json           # 运行时配置
    ├── index.js              # 入口文件
    ├── lib/                  # 库文件
    │   ├── wechat-client.js
    │   └── message-handler.js
    ├── node_modules/         # 依赖
    └── README.md             # 说明文档
```

**plugin.json 格式**:

```json
{
  "name": "clawbot-wechat",
  "version": "1.0.0",
  "description": "微信机器人插件",
  "author": "Tencent",
  "main": "index.js",
  "openclaw": {
    "minVersion": "2026.4.0",
    "maxVersion": "2027.0.0"
  },
  "permissions": [
    "network",
    "filesystem",
    "gateway"
  ],
  "hooks": {
    "onLoad": "onPluginLoad",
    "onUnload": "onPluginUnload",
    "onMessage": "handleMessage"
  },
  "config_schema": {
    "gateway_url": {
      "type": "string",
      "required": true
    },
    "auth_token": {
      "type": "string",
      "required": true
    }
  }
}
```

**config.json 示例**:

```json
{
  "gateway": {
    "url": "http://localhost:18789",
    "token": "your-secure-token",
    "timeout": 30000
  },
  "wechat": {
    "profile": "default",
    "auto_reply": true,
    "allowed_contacts": ["张三", "李四"],
    "command_prefix": "@bot"
  },
  "features": {
    "price_watch": true,
    "auto_checkout": false,
    "voice_reply": false
  },
  "logging": {
    "level": "info",
    "file": "~/.openclaw/logs/clawbot-wechat.log"
  }
}
```

#### 3.2 插件生命周期

```javascript
// index.js - 插件入口
class ClawBotWechatPlugin {
  constructor(openclaw) {
    this.openclaw = openclaw;
    this.config = null;
    this.wechatClient = null;
  }
  
  // 加载时调用
  async onPluginLoad(config) {
    this.config = config;
    this.wechatClient = new WechatClient(config.wechat);
    await this.wechatClient.connect();
    this.openclaw.logger.info('ClawBot WeChat plugin loaded');
  }
  
  // 卸载时调用
  async onPluginUnload() {
    await this.wechatClient.disconnect();
    this.openclaw.logger.info('ClawBot WeChat plugin unloaded');
  }
  
  // 处理消息
  async handleMessage(message) {
    if (message.platform === 'wechat') {
      const response = await this.openclaw.agent.chat(message.content);
      await this.wechatClient.send(message.sender, response);
    }
  }
}

module.exports = ClawBotWechatPlugin;
```

---

## 💾 缓存与日志

### 4. cache/ - 全局缓存目录

**路径**: `~/.openclaw/cache/`

**作用**: 存放临时缓存数据，提升性能

**子目录**:

```
cache/
├── models/                   # 模型缓存
│   ├── embeddings.db         # 向量嵌入数据库
│   └── tokenizer.json        # 分词器缓存
├── http/                     # HTTP 响应缓存
│   ├── taobao/
│   │   ├── search-xxxx.json
│   │   └── detail-xxxx.json
│   └── api/
├── temp/                     # 临时文件
│   ├── uploads/
│   └── downloads/
└── index.db                  # 缓存索引数据库
```

**缓存策略**:

- **TTL (Time To Live)**: 默认 1 小时
- **LRU (Least Recently Used)**: 自动清理最久未用
- **最大容量**: 1GB（可配置）
- **缓存键**: `${service}:${method}:${hash(params)}`

### 5. logs/ - 全局日志目录

**路径**: `~/.openclaw/logs/`

**作用**: 存放系统和应用日志

**日志文件**:

```
logs/
├── gateway.log               # Gateway 主日志
├── gateway-2026-04-13.log    # 按日期归档
├── agent.log                 # Agent 执行日志
├── error.log                 # 错误日志
├── access.log                # HTTP 访问日志
├── mcporter.log              # mcporter 日志
├── clawbot-wechat.log        # 微信插件日志
└── audit.log                 # 审计日志
```

**日志格式** (JSON Lines):

```json
{"timestamp":"2026-04-13T10:00:00.000Z","level":"info","service":"gateway","message":"Server started on port 18789","pid":12345}
{"timestamp":"2026-04-13T10:00:05.123Z","level":"debug","service":"agent","message":"Tool invocation: taobao_search","args":{"keyword":"机械键盘"},"duration":234}
{"timestamp":"2026-04-13T10:00:10.456Z","level":"error","service":"gateway","message":"Connection timeout","error":"ETIMEDOUT","stack":"..."}
```

**日志轮转配置**:

- **最大文件大小**: 10MB
- **保留文件数**: 5 个
- **压缩**: 自动 gzip 归档
- **清理策略**: 保留 30 天

---

## 🔐 守护进程配置

### 6. daemon/ - 守护进程目录

**路径**: `~/.openclaw/daemon/`

**作用**: 管理 OpenClaw Gateway 守护进程

#### macOS LaunchAgent

**文件**: `~/Library/LaunchAgents/com.openclaw.gateway.plist`

**内容**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.gateway</string>
    
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>gateway</string>
        <string>--config</string>
        <string>~/.openclaw/openclaw.json</string>
    </array>
    
    <key>RunAtLoad</key>
    <true/>
    
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <false/>
    </dict>
    
    <key>StandardOutPath</key>
    <string>~/.openclaw/logs/gateway-stdout.log</string>
    
    <key>StandardErrorPath</key>
    <string>~/.openclaw/logs/gateway-stderr.log</string>
    
    <key>WorkingDirectory</key>
    <string>~/.openclaw</string>
    
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin</string>
        <key>OPENCLAW_HOME</key>
        <string>~/.openclaw</string>
    </dict>
</dict>
</plist>
```

**管理命令**:

```bash
# 加载守护进程
launchctl load ~/Library/LaunchAgents/com.openclaw.gateway.plist

# 卸载守护进程
launchctl unload ~/Library/LaunchAgents/com.openclaw.gateway.plist

# 启动服务
launchctl start com.openclaw.gateway

# 停止服务
launchctl stop com.openclaw.gateway

# 查看状态
launchctl list | grep openclaw
```

---

## 📦 备份目录

### 7. backups/ - 备份目录

**路径**: `~/.openclaw/backups/`

**作用**: 存放配置和数据备份

**备份结构**:

```
backups/
├── 2026-04-13_10-00-00/      # 时间戳命名
│   ├── openclaw.json
│   ├── workspace/
│   │   ├── skills/
│   │   ├── config/
│   │   └── data/
│   └── backup-manifest.json
├── 2026-04-12_10-00-00/
└── latest -> 2026-04-13_10-00-00  # 符号链接
```

**自动备份策略**:

```json
{
  "backup": {
    "enabled": true,
    "schedule": "0 2 * * *",  // 每天凌晨 2 点
    "retention": {
      "daily": 7,   // 保留 7 天
      "weekly": 4,  // 保留 4 周
      "monthly": 6  // 保留 6 月
    },
    "include": [
      "openclaw.json",
      "workspace/config/",
      "workspace/agents/",
      "workspace/data/user-data/"
    ],
    "exclude": [
      "workspace/data/cache/",
      "workspace/data/logs/",
      "cache/",
      "logs/"
    ],
    "compression": true,
    "encryption": false
  }
}
```

**手动备份命令**:

```bash
# 创建备份
openclaw backup create --name "before-upgrade"

# 列出备份
openclaw backup list

# 恢复备份
openclaw backup restore --name "2026-04-13_10-00-00"

# 删除备份
openclaw backup delete --name "2026-04-10_10-00-00"
```

---

## 🔍 目录权限与安全

### 文件权限建议

```bash
# 主目录
chmod 700 ~/.openclaw

# 配置文件（包含敏感信息）
chmod 600 ~/.openclaw/openclaw.json
chmod 600 ~/.openclaw/workspace/config/*.json

# 工作空间
chmod 755 ~/.openclaw/workspace
chmod 755 ~/.openclaw/workspace/skills
chmod 644 ~/.openclaw/workspace/skills/*/*.md

# 插件目录
chmod 755 ~/.openclaw/plugins
chmod 755 ~/.openclaw/plugins/*/

# 日志目录
chmod 755 ~/.openclaw/logs
chmod 644 ~/.openclaw/logs/*.log

# 缓存目录
chmod 755 ~/.openclaw/cache
chmod 644 ~/.openclaw/cache/**/*

# 守护进程配置
chmod 644 ~/Library/LaunchAgents/com.openclaw.gateway.plist
```

### 敏感数据保护

**加密配置**:

```json
{
  "security": {
    "encrypt_config": true,
    "encryption_key_env": "OPENCLAW_ENCRYPTION_KEY",
    "sensitive_fields": [
      "gateway.auth.token",
      "gateway.auth.jwt_secret",
      "ollama.api_key",
      "plugins.*.auth_token"
    ]
  }
}
```

**环境变量配置**:

```bash
# ~/.zshrc 或 ~/.bashrc
export OPENCLAW_HOME=~/.openclaw
export OPENCLAW_ENCRYPTION_KEY="your-encryption-key"
export OPENCLAW_LOG_LEVEL="info"
export OPENCLAW_GATEWAY_PORT=18789
```

---

## 📊 目录大小管理

### 磁盘空间监控

**典型空间占用**:

```
~/.openclaw/                  总计: 约 25GB
├── workspace/                约 5GB
│   ├── skills/               约 100MB
│   ├── config/               约 10MB
│   ├── data/                 约 4.5GB
│   │   ├── sessions/         约 2GB
│   │   ├── cache/            约 2GB
│   │   └── logs/             约 500MB
│   └── agents/               约 50MB
├── plugins/                  约 500MB
│   └── clawbot-wechat/       约 500MB
├── cache/                    约 1GB (配置限制)
├── logs/                     约 500MB (自动轮转)
└── backups/                  约 18GB (7 天备份)
```

### 清理策略

**自动清理脚本**:

```bash
#!/bin/bash
# clean-openclaw.sh

OPENCLAW_HOME=~/.openclaw

# 清理过期缓存（超过 7 天）
find $OPENCLAW_HOME/cache/temp -type f -mtime +7 -delete

# 清理旧日志（超过 30 天）
find $OPENCLAW_HOME/logs -name "*.log" -mtime +30 -delete

# 清理旧会话（超过 90 天）
find $OPENCLAW_HOME/workspace/data/sessions -type f -mtime +90 -delete

# 压缩旧备份
find $OPENCLAW_HOME/backups -type d -mtime +7 -exec tar -czf {}.tar.gz {} \; -exec rm -rf {} \;

echo "Cleanup completed"
```

**手动清理命令**:

```bash
# 清理所有缓存
openclaw cache clear --all

# 清理特定缓存
openclaw cache clear --type embeddings

# 清理日志
openclaw logs clean --older-than 30d

# 查看空间占用
openclaw disk-usage

# 导出占用报告
openclaw disk-usage --report > openclaw-disk-report.txt
```

---

## 🛠️ 最佳实践

### 1. 目录组织规范

✅ **推荐做法**:

- Skills 按功能分类命名（如 `shopping/`, `finance/`）
- 配置文件使用语义化命名（如 `taobao-prod.json`）
- 定期归档旧会话数据
- 使用 Git 管理 workspace（可选）

❌ **避免做法**:

- 在根目录放置临时文件
- 手动修改 cache/ 目录
- 直接编辑守护进程配置
- 随意删除日志文件

### 2. 版本控制集成

**初始化 Git**:

```bash
cd ~/.openclaw/workspace
git init
git add skills/ agents/ config/
git commit -m "Initial workspace setup"
```

**.gitignore 示例**:

```gitignore
# 数据文件
data/sessions/
data/cache/
data/logs/

# 临时文件
*.tmp
*.temp
.DS_Store

# 敏感配置
config/*-secret.json
config/*.key

# Node.js
node_modules/
package-lock.json

# 日志
*.log
```

### 3. 迁移与同步

**导出配置**:

```bash
# 导出完整配置
openclaw export --output ~/openclaw-backup.tar.gz

# 仅导出配置文件
openclaw export --config-only --output ~/openclaw-config.tar.gz
```

**导入配置**:

```bash
# 导入到新机器
openclaw import --input ~/openclaw-backup.tar.gz

# 验证导入
openclaw doctor
```

### 4. 性能优化

**缓存优化**:

```json
{
  "cache": {
    "enabled": true,
    "strategy": "lru",
    "ttl": 3600,
    "maxSize": "2gb",
    "preload": [
      "embeddings",
      "frequent-searches"
    ]
  }
}
```

**日志优化**:

```json
{
  "logging": {
    "level": "info",
    "async": true,
    "buffer_size": 1024,
    "sampling": {
      "enabled": true,
      "rate": 0.1  // 仅记录 10% 的 debug 日志
    }
  }
}
```

### 5. 监控与告警

**健康检查脚本**:

```bash
#!/bin/bash
# health-check.sh

# 检查 Gateway 状态
if ! curl -s http://localhost:18789/health > /dev/null; then
    echo "Gateway is down!" | mail -s "OpenClaw Alert" admin@example.com
fi

# 检查磁盘空间
USAGE=$(df -h ~/.openclaw | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $USAGE -gt 80 ]; then
    echo "Disk usage is $USAGE%" | mail -s "OpenClaw Disk Alert" admin@example.com
fi

# 检查日志错误
if grep -i "error" ~/.openclaw/logs/gateway.log | tail -100 | grep -q "$(date +%Y-%m-%d)"; then
    echo "Errors detected in logs" | mail -s "OpenClaw Error Alert" admin@example.com
fi
```

---

## 📚 附录

### A. 快速参考

**常用路径**:

```bash
配置文件:     ~/.openclaw/openclaw.json
工作空间:     ~/.openclaw/workspace
Skills:      ~/.openclaw/workspace/skills
插件:        ~/.openclaw/plugins
日志:        ~/.openclaw/logs
缓存:        ~/.openclaw/cache
```

**常用命令**:

```bash
# 查看配置
openclaw config show

# 验证配置
openclaw config validate

# 编辑配置
openclaw config edit

# 列出 Skills
openclaw skills list

# 检查健康
openclaw doctor

# 查看日志
openclaw logs tail -f

# 清理缓存
openclaw cache clear
```

### B. 故障排查

**配置文件损坏**:

```bash
# 恢复默认配置
openclaw config reset

# 从备份恢复
cp ~/.openclaw/backups/latest/openclaw.json ~/.openclaw/
```

**权限问题**:

```bash
# 修复权限
chmod -R u+rwX ~/.openclaw
find ~/.openclaw -type f -exec chmod 644 {} \;
find ~/.openclaw -type d -exec chmod 755 {} \;
chmod 600 ~/.openclaw/openclaw.json
```

**空间不足**:

```bash
# 分析空间占用
du -sh ~/.openclaw/*

# 清理缓存和日志
openclaw cache clear --all
openclaw logs clean --older-than 7d

# 删除旧备份
rm -rf ~/.openclaw/backups/2026-04-*
```

### C. 相关资源

- **官方文档**: https://docs.openclaw.ai/
- **GitHub**: https://github.com/openclaw/openclaw
- **Discord 社区**: https://discord.gg/clawd
- **问题反馈**: https://github.com/openclaw/openclaw/issues

---

**文档结束**

> 本文档详细分析了 OpenClaw 2026.4.9 的目录结构  
> 如有疑问或建议，请访问官方社区获取支持
