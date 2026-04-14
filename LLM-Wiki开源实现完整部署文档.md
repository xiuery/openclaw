# LLM Wiki 开源实现完整部署文档

> **项目地址**：https://github.com/lucasastorian/llmwiki
> 
> **官方服务**：https://llmwiki.app/
> 
> **基于**：[Karpathy的LLM-Wiki方法论](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
> 
> **编写日期**：2026年4月8日
> 
> **版本**：v1.0
> 
> **Star数量**：56+ | **Fork数量**：10+

---

## 📑 目录

- [1. 项目概述](#1-项目概述)
- [2. 核心理念与架构](#2-核心理念与架构)
- [3. 技术栈详解](#3-技术栈详解)
- [4. 云端服务快速开始](#4-云端服务快速开始)
- [5. 自托管完整指南](#5-自托管完整指南)
- [6. MCP连接配置](#6-mcp连接配置)
- [7. 功能使用指南](#7-功能使用指南)
- [8. 高级配置](#8-高级配置)
- [9. 故障排查](#9-故障排查)
- [10. 性能优化](#10-性能优化)
- [11. 安全最佳实践](#11-安全最佳实践)
- [12. 开发者指南](#12-开发者指南)
- [13. FAQ常见问题](#13-faq常见问题)
- [14. 路线图与贡献](#14-路线图与贡献)

---

## 1. 项目概述

### 1.1 这是什么？

**LLM Wiki** 是 Karpathy 的 LLM Wiki 方法论的**完整Web应用实现**，提供：

- 🌐 **完整的Web界面**（不是命令行工具）
- 📄 **文档管理系统**（上传、查看、组织）
- 🤖 **Claude MCP集成**（AI自动维护Wiki）
- ☁️ **托管服务** + 🏠 **自托管选项**

---

### 1.2 与其他实现的区别

| 维度 | llm-wiki-skill | LLM Wiki (本项目) |
|------|----------------|------------------|
| **类型** | Agent技能/插件 | 完整Web应用 |
| **界面** | 无UI（通过Obsidian查看） | 现代化Web界面 |
| **部署** | 本地安装到Agent | 云端 or 自托管服务器 |
| **文档查看** | Obsidian | 内置PDF/HTML查看器 |
| **数据库** | 文件系统 | PostgreSQL (Supabase) |
| **存储** | 本地文件 | S3对象存储 |
| **多用户** | 单用户 | 支持多用户 |
| **MCP集成** | 间接支持 | 原生MCP服务器 |

---

### 1.3 核心价值

#### 对比传统RAG

```
传统RAG：文档 → 向量索引 → 检索片段 → 拼凑答案
    ↓
问题：无积累，每次重新检索

LLM Wiki：文档 → AI编译成Wiki → 持久化知识 → 复利增长
    ↓
优势：知识编译一次，持续维护
```

#### 三个关键特性

1. **编译而非检索**
   - 知识被编译成结构化Wiki页面
   - 交叉引用、摘要、实体页自动维护

2. **持续积累**
   - 每个新文档让Wiki更丰富
   - 每次查询的答案可归档回Wiki

3. **零维护负担**
   - AI自动更新、链接、检查矛盾
   - 人类专注于策展和思考

---

## 2. 核心理念与架构

### 2.1 三层架构

```
┌────────────────────────────────────────────┐
│           第一层：Raw Sources              │
│      (PDF、文章、笔记、转录文本)            │
│           不可变的真相之源                  │
└────────────────┬───────────────────────────┘
                 ↓
┌────────────────────────────────────────────┐
│            第二层：The Wiki                │
│   (AI生成的Markdown页面、摘要、实体页)      │
│           由LLM完全控制                     │
└────────────────┬───────────────────────────┘
                 ↓
┌────────────────────────────────────────────┐
│           第三层：The Tools                │
│     (搜索、读取、写入工具)                  │
│      Claude通过MCP连接并编排                │
└────────────────────────────────────────────┘
```

---

### 2.2 系统架构图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Next.js   │────▶│   FastAPI   │────▶│  Supabase   │
│   Frontend  │     │   Backend   │     │  (Postgres) │
└─────────────┘     └──────┬──────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    │  MCP Server │◀──── Claude.ai
                    └─────────────┘
                           ↓
                    ┌─────────────┐
                    │ S3 Storage  │
                    └─────────────┘
```

---

### 2.3 核心操作

#### 操作1：Ingest（导入）

**流程**：

```
上传文档 → Claude读取 → 生成摘要 → 更新实体页 → 标记矛盾
```

**特点**：
- 单个文档可能影响10-15个Wiki页面
- 自动识别矛盾信息
- 建立交叉引用

---

#### 操作2：Query（查询）

**流程**：

```
提问 → 读取已编译的Wiki → 综合答案 → (可选)归档为新页面
```

**特点**：
- 知识已预先综合，无需每次重新推导
- 好答案归档回Wiki，形成复利
- 支持引用和溯源

---

#### 操作3：Lint（健康检查）

**流程**：

```
扫描Wiki → 发现不一致 → 标记过时信息 → 找孤立页面 → 建议新问题
```

**特点**：
- 自动维护数据完整性
- 主动发现知识空白
- 建议后续研究方向

---

## 3. 技术栈详解

### 3.1 前端技术栈

#### 核心框架

| 技术 | 版本 | 用途 |
|------|------|------|
| **Next.js** | 16 | React全栈框架 |
| **React** | 19 | UI库 |
| **TypeScript** | Latest | 类型安全 |
| **Tailwind CSS** | Latest | 样式框架 |
| **Radix UI** | Latest | 无障碍组件库 |

#### 主要功能模块

- 📊 **Dashboard** - 知识库概览
- 📄 **PDF Viewer** - 内置PDF查看器
- 🌐 **HTML Viewer** - HTML文档渲染
- 📝 **Wiki Renderer** - Markdown Wiki展示
- 🚀 **Onboarding** - 用户引导

---

### 3.2 后端技术栈

#### 核心框架

| 技术 | 用途 | 特性 |
|------|------|------|
| **FastAPI** | Web框架 | 异步、高性能、自动文档 |
| **asyncpg** | PostgreSQL驱动 | 异步数据库操作 |
| **aioboto3** | AWS SDK | 异步S3操作 |
| **Mistral API** | OCR服务 | PDF文字提取 |

#### 主要功能

- 🔐 **认证授权** - Supabase OAuth
- 📤 **文件上传** - TUS协议（可恢复上传）
- 📄 **文档处理** - PDF提取、HTML处理
- 🔍 **OCR** - Mistral API图像识别
- 🔄 **文档转换** - Office → PDF

---

### 3.3 文档转换服务

#### Converter服务

| 特性 | 值 |
|------|-----|
| **框架** | FastAPI |
| **引擎** | LibreOffice |
| **隔离** | 独立容器，非root运行 |
| **安全** | 零AWS凭证 |

**支持格式**：
- Microsoft Office (docx, xlsx, pptx)
- OpenOffice (odt, ods, odp)
- 其他办公文档

---

### 3.4 MCP服务器

#### 技术组成

| 组件 | 技术 |
|------|------|
| **SDK** | MCP SDK |
| **认证** | Supabase OAuth |
| **协议** | MCP (Model Context Protocol) |

#### 提供的工具

| 工具 | 功能 |
|------|------|
| `guide` | 解释Wiki工作原理，列出可用知识库 |
| `search` | 浏览文件或关键词搜索（PGroonga排序） |
| `read` | 读取文档（PDF带页码范围、内联图片、批量读取） |
| `write` | 创建Wiki页面、字符串替换编辑、追加内容 |
| `delete` | 按路径或通配符归档文档 |

---

### 3.5 数据库

#### Supabase特性

```
PostgreSQL 核心
    +
Row Level Security (RLS)
    +
PGroonga 全文搜索
    +
实时订阅
```

**数据模型**：
- `documents` - 文档元数据
- `chunks` - 文档片段（用于搜索）
- `knowledge_bases` - 知识库
- `users` - 用户信息

---

### 3.6 存储

#### S3兼容存储

**存储内容**：
- 原始上传文件
- 标记的HTML
- 提取的图片
- 生成的Wiki页面

**兼容服务**：
- Amazon S3
- MinIO
- DigitalOcean Spaces
- Cloudflare R2
- 阿里云OSS（S3模式）

---

## 4. 云端服务快速开始

### 4.1 注册账号

#### 步骤1：访问官网

```
https://llmwiki.app/
```

#### 步骤2：创建账号

1. 点击「Sign Up」
2. 使用Email注册或OAuth登录
3. 验证邮箱

---

### 4.2 创建知识库

#### 步骤1：新建Knowledge Base

```
Dashboard → Create Knowledge Base
```

**配置项**：
- **名称**：例如「AI Research」
- **描述**：知识库用途说明
- **可见性**：私有/公开

#### 步骤2：上传文档

**支持的格式**：

| 类型 | 格式 |
|------|------|
| **PDF** | .pdf |
| **Office** | .docx, .xlsx, .pptx |
| **开放文档** | .odt, .ods, .odp |
| **网页** | .html, .htm |
| **文本** | .txt, .md |

**上传方式**：

**方法1：拖放上传**
```
Files 页面 → 拖拽文件到上传区
```

**方法2：选择文件**
```
Files 页面 → Click to upload → 选择文件
```

**方法3：批量上传**
```
支持多文件同时上传（TUS协议，可断点续传）
```

---

### 4.3 连接Claude

#### 步骤1：复制MCP配置

在LLM Wiki中：

```
Settings → MCP Configuration → Copy Config
```

**配置内容示例**：

```json
{
  "name": "LLM Wiki",
  "url": "https://llmwiki.app/mcp",
  "auth": {
    "type": "oauth",
    "provider": "supabase"
  }
}
```

---

#### 步骤2：添加到Claude

1. 打开 **Claude.ai**
2. 进入 **Settings** → **Connectors**
3. 点击 **Add Custom Connector**
4. 粘贴配置
5. 点击 **Connect**

---

#### 步骤3：授权

1. 浏览器弹出授权页面
2. 使用LLM Wiki账号登录
3. 授权Claude访问你的知识库
4. 完成连接

---

### 4.4 开始使用

#### 对Claude说：

```
请使用LLM Wiki的guide工具，展示我的知识库
```

**Claude回复**：

```
✓ 已连接到LLM Wiki
✓ 找到1个知识库：「AI Research」
✓ 包含5个文档

可用操作：
- 搜索文档
- 读取内容
- 生成Wiki页面
```

---

#### 导入第一个文档

```
👤 用户：
请读取「Karpathy LLM Wiki」文档，然后生成摘要

🤖 Claude：
正在读取...
✓ 文档已读取（约5000字）

开始生成Wiki页面...
✓ 创建摘要：wiki/sources/karpathy-llm-wiki.md
✓ 创建实体页：
  - entities/people/andrej-karpathy.md
  - entities/concepts/llm-wiki.md
  - entities/concepts/memex.md
✓ 生成对比页：comparisons/rag-vs-wiki.md

完成！已影响4个Wiki页面。
```

---

## 5. 自托管完整指南

### 5.1 前置条件

#### 系统要求

| 组件 | 最低要求 | 推荐配置 |
|------|---------|---------|
| **OS** | Linux/macOS | Ubuntu 22.04+ |
| **CPU** | 2核 | 4核+ |
| **内存** | 4GB | 8GB+ |
| **存储** | 20GB | 50GB+ SSD |
| **网络** | 公网IP（可选） | 固定IP + 域名 |

---

#### 软件依赖

| 软件 | 版本要求 | 验证命令 |
|------|---------|---------|
| **Python** | 3.11+ | `python --version` |
| **Node.js** | 20+ | `node --version` |
| **npm** | 9+ | `npm --version` |
| **Docker** | 20+ (可选) | `docker --version` |
| **Git** | 2+ | `git --version` |

**安装命令**（Ubuntu示例）：

```bash
# Python 3.11
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.11 python3.11-venv

# Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Docker (可选)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

---

#### 云服务准备

##### 必需服务

**1. Supabase项目**

选项A：Supabase云服务
```
访问：https://supabase.com/
创建免费项目（每月50,000次认证）
```

选项B：本地Docker运行
```bash
# 使用项目自带的docker-compose
docker compose up -d
```

**2. S3兼容存储**

推荐选项：

| 服务 | 免费额度 | 特点 |
|------|---------|------|
| **Cloudflare R2** | 10GB免费 | 零出站费用 |
| **MinIO** | 自托管 | 完全控制 |
| **DigitalOcean Spaces** | $5/月 | 250GB存储 |
| **阿里云OSS** | 按量付费 | 国内访问快 |

**MinIO本地部署**（推荐自托管）：

```bash
# Docker方式
docker run -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=admin \
  -e MINIO_ROOT_PASSWORD=your-password \
  minio/minio server /data --console-address ":9001"

# 访问控制台：http://localhost:9001
```

---

#### 可选服务

**Mistral API**（用于OCR）

```
访问：https://mistral.ai/
注册并获取API Key
免费层：有限额度
```

**如果不使用OCR**：
- PDF需要包含可选择的文本
- 扫描版PDF无法提取内容

---

### 5.2 数据库设置

#### 选项A：使用Supabase云服务

**步骤1：创建项目**

1. 访问 https://supabase.com/
2. 创建新项目
3. 记录以下信息：
   - Project URL: `https://xxx.supabase.co`
   - Anon Key: `eyJhbG...`
   - Service Role Key: `eyJhbG...`
   - Database Password: `your-db-password`

**步骤2：运行迁移**

```bash
# 克隆项目
git clone https://github.com/lucasastorian/llmwiki.git
cd llmwiki

# 获取数据库URL
# 在Supabase Dashboard → Settings → Database
# 复制 Connection string (URI)

# 格式：postgresql://postgres:[password]@xxx.supabase.co:5432/postgres

# 运行迁移
psql "postgresql://postgres:[password]@xxx.supabase.co:5432/postgres" \
  -f supabase/migrations/001_initial.sql
```

---

#### 选项B：本地Docker Supabase

**步骤1：启动服务**

```bash
cd llmwiki
docker compose up -d
```

**包含的服务**：
- PostgreSQL (端口 5432)
- PostgREST API (端口 3000)
- Supabase Studio (端口 3001)
- Auth Server
- Storage Server

**步骤2：访问控制台**

```
http://localhost:3001
```

**步骤3：数据库连接**

```
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/postgres
```

---

### 5.3 后端API部署

#### 步骤1：环境准备

```bash
cd api
python3.11 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
```

#### 步骤2：安装依赖

```bash
pip install -r requirements.txt
```

**依赖列表**（主要）：
- fastapi
- uvicorn[standard]
- asyncpg
- aioboto3
- python-multipart
- pydantic-settings
- httpx

---

#### 步骤3：配置环境变量

```bash
cp ../.env.example .env
```

**编辑 `.env`**：

```bash
# 数据库
DATABASE_URL=postgresql://postgres:password@xxx.supabase.co:5432/postgres

# Supabase
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=eyJhbG...
# 可选：仅旧版HS256项目需要
SUPABASE_JWT_SECRET=your-jwt-secret

# Mistral API (OCR)
MISTRAL_API_KEY=your-mistral-key

# S3存储
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1
S3_BUCKET=your-bucket-name
# 如果使用MinIO或其他S3兼容服务
AWS_ENDPOINT_URL=http://localhost:9000

# 应用设置
APP_URL=http://localhost:3000
CONVERTER_URL=http://localhost:8081  # 可选
```

---

#### 步骤4：启动API服务

**开发模式**：

```bash
uvicorn main:app --reload --port 8000 --host 0.0.0.0
```

**生产模式**：

```bash
uvicorn main:app --workers 4 --port 8000 --host 0.0.0.0
```

**验证**：

```bash
curl http://localhost:8000/health
# 应返回：{"status": "ok"}
```

**访问API文档**：

```
http://localhost:8000/docs
```

---

### 5.4 MCP服务器部署

#### 步骤1：环境准备

```bash
cd ../mcp
python3.11 -m venv .venv
source .venv/bin/activate
```

#### 步骤2：安装依赖

```bash
pip install -r requirements.txt
```

**主要依赖**：
- mcp (Model Context Protocol SDK)
- fastapi
- supabase-py
- asyncpg

---

#### 步骤3：配置

**创建 `.env`**：

```bash
DATABASE_URL=postgresql://postgres:password@xxx.supabase.co:5432/postgres
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=eyJhbG...
```

---

#### 步骤4：启动MCP服务

```bash
uvicorn server:app --reload --port 8080 --host 0.0.0.0
```

**验证**：

```bash
curl http://localhost:8080/mcp
# 应返回MCP协议信息
```

---

### 5.5 前端Web部署

#### 步骤1：环境准备

```bash
cd ../web
npm install
```

#### 步骤2：配置环境变量

```bash
cp .env.example .env.local
```

**编辑 `.env.local`**：

```bash
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_MCP_URL=http://localhost:8080/mcp
```

---

#### 步骤3：启动开发服务器

```bash
npm run dev
```

**访问**：

```
http://localhost:3000
```

---

#### 步骤4：生产构建

```bash
# 构建
npm run build

# 启动生产服务器
npm start
```

**或使用PM2**：

```bash
npm install -g pm2
pm2 start npm --name "llmwiki-web" -- start
```

---

### 5.6 文档转换服务（可选）

#### 作用

将Office文档转换为PDF，便于统一处理。

#### 部署方式

**Docker方式**（推荐）：

```bash
cd ../converter
docker build -t llmwiki-converter .
docker run -d -p 8081:8081 --name converter llmwiki-converter
```

**手动方式**：

```bash
# 安装LibreOffice
sudo apt-get install libreoffice

# 启动服务
cd converter
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --port 8081
```

---

### 5.7 Nginx反向代理配置

#### 安装Nginx

```bash
sudo apt update
sudo apt install nginx
```

#### 配置文件

**创建 `/etc/nginx/sites-available/llmwiki`**：

```nginx
server {
    listen 80;
    server_name your-domain.com;

    # 前端
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # API
    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # MCP
    location /mcp {
        proxy_pass http://localhost:8080/mcp;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 文件上传（大文件支持）
    client_max_body_size 100M;
}
```

**启用配置**：

```bash
sudo ln -s /etc/nginx/sites-available/llmwiki /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

#### SSL证书（Let's Encrypt）

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

---

### 5.8 系统服务配置（Systemd）

#### API服务

**创建 `/etc/systemd/system/llmwiki-api.service`**：

```ini
[Unit]
Description=LLM Wiki API
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/path/to/llmwiki/api
Environment="PATH=/path/to/llmwiki/api/.venv/bin"
ExecStart=/path/to/llmwiki/api/.venv/bin/uvicorn main:app --workers 4 --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

#### MCP服务

**创建 `/etc/systemd/system/llmwiki-mcp.service`**：

```ini
[Unit]
Description=LLM Wiki MCP Server
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/path/to/llmwiki/mcp
Environment="PATH=/path/to/llmwiki/mcp/.venv/bin"
ExecStart=/path/to/llmwiki/mcp/.venv/bin/uvicorn server:app --host 0.0.0.0 --port 8080
Restart=always

[Install]
WantedBy=multi-user.target
```

#### Web服务

**创建 `/etc/systemd/system/llmwiki-web.service`**：

```ini
[Unit]
Description=LLM Wiki Web
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/path/to/llmwiki/web
Environment="PATH=/path/to/llmwiki/web/node_modules/.bin:/usr/bin"
ExecStart=/usr/bin/npm start
Restart=always

[Install]
WantedBy=multi-user.target
```

---

#### 启动服务

```bash
sudo systemctl daemon-reload
sudo systemctl enable llmwiki-api llmwiki-mcp llmwiki-web
sudo systemctl start llmwiki-api llmwiki-mcp llmwiki-web

# 检查状态
sudo systemctl status llmwiki-api
sudo systemctl status llmwiki-mcp
sudo systemctl status llmwiki-web
```

---

## 6. MCP连接配置

### 6.1 MCP协议简介

**MCP (Model Context Protocol)** 是Anthropic开发的协议，让AI模型能够安全地访问外部工具和数据。

#### 工作原理

```
Claude.ai ←→ MCP Server ←→ LLM Wiki
```

#### 优势

- ✅ **标准化**：统一的工具接口
- ✅ **安全**：OAuth认证 + RLS权限
- ✅ **实时**：直接访问最新数据
- ✅ **上下文**：保持会话状态

---

### 6.2 云端服务MCP配置

#### 步骤1：获取配置

登录 https://llmwiki.app/

```
Settings → Connectors → LLM Wiki MCP
```

**复制配置**：

```json
{
  "name": "LLM Wiki",
  "url": "https://llmwiki.app/mcp",
  "auth": {
    "type": "oauth",
    "provider": "supabase",
    "client_id": "auto-detected"
  }
}
```

---

#### 步骤2：添加到Claude

1. 打开 https://claude.ai/
2. **Settings** → **Connectors**
3. **Add Custom Connector**
4. 粘贴配置
5. **Save**

---

#### 步骤3：授权连接

1. 点击 **Connect**
2. 弹出授权窗口
3. 使用LLM Wiki账号登录
4. 授权Claude访问
5. 完成连接

---

### 6.3 自托管MCP配置

#### 配置格式

```json
{
  "name": "My LLM Wiki",
  "url": "https://your-domain.com/mcp",
  "auth": {
    "type": "oauth",
    "provider": "supabase",
    "client_id": "your-supabase-project-ref"
  }
}
```

#### 关键参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `name` | 显示名称 | "My LLM Wiki" |
| `url` | MCP端点 | "https://wiki.example.com/mcp" |
| `client_id` | Supabase项目ID | "abcdefghijk" |

---

### 6.4 验证连接

#### 对Claude说：

```
使用LLM Wiki的guide工具
```

**成功响应**：

```
✓ 已连接到LLM Wiki
✓ 可用知识库：
  1. AI Research (5 documents)
  2. Reading Notes (12 documents)

可用工具：
- guide: 显示帮助和知识库列表
- search: 搜索文档
- read: 读取文档内容
- write: 创建/编辑Wiki页面
- delete: 归档文档
```

---

## 7. 功能使用指南

### 7.1 文档上传

#### 支持的文件类型

| 类型 | 格式 | 处理方式 |
|------|------|---------|
| **PDF** | .pdf | 直接提取文本或OCR |
| **Office** | .docx, .xlsx, .pptx | 转PDF后提取 |
| **网页** | .html, .htm | HTML解析 |
| **文本** | .txt, .md | 直接读取 |
| **开放文档** | .odt, .ods, .odp | 转PDF后提取 |

---

#### 上传方式

**方式1：Web界面拖放**

```
Files页面 → 拖拽文件到上传区 → 自动上传
```

**方式2：通过Claude**

```
👤：我有一个PDF，怎么上传？

🤖：请在Web界面上传，我会自动检测到新文档。
或者，你可以直接把内容粘贴给我，我帮你创建文档。
```

---

#### 上传进度

- ✅ **断点续传**：使用TUS协议
- ✅ **后台处理**：上传后自动提取文本
- ✅ **进度提示**：实时显示上传进度

---

### 7.2 文档查看

#### PDF查看器

**功能**：

- 📄 页面导航
- 🔍 文本搜索
- 📐 缩放控制
- 📑 书签跳转
- 💾 下载原文件

**使用**：

```
Files → 点击PDF文件 → 打开内置查看器
```

---

#### HTML查看器

**功能**：

- 🌐 完整渲染
- 🖼️ 图片加载
- 🔗 链接导航
- 📱 响应式布局

---

### 7.3 Wiki页面浏览

#### Wiki结构

```
wiki/
├── sources/           # 文档摘要
│   └── source-001.md
├── entities/          # 实体页面
│   ├── people/
│   ├── concepts/
│   └── tools/
├── topics/            # 主题综述
├── comparisons/       # 对比分析
└── synthesis/         # 综合分析
```

---

#### 浏览方式

**方式1：目录树**

```
Wiki → 左侧目录 → 点击页面
```

**方式2：搜索**

```
Wiki → 搜索框 → 输入关键词
```

**方式3：链接跳转**

```
Wiki页面中的 [[链接]] → 点击跳转
```

---

### 7.4 使用Claude编译Wiki

#### 场景1：生成文档摘要

**对话**：

```
👤：请读取「Attention Is All You Need」论文，生成摘要

🤖：正在读取PDF...
✓ 读取完成（8页）

开始生成摘要...

# Attention Is All You Need 摘要

## 核心贡献
提出了Transformer架构，完全基于注意力机制...

## 关键创新
1. 自注意力机制
2. 多头注意力
3. 位置编码

## 实验结果
在WMT翻译任务上达到SOTA...

✓ 已保存到：wiki/sources/attention-paper.md
✓ 创建实体页：[[Transformer]]、[[Attention Mechanism]]
```

---

#### 场景2：创建实体页

**对话**：

```
👤：请为「Transformer」概念创建详细的实体页

🤖：正在综合所有相关文档...
✓ 找到3篇相关论文

# Transformer

## 定义
一种基于自注意力机制的深度学习架构...

## 历史
- 2017年：Google提出
- 2018年：BERT应用

## 应用
- NLP（BERT、GPT）
- 计算机视觉（ViT）
- 多模态（CLIP）

## 相关文档
- [[Attention Is All You Need]]
- [[BERT论文]]

✓ 已保存到：wiki/entities/concepts/transformer.md
```

---

#### 场景3：对比分析

**对话**：

```
👤：比较Transformer和RNN的优缺点

🤖：正在综合相关知识...

# Transformer vs RNN 对比

| 维度 | Transformer | RNN |
|------|------------|-----|
| 并行化 | ✅ 完全并行 | ❌ 顺序处理 |
| 长距离依赖 | ✅ 通过注意力 | ⚠️ 梯度消失 |
| 训练速度 | 快 | 慢 | 
| 内存消耗 | 高 | 低 |

## 详细分析
[...]

✓ 已保存到：wiki/comparisons/transformer-vs-rnn.md
```

---

#### 场景4：主题综述

**对话**：

```
👤：总结所有关于「注意力机制」的内容

🤖：正在扫描知识库...
✓ 找到8篇相关文档

# 注意力机制综述

## 发展历程
1. 早期注意力（2014）
2. Transformer（2017）
3. 现代变体

## 核心类型
- 自注意力
- 交叉注意力
- 多头注意力

## 应用领域
[详细内容...]

✓ 已保存到：wiki/topics/attention-mechanisms.md
✓ 更新了5个相关实体页的链接
```

---

### 7.5 搜索功能

#### PGroonga全文搜索

**特性**：

- 🔤 **中英文支持**
- 🎯 **准确性排序**
- ⚡ **高性能**（索引加速）
- 🔍 **模糊匹配**

**使用方式**：

**方式1：Web界面**

```
搜索框 → 输入关键词 → 即时搜索
```

**方式2：通过Claude**

```
👤：搜索包含「transformer」的文档

🤖：搜索中...
✓ 找到5个结果：

1. Attention Is All You Need (PDF, 8页)
   匹配度：95%
   片段："...introduce the Transformer..."

2. BERT论文 (PDF, 16页)
   匹配度：87%
   片段："...based on Transformer architecture..."

[...]
```

---

## 8. 高级配置

### 8.1 OCR配置

#### Mistral OCR

**用途**：从扫描PDF中提取文字

**配置**：

```bash
# api/.env
MISTRAL_API_KEY=your-mistral-api-key
```

**工作流程**：

```
上传扫描PDF → API检测无文本 → 调用Mistral OCR → 提取文字 → 保存
```

**成本**：

- 免费层：有限次数
- 付费：按页计费

---

#### 禁用OCR

如果不需要OCR，留空API Key：

```bash
MISTRAL_API_KEY=
```

**影响**：
- ✅ 可选文字的PDF正常工作
- ❌ 扫描版PDF无法提取

---

### 8.2 S3存储配置

#### MinIO配置示例

```bash
# api/.env
AWS_ENDPOINT_URL=http://localhost:9000
AWS_ACCESS_KEY_ID=minioadmin
AWS_SECRET_ACCESS_KEY=minioadmin
AWS_REGION=us-east-1
S3_BUCKET=llmwiki
```

**创建Bucket**：

```bash
# 安装mc (MinIO Client)
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc

# 配置
./mc alias set local http://localhost:9000 minioadmin minioadmin

# 创建bucket
./mc mb local/llmwiki

# 设置公开访问（可选）
./mc anonymous set download local/llmwiki
```

---

#### Cloudflare R2配置

```bash
# api/.env
AWS_ENDPOINT_URL=https://xxx.r2.cloudflarestorage.com
AWS_ACCESS_KEY_ID=your-r2-access-key
AWS_SECRET_ACCESS_KEY=your-r2-secret-key
AWS_REGION=auto
S3_BUCKET=llmwiki
```

---

### 8.3 数据库优化

#### 连接池配置

**asyncpg配置**（在`api/database.py`中）：

```python
pool = await asyncpg.create_pool(
    database_url,
    min_size=5,      # 最小连接数
    max_size=20,     # 最大连接数
    command_timeout=60,  # 命令超时
)
```

---

#### PGroonga索引优化

**重建索引**：

```sql
-- 如果搜索性能下降
REINDEX INDEX documents_content_idx;
```

**查看索引大小**：

```sql
SELECT pg_size_pretty(pg_total_relation_size('documents_content_idx'));
```

---

### 8.4 文件上传优化

#### TUS协议配置

**最大文件大小**（在`api/config.py`中）：

```python
MAX_UPLOAD_SIZE = 100 * 1024 * 1024  # 100MB
```

**分片大小**：

```python
CHUNK_SIZE = 1024 * 1024  # 1MB
```

---

#### Nginx上传配置

```nginx
location /api/upload {
    proxy_pass http://localhost:8000/upload;
    
    # 大文件支持
    client_max_body_size 500M;
    
    # 超时设置
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    
    # 缓冲设置
    proxy_request_buffering off;
}
```

---

### 8.5 性能监控

#### 日志配置

**启用详细日志**：

```bash
# api/.env
LOG_LEVEL=DEBUG  # DEBUG, INFO, WARNING, ERROR
```

**查看日志**：

```bash
# 实时查看API日志
tail -f api/logs/app.log

# MCP日志
tail -f mcp/logs/mcp.log
```

---

#### 性能指标

**PostgreSQL慢查询**：

```sql
-- 启用慢查询日志
ALTER DATABASE postgres SET log_min_duration_statement = 1000; -- 1秒

-- 查看慢查询
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

---

## 9. 故障排查

### 9.1 数据库连接问题

#### 问题1：无法连接数据库

**症状**：

```
asyncpg.exceptions.InvalidPasswordError
```

**检查**：

```bash
# 测试连接
psql "postgresql://user:pass@host:5432/db"
```

**解决方案**：

1. 检查密码是否正确
2. 检查防火墙规则
3. 如果是Supabase，检查IP白名单

---

#### 问题2：迁移失败

**症状**：

```
ERROR: relation "documents" already exists
```

**解决方案**：

```sql
-- 检查已有表
\dt

-- 如果需要重建
DROP TABLE IF EXISTS documents CASCADE;

-- 重新运行迁移
\i supabase/migrations/001_initial.sql
```

---

### 9.2 上传问题

#### 问题3：文件上传失败

**症状**：

```
Upload failed: S3 connection error
```

**检查清单**：

1. **S3凭证**：
```bash
# 测试S3连接（使用aws cli）
aws s3 ls --endpoint-url=http://localhost:9000

# 或使用curl
curl http://localhost:9000
```

2. **Bucket权限**：
```bash
# MinIO
./mc ls local/llmwiki
```

3. **防火墙**：
确保端口9000可访问

---

#### 问题4：大文件上传超时

**症状**：

```
RequestTimeout: Your socket connection to the server was not read
```

**解决方案**：

**增加Nginx超时**：

```nginx
proxy_read_timeout 600s;
proxy_send_timeout 600s;
client_body_timeout 600s;
```

**增加API超时**：

```python
# api/config.py
UPLOAD_TIMEOUT = 600  # 10分钟
```

---

### 9.3 MCP连接问题

#### 问题5：Claude无法连接MCP

**症状**：

```
Connection refused
```

**检查**：

1. **MCP服务状态**：
```bash
curl http://localhost:8080/mcp
```

2. **防火墙**：
```bash
sudo ufw allow 8080
```

3. **URL配置**：
确保Claude配置中的URL正确

---

#### 问题6：授权失败

**症状**：

```
OAuth error: Invalid client
```

**解决方案**：

1. **检查Supabase URL**：
```bash
# 在.env中
SUPABASE_URL=https://xxx.supabase.co  # 注意https
```

2. **检查Anon Key**：
在Supabase Dashboard验证

3. **重新授权**：
在Claude中删除连接，重新添加

---

### 9.4 OCR问题

#### 问题7：OCR失败

**症状**：

```
Mistral API error: Unauthorized
```

**解决方案**：

1. **验证API Key**：
```bash
curl -H "Authorization: Bearer $MISTRAL_API_KEY" \
  https://api.mistral.ai/v1/models
```

2. **检查额度**：
登录Mistral控制台查看剩余额度

3. **降级处理**：
如果不需要OCR，删除`MISTRAL_API_KEY`

---

### 9.5 性能问题

#### 问题8：搜索缓慢

**症状**：

搜索超过3秒无结果

**诊断**：

```sql
-- 检查索引
SELECT * FROM pg_indexes WHERE tablename = 'documents';

-- 查看查询计划
EXPLAIN ANALYZE
SELECT * FROM documents
WHERE content @@ to_tsquery('transformer');
```

**解决方案**：

```sql
-- 重建PGroonga索引
REINDEX INDEX documents_content_idx;

-- 或更新统计信息
ANALYZE documents;
```

---

#### 问题9：内存占用高

**症状**：

API进程占用超过2GB内存

**检查**：

```bash
# 查看进程内存
ps aux | grep uvicorn

# 如果使用Docker
docker stats
```

**优化**：

```python
# 减少worker数量
uvicorn main:app --workers 2

# 或减少数据库连接池
max_size=10  # 在database.py中
```

---

## 10. 性能优化

### 10.1 数据库优化

#### 索引优化

```sql
-- 为常用查询字段添加索引
CREATE INDEX idx_documents_user_id ON documents(user_id);
CREATE INDEX idx_documents_kb_id ON documents(knowledge_base_id);
CREATE INDEX idx_documents_created_at ON documents(created_at DESC);

-- 复合索引
CREATE INDEX idx_docs_user_kb ON documents(user_id, knowledge_base_id);
```

---

#### 查询优化

**使用连接池**：

```python
# 避免每次创建连接
async with pool.acquire() as conn:
    result = await conn.fetch("SELECT ...")
```

**批量操作**：

```python
# 批量插入
await conn.executemany(
    "INSERT INTO chunks (doc_id, content) VALUES ($1, $2)",
    [(doc_id, chunk) for chunk in chunks]
)
```

---

### 10.2 存储优化

#### 压缩上传

**启用Gzip**：

```python
# 在上传前压缩
import gzip
compressed = gzip.compress(content)
await s3.upload(compressed)
```

---

#### CDN加速

**使用CloudFront或Cloudflare**：

```
S3/R2 → CDN → 用户
```

**配置示例**（Cloudflare）：

```
1. 添加你的域名到Cloudflare
2. 创建CNAME记录指向R2 bucket
3. 启用缓存规则
```

---

### 10.3 前端优化

#### 代码分割

**Next.js自动优化**：

```typescript
// 动态导入大组件
const PDFViewer = dynamic(() => import('@/components/PDFViewer'), {
  loading: () => <Loading />,
  ssr: false
});
```

---

#### 图片优化

**使用Next.js Image组件**：

```tsx
import Image from 'next/image';

<Image
  src="/wiki-page.png"
  width={800}
  height={600}
  alt="Wiki Page"
  loading="lazy"
/>
```

---

### 10.4 缓存策略

#### Redis缓存（可选）

**安装**：

```bash
docker run -d -p 6379:6379 redis:alpine
```

**使用**：

```python
import redis
r = redis.Redis(host='localhost', port=6379)

# 缓存搜索结果
cache_key = f"search:{query}"
cached = r.get(cache_key)
if cached:
    return json.loads(cached)

# ... 执行搜索 ...
r.setex(cache_key, 300, json.dumps(results))  # 5分钟过期
```

---

## 11. 安全最佳实践

### 11.1 认证与授权

#### Row Level Security (RLS)

Supabase RLS自动生效：

```sql
-- 示例：用户只能看自己的文档
CREATE POLICY "Users can view own documents"
ON documents FOR SELECT
USING (auth.uid() = user_id);

-- 示例：用户只能修改自己的Wiki
CREATE POLICY "Users can edit own wikis"
ON wiki_pages FOR UPDATE
USING (auth.uid() = user_id);
```

---

#### API认证

**JWT验证**：

```python
from fastapi import Depends, HTTPException
from supabase import Client

async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        user = supabase.auth.get_user(token)
        return user
    except:
        raise HTTPException(status_code=401, detail="Invalid token")
```

---

### 11.2 数据保护

#### 加密存储

**S3加密**：

```python
# 服务器端加密
await s3_client.upload_file(
    file,
    bucket,
    key,
    ExtraArgs={'ServerSideEncryption': 'AES256'}
)
```

---

#### 敏感信息处理

**环境变量管理**：

```bash
# 使用.env文件，永远不要提交到Git
echo ".env" >> .gitignore

# 使用secrets管理器（生产环境）
# AWS Secrets Manager
# Azure Key Vault
# Google Secret Manager
```

---

### 11.3 网络安全

#### HTTPS强制

**Nginx配置**：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # 安全头
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
}
```

---

#### CORS配置

**FastAPI CORS**：

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://your-domain.com"],  # 不要用*
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)
```

---

### 11.4 输入验证

#### 文件上传验证

```python
ALLOWED_EXTENSIONS = {'.pdf', '.docx', '.txt', '.md', '.html'}
MAX_FILE_SIZE = 100 * 1024 * 1024  # 100MB

def validate_upload(file):
    # 检查扩展名
    ext = os.path.splitext(file.filename)[1].lower()
    if ext not in ALLOWED_EXTENSIONS:
        raise ValueError("File type not allowed")
    
    # 检查大小
    if file.size > MAX_FILE_SIZE:
        raise ValueError("File too large")
    
    # 检查MIME类型
    # ...
```

---

## 12. 开发者指南

### 12.1 本地开发环境

#### 快速启动

```bash
# 克隆项目
git clone https://github.com/lucasastorian/llmwiki.git
cd llmwiki

# 启动数据库（Docker）
docker compose up -d

# 后端
cd api
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000

# MCP（新终端）
cd ../mcp
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn server:app --reload --port 8080

# 前端（新终端）
cd ../web
npm install
npm run dev
```

**访问**：
- 前端：http://localhost:3000
- API文档：http://localhost:8000/docs
- MCP：http://localhost:8080/mcp

---

### 12.2 项目结构

```
llmwiki/
├── api/                    # FastAPI后端
│   ├── main.py            # 主应用
│   ├── routes/            # API路由
│   │   ├── auth.py
│   │   ├── documents.py
│   │   ├── upload.py
│   │   └── wiki.py
│   ├── models/            # 数据模型
│   ├── services/          # 业务逻辑
│   │   ├── ocr.py
│   │   ├── pdf.py
│   │   └── s3.py
│   ├── database.py        # 数据库连接
│   └── config.py          # 配置
│
├── mcp/                   # MCP服务器
│   ├── server.py          # MCP主服务
│   ├── tools/             # MCP工具
│   │   ├── guide.py
│   │   ├── search.py
│   │   ├── read.py
│   │   ├── write.py
│   │   └── delete.py
│   └── auth.py            # OAuth认证
│
├── web/                   # Next.js前端
│   ├── app/               # App Router
│   │   ├── page.tsx       # 首页
│   │   ├── files/         # 文件管理
│   │   ├── wiki/          # Wiki浏览
│   │   └── settings/      # 设置
│   ├── components/        # React组件
│   │   ├── PDFViewer.tsx
│   │   ├── WikiRenderer.tsx
│   │   └── FileUpload.tsx
│   └── lib/               # 工具函数
│
├── converter/             # 文档转换服务
│   └── main.py
│
├── supabase/
│   └── migrations/        # 数据库迁移
│       └── 001_initial.sql
│
├── tests/                 # 测试
│   ├── test_api.py
│   ├── test_mcp.py
│   └── test_converter.py
│
├── docker-compose.yml     # Docker配置
├── .env.example           # 环境变量模板
└── README.md
```

---

### 12.3 添加新的MCP工具

#### 示例：添加导出工具

**Step 1：创建工具文件**

`mcp/tools/export.py`：

```python
from mcp import Tool

class ExportTool(Tool):
    name = "export"
    description = "Export wiki pages to various formats"
    
    input_schema = {
        "type": "object",
        "properties": {
            "format": {
                "type": "string",
                "enum": ["pdf", "html", "markdown"]
            },
            "pages": {
                "type": "array",
                "items": {"type": "string"}
            }
        },
        "required": ["format", "pages"]
    }
    
    async def execute(self, format: str, pages: list):
        # 实现导出逻辑
        if format == "pdf":
            return await self._export_pdf(pages)
        # ...
```

---

**Step 2：注册工具**

`mcp/server.py`：

```python
from tools.export import ExportTool

# 注册工具
mcp_server.add_tool(ExportTool())
```

---

### 12.4 数据库迁移

#### 创建新迁移

```sql
-- supabase/migrations/002_add_tags.sql

-- 添加标签表
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    created_at TIMESTAMPTZ DEFAULT now()
);

-- 文档标签关联表
CREATE TABLE document_tags (
    document_id UUID REFERENCES documents(id),
    tag_id UUID REFERENCES tags(id),
    PRIMARY KEY (document_id, tag_id)
);

-- RLS策略
ALTER TABLE tags ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public tags" ON tags FOR SELECT USING (true);
```

---

#### 运行迁移

```bash
psql $DATABASE_URL -f supabase/migrations/002_add_tags.sql
```

---

### 12.5 测试

#### 运行测试

```bash
# 安装测试依赖
pip install pytest pytest-asyncio httpx

# 运行所有测试
pytest

# 运行特定测试
pytest tests/test_api.py::test_upload

# 带覆盖率
pytest --cov=api --cov-report=html
```

---

#### 编写测试

`tests/test_mcp.py`：

```python
import pytest
from httpx import AsyncClient
from mcp.server import app

@pytest.mark.asyncio
async def test_guide_tool():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/mcp/tools/guide")
        assert response.status_code == 200
        data = response.json()
        assert "knowledge_bases" in data
```

---

## 13. FAQ常见问题

### Q1：LLM Wiki和Obsidian有什么区别？

**A**：

| 维度 | LLM Wiki | Obsidian |
|------|---------|----------|
| **定位** | AI驱动的知识管理平台 | 个人笔记工具 |
| **维护** | AI自动维护 | 人工维护 |
| **协作** | 支持多用户 | 主要单用户 |
| **存储** | 云端+数据库 | 本地文件 |
| **编辑** | 主要通过Claude | 直接编辑 |

**可以结合使用**：用LLM Wiki管理文档和AI生成内容，用Obsidian做个人笔记。

---

### Q2：需要付费吗？

**A**：

| 部署方式 | 成本 |
|---------|------|
| **云端服务** | 免费层 + 付费套餐 |
| **自托管** | 基础设施成本（服务器、Supabase、S3） |
| **Claude使用** | Claude Pro订阅（$20/月） |

**自托管成本估算**：
- VPS：$5-20/月
- Supabase：免费层或$25/月
- S3（R2）：$5/月（50GB）
- **总计**：约$10-50/月

---

### Q3：支持离线使用吗？

**A**：

**云端服务**：❌ 需要网络连接

**自托管**：
- ⚠️ 需要连接到你的服务器
- ✅ 可以在内网部署
- ❌ MCP需要Claude.ai，必须联网

**完全离线**：不支持（因为依赖Claude）

---

### Q4：数据安全吗？

**A**：

**云端服务**：
- ✅ 数据加密传输（HTTPS）
- ✅ 数据库RLS保护
- ✅ 只有你能访问自己的知识库
- ⚠️ 数据存在第三方服务器

**自托管**：
- ✅ 完全控制数据
- ✅ 可以使用私有S3
- ✅ 数据库在你的控制下
- ⚠️ 需要自己负责安全配置

---

### Q5：可以导入现有的Obsidian笔记吗？

**A**：

**目前**：需要逐个上传Markdown文件

**未来计划**：批量导入Obsidian Vault

**临时方案**：

```bash
# 批量转PDF（通过脚本）
for file in vault/*.md; do
    pandoc "$file" -o "${file%.md}.pdf"
done

# 然后上传PDF
```

---

### Q6：支持中文吗？

**A**：

✅ **完全支持**：
- Web界面中文
- 中文文档识别
- 中文搜索（PGroonga）
- Claude中文对话

---

### Q7：如何备份数据？

**A**：

**云端服务**：

```
Settings → Export → Download All Data
```

**自托管**：

```bash
# 备份数据库
pg_dump $DATABASE_URL > backup.sql

# 备份S3（使用rclone）
rclone sync minio:llmwiki /backup/s3/
```

---

### Q8：可以和团队共享知识库吗？

**A**：

**目前**：每个用户独立的知识库

**未来计划**：
- 共享知识库
- 协作编辑
- 权限管理

**临时方案**：导出后分享文件

---

### Q9：如何删除知识库？

**A**：

**云端服务**：

```
Settings → Knowledge Bases → [选择知识库] → Delete
```

**自托管**：

```sql
-- 删除知识库及关联数据
DELETE FROM wiki_pages WHERE knowledge_base_id = 'xxx';
DELETE FROM documents WHERE knowledge_base_id = 'xxx';
DELETE FROM knowledge_bases WHERE id = 'xxx';
```

---

### Q10：性能如何？支持多大规模？

**A**：

**测试数据**：

| 文档数量 | 搜索速度 | Wiki生成速度 |
|---------|---------|-------------|
| <100 | <100ms | ~5s/页 |
| 100-500 | <300ms | ~5s/页 |
| 500-1000 | <500ms | ~5s/页 |
| >1000 | 需优化 | ~5s/页 |

**优化建议**：
- 1000+文档：添加Redis缓存
- 5000+文档：考虑分库或企业方案

---

## 14. 路线图与贡献

### 14.1 当前版本特性

**v1.0 特性**（当前）：

- ✅ PDF/Office文档上传
- ✅ MCP集成
- ✅ Claude自动编译Wiki
- ✅ 全文搜索（PGroonga）
- ✅ 内置PDF/HTML查看器
- ✅ 多用户支持
- ✅ S3存储
- ✅ OCR（Mistral）

---

### 14.2 计划功能

**v1.1** (2026 Q2)：

- 📅 批量导入Obsidian Vault
- 🔄 Wiki版本历史
- 📊 知识图谱可视化
- 🏷️ 标签系统

**v1.2** (2026 Q3)：

- 👥 共享知识库
- 💬 评论和注释
- 🔔 变更通知
- 📱 移动应用

**v2.0** (2026 Q4)：

- 🤖 多LLM支持（GPT-4、Gemini）
- 🌐 多语言Wiki自动翻译
- 📊 高级分析和洞察
- 🔌 插件系统

---

### 14.3 贡献指南

#### 报告问题

```
https://github.com/lucasastorian/llmwiki/issues
```

**问题模板**：
- 环境信息（OS、Python版本、Node版本）
- 重现步骤
- 预期行为 vs 实际行为
- 错误日志

---

#### 提交代码

**流程**：

```bash
# 1. Fork项目
# 2. 创建分支
git checkout -b feature/your-feature

# 3. 开发和测试
pytest

# 4. 提交
git commit -m "Add: your feature"
git push origin feature/your-feature

# 5. 创建Pull Request
```

**代码规范**：
- Python：Black格式化
- TypeScript：Prettier格式化
- 提交信息：遵循Conventional Commits

---

#### 文档贡献

欢迎改进：
- README翻译
- 使用教程
- 最佳实践
- 视频演示

---

## 附录

### A. 环境变量完整列表

#### API环境变量

```bash
# 数据库
DATABASE_URL=postgresql://user:pass@host:5432/db

# Supabase
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=eyJhbG...
SUPABASE_JWT_SECRET=your-secret  # 可选

# Mistral OCR
MISTRAL_API_KEY=your-key

# S3存储
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_REGION=us-east-1
S3_BUCKET=bucket-name
AWS_ENDPOINT_URL=http://localhost:9000  # 可选（MinIO等）

# 应用
APP_URL=http://localhost:3000
CONVERTER_URL=http://localhost:8081  # 可选

# 日志
LOG_LEVEL=INFO  # DEBUG, INFO, WARNING, ERROR
```

---

#### Web环境变量

```bash
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_MCP_URL=http://localhost:8080/mcp
```

---

### B. 常用命令参考

#### 开发命令

```bash
# API
cd api && uvicorn main:app --reload --port 8000

# MCP
cd mcp && uvicorn server:app --reload --port 8080

# Web
cd web && npm run dev

# 数据库迁移
psql $DATABASE_URL -f supabase/migrations/001_initial.sql

# 运行测试
pytest
```

---

#### 生产命令

```bash
# API（生产）
uvicorn main:app --workers 4 --host 0.0.0.0 --port 8000

# Web（生产）
cd web && npm run build && npm start

# 系统服务
sudo systemctl start llmwiki-api
sudo systemctl start llmwiki-mcp
sudo systemctl start llmwiki-web
```

---

### C. 资源链接

#### 官方资源

- **项目主页**：https://github.com/lucasastorian/llmwiki
- **在线演示**：https://llmwiki.app/
- **Karpathy原文**：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

#### 技术文档

- **FastAPI**：https://fastapi.tiangolo.com/
- **Next.js**：https://nextjs.org/docs
- **Supabase**：https://supabase.com/docs
- **MCP协议**：https://www.anthropic.com/mcp
- **PGroonga**：https://pgroonga.github.io/

#### 云服务

- **Supabase**：https://supabase.com/
- **Cloudflare R2**：https://www.cloudflare.com/products/r2/
- **MinIO**：https://min.io/
- **Mistral AI**：https://mistral.ai/

---

### D. 术语表

| 术语 | 说明 |
|------|------|
| **MCP** | Model Context Protocol - AI模型上下文协议 |
| **RLS** | Row Level Security - 行级安全 |
| **TUS** | Tus Upload Protocol - 可恢复上传协议 |
| **PGroonga** | PostgreSQL全文搜索扩展 |
| **OCR** | Optical Character Recognition - 光学字符识别 |
| **S3** | Simple Storage Service - 对象存储服务 |
| **Supabase** | 开源Firebase替代方案 |

---

## 文档信息

| 项目 | 内容 |
|------|------|
| **文档版本** | v1.0 |
| **编写日期** | 2026年4月8日 |
| **最后更新** | 2026年4月8日 |
| **字数统计** | 约30000字 |
| **适用项目版本** | LLM Wiki main分支 |
| **许可证** | Apache 2.0 |

---

**文档完整结束**

🎉 祝你使用LLM Wiki愉快！

有问题请提交Issue：https://github.com/lucasastorian/llmwiki/issues

---

## 快速开始总结

### 云端使用（最简单）

```
1. 访问 https://llmwiki.app/
2. 注册账号
3. 创建知识库
4. 上传文档
5. 连接Claude
6. 开始构建Wiki
```

### 自托管（完全控制）

```bash
# 1. 克隆项目
git clone https://github.com/lucasastorian/llmwiki.git
cd llmwiki

# 2. 配置环境
cp .env.example .env
# 编辑.env填入配置

# 3. 启动服务
docker compose up -d  # 数据库
cd api && uvicorn main:app --reload --port 8000  # API
cd mcp && uvicorn server:app --reload --port 8080  # MCP
cd web && npm run dev  # Web

# 4. 访问 http://localhost:3000
```

享受AI驱动的知识管理新体验！✨
