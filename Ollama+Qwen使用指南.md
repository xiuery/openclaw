# Ollama + Qwen 使用指南

> **编写日期**：2026年4月10日  
> **适用平台**：MacBook Pro (Apple Silicon / Intel)  
> **模型**：Qwen2.5-Coder / Qwen3.5

---

## 📋 目录

1. [Ollama 简介](#1-ollama-简介)
2. [系统要求](#2-系统要求)
3. [安装 Ollama](#3-安装-ollama)
4. [Qwen 模型下载](#4-qwen-模型下载)
5. [基础操作命令](#5-基础操作命令)
6. [模型配置与优化](#6-模型配置与优化)
7. [API 使用](#7-api-使用)
8. [集成应用](#8-集成应用)
9. [性能优化](#9-性能优化)
10. [故障排查](#10-故障排查)
11. [实战案例](#11-实战案例)

---

## 1. Ollama 简介

### 1.1 什么是 Ollama？

**Ollama** 是一个开源的本地大语言模型运行平台：

- 🚀 **易于使用**：一键安装，命令行操作简单
- 💻 **本地运行**：完全离线，数据隐私安全
- 🔧 **模型管理**：轻松下载、切换、删除模型
- 🌐 **API 支持**：OpenAI 兼容的 REST API
- 🍎 **原生支持**：针对 Apple Silicon 优化

### 1.2 Qwen 模型系列

| 模型 | 说明 | 参数量 | 使用场景 |
|------|------|--------|----------|
| **Qwen2.5-Coder** | 代码专用 | 1.5B-32B | 代码生成、补全、技术问答 |
| **Qwen3.5** | 通用聊天 | 0.5B-110B | 对话、文本生成、知识问答 |

---

## 2. 系统要求

### 2.1 硬件要求

#### Apple Silicon (推荐)

| 组件 | 最低 | 推荐 | 最佳 |
|------|------|------|------|
| 芯片 | M1 | M2/M3 | M3 Max/Ultra |
| 内存 | 8GB | 16GB | 32GB+ |
| 存储 | 20GB | 50GB+ | 100GB+ |
| macOS | 12.0+ | 13.0+ | 14.0+ |

#### Intel Mac

| 组件 | 最低 | 推荐 |
|------|------|------|
| CPU | i5 | i7/i9 |
| 内存 | 16GB | 32GB+ |
| 存储 | 20GB | 50GB+ |

### 2.2 模型大小参考

| 模型 | 大小 | 推荐内存 |
|------|------|----------|
| qwen2.5-coder:1.5b | ~1GB | 8GB |
| qwen2.5-coder:7b | ~4.7GB | 16GB |
| qwen2.5-coder:14b | ~8.5GB | 24GB |
| qwen2.5-coder:32b | ~19GB | 32GB+ |
| qwen3.5:7b | ~4.7GB | 16GB |
| qwen3.5:72b | ~45GB | 64GB+ |

---

## 3. 安装 Ollama

### 3.1 方法一：官方安装包（推荐）

```bash
# 下载
curl -L https://ollama.ai/download/Ollama-darwin.zip -o ~/Downloads/Ollama.zip

# 解压到 Applications
unzip ~/Downloads/Ollama.zip -d /Applications/

# 验证
ollama --version
```

### 3.2 方法二：Homebrew

```bash
# 安装
brew install ollama

# 验证
ollama --version
```

### 3.3 启动服务

#### 菜单栏应用（推荐）
- 打开 Applications → Ollama
- 菜单栏出现图标即表示服务运行

#### 命令行启动

```bash
# 前台运行
ollama serve

# 后台运行
nohup ollama serve > /dev/null 2>&1 &

# 检查状态
curl http://localhost:11434/api/tags
```

#### 开机自启（launchd）

```bash
cat > ~/Library/LaunchAgents/com.ollama.server.plist <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.ollama.server</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/ollama</string>
        <string>serve</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
EOF

# 加载服务
launchctl load ~/Library/LaunchAgents/com.ollama.server.plist

# 卸载服务
# launchctl unload ~/Library/LaunchAgents/com.ollama.server.plist
```

---

## 4. Qwen 模型下载

### 4.1 Qwen2.5-Coder 系列

```bash
# 1.5B（轻量级）
ollama run qwen2.5-coder:1.5b

# 7B（推荐）
ollama run qwen2.5-coder:7b

# 14B（高性能）
ollama run qwen2.5-coder:14b

# 32B（最强）
ollama run qwen2.5-coder:32b
```

### 4.2 Qwen3.5 系列

```bash
# 0.5B（超轻量）
ollama run qwen3.5:0.5b

# 7B（推荐）
ollama run qwen3.5:7b

# 14B
ollama run qwen3.5:14b

# 32B
ollama run qwen3.5:32b

# 72B（旗舰）
ollama run qwen3.5:72b
```

### 4.3 量化版本

| 后缀 | 量化 | 精度 | 大小 | 速度 |
|------|------|------|------|------|
| 默认 | Q4_K_M | 中 | ~50% | 快 |
| :q8 | Q8_0 | 高 | ~80% | 中 |
| :q4 | Q4_0 | 低 | ~40% | 很快 |

```bash
# 高精度版本
ollama run qwen2.5-coder:7b-q8

# 快速版本
ollama run qwen2.5-coder:7b-q4
```

---

## 5. 基础操作命令

### 5.1 模型管理

```bash
# 下载模型
ollama pull qwen2.5-coder:7b

# 运行模型（交互式）
ollama run qwen2.5-coder:7b

# 运行模型（单次提示）
ollama run qwen2.5-coder:7b "写一个Python快速排序"

# 列出已安装模型
ollama list

# 查看模型详情
ollama show qwen2.5-coder:7b

# 删除模型
ollama rm qwen2.5-coder:7b

# 清理未使用模型
ollama prune
```

### 5.2 对话操作

#### 交互模式

```bash
ollama run qwen2.5-coder:7b

>>> 写一个冒泡排序
# ... AI 回复 ...

>>> 优化这个代码
# ... AI 继续优化 ...

>>> /bye  # 退出
```

#### 特殊命令

| 命令 | 功能 |
|------|------|
| `/bye` | 退出 |
| `/clear` | 清除上下文 |
| `/set parameter value` | 设置参数 |
| `/show info` | 显示信息 |

### 5.3 系统管理

```bash
# 查看运行状态
ollama ps

# 更新 Ollama
brew upgrade ollama  # Homebrew 安装的

# 重启服务
pkill ollama && ollama serve
```

---

## 6. 模型配置与优化

### 6.1 创建自定义模型

#### 步骤 1：创建 Modelfile

```bash
cat > Modelfile <<'EOF'
FROM qwen2.5-coder:7b

SYSTEM """
你是一个专业的 Python 编程助手。
回答要求：
1. 代码简洁清晰
2. 添加详细注释
3. 包含使用示例
"""

PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER top_k 40
PARAMETER num_ctx 4096
PARAMETER repeat_penalty 1.1

TEMPLATE """{{ .System }}

User: {{ .Prompt }}
Assistant:"""
EOF
```

#### 步骤 2：构建自定义模型

```bash
# 创建名为 my-python-coder 的模型
ollama create my-python-coder -f Modelfile

# 运行自定义模型
ollama run my-python-coder
```

### 6.2 参数详解

| 参数 | 说明 | 默认值 | 范围 | 用途 |
|------|------|--------|------|------|
| `temperature` | 随机性 | 0.8 | 0-2 | 越高越创造性 |
| `top_p` | 核采样 | 0.9 | 0-1 | 控制多样性 |
| `top_k` | 候选词数 | 40 | 1-100 | 限制选择范围 |
| `num_ctx` | 上下文长度 | 2048 | 512-32768 | Token 窗口大小 |
| `repeat_penalty` | 重复惩罚 | 1.1 | 1.0-2.0 | 避免重复 |
| `num_predict` | Max tokens | -1 | -1/1-2048 | 最大生成长度 |

#### 不同场景的参数配置

**代码生成（精确）**：
```bash
PARAMETER temperature 0.3
PARAMETER top_p 0.7
PARAMETER repeat_penalty 1.1
```

**创意写作**：
```bash
PARAMETER temperature 1.2
PARAMETER top_p 0.95
PARAMETER repeat_penalty 1.05
```

**技术文档**：
```bash
PARAMETER temperature 0.5
PARAMETER top_p 0.85
PARAMETER num_ctx 8192
```

### 6.3 高级配置示例

#### 示例 1：代码审查助手

```bash
cat > CodeReviewer <<'EOF'
FROM qwen2.5-coder:7b

SYSTEM """
你是一个资深代码审查专家。
审查重点：
1. 代码规范性
2. 性能优化点
3. 潜在 bug
4. 安全隐患
5. 改进建议
"""

PARAMETER temperature 0.4
PARAMETER num_ctx 8192
EOF

ollama create code-reviewer -f CodeReviewer
```

#### 示例 2：中英翻译助手

```bash
cat > Translator <<'EOF'
FROM qwen3.5:7b

SYSTEM """
你是专业的中英文翻译。
翻译要求：
1. 准确传达原意
2. 符合目标语言习惯
3. 保持专业术语准确性
"""

PARAMETER temperature 0.3
PARAMETER top_p 0.8
EOF

ollama create translator -f Translator
```

---

## 7. API 使用

### 7.1 REST API 基础

Ollama 提供兼容 OpenAI 的 REST API，默认运行在 `http://localhost:11434`。

#### 生成补全

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "qwen2.5-coder:7b",
  "prompt": "写一个 Python 快速排序",
  "stream": false
}'
```

#### 流式响应

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "qwen2.5-coder:7b",
  "prompt": "解释什么是递归",
  "stream": true
}'
```

#### 聊天接口

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "qwen3.5:7b",
  "messages": [
    {
      "role": "system",
      "content": "你是一个helpful助手"
    },
    {
      "role": "user",
      "content": "什么是机器学习？"
    }
  ],
  "stream": false
}'
```

### 7.2 Python SDK

#### 安装

```bash
pip install ollama
```

#### 基础使用

```python
import ollama

# 单次对话
response = ollama.chat(model='qwen2.5-coder:7b', messages=[
  {
    'role': 'user',
    'content': '写一个 Python 快速排序',
  },
])
print(response['message']['content'])

# 流式响应
stream = ollama.chat(
    model='qwen3.5:7b',
    messages=[{'role': 'user', 'content': '解释什么是深度学习？'}],
    stream=True,
)

for chunk in stream:
  print(chunk['message']['content'], end='', flush=True)
```

#### 高级示例：代码生成器

```python
import ollama

def generate_code(description, language="Python"):
    prompt = f"用 {language} 编写：{description}\n要求：添加注释，包含示例"
    
    response = ollama.chat(
        model='qwen2.5-coder:7b',
        messages=[
            {
                'role': 'system',
                'content': '你是专业代码生成助手，代码要清晰、高效、有注释。'
            },
            {
                'role': 'user',
                'content': prompt
            }
        ],
        options={
            'temperature': 0.3,
            'num_ctx': 4096
        }
    )
    
    return response['message']['content']

# 使用示例
code = generate_code("二分查找算法")
print(code)
```

#### 多轮对话示例

```python
import ollama

messages = []

def chat(user_input):
    messages.append({'role': 'user', 'content': user_input})
    
    response = ollama.chat(
        model='qwen3.5:7b',
        messages=messages
    )
    
    assistant_message = response['message']['content']
    messages.append({'role': 'assistant', 'content': assistant_message})
    
    return assistant_message

# 对话示例
print(chat("什么是快速排序？"))
print(chat("它的时间复杂度是多少？"))
print(chat("给我写一个 Python 实现"))
```

### 7.3 JavaScript/Node.js SDK

#### 安装

```bash
npm install ollama
```

#### 基础使用

```javascript
import ollama from 'ollama'

// 单次对话
const response = await ollama.chat({
  model: 'qwen2.5-coder:7b',
  messages: [{ role: 'user', content: '写一个 JavaScript 快速排序' }],
})
console.log(response.message.content)

// 流式响应
const stream = await ollama.chat({
  model: 'qwen3.5:7b',
  messages: [{ role: 'user', content: '什么是 Promise？' }],
  stream: true,
})

for await (const chunk of stream) {
  process.stdout.write(chunk.message.content)
}
```

### 7.4 OpenAI 兼容 API

Ollama 兼容 OpenAI API 格式：

```python
from openai import OpenAI

client = OpenAI(
    base_url = 'http://localhost:11434/v1',
    api_key='ollama',  # 必填但可以是任意值
)

response = client.chat.completions.create(
  model="qwen2.5-coder:7b",
  messages=[
    {"role": "system", "content": "你是一个编程助手"},
    {"role": "user", "content": "写一个 Fibonacci 函数"}
  ]
)

print(response.choices[0].message.content)
```

---

## 8. 集成应用

### 8.1 VS Code 集成

#### 方法 1：Continue 插件（推荐）

**安装步骤**：

1. 打开 VS Code
2. 安装 Continue 插件
3. 配置 `~/.continue/config.json`：

```json
{
  "models": [
    {
      "title": "Qwen2.5-Coder",
      "provider": "ollama",
      "model": "qwen2.5-coder:7b",
      "apiBase": "http://localhost:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen2.5-Coder Tab",
    "provider": "ollama",
    "model": "qwen2.5-coder:1.5b"
  }
}
```

**使用**：
- `Cmd+I`：代码生成
- `Cmd+Shift+I`：聊天窗口
- Tab：自动补全

#### 方法 2：Ollama Autocoder

```bash
# 在 VS Code 中安装 Ollama Autocoder 插件

# 配置 settings.json
{
  "ollama.model": "qwen2.5-coder:7b",
  "ollama.url": "http://localhost:11434"
}
```

### 8.2 终端集成

#### Shell 助手

创建 `~/.ollama_helper.sh`：

```bash
#!/bin/bash

# AI 命令助手
aicmd() {
    local question="$*"
    ollama run qwen2.5-coder:7b "作为Linux专家，回答命令相关问题：$question。只返回命令，不要解释。"
}

# AI 代码助手
aicode() {
    local description="$*"
    ollama run qwen2.5-coder:7b "$description"
}

# Git commit 消息生成
aigit() {
    local diff=$(git diff --cached)
    ollama run qwen3.5:7b "根据以下代码变更生成简洁的commit消息（中文）：\n$diff"
}
```

加载到 shell：
```bash
source ~/.ollama_helper.sh

# 使用示例
aicmd "如何查找大于100MB的文件"
aicode "写一个Python读取CSV的脚本"
aigit
```

### 8.3 Raycast 集成

#### 安装 AI Commands

1. 安装 Raycast
2. 安装 Ollama extension
3. 配置模型：Settings → Extensions → Ollama

#### 自定义命令

创建快捷指令：
- **Explain Code**: 解释选中的代码
- **Fix Code**: 修复代码错误
- **Write Tests**: 生成单元测试
- **Translate**: 中英翻译

### 8.4 Open WebUI（Web 界面）

#### 安装

```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

#### 配置

1. 访问 `http://localhost:3000`
2. 注册账号
3. 设置 → Models → 添加 Ollama endpoint：`http://host.docker.internal:11434`

**功能**：
- ChatGPT 风格界面
- 多模型切换
- 对话历史管理
- Prompt 模板库
- 文档上传

### 8.5 n8n 工作流集成

#### Docker Compose 配置

```yaml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

#### 工作流示例：自动代码审查

1. Webhook 接收 GitHub PR
2. Ollama 节点分析代码
3. Comment 回 GitHub

```json
{
  "nodes": [
    {
      "name": "Ollama",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "http://host.docker.internal:11434/api/generate",
        "method": "POST",
        "bodyParameters": {
          "model": "qwen2.5-coder:7b",
          "prompt": "审查以下代码：{{ $json.code }}"
        }
      }
    }
  ]
}
```

---

## 9. 性能优化

### 9.1 硬件加速

#### Apple Silicon 优化

Ollama 自动使用 Metal API 加速，无需额外配置。

**验证 GPU 使用**：

```bash
# 运行模型时打开活动监视器
# GPU 历史记录会显示使用情况

# 或使用命令行
sudo powermetrics --samplers gpu_power -i1000 -n1
```

#### 环境变量优化

```bash
# 设置 GPU 层数（-1 = 全部使用 GPU）
export OLLAMA_NUM_GPU=-1

# 设置上下文大小
export OLLAMA_NUM_CONTEXT=4096

# 启动服务
ollama serve
```

### 9.2 内存管理

#### 模型卸载策略

```bash
# 设置模型保持时间（默认5分钟）
export OLLAMA_KEEP_ALIVE=10m

# 立即卸载
export OLLAMA_KEEP_ALIVE=0

# 永久保持
export OLLAMA_KEEP_ALIVE=-1
```

#### 监控内存使用

```bash
# 查看模型内存占用
ollama ps

# 输出示例：
# NAME                SIZE    PROCESSOR    UNTIL
# qwen2.5-coder:7b   4.7GB   100% GPU     4 min
```

### 9.3 并发配置

```bash
# 限制并发请求数（默认为CPU核心数）
export OLLAMA_MAX_LOADED_MODELS=2

# 设置每个模型的线程数
export OLLAMA_NUM_THREAD=8
```

### 9.4 批处理优化

```python
import ollama
import concurrent.futures

def process_prompt(prompt):
    return ollama.generate(model='qwen2.5-coder:7b', prompt=prompt)

prompts = [
    "写一个快速排序",
    "写一个冒泡排序",
    "写一个归并排序"
]

# 并发处理
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(process_prompt, prompts))

for result in results:
    print(result['response'])
```

### 9.5 缓存策略

#### Prompt 缓存

```python
# 使用相同的 system prompt 可以复用缓存
system_prompt = "你是一个Python专家"

messages_base = [{'role': 'system', 'content': system_prompt}]

# 多个请求共享 system prompt
for question in questions:
    messages = messages_base + [{'role': 'user', 'content': question}]
    response = ollama.chat(model='qwen2.5-coder:7b', messages=messages)
```

### 9.6 性能基准测试

```bash
# 测试推理速度
time ollama run qwen2.5-coder:7b "写一个100行的Python脚本" --verbose

# 输出包括：
# - Tokens per second
# - Total duration
# - Load duration
# - Prompt eval duration
```

**性能参考**（M2 Max 64GB）：

| 模型 | 加载时间 | 生成速度 | 内存占用 |
|------|---------|---------|----------|
| qwen2.5-coder:1.5b | ~1s | 80-100 t/s | ~1GB |
| qwen2.5-coder:7b | ~3s | 40-60 t/s | ~5GB |
| qwen2.5-coder:14b | ~5s | 20-30 t/s | ~9GB |
| qwen3.5:7b | ~3s | 40-60 t/s | ~5GB |

---

## 10. 故障排查

### 10.1 常见问题

#### 问题 1：服务无法启动

**症状**：
```
Error: listen tcp 127.0.0.1:11434: bind: address already in use
```

**解决**：
```bash
# 查找占用端口的进程
lsof -i :11434

# 杀死进程
kill -9 <PID>

# 重启服务
ollama serve
```

#### 问题 2：模型下载失败

**症状**：
```
Error: failed to download model
```

**解决**：
```bash
# 检查网络连接
ping ollama.ai

# 使用代理
export https_proxy=http://127.0.0.1:7890
ollama pull qwen2.5-coder:7b

# 手动指定 mirror（如果有）
export OLLAMA_HOST=https://mirror.example.com
```

#### 问题 3：内存不足

**症状**：
```
Error: failed to load model: out of memory
```

**解决**：
```bash
# 使用更小的模型
ollama run qwen2.5-coder:1.5b

# 或使用更激进的量化
ollama run qwen2.5-coder:7b-q4

# 清理未使用的模型
ollama prune
```

#### 问题 4：响应速度慢

**解决**：
```bash
# 检查是否使用 GPU
ollama ps  # 查看 PROCESSOR 列

# 减小上下文窗口
export OLLAMA_NUM_CONTEXT=2048

# 使用更小的模型
ollama run qwen2.5-coder:7b  # 而非 14b
```

#### 问题 5：生成质量差

**解决**：
```bash
# 调整 temperature（降低随机性）
ollama run qwen2.5-coder:7b
>>> /set temperature 0.3

# 使用更大的模型
ollama run qwen2.5-coder:14b

# 优化 prompt
# 不好的：
"写代码"

# 好的：
"用 Python 编写一个快速排序函数，要求：
1. 添加详细注释
2. 处理边界情况
3. 包含时间复杂度分析
4. 提供使用示例"
```

### 10.2 诊断命令

```bash
# 查看服务日志
tail -f /tmp/ollama.log  # 如果配置了 launchd

# 查看模型详情
ollama show qwen2.5-coder:7b --modelfile

# 查看系统信息
ollama --version
system_profiler SPHardwareDataType

# 测试 API
curl http://localhost:11434/api/tags

# 检查模型完整性
ollama list
```

### 10.3 日志分析

#### 启用详细日志

```bash
# 设置日志级别
export OLLAMA_DEBUG=1

ollama serve
```

#### 常见错误代码

| 错误码 | 含义 | 解决方法 |
|-------|------|---------|
| 404 | 模型不存在 | 检查模型名称，重新 pull |
| 500 | 服务器错误 | 查看日志，重启服务 |
| 503 | 服务不可用 | 检查服务是否运行 |

### 10.4 性能调优检查清单

- [ ] 使用 Apple Silicon Mac
- [ ] 安装最新版本 Ollama
- [ ] 选择合适大小的模型
- [ ] 设置合理的 `OLLAMA_NUM_CONTEXT`
- [ ] 配置 `OLLAMA_KEEP_ALIVE`
- [ ] 关闭不必要的后台程序
- [ ] 确保充足的可用内存
- [ ] 使用量化模型（如需要）
- [ ] 优化 prompt 质量

---

## 11. 实战案例

### 11.1 案例 1：智能代码助手

#### 需求
创建一个 CLI 工具，通过自然语言生成代码。

#### 实现

```python
#!/usr/bin/env python3
# code_assistant.py

import ollama
import sys

def generate_code(description, language="Python"):
    """生成代码"""
    prompt = f"""
作为专业{language}开发者，根据以下需求编写代码：

需求：{description}

要求：
1. 代码简洁高效
2. 添加详细注释
3. 包含错误处理
4. 提供使用示例
"""
    
    response = ollama.chat(
        model='qwen2.5-coder:7b',
        messages=[
            {'role': 'system', 'content': f'你是{language}专家，编写高质量代码。'},
            {'role': 'user', 'content': prompt}
        ],
        options={
            'temperature': 0.3,
            'num_ctx': 4096
        }
    )
    
    return response['message']['content']

def main():
    if len(sys.argv) < 2:
        print("用法: python code_assistant.py '需求描述' [语言]")
        sys.exit(1)
    
    description = sys.argv[1]
    language = sys.argv[2] if len(sys.argv) > 2 else "Python"
    
    print(f"正在生成 {language} 代码...\n")
    code = generate_code(description, language)
    print(code)

if __name__ == "__main__":
    main()
```

#### 使用

```bash
# 生成 Python 代码
python code_assistant.py "读取CSV文件并计算平均值"

# 生成 JavaScript 代码
python code_assistant.py "实现防抖函数" JavaScript
```

### 11.2 案例 2：自动化代码审查

```python
#!/usr/bin/env python3
# code_reviewer.py

import ollama
import sys
import os

def review_code(file_path):
    """审查代码文件"""
    with open(file_path, 'r') as f:
        code = f.read()
    
    prompt = f"""
审查以下代码，提供详细分析：

```
{code}
```

请从以下方面审查：
1. 代码质量和规范性
2. 性能优化建议
3. 潜在的bug
4. 安全隐患
5. 可读性改进建议

输出格式：
## 整体评分：X/10

## 优点：
- ...

## 问题：
- ...

## 改进建议：
- ...
"""
    
    response = ollama.chat(
        model='qwen2.5-coder:7b',
        messages=[
            {'role': 'system', 'content': '你是资深代码审查专家。'},
            {'role': 'user', 'content': prompt}
        ],
        options={'temperature': 0.4}
    )
    
    return response['message']['content']

def main():
    if len(sys.argv) < 2:
        print("用法: python code_reviewer.py <代码文件>")
        sys.exit(1)
    
    file_path = sys.argv[1]
    
    if not os.path.exists(file_path):
        print(f"错误：文件 {file_path} 不存在")
        sys.exit(1)
    
    print(f"正在审查 {file_path}...\n")
    review = review_code(file_path)
    print(review)

if __name__ == "__main__":
    main()
```

### 11.3 案例 3：Git Commit 消息生成器

```bash
#!/bin/bash
# git-ai-commit.sh

# 获取暂存的改动
DIFF=$(git diff --cached)

if [ -z "$DIFF" ]; then
    echo "没有暂存的改动"
    exit 1
fi

# 使用 Ollama 生成 commit 消息
COMMIT_MSG=$(ollama run qwen3.5:7b "根据以下 git diff 生成简洁的中文 commit 消息（50字以内）：

$DIFF

只返回 commit 消息，不要额外解释。" | head -n 1)

echo "建议的 commit 消息："
echo "  $COMMIT_MSG"
echo ""
read -p "使用此消息提交? (y/n) " -n 1 -r
echo

if [[ $REPLY =~ ^[Yy]$ ]]; then
    git commit -m "$COMMIT_MSG"
    echo "已提交！"
else
    echo "已取消"
fi
```

#### 使用

```bash
# 赋予执行权限
chmod +x git-ai-commit.sh

# 暂存改动
git add .

# 自动生成并提交
./git-ai-commit.sh
```

### 11.4 案例 4：文档翻译工具

```python
#!/usr/bin/env python3
# translator.py

import ollama
import sys

def translate(text, target_lang="英文"):
    """翻译文本"""
    response = ollama.chat(
        model='qwen3.5:7b',
        messages=[
            {
                'role': 'system',
                'content': f'你是专业翻译，将文本翻译成{target_lang}。保持格式，准确传达原意。'
            },
            {
                'role': 'user',
                'content': text
            }
        ],
        options={'temperature': 0.3}
    )
    
    return response['message']['content']

def translate_file(input_file, output_file, target_lang="英文"):
    """翻译整个文件"""
    with open(input_file, 'r', encoding='utf-8') as f:
        content = f.read()
    
    print(f"正在翻译到{target_lang}...")
    translated = translate(content, target_lang)
    
    with open(output_file, 'w', encoding='utf-8') as f:
        f.write(translated)
    
    print(f"已保存到 {output_file}")

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("用法: python translator.py <输入文件> <输出文件> [目标语言]")
        sys.exit(1)
    
    input_file = sys.argv[1]
    output_file = sys.argv[2]
    target_lang = sys.argv[3] if len(sys.argv) > 3 else "英文"
    
    translate_file(input_file, output_file, target_lang)
```

### 11.5 案例 5：交互式学习助手

```python
#!/usr/bin/env python3
# learning_assistant.py

import ollama

class LearningAssistant:
    def __init__(self, topic, model='qwen3.5:7b'):
        self.topic = topic
        self.model = model
        self.messages = [
            {
                'role': 'system',
                'content': f'你是{topic}领域的专业导师，擅长深入浅出地解释概念，并提供实例。'
            }
        ]
    
    def ask(self, question):
        """提问"""
        self.messages.append({'role': 'user', 'content': question})
        
        response = ollama.chat(
            model=self.model,
            messages=self.messages,
            options={'temperature': 0.7}
        )
        
        answer = response['message']['content']
        self.messages.append({'role': 'assistant', 'content': answer})
        
        return answer
    
    def quiz(self):
        """生成测验题"""
        question = f"根据我们之前讨论的{self.topic}内容，出3道选择题来测试理解。"
        return self.ask(question)
    
    def summarize(self):
        """总结学习内容"""
        return self.ask(f"总结我们关于{self.topic}的讨论要点。")

def main():
    print("=== AI 学习助手 ===\n")
    topic = input("你想学习什么主题？ ")
    
    assistant = LearningAssistant(topic)
    print(f"\n开始学习 {topic}！输入 'quit' 退出，'quiz' 测验，'summary' 总结。\n")
    
    while True:
        question = input("你: ").strip()
        
        if question.lower() == 'quit':
            print("再见！")
            break
        elif question.lower() == 'quiz':
            print("\nAI: " + assistant.quiz() + "\n")
        elif question.lower() == 'summary':
            print("\nAI: " + assistant.summarize() + "\n")
        elif question:
            print("\nAI: " + assistant.ask(question) + "\n")

if __name__ == "__main__":
    main()
```

#### 使用示例

```bash
python learning_assistant.py

# 输入：
# 你想学习什么主题？ Python装饰器
# 
# 你: 什么是装饰器？
# AI: 装饰器是Python的一个强大特性...
# 
# 你: 给我一个例子
# AI: 好的，这是一个简单的例子...
# 
# 你: quiz
# AI: 好的，这里有3道选择题...
```

---

## 📚 附录

### A. 快速参考

#### 常用命令速查

```bash
# 模型管理
ollama pull <model>          # 下载模型
ollama list                  # 列出模型
ollama rm <model>            # 删除模型
ollama show <model>          # 显示详情

# 运行
ollama run <model>           # 交互模式
ollama run <model> "prompt"  # 单次运行

# 服务
ollama serve                 # 启动服务
ollama ps                    # 查看状态
pkill ollama                 # 停止服务

# API
curl http://localhost:11434/api/generate -d '{...}'
curl http://localhost:11434/api/chat -d '{...}'
```

#### 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `OLLAMA_HOST` | 服务地址 | `127.0.0.1:11434` |
| `OLLAMA_MODELS` | 模型目录 | `~/.ollama/models` |
| `OLLAMA_NUM_GPU` | GPU层数 | `-1`（全部） |
| `OLLAMA_NUM_CONTEXT` | 上下文长度 | `2048` |
| `OLLAMA_KEEP_ALIVE` | 保持时间 | `5m` |

### B. 推荐资源

- **官网**: https://ollama.ai
- **GitHub**: https://github.com/ollama/ollama
- **模型库**: https://ollama.ai/library
- **Qwen官网**: https://qwenlm.github.io
- **Discord社区**: https://discord.gg/ollama

### C. 更新日志

- **2026-04-10**: 创建文档，基于 Ollama 0.1.29
- 未来更新将包含最新版本特性

---

**文档终结** | 共计 11 个章节 | 适用于 MacBook Pro 用户