# Karpathy LLM-Wiki - 资源清单

> **基于**：[Andrej Karpathy的LLM-Wiki方法论](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
> 
> **编写日期**：2026年4月8日
> 
> **版本**：v1.0
> 
> **文档性质**：资源清单

---

## 📑 目录

- [第一部分：核心概念](#第一部分核心概念)
- [第二部分：LLM-Wiki Skill实现资源清单](#第二部分llm-wiki-skill实现资源清单)
- [第三部分：LLM-Wiki开源实现资源清单](#第三部分llm-wiki开源实现资源清单)

---

## 第一部分：核心概念

### 1.1 什么是LLM-Wiki？

**LLM Wiki** 是一种使用大语言模型（LLM）构建和维护**个人知识库**的创新模式，由Andrej Karpathy于2026年4月提出。

**核心思想**：

> 让AI持续编译和维护你的知识库，而不是每次查询时重新检索

---

### 1.2 LLM-Wiki vs 传统RAG

| 特性 | 传统RAG | LLM-Wiki |
|------|---------|----------|
| **工作方式** | 每次查询时检索片段 | AI编译和维护Wiki页面 |
| **知识积累** | ❌ 无积累 | ✅ 持续积累 |
| **知识连接** | ❌ 需要重新发现 | ✅ 自动交叉引用 |
| **记忆** | ❌ 答案消失在历史中 | ✅ 持久化Wiki页面 |
| **进化** | ❌ 不会改进 | ✅ 随时间复利增长 |

---

## 第二部分：LLM-Wiki Skill实现资源清单

### 2.1 项目信息

| 项目 | 信息 |
|------|------|
| **类型** | Agent技能/CLI工具 |
| **目标用户** | 个人开发者、研究人员 |
| **部署方式** | 本地安装 |
| **界面** | 无UI，通过Obsidian查看 |
| **数据存储** | 本地文件系统（Markdown） |
| **参考地址** | https://github.com/sdyckjq-lab/llm-wiki-skill |

---

### 2.2 模型资源

#### 选项1：Claude（推荐）

| 套餐 | 说明 |
|------|------|
| **Claude Pro** | 消息限制较少，优先访问新模型 |
| **Claude API** | 更灵活，适合重度使用 |

#### 选项2：OpenAI

| 套餐 | 说明 |
|------|------|
| **ChatGPT Plus** | 包含Code Interpreter |
| **OpenAI API** | GPT-4按token计费 |

#### 选项3：本地模型（免费）

需要独立部署本地LLM（如Ollama、LM Studio）

---

### 2.3 硬件要求

#### 云端模型方案

| 组件 | 最低要求 | 推荐配置 |
|------|---------|---------||
| **CPU** | 双核 2GHz | 四核 3GHz+ |
| **内存** | 8GB | 16GB+ |
| **存储** | 10GB可用空间 | 50GB+ SSD |
| **网络** | 稳定网络连接 | 10Mbps+ |

#### 本地模型方案

需要额外GPU资源（根据模型大小而定）

---

### 2.4 软件依赖

#### 核心组件

| 软件 | 用途 | 安装方式 |
|------|------|---------|
| **Bash** | 运行LLM-Wiki-Skill脚本 | 系统自带（Mac/Linux） |
| **AI编程助手** | Claude Code/Codex/OpenClaw | 订阅服务 |
| **Obsidian** | Wiki查看和导航 | 下载安装 |

#### 内容处理工具

| 工具 | 用途 | 安装方式 |
|------|------|---------|
| **baoyu-url-to-markdown** | 网页转Markdown | npm install |
| **wechat-article-to-markdown** | 微信文章转换 | npm install |
| **yt-dlp** | YouTube视频下载 | pip install |
| **Whisper** | 音视频转录 | pip install openai-whisper |

---

### 2.5 系统权限要求

| 权限 | 说明 |
|------|------|
| **文件系统读写** | 读取raw-sources/，写入wiki/ |
| **网络访问** | 调用LLM API，下载网页内容 |
| **命令行执行** | 运行Bash脚本和处理工具 |

---

### 2.6 目录结构

```
llm-wiki/
├── raw-sources/        # 原始资料（只读）
│   ├── pdfs/
│   ├── articles/
│   └── notes/
├── wiki/               # AI生成Wiki（可写）
│   ├── entities/
│   ├── concepts/
│   └── summaries/
└── AGENTS.md           # Schema配置
```

---

## 第三部分：LLM-Wiki开源实现资源清单

### 3.1 项目信息

| 项目 | 信息 |
|------|------|
| **类型** | 全栈Web应用 |
| **目标用户** | 团队、企业、高级用户 |
| **部署方式** | 自托管（云服务器/本地服务器） |
| **界面** | 现代化Web UI |
| **数据存储** | PostgreSQL + S3 |
| **参考地址** | https://github.com/lucasastorian/llmwiki |
| **官方服务** | https://llmwiki.app/ |

---

### 3.2 模型资源

#### Claude API（主要使用）

| 用途 | 说明 |
|------|------|
| **MCP集成** | 通过MCP协议调用Claude |
| **文档处理** | OCR使用Mistral API |

---

### 3.3 硬件要求

#### 服务器配置

| 组件 | 配置 | 说明 |
|------|------|------|
| **Web服务器** | 8核16GB，1-2台 | 运行Next.js前端 |
| **API服务器** | 8核32GB，1-2台 | 运行FastAPI后端 |
| **数据库** | 根据规模选择 | PostgreSQL单机或集群 |
| **对象存储** | 1TB+ | S3或兼容服务 |

---

### 3.4 软件依赖

#### 前端技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| **Next.js** | 16 | React框架 |
| **React** | 19 | UI库 |
| **TypeScript** | 最新 | 类型安全 |
| **TailwindCSS** | 最新 | 样式框架 |

#### 后端技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| **Python** | 3.11+ | 后端语言 |
| **FastAPI** | 最新 | Web框架 |
| **MCP SDK** | 最新 | Claude集成 |
| **Asyncio** | 内置 | 异步处理 |

#### 数据库与存储

| 技术 | 用途 |
|------|------|
| **Supabase** | PostgreSQL托管服务（可选） |
| **PostgreSQL** | 主数据库 |
| **PGroonga** | 中英文全文搜索扩展 |
| **S3存储** | MinIO/Cloudflare R2/AWS S3 |

#### MCP服务器

| 工具 | 功能 |
|------|------|
| **guide** | 获取Wiki使用指南 |
| **search** | 搜索文档和Wiki页面 |
| **read** | 读取文档内容 |
| **write** | 创建/更新Wiki页面 |
| **delete** | 删除Wiki页面 |

---

### 3.5 系统权限要求

| 权限类型 | 说明 |
|----------|------|
| **服务器管理** | 安装配置Web/API服务器 |
| **数据库管理** | 创建数据库、安装扩展（PGroonga） |
| **对象存储** | 配置S3 bucket和访问策略 |
| **网络配置** | 域名、SSL证书、防火墙规则 |
| **Docker权限** | （可选）用于容器化部署 |

---

**文档结束**
