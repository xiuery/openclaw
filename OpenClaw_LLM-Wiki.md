# OpenClaw LLM-Wiki 指南

> **项目地址**：https://github.com/sdyckjq-lab/llm-wiki-skill
> 
> **基于**：[Karpathy的LLM-Wiki方法论](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
> 
> **适用平台**：macOS + OpenClaw
> 
> **编写日期**：2026年4月14日
> 
> **版本**：v1.0

---

## 📑 目录

- [1. OpenClaw与LLM-Wiki概述](#1-openclaw与llm-wiki概述)
- [2. 为什么选择LLM-Wiki](#2-为什么选择llm-wiki)
- [3. 系统要求](#3-系统要求)
- [4. 前置条件检查](#4-前置条件检查)
- [5. 安装部署](#5-安装部署)
- [6. OpenClaw配置](#6-openclaw配置)
- [7. 目录结构](#7-目录结构)
- [8. 功能特性](#8-功能特性)
- [9. 使用指南](#9-使用指南)
- [10. 素材来源支持](#10-素材来源支持)
- [11. 故障排查](#11-故障排查)
- [12. 最佳实践](#12-最佳实践)
- [13. 常见问题FAQ](#13-常见问题faq)
- [附录](#附录)

---

## 1. OpenClaw与LLM-Wiki概述

### 1.1 什么是OpenClaw？

**OpenClaw** 是一个开源的、灵活的AI Agent平台，具有以下特点：

- 🟩 **开源灵活**：完全开源，可自定义扩展
- 🔧 **多模型支持**：支持OpenAI、Ollama等多种模型
- 🚀 **社区活跃**：活跃的开源社区，持续更新
- 💪 **本地部署**：支持完全本地化运行
- 🛠️ **技能系统**：强大的技能扩展机制

**默认安装路径**：`~/.openclaw/skills/llm-wiki`

---

### 1.2 什么是LLM-Wiki？

**LLM-Wiki** 是基于Karpathy方法论的知识库构建系统，核心理念是：

> **把碎片化的信息变成持续积累、互相链接的知识库。**

**核心优势**：

```
传统RAG：查询 → 检索片段 → 拼凑答案（无积累）
    ↓
LLM-Wiki：素材 → 编译成Wiki → 持续维护（知识复利）
```

---

### 1.3 OpenClaw + LLM-Wiki的价值

| 优势 | 说明 |
|------|------|
| 🔓 **完全开源** | 代码透明，可自主控制 |
| 💰 **成本可控** | 支持本地模型（Ollama），无API费用 |
| 🔒 **隐私安全** | 数据完全本地化，不上传云端 |
| 🎯 **深度定制** | 可根据需求调整配置和工作流 |
| 📈 **持续积累** | 知识库随使用而自动增长 |

---

## 2. 为什么选择LLM-Wiki

### 2.1 RAG vs LLM-Wiki

| 对比维度 | RAG模式 | LLM-Wiki模式 |
|---------|--------|-------------|
| **知识表现** | 向量片段 | 结构化页面 |
| **查询方式** | 每次检索 | 导航阅读 |
| **积累性** | ❌ 无 | ✅ 有 |
| **维护** | 自动索引 | AI主动维护 |
| **成本** | 查询时高 | 查询时低 |
| **上下文完整性** | ⚠️ 可能碎片化 | ✅ 完整保留 |

---

### 2.2 三个关键原则

#### 原则1：编译而非检索

```
RAG：每次查询重新检索
Wiki：知识编译一次，持续维护
```

#### 原则2：持久化积累

```
每次添加素材 → Wiki变得更丰富
每次提问 → 答案归档回Wiki
```

#### 原则3：自动化维护

```
人类：策展、思考、提问
Agent：摘要、链接、一致性维护
```

---

## 3. 系统要求

### 3.1 操作系统

| 系统 | 支持状态 | 说明 |
|------|---------|------|
| ✅ **macOS** | 完全支持 | 推荐使用 |
| ✅ **Linux** | 完全支持 | 包括Ubuntu、Debian等 |
| ⚠️ **Windows** | 部分支持 | 建议使用WSL2或Git Bash |

---

### 3.2 必需组件

| 组件 | 作用 | 检查命令 | 安装方法 |
|------|------|---------|---------|
| **Bash** | 运行安装脚本 | `bash --version` | 系统自带 |
| **Git** | 克隆仓库 | `git --version` | `apt install git` / `brew install git` |
| **OpenClaw** | AI Agent平台 | `openclaw --version` | 见OpenClaw官方文档 |

---

### 3.3 可选组件

| 组件 | 作用 | 必需场景 | 安装方法 |
|------|------|---------|---------|
| **Chrome** | 网页提取 | 自动提取网页、X/Twitter | 官网下载 |
| **uv** | Python环境 | 微信公众号提取 | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **bun/npm** | JS依赖 | 网页内容提取 | `curl -fsSL https://bun.sh/install \| bash` |
| **Obsidian** | 可视化界面 | 图形化浏览Wiki | https://obsidian.md/ |

---

## 4. 前置条件检查

### 4.1 检查OpenClaw安装

```bash
# 检查OpenClaw版本
openclaw --version

# 检查OpenClaw配置目录
ls -la ~/.openclaw/

# 检查Skills目录
ls -la ~/.openclaw/skills/
```

**预期结果**：
- OpenClaw已安装并可正常运行
- `~/.openclaw/` 目录存在
- `~/.openclaw/skills/` 目录存在（如果没有会自动创建）

---

### 4.2 检查Agent能力

#### 必须具备的能力

| 能力 | 测试方法 | 说明 |
|------|---------|------|
| **Shell命令执行** | 让Agent执行`echo "test"` | 能运行bash脚本 |
| **文件读写** | 让Agent创建测试文件 | 能创建和修改文件 |
| **工作目录理解** | 让Agent列出当前目录 | 能操作相对路径 |

**测试对话示例**：

```
👤 你好，请执行命令：echo "test"

🤖 [执行成功并显示结果]
```

如果以上测试都通过，说明OpenClaw具备运行LLM-Wiki的能力。

---

### 4.3 检查可选工具

#### Chrome调试模式（用于网页提取）

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 &

# Linux
google-chrome --remote-debugging-port=9222 &

# 验证连接
curl http://localhost:9222/json
```

#### uv安装（用于微信公众号）

```bash
# 安装uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 验证安装
uv --version
```

#### bun或npm（用于JS依赖）

```bash
# 检查npm（Node.js自带）
npm --version

# 或安装bun（更快）
curl -fsSL https://bun.sh/install | bash

# 验证
bun --version
```

---

## 5. 安装部署

### 5.1 快速安装（推荐）

#### 方法1：让OpenClaw Agent自动安装

这是**最简单**的方法，直接与你的OpenClaw Agent对话：

**步骤1：发送安装指令**

```
请帮我安装 llm-wiki-skill：
https://github.com/sdyckjq-lab/llm-wiki-skill

使用OpenClaw平台，运行：
bash install.sh --platform openclaw
```

**步骤2：Agent会自动执行**

```bash
# Agent会自动执行以下操作：
git clone https://github.com/sdyckjq-lab/llm-wiki-skill.git
cd llm-wiki-skill
bash install.sh --platform openclaw
```

**步骤3：验证安装**

```
请检查 llm-wiki 技能是否已安装
```

**预期结果**：
```
✓ 已安装到：~/.openclaw/skills/llm-wiki/
✓ 技能已激活
```

---

#### 方法2：手动安装（完全控制）

如果你想完全掌控安装过程，可以手动操作：

**步骤1：克隆仓库**

```bash
# 克隆到任意目录
cd ~/Downloads  # 或其他临时目录
git clone https://github.com/sdyckjq-lab/llm-wiki-skill.git
cd llm-wiki-skill
```

**步骤2：检查目录结构**

```bash
# 验证关键文件存在
ls -la install.sh
ls -la scripts/
ls -la platforms/openclaw/
```

**步骤3：运行安装脚本**

```bash
# 安装到OpenClaw
bash install.sh --platform openclaw
```

**安装脚本会自动**：
1. 检测OpenClaw配置目录（`~/.openclaw/`）
2. 创建Skills目录（如果不存在）
3. 复制必要文件到 `~/.openclaw/skills/llm-wiki/`
4. 设置正确的权限
5. 配置OpenClaw以识别新技能

**步骤4：验证安装**

```bash
# 检查安装文件
ls -la ~/.openclaw/skills/llm-wiki/

# 预期看到：
# README.md
# platforms/
# scripts/
# examples/
```

---

### 5.2 自定义安装路径

如果你的OpenClaw使用非标准路径，可以自定义安装位置：

```bash
bash install.sh \
  --platform openclaw \
  --target-dir /your/custom/path/skills
```

**示例**：

```bash
# 安装到自定义位置
bash install.sh \
  --platform openclaw \
  --target-dir ~/MyOpenClaw/skills
```

---

### 5.3 验证安装

#### 验证文件结构

```bash
# 检查主目录
ls ~/.openclaw/skills/llm-wiki/

# 应该看到：
# ├── platforms/
# │   └── openclaw/
# │       └── README.md
# ├── scripts/
# ├── examples/
# └── README.md
```

#### 验证OpenClaw识别

**对Agent说**：

```
请列出你当前可用的技能
```

**预期响应**：

```
当前可用的技能：
- llm-wiki: 知识库构建系统
- [其他技能...]
```

如果`llm-wiki`在列表中，说明安装成功！

---

## 6. OpenClaw配置

### 6.1 核心配置文件

OpenClaw通过以下文件识别和配置LLM-Wiki Skill：

```
~/.openclaw/skills/llm-wiki/platforms/openclaw/README.md
```

这是OpenClaw与LLM-Wiki的接口定义文件。

---

### 6.2 配置内容详解

**文件位置**：`~/.openclaw/skills/llm-wiki/platforms/openclaw/README.md`

**核心内容**：

```markdown
# OpenClaw LLM-Wiki Skill

## Skill元数据
- **ID**: llm-wiki
- **Name**: 知识库构建系统
- **Author**: sdyckjq-lab
- **Version**: 1.0
- **Description**: 基于Karpathy方法论的个人Wiki构建

## 能力清单
- 素材导入（网页、PDF、文本、视频字幕）
- Wiki页面自动生成
- 双向链接管理
- 知识库健康检查
- 综合分析与查询

## 命令注册

### /wiki-init
**用途**：初始化新的知识库
**参数**：<path> - 知识库根目录路径
**示例**：`/wiki-init ~/Documents/MyWiki`

### /wiki-ingest
**用途**：导入素材到知识库
**参数**：<source> - URL、文件路径或文本内容
**示例**：
- `/wiki-ingest https://example.com/article`
- `/wiki-ingest ~/Downloads/paper.pdf`

### /wiki-query
**用途**：查询知识库
**参数**：<question> - 查询问题
**示例**：`/wiki-query RAG和Wiki有什么区别？`

### /wiki-lint
**用途**：知识库健康检查
**参数**：无
**示例**：`/wiki-lint`

### /wiki-status
**用途**：查看知识库状态
**参数**：无
**示例**：`/wiki-status`

## 触发条件
- 用户说"初始化知识库"、"创建Wiki"
- 用户提供URL或文件路径并说"导入"、"添加"
- 用户说"查询"、"找"、"搜索Wiki"
- 用户说"检查"、"lint"、"健康检查"

## 工作流程

### 初始化流程（Init）
1. 创建目录结构（raw/, wiki/, 等）
2. 生成模板文件（index.md, log.md, .wiki-schema.md）
3. 记录初始化日志

### 导入流程（Ingest）
1. 识别素材类型（URL、PDF、文本等）
2. 路由到合适的提取器
3. 保存到raw/相应子目录
4. 生成摘要页（wiki/sources/）
5. 更新/创建实体页（wiki/entities/）
6. 更新主题页（wiki/topics/）
7. 更新索引（index.md）
8. 记录到日志（log.md）

### 查询流程（Query）
1. 读取index.md定位相关页面
2. 读取多个Wiki页面
3. 综合信息生成答案
4. 评估答案价值
5. 如有必要，归档为新页面
6. 记录到log.md

### 健康检查流程（Lint）
1. 扫描所有Wiki页面
2. 检查孤立页面（无入站链接）
3. 检查断开链接（指向不存在的页面）
4. 检查信息矛盾
5. 建议新页面
6. 自动修复可修复的问题
7. 生成报告

## Hook点（OpenClaw特有）
- **on_source_added**: 新素材添加时触发导入流程
- **on_query**: 用户查询时触发查询流程
- **on_periodic**: 定期触发健康检查（可配置）
- **on_startup**: OpenClaw启动时加载配置

## 配置选项

用户可在知识库根目录的 `.wiki-schema.md` 中配置：

### 页面创建规则
- 实体页：概念出现≥3次时创建
- 主题页：相关素材≥5篇时创建
- 摘要页：每个素材自动创建

### 链接规则
- 使用`[[双向链接]]`语法
- 人物名必须链接
- 重要概念首次出现时链接

### 自动化规则
- 每导入10个素材自动触发Lint
- 每周定期健康检查
- 发现矛盾立即标记

## 依赖
- OpenClaw核心功能：文件读写、Shell执行
- 可选：Chrome（网页提取）
- 可选：uv（微信公众号提取）
- 可选：bun/npm（JS内容处理）
```

---

### 6.3 技能配置文件（.wiki-schema.md）

每个知识库根目录都应该有一个 `.wiki-schema.md` 文件，定义Agent的行为规则。

**默认模板**：

```markdown
# Wiki Schema配置

## 页面创建规则
- **实体页**：当概念在素材中出现≥3次时创建独立页面
- **主题页**：某主题有≥5篇相关素材时创建综述页
- **摘要页**：每个导入的素材自动创建

## 链接规则
- 使用`[[双向链接]]`语法
- 人物名、重要概念必须链接
- 工具、产品名称建议链接

## 摘要规则
- **长文**（>2000字）：完整摘要 + 关键点提炼 + 分节处理
- **中等**（500-2000字）：标准摘要 + 实体抽取
- **短文**（<500字）：简化摘要 + 关键点

## 更新触发
- 新素材导入：更新相关实体页、主题页
- 发现矛盾：标记并记录到log.md
- Lint检查：每10个素材后自动触发

## 自动化维护
- 每周一自动健康检查
- 发现孤立页面自动建议链接
- 断开链接自动修复（如果可能）

## OpenClaw特定配置
- 使用 `/wiki-` 命令前缀
- 支持批量操作
- 自动日志记录
```

---

## 7. 目录结构

### 7.1 完整目录树

```
你的知识库根目录/
├── raw/                          # 原始素材（只读）
│   ├── articles/                 # 网页文章
│   │   ├── 2026-04-01-article1.md
│   │   └── 2026-04-02-article2.md
│   ├── tweets/                   # X/Twitter
│   │   └── @username-tweet.md
│   ├── wechat/                   # 微信公众号
│   │   └── account-article.md
│   ├── xiaohongshu/              # 小红书（手动）
│   │   └── note-title.md
│   ├── zhihu/                    # 知乎
│   │   └── question-answer.md
│   ├── youtube/                  # YouTube字幕
│   │   └── video-title.md
│   ├── pdfs/                     # PDF文档
│   │   └── research-paper.pdf
│   ├── notes/                    # 个人笔记
│   │   └── daily-notes.md
│   └── assets/                   # 附件（图片等）
│       ├── images/
│       └── files/
│
├── wiki/                         # AI生成的知识库
│   ├── entities/                 # 实体页面
│   │   ├── people/               # 人物
│   │   │   ├── karpathy.md
│   │   │   └── hinton.md
│   │   ├── concepts/             # 概念
│   │   │   ├── rag.md
│   │   │   └── transformer.md
│   │   └── tools/                # 工具
│   │       ├── openslaw.md
│   │       └── ollama.md
│   │
│   ├── topics/                   # 主题页面
│   │   ├── ai-agents.md
│   │   ├── knowledge-management.md
│   │   └── llm-applications.md
│   │
│   ├── sources/                  # 素材摘要
│   │   ├── article-001.md
│   │   ├── paper-002.md
│   │   └── tweet-003.md
│   │
│   ├── comparisons/              # 对比分析
│   │   ├── rag-vs-wiki.md
│   │   └── gpt4-vs-claude.md
│   │
│   └── synthesis/                # 综合分析
│       ├── ai-trend-2026.md
│       └── knowledge-architecture.md
│
├── index.md                      # 总索引（导航）
├── log.md                        # 操作日志
└── .wiki-schema.md               # 配置文件
```

---

### 7.2 目录职责说明

#### raw/ - 原始素材层

**特性**：
- 🔒 **不可变**：Agent只读，从不修改
- 📂 **分类存储**：按来源类型组织
- 🎯 **真相之源**：所有知识的起点

**命名规范**：
```
格式：[日期]-[标题].md
示例：2026-04-14-karpathy-llm-wiki.md
```

---

#### wiki/ - 知识库层

**特性**：
- ✍️ **Agent拥有**：完全由AI生成和维护
- 🔗 **互相链接**：使用`[[双向链接]]`语法
- 📈 **持续进化**：随素材增加而丰富

**页面类型详解**：

| 类型 | 目录 | 触发条件 | 用途 |
|------|------|---------|------|
| **实体页** | `entities/` | 概念出现≥3次 | 人物、概念、工具的详细页 |
| **主题页** | `topics/` | 素材≥5篇 | 领域主题综述 |
| **摘要页** | `sources/` | 每个素材自动 | 素材精华提炼 |
| **对比页** | `comparisons/` | 用户提问对比 | 横向对比分析 |
| **综合页** | `synthesis/` | 综合查询 | 跨素材综合分析 |

---

## 8. 功能特性

### 8.1 零配置初始化

**OpenClaw命令**：

```
/wiki-init ~/Documents/MyWiki
```

**或自然语言**：

```
请在 ~/Documents/MyWiki 初始化一个新的知识库
```

**Agent会自动**：
1. 创建完整目录结构
2. 生成模板文件（index.md、log.md、.wiki-schema.md）
3. 记录初始化日志
4. 返回成功确认

---

### 8.2 智能素材路由

**支持的来源**：

| 来源类型 | URL模式 | 提取器 | 依赖 |
|---------|---------|-------|------|
| **X/Twitter** | `x.com/*` | baoyu-url-to-markdown | Chrome调试 |
| **微信公众号** | `mp.weixin.qq.com/*` | wechat-article-to-markdown | uv |
| **知乎** | `zhihu.com/*` | baoyu-url-to-markdown | Chrome调试 |
| **YouTube** | `youtube.com/*` | youtube-transcript | 无 |
| **通用网页** | 其他URL | baoyu-url-to-markdown | Chrome调试 |
| **PDF** | `.pdf` | 内置PDF解析 | 无 |
| **纯文本** | 直接粘贴 | 无需提取 | 无 |

**自动识别示例**：

```
/wiki-ingest https://x.com/karpathy/status/123456789
↓
Agent识别为Twitter → 使用baoyu提取器 → 提取成功
```

---

### 8.3 内容分级处理

根据内容长度自动采用不同策略：

| 长度 | 类型 | 处理策略 | Token节省 |
|------|------|---------|----------|
| <500字 | 短内容 | 简化摘要 | ~70% |
| 500-2000字 | 中等 | 标准摘要 | ~40% |
| >2000字 | 长内容 | 完整摘要+分节 | ~20% |

**对于Ollama本地模型用户**，这个特性可以显著提升处理速度。

---

### 8.4 批量消化

**命令**：

```
/wiki-ingest ~/Downloads/papers/
```

**或**：

```
请批量处理 ~/Downloads/papers/ 目录下的所有PDF
```

**Agent工作流**：
1. 扫描目录
2. 识别文件类型
3. 逐个提取（或并行）
4. 生成摘要
5. 更新Wiki
6. 生成批处理报告

---

### 8.5 知识库健康检查

**命令**：

```
/wiki-lint
```

**或**：

```
请对知识库进行健康检查
```

**检查项目**：

| 检查类型 | 说明 | 自动修复 |
|---------|------|---------|
| **孤立页面** | 无入站链接 | ✅ 建议链接位置 |
| **断开链接** | 指向不存在页面 | ✅ 创建页面或修复 |
| **信息矛盾** | 不同页面冲突 | ⚠️ 标记待人工审核 |
| **过时内容** | 被新素材推翻 | ⚠️ 标记待更新 |
| **缺失页面** | 重要概念未建页 | ✅ 建议创建 |

---

### 8.6 Obsidian集成

所有内容都是标准Markdown，可直接用Obsidian打开：

**步骤**：

1. **安装Obsidian**：https://obsidian.md/

2. **打开知识库**：
   ```
   Obsidian → Open folder as vault → 选择Wiki根目录
   ```

3. **查看图谱**：
   ```
   Ctrl/Cmd + G → 查看知识网络
   ```

**Obsidian优势**：

- 📊 **图谱视图**：可视化知识网络
- 🔗 **双向链接**：点击`[[链接]]`直接跳转
- 🔍 **全文搜索**：快速定位内容
- 🏷️ **标签过滤**：`#标签` 分类查看
- 🔌 **插件扩展**：Dataview、Marp等

---

## 9. 使用指南

### 9.1 基础工作流

```
1. 初始化知识库
   ↓
2. 导入素材（Ingest）
   ↓
3. Agent自动编译为Wiki
   ↓
4. 查询使用（Query）
   ↓
5. 好答案归档回Wiki
   ↓
6. 定期健康检查（Lint）
   ↓
循环：持续积累，知识复利
```

---

### 9.2 初始化知识库

**OpenClaw命令**：

```
/wiki-init ~/Documents/AIResearch
```

**自然语言**：

```
请在 ~/Documents/AIResearch 初始化一个知识库，
主题是AI研究，包括：
- 论文阅读
- 技术文章
- 会议笔记
- 实验记录
```

**Agent响应**：

```
✓ 创建目录结构
✓ 生成模板文件
✓ 初始化日志
✓ 配置.wiki-schema.md（已针对AI研究主题优化）

知识库已准备就绪：~/Documents/AIResearch
```

---

### 9.3 导入素材（Ingest）

#### 场景1：导入网页文章

**命令**：

```
/wiki-ingest https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
```

**或自然语言**：

```
请导入这篇文章到我的Wiki：
https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
```

**Agent工作流**：

```
正在提取网页...
✓ 使用 baoyu-url-to-markdown 提取成功
✓ 保存到：raw/articles/2026-04-14-karpathy-llm-wiki.md
✓ 文章长度：4200字（长文）

编译到Wiki...
✓ 生成摘要：wiki/sources/article-001.md
✓ 识别关键实体：
  - [[Karpathy]]（新建）
  - [[LLM Wiki]]（新建）
  - [[RAG]]（新建）
  - [[Memex]]（新建）
✓ 创建主题页：topics/knowledge-management.md
✓ 更新索引：index.md
✓ 记录日志：log.md

完成！本次影响8个页面。
```

---

#### 场景2：导入PDF论文

**命令**：

```
/wiki-ingest ~/Downloads/attention-paper.pdf
```

**或**：

```
我下载了一篇PDF论文，路径是：
~/Downloads/attention-paper.pdf
请帮我导入到Wiki
```

**Agent工作流**：

```
读取PDF...
✓ 检测：43页，英文学术论文
✓ 保存到：raw/pdfs/attention-is-all-you-need.pdf
✓ 提取文本成功

编译到Wiki...
✓ 生成摘要：wiki/sources/paper-001.md
  - 结构：问题→方法→实验→结论
✓ 创建实体页：
  - entities/concepts/attention-mechanism.md
  - entities/concepts/transformer.md
✓ 更新主题页：topics/deep-learning.md
✓ 更新索引和日志

完成！
```

---

#### 场景3：导入X/Twitter内容

**前置要求**：Chrome调试模式

```bash
# macOS启动Chrome调试模式
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 &

# Linux
google-chrome --remote-debugging-port=9222 &
```

**导入推文**：

```
/wiki-ingest https://x.com/karpathy/status/2040470801506541998
```

**Agent工作流**：

```
识别为Twitter链接...
✓ 连接Chrome（端口9222）
✓ 提取推文内容
✓ 保存到：raw/tweets/karpathy-2040470801.md

短内容（<500字），简化处理
✓ 生成简要摘要：wiki/sources/tweet-001.md
✓ 更新 [[Karpathy]] 实体页

完成！
```

---

#### 场景4：批量导入

**命令**：

```
/wiki-ingest ~/Documents/papers/
```

**或**：

```
请批量处理 ~/Documents/papers/ 下的所有PDF
```

**Agent工作流**：

```
扫描目录...
✓ 发现15个PDF文件

开始批量处理：
[1/15] attention-paper.pdf ✓ (3分钟)
[2/15] bert-paper.pdf ✓ (2分钟)
[3/15] gpt3-paper.pdf ✓ (5分钟)
...
[15/15] llama-paper.pdf ✓ (3分钟)

批处理报告：
- 新增摘要：15个
- 新增实体页：12个
- 更新主题页：5个
- 总耗时：约45分钟

详见：wiki/batch-report-2026-04-14.md
```

---

### 9.4 查询知识库（Query）

#### 简单查询

**命令**：

```
/wiki-query RAG和Wiki模式有什么区别？
```

**或**：

```
RAG和Wiki模式有什么区别？
```

**Agent工作流**：

```
查询index.md...找到相关页面
读取：
- wiki/comparisons/rag-vs-wiki.md
- wiki/entities/concepts/rag.md
- wiki/topics/knowledge-management.md

根据Wiki内容回答：

【核心区别】

1. **知识表现**
   - RAG：向量片段
   - Wiki：结构化页面

2. **积累性**
   - RAG：❌ 无积累
   - Wiki：✅ 复利增长

3. **查询方式**
   - RAG：检索拼凑
   - Wiki：导航阅读

详见：[[rag-vs-wiki]]

是否需要将此回答归档为新页面？
```

---

#### 综合分析查询

**命令**：

```
/wiki-query 总结一下2026年AI Agent的发展趋势
```

**或**：

```
请基于Wiki内容，总结2026年AI Agent的发展趋势
```

**Agent工作流**：

```
查询相关素材...找到18篇

综合分析中...
✓ 提取关键趋势
✓ 整理时间线
✓ 识别争议点

生成综合分析：wiki/synthesis/ai-agent-trend-2026.md

【主要趋势】

1. **从单轮到持久化记忆**
   - 例证：Claude Projects、OpenAI Memory
   - 素材来源：5篇文章

2. **从工具调用到工作流编排**
   - 例证：LangGraph、AutoGPT
   - 素材来源：7篇文章

3. **从文本到多模态**
   - 例证：GPT-4V、Claude Vision
   - 素材来源：6篇文章

[完整分析已归档]
可通过 [[AI Agent 2026趋势]] 访问
```

---

### 9.5 健康检查（Lint）

**命令**：

```
/wiki-lint
```

**或**：

```
请对知识库进行健康检查
```

**Agent工作流**：

```
开始全库扫描...

【检查结果】

孤立页面（3个）：
- entities/concepts/memex.md - 无入站链接
- topics/zettelkasten.md - 无引用
- sources/article-025.md - 未被主题页引用

断开链接（5个）：
- topics/ai-agents.md → [[GPT-4]]（页面不存在）
- entities/people/karpathy.md → [[Tesla AI]]（不存在）
- [已自动修复3个]

信息矛盾（2处）：
- article-001: "RAG无积累"
- article-015: "RAG可缓存积累"
→ 已标记，待人工审核

建议创建的页面（4个）：
- [[GPT-4]] - 被引用8次
- [[Tesla AI]] - 被引用5次
- [[向量数据库]] - 被引用6次
- [[语义搜索]] - 被引用4次

【修复操作】

✓ 自动修复：断开链接3个
✓ 自动建议：链接孤立页面到相关主题
⚠️ 待人工确认：信息矛盾2处

详细报告：wiki/lint-report-2026-04-14.md
```

---

### 9.6 查看知识库状态

**命令**：

```
/wiki-status
```

**或**：

```
我的知识库现在是什么状态？
```

**Agent响应**：

```
【知识库状态】

📍 位置：~/Documents/AIResearch

📊 统计：
- 素材数：48篇
  - 文章：32篇
  - 论文：12篇
  - 推文：4条
- Wiki页面：85个
  - 实体页：35个
  - 主题页：15个
  - 摘要页：48个
  - 对比页：3个
  - 综合页：4个

📈 增长趋势：
- 近7天新增：12篇素材
- 本月累计：48篇
- 活跃主题：AI Agents, Knowledge Management

⚠️ 待处理：
- 孤立页面：3个
- 信息矛盾：2处
- 建议健康检查

最后更新：2026-04-14 15:30
```

---

## 10. 素材来源支持

### 10.1 核心主线（无外部依赖）

| 类型 | 支持格式 | 操作方式 | 依赖 |
|------|---------|---------|------|
| **本地文件** | PDF、Markdown、TXT | 直接读取 | ❌ 无 |
| **纯文本** | 粘贴内容 | 即时处理 | ❌ 无 |

**优势**：
- ✅ 100%可靠
- ✅ 离线可用
- ✅ 无需配置

---

### 10.2 可选外挂（需额外工具）

| 来源 | 提取器 | 依赖 | 安装方法 |
|------|-------|------|---------|
| **通用网页** | baoyu-url-to-markdown | Chrome调试 | 见下文 |
| **X/Twitter** | baoyu-url-to-markdown | Chrome调试+登录 | 见下文 |
| **微信公众号** | wechat-article-to-markdown | uv | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **YouTube** | youtube-transcript | ❌ 无 | 内置 |
| **知乎** | baoyu-url-to-markdown | Chrome调试 | 见下文 |

---

### 10.3 手动回退

| 来源 | 原因 | 操作 |
|------|------|------|
| **小红书** | 无可靠提取器 | 复制内容粘贴给Agent |
| **提取失败的网页** | 反爬/登录墙 | 复制内容粘贴给Agent |

**手动粘贴示例**：

```
这是我从小红书复制的内容，请导入到Wiki：

[粘贴内容]
标题：如何构建个人知识库
作者：XXX
链接：https://xiaohongshu.com/...
```

---

### 10.4 Chrome调试模式配置

**macOS**：

```bash
# 关闭所有Chrome实例
killall "Google Chrome"

# 以调试模式启动
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 &
```

**Linux**：

```bash
google-chrome --remote-debugging-port=9222 &
```

**Windows（PowerShell）**：

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" `
  --remote-debugging-port=9222
```

**验证连接**：

```bash
curl http://localhost:9222/json
```

**应该返回JSON数据，包含当前打开的标签页信息。**

---

## 11. 故障排查

### 11.1 安装问题

#### 问题1：install.sh权限不足

**症状**：
```bash
bash: ./install.sh: Permission denied
```

**解决方案**：
```bash
chmod +x install.sh
bash install.sh --platform openclaw
```

---

#### 问题2：OpenClaw无法识别技能

**症状**：
Agent说"我不知道llm-wiki技能"

**检查步骤**：

1. **验证文件存在**：
```bash
ls -la ~/.openclaw/skills/llm-wiki/
```

2. **检查配置文件**：
```bash
cat ~/.openclaw/skills/llm-wiki/platforms/openclaw/README.md
```

3. **重启OpenClaw**（某些情况需要）：
```bash
openclaw restart
# 或重启Agent进程
```

4. **让Agent重新加载技能**：
```
请重新加载你的技能列表
```

---

### 11.2 提取器问题

#### 问题3：Chrome连接失败

**症状**：
```
Error: Cannot connect to Chrome on port 9222
```

**解决方案**：

1. **检查Chrome是否运行**：
```bash
curl http://localhost:9222/json
```

2. **重启Chrome调试模式**：
```bash
# macOS
killall "Google Chrome"
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 &
```

3. **检查端口占用**：
```bash
lsof -i :9222
```

---

#### 问题4：Twitter提取失败

**症状**：
```
Error: Cannot extract Twitter content
```

**解决方案**：

1. **确保Chrome已登录X**：
   - 在Chrome中访问 https://x.com
   - 登录账号
   - 保持标签页打开

2. **手动复制回退**：
```
自动提取失败，我把推文内容粘贴给你：
[粘贴推文内容]
```

---

#### 问题5：微信公众号提取失败

**症状**：
```
Error: wechat-article-to-markdown not found
```

**解决方案**：

1. **安装uv**：
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc  # 或 ~/.zshrc
```

2. **验证安装**：
```bash
uv --version
```

3. **重新运行导入**：
```
/wiki-ingest [微信公众号URL]
```

---

### 11.3 使用问题

#### 问题6：Obsidian无法打开Wiki

**症状**：
Obsidian报错或显示空白

**解决方案**：

1. **验证目录结构**：
```bash
ls -la ~/path/to/wiki/
# 应该看到 raw/, wiki/, index.md, log.md
```

2. **检查权限**：
```bash
chmod -R 755 ~/path/to/wiki/
```

3. **在Obsidian中重新打开**：
```
Obsidian → Open folder as vault → 选择正确路径
```

---

#### 问题7：双向链接不工作

**症状**：
点击`[[链接]]`无反应

**解决方案**：

1. **检查目标文件是否存在**：
```bash
find ~/path/to/wiki -name "*karpathy*.md"
```

2. **运行健康检查**：
```
/wiki-lint
```

3. **让Agent修复断链**：
```
请修复所有断开的链接
```

---

#### 问题8：导入后Wiki没有更新

**症状**：
素材导入成功，但Wiki页面没有变化

**检查步骤**：

1. **查看log.md**：
```bash
tail -n 50 ~/path/to/wiki/log.md
```

2. **手动触发编译**：
```
刚才导入的素材似乎没有编译，请重新处理：
raw/articles/2026-04-14-xxx.md
```

3. **检查.wiki-schema.md配置**：
```bash
cat ~/path/to/wiki/.wiki-schema.md
```

---

### 11.4 性能问题

#### 问题9：处理速度慢（使用Ollama时）

**优化方案**：

1. **使用更小的模型**：
```bash
# 从14b降级到7b
ollama pull qwen3.5:7b
```

2. **启用GPU加速**（如果有独显）：
```bash
# 检查GPU是否被使用
nvidia-smi  # NVIDIA GPU
```

3. **增加并发**（OpenClaw配置）：
```json
{
  "concurrency": 2
}
```

4. **批量处理时降低质量要求**：
```
批量处理时使用简化模式
```

---

#### 问题10：内存占用过高

**症状**：
系统内存不足

**解决方案**：

1. **使用更小的模型**（Ollama）：
```bash
# qwen3.5:7b 只需约5GB内存
ollama pull qwen3.5:7b
```

2. **关闭其他应用**

3. **分批处理大量素材**：
```
请分3批处理这个目录，每批5个文件
```

---

## 12. 最佳实践

### 12.1 初始化最佳实践

#### 实践1：选择合适的根目录

**推荐位置**：

```bash
# 个人知识库
~/Documents/PersonalWiki/

# 项目特定
~/Projects/ProjectName/wiki/

# 研究主题
~/Research/AIResearch/
```

**避免**：
- ❌ 系统目录
- ❌ 云盘根目录（可能冲突）
- ❌ 包含空格的路径

---

#### 实践2：规划主题结构

**初始化时明确主题**：

```
请在 ~/Documents/AIWiki 初始化知识库，
主题是AI研究，包括：
- 论文阅读
- 技术博客
- 会议笔记
- 实验记录

请在.wiki-schema.md中配置相应的规则
```

---

### 12.2 素材导入最佳实践

#### 实践3：保持素材可追溯

**好的做法**：

```markdown
---
source: https://gist.github.com/karpathy/xxx
author: Andrej Karpathy
date: 2026-04-04
type: Gist
tags: [knowledge-management, llm-wiki]
---

# 文章标题

[内容...]
```

**为什么重要**：
- 方便后续验证
- 溯源分析
- 更新追踪

---

#### 实践4：批量导入前先测试

**工作流**：

```
1. 先导入1-2个样本
   /wiki-ingest [样本URL]

2. 检查生成的Wiki质量
   [查看wiki/sources/和entities/]

3. 调整.wiki-schema.md配置
   [如不满意，修改规则]

4. 批量导入剩余素材
   /wiki-ingest [目录路径]
```

---

#### 实践5：使用标签分类

**在素材中添加标签**：

```markdown
---
tags: [ai-agent, knowledge-management, paper]
---
```

**Agent会在Wiki页面中保留这些标签，方便Obsidian过滤。**

---

### 12.3 维护最佳实践

#### 实践6：定期健康检查

**建议频率**：

| 知识库规模 | Lint频率 |
|-----------|---------|
| <50篇素材 | 每周一次 |
| 50-200篇 | 每3天一次 |
| >200篇 | 每天一次（自动化） |

**自动化配置**（.wiki-schema.md）：

```markdown
## 自动化维护
- 每10个素材导入后自动Lint
- 每周一上午自动健康检查
- 发现矛盾立即标记提醒
```

---

#### 实践7：人工审核关键内容

**需要人工审核**：

- ✅ 信息矛盾标记
- ✅ 重要人物/概念的实体页
- ✅ 综合分析结论
- ✅ 对比分析页面

**工作流**：

```
1. Agent标记需人工确认的内容
2. 定期（每周末）集中审核
3. 确认or修改
4. 让Agent更新到Wiki
```

---

#### 实践8：备份策略

**方案1：Git版本控制（推荐）**

```bash
cd ~/path/to/wiki
git init
git add .
git commit -m "Initial wiki"

# 推送到私有仓库
git remote add origin git@github.com:yourname/private-wiki.git
git push -u origin main
```

**方案2：定期打包**

```bash
#!/bin/bash
# backup-wiki.sh

DATE=$(date +%Y%m%d)
tar -czf ~/Backups/wiki-backup-$DATE.tar.gz ~/path/to/wiki/
```

**方案3：云端同步**

将Wiki放在：
- iCloud Drive
- Dropbox
- OneDrive

---

### 12.4 查询最佳实践

#### 实践9：善用回填机制

**好的查询**：

```
/wiki-query 请分析AI Agent在2026年的三大技术突破

[Agent生成深度分析...]

👤：很好，请将这份分析归档为主题页
```

**避免归档**：
- ❌ 简单事实查询
- ❌ 重复内容
- ❌ 临时性问题

---

#### 实践10：构建查询记录

**价值**：
- 追溯思考过程
- 避免重复查询
- 识别知识盲<br>

**log.md会自动记录所有查询，定期回顾可以发现知识盲点。**

---

## 13. 常见问题FAQ

### 13.1 一般问题

#### Q1：为什么用Markdown而不是数据库？

**A**：

**Markdown优势**：

1. **人类可读**：直接查看编辑
2. **工具无关**：任何编辑器都可用
3. **版本控制**：Git原生支持
4. **未来证明**：永远不会过时
5. **跨平台**：完全可移植

**数据库的问题**：
- ❌ 供应商锁定
- ❌ 迁移困难
- ❌ 需要专门工具

---

#### Q2：怎么处理敏感信息？

**A**：

**隐私保护策略**：

1. **本地运行**：
   - 所有数据在本地
   - 使用Ollama模型，不上传云端

2. **敏感内容标记**（.wiki-schema.md）：
   ```markdown
   ## 敏感信息处理
   - 标记为sensitive的页面不自动链接
   - 不包含在公开导出中
   ```

3. **分离知识库**：
   - 公开知识库 vs 私密知识库
   - 不同根目录

---

#### Q3：能和团队共享吗？

**A**：

**可以，方式**：

**方案1：私有Git仓库**

```bash
git init
git add .
git commit -m "Team wiki"
git remote add origin git@github.com:team/private-wiki.git
git push
```

**方案2：共享文件夹**
- 企业网盘
- NAS

**注意事项**：
- ✅ 需要访问控制
- ✅ 需要协作规范（避免冲突）
- ⚠️ Agent操作可能冲突（建议轮流使用）

---

#### Q4：如何迁移到新电脑？

**A**：

**Git方法（推荐）**：

```bash
# 旧电脑
git push

# 新电脑
git clone <repo-url>
cd llm-wiki-skill
bash install.sh --platform openclaw
```

**打包方法**：

```bash
# 旧电脑
tar -czf my-wiki.tar.gz ~/path/to/wiki/

# 新电脑
tar -xzf my-wiki.tar.gz
# 重新安装llm-wiki-skill
```

---

### 13.2 使用问题

#### Q5：如何删除错误导入的素材？

**A**：

**步骤**：

1. **删除原始文件**：
```bash
rm ~/path/to/wiki/raw/articles/wrong-article.md
```

2. **让Agent清理**：
```
刚才导入的wrong-article.md是错误的，
请删除相关摘要页、移除链接、更新索引
```

3. **运行Lint**：
```
/wiki-lint
```

---

#### Q6：能自动去重吗？

**A**：

**部分支持**：

- ✅ URL去重（同一URL不重复导入）
- ✅ 文件名去重（检测同名文件）
- ⚠️ 内容相似需人工判断

**内容相似处理**：

```
发现两篇文章内容重复，请合并：
- article-001.md
- article-015.md

保留更完整的一篇，删除另一篇
```

---

## 附录

### A. 常用命令速查

#### OpenClaw + LLM-Wiki命令

```bash
# 初始化知识库
/wiki-init <path>

# 导入素材
/wiki-ingest <url|file|directory>

# 查询知识库
/wiki-query <question>

# 健康检查
/wiki-lint

# 查看状态
/wiki-status
```

#### 自然语言触发

```
# 初始化
"请在~/Documents/MyWiki初始化知识库"

# 导入
"请导入这篇文章：[URL]"
"请批量处理~/Downloads/papers/"

# 查询
"RAG和Wiki有什么区别？"
"总结AI Agent发展趋势"

# 维护
"请进行健康检查"
"将刚才的回答归档到Wiki"
```

---

### B. 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| **原始素材** | Raw Sources | 未处理的原始文件 |
| **Wiki编译** | Wiki Compilation | 素材转换为结构化Wiki |
| **实体页** | Entity Page | 人物/概念/工具页面 |
| **主题页** | Topic Page | 主题综述页 |
| **双向链接** | Backlink | `[[页面]]`互相引用 |
| **Lint** | Lint | 健康检查 |
| **回填** | File Back | 查询结果归档回Wiki |
| **孤立页面** | Orphan Page | 无入站链接的页面 |

---

### C. 资源链接

#### 官方资源

- **项目主页**: https://github.com/sdyckjq-lab/llm-wiki-skill
- **Karpathy原文**: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- **OpenClaw官网**: https://openclaw.ai/

#### 工具下载

- **Obsidian**: https://obsidian.md/
- **Ollama**: https://ollama.com/
- **uv**: https://github.com/astral-sh/uv
- **bun**: https://bun.sh/

---

**文档结束**
