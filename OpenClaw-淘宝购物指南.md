# OpenClaw 淘宝购物指南

> **更新日期**: 2026年4月13日  
> **适用设备**: MacBook Pro M4  
> **文档版本**: v1.0

本文档提供从零开始在 MacBook Pro M4 上部署 OpenClaw + Ollama + Qwen3.5 本地大模型的完整指南，并演示如何使用 AI 助手进行淘宝购物。

---

## 📋 目录

- [系统要求](#系统要求)
- [一、部署 Ollama + Qwen3.5 本地大模型](#一部署-ollama--qwen35-本地大模型)
- [二、部署与配置 OpenClaw](#二部署与配置-openclaw)
- [三、配置 taobao-native](#三配置-taobao-native)
- [四、安装微信 ClawBot 插件](#四安装微信-clawbot-插件)
- [附录](#附录)
- [更新日志](#更新日志)

---

## 💻 系统要求

### 硬件配置
- **设备**: MacBook Pro M4 (或 M1/M2/M3 系列)
- **内存**: 建议 32GB+ (Qwen3.5-14B 需要约 8GB)
- **存储**: 至少 500GB 可用空间
- **系统**: macOS 14.0+

### 软件依赖
- VPN（可选）：下载软件、大模型
- Homebrew
- Node.js v24 或 v22.16+
- Git
- 淘宝桌面客户端 2.5.1+
- 微信 Mac 版 3.8.0+（可选，使用 ClawBot 插件时需要）

---

# 一、部署 Ollama + Qwen3.5 本地大模型

## 为什么不用 OpenAI Codex

在开始部署本地大模型之前，我们需要理解为什么选择本地方案而非 OpenAI Codex 或其他云端 API。

```
OpenAI API 在中国大陆的使用障碍:

❌ 网络访问限制:
- OpenAI API 在大陆无法直接访问
- 必须使用科学上网工具 (VPN/代理)
- VPN 稳定性影响服务可用性
- 企业环境可能禁止使用 VPN

❌ 支付方式限制:
- 不支持大陆银行卡支付
- 需要 Visa/MasterCard 国际信用卡
- 或使用虚拟信用卡 (额外成本)
- 部分地区可能面临封号风险

✅ 本地 Ollama 部署:
- 无需科学上网，本地直接运行
- 无需国际信用卡
- 不受地域网络限制
```

### 本地模型对比

| 对比维度 | Qwen3.5:9b<br/>⭐推荐 | Qwen3:8b | Qwen:7b | DeepSeek-R1:7b |
|---------|---------------------|----------|---------|----------------|
| **参数量** | 9B | 8B | 7B | 7B |
| **模型大小** | 6.6 GB | 5.2 GB | 4.5 GB | 4.7 GB |
| **内存需求** | ~20GB | ~18GB | ~10GB | ~6GB |
| **中文理解** | ⭐⭐⭐⭐⭐ 优秀 | ⭐⭐⭐⭐ 良好 | ⭐⭐⭐ 中等 | ⭐⭐⭐⭐ 良好 |
| **工具调用** | ⭐⭐⭐⭐⭐ 原生支持 | ⭐⭐⭐⭐⭐ 原生支持 | ❌ 不支持 | ❌ 不支持 |
| **响应速度** | ⭐⭐⭐⭐ 快 | ⭐⭐⭐ 一般 | ⭐⭐⭐ 一般 | ⭐⭐⭐ 一般 |
| **推理能力** | ⭐⭐⭐⭐⭐ 优秀 | ⭐⭐⭐⭐ 良好 | ⭐⭐⭐ 中等 | ⭐⭐⭐⭐⭐ 优秀 |
| **多轮对话** | ⭐⭐⭐⭐⭐ 优秀 | ⭐⭐⭐⭐ 良好 | ⭐⭐⭐ 中等 | ⭐⭐⭐⭐ 良好 |
| **上下文长度** | 32K tokens | 32K tokens | 8K tokens | 128K tokens |

---

## 1.1 安装 Ollama

### 方法一：官网下载（推荐）

1. **访问 Ollama 官网**
   - 打开 https://ollama.com/download/mac
   - 下载 macOS 版本 (支持 Apple Silicon)

2. **安装 Ollama**
   ```bash
   # 打开下载的 .dmg 文件
   # 拖拽 Ollama 到 Applications 文件夹
   ```

3. **启动 Ollama**
   ```bash
   # 从 Launchpad 或 Applications 启动 Ollama
   # 或使用命令行
   ollama serve
   ```

### 方法二：Homebrew 安装

```bash
# 安装 Homebrew (如果未安装)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 使用 Homebrew 安装 Ollama
brew install ollama

# 启动 Ollama 服务
ollama serve
```

---

## 1.2 下载 Qwen3.5 模型

### 为什么选择 Qwen3.5

Qwen3.5 是阿里巴巴通义千问团队推出的最新一代**多模态大语言模型**，具有以下优势：

- **🎯 多模态能力**: 不仅支持文本理解与生成，还能处理图像、音频等多种模态数据
- **🚀 性能优异**: 在中文和英文基准测试中表现出色，特别是中文理解能力业界领先
- **💡 推理能力强**: 擅长复杂逻辑推理、数学计算、代码生成等任务
- **🔧 工具调用**: 原生支持 Function Calling，能够准确调用外部工具（如 taobao-native）
- **⚡ 开源免费**: 模型完全开源，可本地部署，无需付费 API
- **🍎 Apple Silicon 优化**: 在 M 系列芯片上运行流畅，充分利用 Metal GPU 加速

对于淘宝购物助手场景，Qwen3.5 的工具调用能力和中文理解能力尤为重要。

### 推荐模型选择

| 模型 | 参数量 | 内存需求 | 推荐用途 |
|------|--------|----------|----------|
| **qwen3.5:7b** | 7B | ~5GB | 日常对话、轻量任务 |
| **qwen3.5:14b** | 14B | ~8GB | 复杂任务、购物助手 |
| **qwen3.5:32b** | 32B | ~20GB | 专业级任务 |

### 下载模型

```bash
# 推荐：14B 版本 (性能与资源平衡)
ollama pull qwen3.5:14b

# 或选择 7B 版本 (更轻量)
ollama pull qwen3.5:7b

# 或选择最新版本
ollama pull qwen3.5:latest
```

下载完成后验证：

```bash
# 列出已安装模型
ollama list

# 测试模型
ollama run qwen3.5:14b "你好，请介绍一下自己"
```

---

## 1.3 配置 Ollama

### 设置环境变量

```bash
# 编辑 ~/.zshrc (macOS 默认 shell)
nano ~/.zshrc

# 添加以下内容
export OLLAMA_HOST="127.0.0.1:11434"
export OLLAMA_MODELS="/Users/$(whoami)/.ollama/models"

# 保存后重新加载
source ~/.zshrc
```

### 验证 Ollama 服务

```bash
# 检查服务状态
curl http://localhost:11434/api/tags

# 应返回已安装模型列表
```

---

## 1.4 性能优化 (M4 芯片)

### 启用 Metal 加速

Ollama 在 Apple Silicon 上默认使用 Metal GPU 加速，无需额外配置。

### 调整并发设置

```bash
# 设置最大并发请求数
export OLLAMA_MAX_LOADED_MODELS=1
export OLLAMA_NUM_PARALLEL=2
```

### 模型预加载

```bash
# 预加载模型到内存
ollama run qwen3.5:14b ""
```

---

# 二、部署与配置 OpenClaw

## 2.1 安装 Node.js

### 使用 NVM (推荐)

```bash
# 安装 NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 重新加载 shell 配置
source ~/.zshrc

# 安装 Node.js v24
nvm install 24
nvm use 24
nvm alias default 24

# 验证安装
node --version  # v24.x.x
npm --version   # 10.x.x
```

---

## 2.2 全局安装 OpenClaw

```bash
# 使用 npm 全局安装
npm install -g openclaw@latest

# 或使用 pnpm (更快)
npm install -g pnpm
pnpm add -g openclaw@latest

# 验证安装
openclaw --version
```

---

## 2.3 初始化 OpenClaw

```bash
# 运行引导安装
openclaw onboard

# 按提示操作：
# 1. 选择工作空间位置 (默认: ~/.openclaw/workspace)
# 2. 配置 Gateway 端口 (默认: 18789)
# 3. 选择是否安装守护进程 (推荐: Y)
```

---

## 2.4 配置 OpenClaw 使用 Ollama

### 编辑配置文件

```bash
# 打开配置文件
nano ~/.openclaw/openclaw.json
```

### 配置内容

```json
{
  "agent": {
    "model": "ollama/qwen3.5:14b",
    "workspace": "~/.openclaw/workspace"
  },
  
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your-secure-token-here"
    }
  },
  
  "browser": {
    "enabled": true,
    "color": "#FF4500"
  },
  
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "allowlist": ["bash", "read", "write", "browser", "mcporter", "taobao-native"]
      }
    }
  },
  
  "ollama": {
    "baseUrl": "http://localhost:11434"
  }
}
```

### 配置说明

| 配置项 | 说明 | 备注 |
|--------|------|------|
| `model` | 使用 `ollama/` 前缀指定本地模型 | 格式: `ollama/模型名` |
| `baseUrl` | Ollama API 地址 | 默认 11434 端口 |
| `sandbox.allowlist` | 允许的工具列表 | 包含 taobao-native |

---

## 2.5 启动 OpenClaw Gateway

```bash
# 前台启动 (调试模式)
openclaw gateway --verbose

# 或后台运行 (守护进程)
# 安装守护进程后自动启动
openclaw onboard --install-daemon

# 检查状态
openclaw doctor

# dashboard
openclaw dashboard
```

### 验证 Gateway

```bash
# 健康检查
curl http://localhost:18789/health

# 应返回: {"status":"ok"}
```

---

## 2.6 重置配置（保留应用）

如果需要重新配置 OpenClaw，但不想卸载应用程序，可以只删除配置文件。

### 方法一：完全重置（删除整个目录）

如果遇到严重问题，可以删除整个 `~/.openclaw` 目录，从零开始配置。

**适用场景**：
- Gateway 无法正常启动
- Skills 加载错误且无法修复
- 配置文件严重损坏
- 需要彻底清理所有本地数据

**操作步骤**：

```bash
# 1. 停止 Gateway
openclaw gateway stop
# 或强制停止所有 openclaw 进程
pkill -f openclaw

# 2. 备份整个目录（可选但推荐）
cp -r ~/.openclaw ~/.openclaw.backup.$(date +%Y%m%d_%H%M%S)

# 3. 删除整个 .openclaw 目录
rm -rf ~/.openclaw

# 4. 重新初始化
openclaw onboard

# 5. 按照 2.4 节重新配置 Ollama
nano ~/.openclaw/openclaw.json
# 添加 Ollama 配置

# 6. 启动 Gateway
openclaw gateway --verbose

# 7. 验证配置
openclaw doctor

# 8. 按需重新安装 Skills 和 mcporter（参考第三章）
```

### 方法二：仅删除配置文件（保留 Skills）

如果只是配置出错，可以只删除配置文件，保留已下载的 Skills。

**适用场景**：
- 仅修改模型配置
- 端口冲突需要更换
- 认证配置错误
- Skills 正常，只是配置文件问题

**操作步骤**：

```bash
# 1. 停止 Gateway
openclaw gateway stop
# 或强制停止
pkill -f openclaw

# 2. 备份当前配置（可选）
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 3. 删除配置文件
rm ~/.openclaw/openclaw.json

# 4. (可选) 清理工作空间
# 注意：这会删除所有 Skills 和本地数据
# rm -rf ~/.openclaw/workspace/*

# 5. (可选) 仅删除 Skills
rm -rf ~/.openclaw/workspace/skills/*
```

**重新初始化**

```bash
# 重新运行引导配置
openclaw onboard

# 按之前的步骤重新配置：
# 1. 选择工作空间
# 2. 配置端口
# 3. 安装守护进程
```

**重新应用配置**

```bash
# 编辑新生成的配置文件
nano ~/.openclaw/openclaw.json

# 按照 2.4 节的内容重新配置 Ollama
# 确保添加:
# - "model": "ollama/qwen3.5:14b"
# - "ollama.baseUrl": "http://localhost:11434"
# - sandbox.allowlist 包含 "mcporter" 和 "taobao-native"
```

**重启服务**

```bash
# 重启 Gateway
openclaw gateway restart

# 验证配置
openclaw doctor

# 根据需要重新安装 Skills 和 mcporter（参考第三章）
```

---

## 2.7 配置 WebChat UI (可选)

```bash
# 启动 Web 界面
openclaw webchat --port 3000

# 浏览器访问
open http://localhost:3000
```

---

# 三、配置 taobao-native

## 3.1 前置准备

### 下载淘宝桌面客户端

1. **访问淘宝官网**
   - 打开 https://tbbbk.com/
   - 下载 macOS 版本 (2.5.1 或更高)

2. **安装并登录**
   - 打开 .dmg 安装包
   - 拖拽到 Applications
   - 启动并用淘宝账号登录

---

## 3.2 开启 MCP 服务

1. **进入淘宝设置**
   ```
   淘宝桌面客户端 → 右上角"设置" → "AI 设置"
   ```

2. **启用 MCP**
   ```
   勾选 ✅ "启用 MCP 服务"
   ```

3. **确认端口**
   - 默认端口: `3654`
   - 记录实际端口号 (一般不会改变)

---

## 3.3 安装 mcporter

```bash
# 全局安装 mcporter
npm install -g mcporter

# 验证安装
mcporter --version
```

---

## 3.4 配置 mcporter

### 创建配置文件

```bash
# 创建配置目录
mkdir -p ~/.openclaw/workspace/config

# 创建配置文件
nano ~/.openclaw/workspace/config/mcporter.json
```

### 配置内容

```json
{
  "mcpServers": {
    "taobao-native": {
      "type": "streamableHttp",
      "url": "http://localhost:3654/mcp",
      "timeout": 30000
    }
  }
}
```

---

## 3.5 安装 OpenClaw Skills

### 安装 mcporter Skill

```bash
# 创建目录
mkdir -p ~/.openclaw/workspace/skills/mcporter
cd ~/.openclaw/workspace/skills/mcporter

# 下载 Skill 文件
curl -O https://tblifecdn.taobao.com/taobaopc/skills/mcporter/SKILL.md
```

### 安装 taobao-native Skill

```bash
# 创建目录
mkdir -p ~/.openclaw/workspace/skills/taobao-native
cd ~/.openclaw/workspace/skills/taobao-native

# 下载 Skill 文件
curl -O https://tblifecdn.taobao.com/taobaopc/skills/taobao-native/SKILL.md
```

### 验证 Skills

```bash
# 重启 Gateway
openclaw gateway restart

# 检查 Skills
openclaw skills list

# 应看到:
# ✅ mcporter
# ✅ taobao-native
```

---

## 3.6 测试 taobao-native 连接

```bash
# 测试 MCP 连接
curl http://localhost:3654/mcp

# 或使用 OpenClaw 诊断
openclaw doctor --verbose
```

---

# 四、安装微信 ClawBot 插件

## 4.1 什么是 ClawBot

ClawBot 是 OpenClaw 官方推出的微信机器人插件，允许你通过微信与 AI 助手进行对话，实现移动端随时随地使用购物助手的功能。

**主要特性：**
- 📱 微信原生集成，无需切换应用
- 🔐 端到端加密，保护隐私安全
- 🚀 实时响应，体验流畅
- 🛍️ 支持淘宝购物全流程操作
- 🔔 支持价格监控推送通知

---

## 4.2 前置要求

### 系统要求
- macOS 14.0+ (M 系列芯片)
- 微信 Mac 版 3.8.0+
- OpenClaw Gateway 已正常运行

### 网络要求
- 本地网络畅通（微信插件与 Gateway 通信）
- 可选：Tailscale 配置（远程访问）

---

## 4.3 安装 ClawBot 插件

### 一键安装（推荐）

使用官方 CLI 工具一键安装微信 ClawBot 插件：

```bash
# 运行官方安装命令
npx -y @tencent-weixin/openclaw-weixin-cli@latest install

# 按提示操作：
# 1. 选择安装路径（默认：~/.openclaw/plugins）
# 2. 授予必要的系统权限
# 3. 同意安装微信插件注入器
# 4. 等待安装完成
```

### 验证安装

```bash
# 检查插件是否安装成功
ls ~/.openclaw/plugins/clawbot-wechat

# 启动微信并测试
# 在微信中发送消息给自己：@bot 你好
# 如果收到 AI 回复，说明安装成功
```

---

## 4.4 高级配置

### 远程访问配置（可选）

如果需要在外网使用，可以配置 Tailscale：

```bash
# 编辑 OpenClaw 配置
nano ~/.openclaw/openclaw.json

# 添加 Tailscale 配置
{
  "gateway": {
    "tailscale": {
      "mode": "on",
      "hostname": "openclaw-gateway"
    }
  }
}

# 重启 Gateway
openclaw gateway restart
```

### 白名单配置

限制特定联系人使用：

```json
{
  "wechat": {
    "allowedContacts": [
      "张三",
      "李四",
      "wxid_abc123"
    ]
  }
}
```

---

## 4.5 常见问题

### 问题 1: 微信插件未生效

```bash
# 检查微信版本
# 确保使用 Mac 版 3.8.0+

# 重新安装 ClawBot
npx -y @tencent-weixin/openclaw-weixin-cli@latest uninstall
npx -y @tencent-weixin/openclaw-weixin-cli@latest install

# 完全退出微信后重启
```

### 问题 2: 无法连接 Gateway

```bash
# 检查 Gateway 状态
openclaw doctor

# 检查网络连接
curl http://localhost:18789/health

# 检查插件配置
cat ~/.openclaw/plugins/clawbot-wechat/config.json
```

### 问题 3: 消息无响应

```bash
# 检查插件状态
openclaw plugin status

# 查看插件日志
openclaw plugin logs clawbot-wechat --tail 50

# 重启插件
openclaw plugin restart clawbot-wechat
```

---

## 4.6 隐私与安全

### 数据隐私
- 所有对话数据在本地处理
- 不会上传到云端服务器
- 支持端到端加密

### 安全建议
- 定期更换 Gateway 认证令牌
- 不要在群聊中使用敏感功能
- 定期检查插件更新

### 卸载插件

如果需要卸载：

```bash
# 使用官方 CLI 卸载
npx -y @tencent-weixin/openclaw-weixin-cli@latest uninstall

# 或手动卸载
# 1. 停止插件服务
openclaw plugin stop clawbot-wechat

# 2. 删除插件文件
rm -rf ~/.openclaw/plugins/clawbot-wechat

# 3. 重启微信
```

---

# 附录

## A. 常用命令速查

### Ollama

```bash
# 服务管理
ollama serve                    # 启动服务
ollama list                     # 列出模型
ollama pull <model>             # 下载模型
ollama rm <model>               # 删除模型
ollama run <model> "<prompt>"   # 运行模型

# 模型管理
ollama show qwen3.5:14b         # 查看模型信息
ollama cp qwen3.5:14b my-model  # 复制模型
```

### OpenClaw

```bash
# 基础命令
openclaw --version              # 查看版本
openclaw onboard                # 引导配置
openclaw doctor                 # 健康检查

# Gateway 管理
openclaw gateway                # 启动 Gateway
openclaw gateway restart        # 重启 Gateway
openclaw logs --tail 100        # 查看日志

# Skills 管理
openclaw skills list            # 列出 Skills
openclaw skills search <query>  # 搜索 Skills

# Agent 交互
openclaw agent --message "..."  # 发送消息
openclaw chat                   # 交互模式

# Web 界面
openclaw webchat --port 3000    # 启动 Web UI
```

---

## B. 快速启动脚本

### 一键启动环境

创建 `~/start-openclaw.sh`:

```bash
#!/bin/bash

echo "🚀 启动 OpenClaw 购物助手环境..."

# 1. 启动 Ollama
echo "📦 启动 Ollama..."
ollama serve > /dev/null 2>&1 &
sleep 2

# 2. 启动淘宝客户端
echo "🛒 启动淘宝..."
open -a "淘宝"
sleep 3

# 3. 启动 OpenClaw Gateway
echo "🤖 启动 OpenClaw Gateway..."
openclaw gateway --verbose &
sleep 2

# 4. 启动 ClawBot 插件（可选）
echo "💬 启动 ClawBot 微信插件..."
openclaw plugin start clawbot-wechat
sleep 1

# 5. 启动 Web UI
echo "🌐 启动 Web 界面..."
openclaw webchat --port 3000 &
sleep 2

echo "✅ 环境启动完成！"
echo ""
echo "访问 http://localhost:3000 开始对话"
echo "或在微信中发送 @bot 与 AI 助手对话"
echo ""
```

使用：

```bash
# 添加执行权限
chmod +x ~/start-openclaw.sh

# 运行
~/start-openclaw.sh
```

---

## C. 资源链接

### 官方文档
- [Ollama 官网](https://ollama.com/)
- [OpenClaw 官方文档](https://docs.openclaw.ai/)
- [Qwen 模型库](https://ollama.com/library/qwen3.5)
- [淘宝桌面客户端](https://tbbbk.com/)

### 社区资源
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Discord](https://discord.gg/clawd)
- [Ollama GitHub](https://github.com/ollama/ollama)

---

# 更新日志

## Feature（计划中）
- [ ] Karpathy LLM-Wik

---

## 2026-04-13
- 🔄 调整为本地化部署方案
- 📝 集成 Ollama + Qwen3.5 本地大模型方案
- 🔧 添加 taobao-native MCP 服务配置
- 💬 集成微信 ClawBot 插件支持
- 📦 提供完整的部署与配置指南
- 🛍️ 添加 7 个实战购物对话示例
- 🔨 完善故障排查指南
- ✅ 完成《OpenClaw 淘宝购物指南》文档

## 2026-04-09
- 🎯 项目定项：OpenAI Codex
- 📋 确定使用 OpenAI API 作为初始方案

---

**文档结束**  

现在您可以开始使用 OpenClaw + Ollama + Qwen 本地大模型进行智能购物了！

如有问题，请参考[故障排查](#六故障排查)章节或访问[官方文档](https://docs.openclaw.ai/)
