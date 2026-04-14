# LLM-Wiki Skill 详细部署文档

> **项目地址**：https://github.com/sdyckjq-lab/llm-wiki-skill
> 
> **基于**：[Karpathy的LLM-Wiki方法论](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
> 
> **编写日期**：2026年4月8日
> 
> **版本**：v1.0

---

## 📑 目录

- [1. 项目概述](#1-项目概述)
- [2. 核心理念](#2-核心理念)
- [3. 系统要求](#3-系统要求)
- [4. 支持的平台](#4-支持的平台)
- [5. 前置条件详解](#5-前置条件详解)
- [6. 安装部署](#6-安装部署)
- [7. 目录结构详解](#7-目录结构详解)
- [8. 平台配置指南](#8-平台配置指南)
- [9. 功能特性详解](#9-功能特性详解)
- [10. 使用指南](#10-使用指南)
- [11. 素材来源支持](#11-素材来源支持)
- [12. 故障排查](#12-故障排查)
- [13. 最佳实践](#13-最佳实践)
- [14. 常见问题FAQ](#14-常见问题faq)
- [15. 技术架构](#15-技术架构)
- [16. 贡献与致谢](#16-贡献与致谢)

---

## 1. 项目概述

### 1.1 这是什么？

**llm-wiki-skill** 是一个基于 Karpathy 的 llm-wiki 方法论的**多平台知识库构建系统**，专为以下AI Agent设计：

- 🟦 **Claude Code**
- 🟨 **Codex** 
- 🟩 **OpenClaw**

### 1.2 它解决什么问题？

#### 传统RAG的困境

```
传统RAG：查询 → 检索片段 → 拼凑答案
    ↓
问题：每次都从零开始，无积累
```

#### LLM-Wiki的解决方案

```
LLM-Wiki：素材 → 编译成Wiki → 持续维护 → 知识积累
    ↓
优势：知识被编译一次，持续更新，复利增长
```

### 1.3 核心价值

> **把碎片化的信息变成持续积累、互相链接的知识库。**

**你的角色**：提供素材
**Agent的角色**：整理成Wiki页面，维护链接和结构

---

## 2. 核心理念

### 2.1 三个关键原则

#### 原则1：编译而非检索

```
RAG模式：每次查询重新检索
Wiki模式：知识编译一次，持续维护
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

### 2.2 与Karpathy方法论的关系

| Karpathy原文 | 本项目实现 |
|-------------|-----------|
| 理念文档 | 可执行的多平台Skill |
| 需要自己实现 | 开箱即用的安装脚本 |
| 单一Agent | 支持3个主流Agent |
| 手动配置 | 自动化安装和配置 |

---

## 3. 系统要求

### 3.1 操作系统

| 系统 | 支持状态 | 说明 |
|------|---------|------|
| ✅ **macOS** | 完全支持 | 推荐使用 |
| ✅ **Linux** | 完全支持 | 包括 WSL2 |
| ⚠️ **Windows** | 部分支持 | 建议使用 WSL2 或 Git Bash |

### 3.2 必需组件

| 组件 | 作用 | 检查命令 |
|------|------|---------|
| **Bash** | 运行安装脚本 | `bash --version` |
| **Git** | 克隆仓库 | `git --version` |
| **Agent** | 三选一即可 | 见下节 |

### 3.3 可选组件

| 组件 | 作用 | 必需场景 |
|------|------|---------|
| **Chrome** | 网页提取 | 自动提取网页、X/Twitter、知乎 |
| **uv** | Python环境管理 | 微信公众号提取 |
| **bun/npm** | JS依赖管理 | 自动提取网页内容 |

---

## 4. 支持的平台

### 4.1 Claude Code

**开发商**：Anthropic  
**版本要求**：支持技能系统的版本  
**默认安装路径**：`~/.claude/skills/llm-wiki`

#### 特点

- ✅ 与Claude Sonnet 4深度集成
- ✅ 支持文件读写
- ✅ 支持Shell命令执行
- ✅ 原生Markdown支持

### 4.2 Codex

**开发商**：OpenAI  
**版本要求**：支持Agent模式  
**默认安装路径**：`~/.codex/skills/llm-wiki`（兼容旧路径`~/.Codex/skills`）

#### 特点

- ✅ GPT-4模型支持
- ✅ 代码理解能力强
- ✅ 支持技能扩展
- ✅ 与VS Code集成

### 4.3 OpenClaw

**开发商**：社区开源  
**版本要求**：最新稳定版  
**默认安装路径**：`~/.openclaw/skills/llm-wiki`

#### 特点

- ✅ 开源灵活
- ✅ 可自定义扩展
- ✅ 支持多模型切换
- ✅ 社区活跃

---

## 5. 前置条件详解

### 5.1 Agent能力要求

#### ✅ 必须具备

| 能力 | 说明 | 测试方法 |
|------|------|---------|
| **Shell命令执行** | 能运行bash脚本 | 让Agent执行`echo "test"` |
| **文件读写** | 能创建和修改文件 | 让Agent创建测试文件 |
| **工作目录理解** | 能操作相对路径 | 让Agent列出当前目录 |

#### ⭕ 可选但推荐

| 能力 | 优势 |
|------|------|
| **网络访问** | 可以自动下载依赖 |
| **多线程** | 提升批量处理速度 |
| **长文本处理** | 处理大型PDF和长文章 |

---

### 5.2 环境准备检查清单

#### 第一步：检查Shell

```bash
# 检查bash版本（需要4.0+）
bash --version

# 如果版本过低（macOS常见）
brew install bash  # macOS
```

#### 第二步：检查Agent安装

```bash
# Claude Code
ls ~/.claude/

# Codex
ls ~/.codex/ 或 ls ~/.Codex/

# OpenClaw
ls ~/.openclaw/
```

#### 第三步：检查可选组件

**Chrome调试模式**（用于网页提取）：

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 &

# Linux
google-chrome --remote-debugging-port=9222 &

# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```

**uv安装**（用于微信公众号）：

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# 验证安装
uv --version
```

**bun或npm**（用于JS依赖）：

```bash
# 检查npm（Node.js自带）
npm --version

# 或安装bun（更快）
curl -fsSL https://bun.sh/install | bash

# 验证
bun --version
```

---

## 6. 安装部署

### 6.1 快速安装（推荐）

#### 方法1：Agent自动安装（最简单）

**步骤**：

1. 将以下内容发送给你的Agent：

```
请帮我安装 llm-wiki-skill：
https://github.com/sdyckjq-lab/llm-wiki-skill

根据你的平台自动选择：
- Claude Code 使用：bash install.sh --platform claude
- Codex 使用：bash install.sh --platform codex  
- OpenClaw 使用：bash install.sh --platform openclaw
```

2. Agent会自动：
   - 克隆仓库
   - 检测平台
   - 运行安装脚本
   - 配置技能系统

---

### 6.2 手动安装（完全控制）

#### 步骤详解

**第1步：克隆仓库**

```bash
# 克隆到任意目录（建议用临时目录）
git clone https://github.com/sdyckjq-lab/llm-wiki-skill.git
cd llm-wiki-skill
```

**第2步：检查目录结构**

```bash
# 验证关键文件存在
ls -la install.sh
ls -la scripts/
ls -la platforms/
```

**第3步：选择平台并安装**

```bash
# Claude Code
bash install.sh --platform claude

# Codex
bash install.sh --platform codex

# OpenClaw
bash install.sh --platform openclaw

# 自动检测（仅在单一平台环境）
bash install.sh --platform auto
```

**第4步：验证安装**

```bash
# Claude Code
ls -la ~/.claude/skills/llm-wiki/

# Codex
ls -la ~/.codex/skills/llm-wiki/

# OpenClaw
ls -la ~/.openclaw/skills/llm-wiki/
```

**第5步：测试Agent识别**

发送给你的Agent：

```
请检查 llm-wiki 技能是否已安装
```

---

### 6.3 自定义安装路径

#### 适用场景

- OpenClaw使用非标准路径
- 多用户环境
- 企业定制部署

#### 命令

```bash
bash install.sh \
  --platform openclaw \
  --target-dir /your/custom/skills/path
```

---

### 6.4 旧版兼容说明

#### Claude旧版安装方式

```bash
# 保留给旧用户（不推荐新用户使用）
bash setup.sh
```

**说明**：`setup.sh` 现在只是 `install.sh --platform claude` 的兼容入口。

#### Codex旧路径兼容

安装器会自动检测和支持：
- 新路径：`~/.codex/skills/`
- 旧路径：`~/.Codex/skills/`

---

## 7. 目录结构详解

### 7.1 完整目录树

```
你的知识库根目录/
├── raw/                          # 原始素材（不可变，Agent只读）
│   ├── articles/                 # 网页文章
│   │   ├── 2026-04-01-article1.md
│   │   └── 2026-04-02-article2.md
│   ├── tweets/                   # X/Twitter内容
│   │   └── @username-tweet.md
│   ├── wechat/                   # 微信公众号
│   │   └── account-article.md
│   ├── xiaohongshu/              # 小红书（手动粘贴）
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
├── wiki/                         # AI生成的知识库（Agent完全控制）
│   ├── entities/                 # 实体页面
│   │   ├── people/               # 人物
│   │   │   ├── karpathy.md
│   │   │   └── hinton.md
│   │   ├── concepts/             # 概念
│   │   │   ├── rag.md
│   │   │   └── transformer.md
│   │   └── tools/                # 工具
│   │       ├── obsidian.md
│   │       └── claude.md
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
│   │   └── claude-vs-gpt4.md
│   │
│   └── synthesis/                # 综合分析
│       ├── ai-trend-2026.md
│       └── knowledge-architecture.md
│
├── index.md                      # 总索引（导航地图）
├── log.md                        # 操作日志（时间线）
└── .wiki-schema.md               # 配置文件（Agent行为规则）
```

---

### 7.2 目录说明

#### 7.2.1 raw/ - 原始素材层

**特性**：
- 🔒 **不可变**：Agent只读，从不修改
- 📂 **分类存储**：按来源类型组织
- 🎯 **真相之源**：所有知识的起点

**命名规范**：
```
格式：[日期]-[标题].md
示例：2026-04-08-karpathy-llm-wiki.md
```

---

#### 7.2.2 wiki/ - 知识库层

**特性**：
- ✍️ **Agent拥有**：完全由AI生成和维护
- 🔗 **互相链接**：使用`[[双向链接]]`语法
- 📈 **持续进化**：随素材增加而丰富

**页面类型**：

| 类型 | 目录 | 用途 | 示例 |
|------|------|------|------|
| **实体页** | `entities/` | 人物、概念、工具 | `[[Karpathy]]` |
| **主题页** | `topics/` | 领域主题综述 | `[[AI代理系统]]` |
| **摘要页** | `sources/` | 素材精华提炼 | `article-001.md` |
| **对比页** | `comparisons/` | 横向对比分析 | `rag-vs-wiki.md` |
| **综合页** | `synthesis/` | 跨素材综合 | `ai-trend-2026.md` |

---

#### 7.2.3 特殊文件

##### index.md - 总索引

**作用**：
- 📖 内容目录
- 🗺️ 导航地图
- 🔍 检索入口

**结构示例**：

```markdown
# Wiki索引

## 实体页面（32个）
- [[Andrej Karpathy]] - AI研究者，Tesla前AI总监
- [[Transformer]] - 注意力机制的深度学习架构
- [[Obsidian]] - 知识管理工具

## 主题页面（15个）
- [[知识管理]] - 个人知识库构建方法论
- [[AI代理]] - LLM驱动的自主代理系统

## 素材摘要（48个）
- [[article-001]] - Karpathy的LLM Wiki方法论
- [[paper-002]] - Attention Is All You Need

最后更新：2026-04-08
```

---

##### log.md - 操作日志

**作用**：
- ⏰ 时间线记录
- 📝 变更追踪
- 🔍 历史回溯

**格式示例**：

```markdown
# Wiki操作日志

## [2026-04-08 14:30] ingest | Karpathy LLM Wiki文章
- 创建素材摘要：`sources/article-048.md`
- 更新实体页：`[[Karpathy]]`、`[[Memex]]`
- 创建新主题页：`topics/wiki-methodology.md`
- 影响页面：7个

## [2026-04-08 10:15] query | RAG与Wiki对比
- 生成对比分析：`comparisons/rag-vs-wiki.md`
- 归档回Wiki

## [2026-04-07 16:45] lint | 健康检查
- 发现3个孤立页面
- 修复5个断开的链接
- 标记2处信息矛盾
```

---

##### .wiki-schema.md - 配置文件

**作用**：
- 📋 Agent行为规则
- 🎨 页面格式规范
- 🔄 工作流定义

**内容框架**：

```markdown
# Wiki Schema配置

## 页面创建规则
- 实体页：当概念出现3次以上时创建独立页面
- 主题页：素材达到5篇以上时创建主题综述
- 摘要页：每个素材导入时自动创建

## 链接规则
- 使用`[[双向链接]]`语法
- 人物名必须链接
- 重要概念首次出现时链接

## 摘要规则
- 长文（>2000字）：完整摘要 + 关键点
- 短文（<500字）：简化处理
- 中等：标准摘要

## 更新触发
- 新素材导入：更新相关实体页、主题页
- 发现矛盾：标记并记录到log
- Lint检查：每10个素材后自动触发
```

---

## 8. 平台配置指南

### 8.1 Claude Code配置

#### 8.1.1 配置文件位置

```
~/.claude/skills/llm-wiki/platforms/claude/CLAUDE.md
```

#### 8.1.2 核心配置示例

```markdown
# Claude Code LLM-Wiki Skill

## 你的职责
你是一个知识库管理专家，负责：
1. 将用户提供的素材编译成结构化Wiki
2. 维护页面间的链接和一致性
3. 定期进行健康检查

## 工作流程

### 导入素材（Ingest）
1. 识别素材类型（URL、文件、纯文本）
2. 路由到合适的提取器
3. 生成摘要页
4. 更新相关实体页
5. 记录到log.md

### 查询（Query）
1. 读取index.md定位页面
2. 综合相关信息
3. 生成答案并引用
4. 好答案归档回Wiki

### 自检（Lint）
1. 检查孤立页面
2. 发现矛盾信息
3. 修复断开链接
4. 建议新页面
```

---

### 8.2 Codex配置

#### 8.2.1 配置文件位置

```
~/.codex/skills/llm-wiki/platforms/codex/AGENTS.md
```

#### 8.2.2 核心配置示例

```markdown
# Codex LLM-Wiki Agent

## 技能定义
名称：llm-wiki
版本：1.0
类型：知识管理

## 能力清单
- 素材提取（网页、PDF、文本）
- Wiki页面生成
- 链接管理
- 健康检查

## 触发条件
- 用户说"导入"、"添加素材"
- 用户提供URL或文件路径
- 用户问"查询"、"找"
- 用户说"检查"、"lint"

## 执行流程
[见Claude配置，流程相同]
```

---

### 8.3 OpenClaw配置

#### 8.3.1 配置文件位置

```
~/.openclaw/skills/llm-wiki/platforms/openclaw/README.md
```

#### 8.3.2 核心配置示例

```markdown
# OpenClaw LLM-Wiki Skill

## Skill元数据
ID: llm-wiki
Name: 知识库构建
Author: sdyckjq-lab
Version: 1.0

## 命令注册
- `/wiki-ingest <source>` - 导入素材
- `/wiki-query <question>` - 查询知识库
- `/wiki-lint` - 健康检查
- `/wiki-init <path>` - 初始化知识库

## Hook点
- on_source_added: 触发导入流程
- on_query: 触发查询流程
- on_periodic: 定期健康检查
```

---

## 9. 功能特性详解

### 9.1 零配置初始化

#### 功能描述

一句话创建知识库，自动生成完整的目录结构和模板文件。

#### 使用方法

**对Agent说**：

```
请在 ~/Documents/MyWiki 初始化一个新的知识库
```

**Agent会自动**：

```bash
mkdir -p ~/Documents/MyWiki/{raw,wiki}/{articles,tweets,wechat,zhihu,pdfs,notes,assets}
mkdir -p ~/Documents/MyWiki/wiki/{entities,topics,sources,comparisons,synthesis}
touch ~/Documents/MyWiki/{index.md,log.md,.wiki-schema.md}
```

**生成的文件**：
- `index.md` - 带模板的索引文件
- `log.md` - 首条日志记录
- `.wiki-schema.md` - 默认配置规则

---

### 9.2 智能素材路由

#### 功能描述

根据URL域名自动选择最佳提取方式，无需手动指定。

#### 路由规则

| URL模式 | 提取器 | 自动/手动 |
|---------|-------|----------|
| `x.com/*` | baoyu-url-to-markdown | 自动（需Chrome） |
| `twitter.com/*` | baoyu-url-to-markdown | 自动（需Chrome） |
| `mp.weixin.qq.com/*` | wechat-article-to-markdown | 自动（需uv） |
| `zhihu.com/*` | baoyu-url-to-markdown | 自动（需Chrome） |
| `youtube.com/*` | youtube-transcript | 自动 |
| `xiaohongshu.com/*` | - | 手动粘贴 |
| 其他网页 | baoyu-url-to-markdown | 自动（需Chrome） |

#### 示例对话

**用户**：
```
请帮我导入这篇文章：
https://x.com/karpathy/status/123456789
```

**Agent**：
```
识别为X/Twitter，使用baoyu-url-to-markdown提取...
✓ 提取成功
✓ 保存到 raw/tweets/karpathy-123456789.md
✓ 生成摘要 wiki/sources/tweet-001.md
✓ 更新实体页 [[Karpathy]]
```

---

### 9.3 内容分级处理

#### 功能描述

根据内容长度采用不同处理策略，避免浪费Token。

#### 分级规则

| 长度 | 类型 | 处理策略 |
|------|------|---------|
| <500字 | 短内容 | 简化摘要，快速提取关键点 |
| 500-2000字 | 中等内容 | 标准摘要 + 实体抽取 |
| >2000字 | 长内容 | 完整摘要 + 分节处理 + 深度分析 |

#### 处理差异

**短内容示例**（Twitter）：
```markdown
# 素材摘要：Karpathy关于LLM Wiki的推文

## 核心观点
- Wiki优于RAG，因为有持续积累
- 维护成本接近零
- 适合个人知识管理

## 相关实体
- [[Karpathy]]
- [[LLM Wiki]]
```

**长内容示例**（技术文章）：
```markdown
# 素材摘要：深度理解LLM Wiki方法论

## 文章结构
1. RAG的局限性分析
2. Wiki编译范式介绍
3. 三层架构设计
4. 四大核心操作
5. 实践案例

## 详细摘要
### 第一部分：RAG的局限
[详细内容...]

### 第二部分：Wiki范式
[详细内容...]

## 关键引用
> "知识被编译一次，持续维护..."

## 相关实体
- [[Karpathy]]
- [[RAG]]
- [[LLM Wiki]]
- [[Memex]]
- [[Obsidian]]

## 主题标签
#知识管理 #AI代理 #个人Wiki
```

---

### 9.4 批量消化

#### 功能描述

给定一个文件夹路径，自动批量处理所有文件。

#### 使用方法

**对Agent说**：

```
请批量处理 ~/Downloads/papers/ 目录下的所有PDF
```

**Agent工作流**：

```
扫描目录 → 识别文件类型 → 逐个提取 → 生成摘要 → 更新Wiki
```

#### 支持的文件类型

- ✅ PDF（`.pdf`）
- ✅ Markdown（`.md`）
- ✅ 文本文件（`.txt`）
- ✅ HTML（`.html`、`.htm`）
- ✅ Word文档（`.docx`，部分支持）

#### 批量处理策略

```
并行处理：同时处理多个文件（如果Agent支持）
顺序处理：逐个处理，确保质量
增量更新：只处理新增和变更的文件
```

---

### 9.5 结构化Wiki生成

#### 功能描述

自动生成五种类型的Wiki页面，并用双向链接互相关联。

#### 页面生成规则

##### 规则1：实体页创建

**触发条件**：
- 概念在素材中出现≥3次
- 人物被多次提及
- 工具/产品被讨论

**示例**（`entities/concepts/rag.md`）：

```markdown
# RAG（检索增强生成）

## 定义
一种将检索系统与大语言模型结合的方法，通过检索相关文档片段来增强生成质量。

## 工作原理
1. 将文档切成片段
2. 存入向量数据库
3. 查询时检索相关片段
4. 将片段作为上下文给LLM

## 优势
- 可扩展到大量文档
- 无需重新训练模型

## 局限
- 每次查询重新检索（无积累）
- 依赖检索质量
- 片段化可能丢失上下文

## 相关页面
- [[LLM Wiki]] - 替代方案
- [[向量数据库]]
- [[语义搜索]]

## 来源
- [[article-001]] - Karpathy的对比分析
- [[paper-002]] - RAG原始论文

最后更新：2026-04-08
```

---

##### 规则2：主题页创建

**触发条件**：
- 某主题有≥5篇相关素材
- 用户明确要求综述

**示例**（`topics/ai-agents.md`）：

```markdown
# AI代理系统

## 概述
基于大语言模型的自主代理系统，能够理解任务、制定计划、执行操作。

## 核心能力
- 任务理解与分解
- 工具使用
- 长期记忆
- 自我反思

## 当前挑战
1. 可靠性问题
2. 成本控制
3. 安全性保障

## 主要实现
- [[Claude Code]] - Anthropic
- [[Codex]] - OpenAI
- [[OpenClaw]] - 开源社区

## 相关素材
- [[article-012]] - Agent架构设计
- [[article-023]] - 记忆系统实现
- [[article-034]] - 工具调用机制

## 发展趋势
[基于素材的综合分析...]

最后更新：2026-04-08
标签：#AI #Agent #LLM
```

---

##### 规则3：对比页创建

**触发条件**：
- 用户提问"A和B有什么区别"
- 素材中反复对比两个概念

**示例**（`comparisons/rag-vs-wiki.md`）：

```markdown
# RAG vs LLM Wiki 对比分析

## 生成背景
询问：RAG和Wiki模式的核心区别是什么？
日期：2026-04-08

## 核心差异

| 维度 | RAG | LLM Wiki |
|------|-----|----------|
| **知识表现** | 向量片段 | 结构化页面 |
| **查询方式** | 检索+拼凑 | 导航+阅读 |
| **积累性** | 无 | 有 |
| **维护** | 自动索引 | AI主动维护 |
| **成本** | 查询时高 | 维护时高，查询时低 |

## 详细分析

### 工作流程差异
[详细对比...]

### 适用场景
[场景分析...]

## 结论
[综合评价...]

## 来源
- [[article-001]] - Karpathy的分析
- 用户查询-2026-04-08

最后更新：2026-04-08
```

---

### 9.6 知识库健康检查

#### 功能描述

定期自动检测知识库的完整性和一致性，提出改进建议。

#### 检查项目

| 检查类型 | 说明 | 修复方式 |
|---------|------|---------|
| **孤立页面** | 无入站链接的页面 | 添加到相关主题页 |
| **断开链接** | 指向不存在页面的链接 | 创建页面或修复链接 |
| **信息矛盾** | 不同页面信息冲突 | 标记并记录 |
| **过时内容** | 被新素材推翻的旧信息 | 更新并注明 |
| **缺失页面** | 重要概念未建页 | 创建实体页 |

#### 使用方法

**手动触发**：

```
请对知识库进行健康检查
```

**自动触发**：
- 每导入10个素材自动触发
- 每周定期检查
- 用户查询发现矛盾时触发

#### 检查报告示例

```markdown
# Wiki健康检查报告
时间：2026-04-08 15:30

## 发现的问题

### 1. 孤立页面（3个）
- `entities/concepts/memex.md` - 无入站链接
- `topics/zettelkasten.md` - 无引用
- `sources/article-025.md` - 未被任何主题页引用

### 2. 断开链接（5个）
- `topics/ai-agents.md` → `[[GPT-4]]` (页面不存在)
- `entities/people/karpathy.md` → `[[Tesla AI]]` (页面不存在)

### 3. 信息矛盾（2处）
- `article-001.md` 称"RAG无积累"
- `article-015.md` 称"RAG可以通过缓存积累"
- 需人工判断

### 4. 建议创建的页面（4个）
- `[[GPT-4]]` - 被引用8次
- `[[Tesla AI]]` - 被引用5次
- `[[向量数据库]]` - 被引用6次

## 修复计划
1. 将Memex页面连接到知识管理主题
2. 创建缺失的实体页
3. 标记信息矛盾供人工审核

已自动修复：3项
待人工确认：2项
```

---

### 9.7 Obsidian兼容

#### 功能描述

所有内容都是本地Markdown，可直接用Obsidian打开，享受原生图形界面。

#### Obsidian优势

| 功能 | 说明 |
|------|------|
| **图谱视图** | 可视化页面间的链接关系 |
| **双向链接** | 点击`[[链接]]`直接跳转 |
| **搜索** | 全库搜索 |
| **标签** | `#标签` 过滤 |
| **插件** | Dataview、Marp等扩展 |

#### 配置步骤

1. **安装Obsidian**：https://obsidian.md/

2. **打开知识库**：
   ```
   Obsidian → Open folder as vault → 选择你的Wiki根目录
   ```

3. **推荐插件**：
   - **Dataview**：动态查询和统计
   - **Graph View**：知识网络可视化
   - **Canvas**：思维导图
   - **Marp**：Markdown转PPT

4. **查看图谱**：
   ```
   Ctrl/Cmd + G → 查看全局关系图
   ```

#### 示例：Dataview查询

在任意Markdown文件中：

````markdown
```dataview
TABLE 
  file.ctime AS "创建日期",
  length(file.outlinks) AS "链接数"
FROM "wiki/entities"
WHERE contains(file.tags, "#AI")
SORT file.ctime DESC
```
````

**结果**：显示所有带`#AI`标签的实体页，按创建日期排序。

---

## 10. 使用指南

### 10.1 基础工作流

#### 流程图

```
素材获取 → 导入Wiki → Agent处理 → 查询使用 → 回填归档 → 定期维护
   ↑                                                              ↓
   └──────────────────── 循环复利增长 ──────────────────────────┘
```

---

### 10.2 导入素材（Ingest）

#### 场景1：导入网页文章

**对话示例**：

```
👤 用户：
请帮我导入这篇文章：
https://www.example.com/article

🤖 Agent：
正在提取网页内容...
✓ 使用 baoyu-url-to-markdown 提取成功
✓ 保存到：raw/articles/2026-04-08-example-article.md
✓ 文章长度：3500字（长文）

开始编译到Wiki...
✓ 生成摘要：wiki/sources/article-049.md
✓ 识别关键实体：
  - [[Karpathy]]（已存在，已更新）
  - [[LLM Wiki]]（已存在，已更新）
  - [[知识编译]]（新概念，已创建）
✓ 更新主题页：topics/knowledge-management.md
✓ 记录到日志：log.md

完成！本次影响6个页面。
```

---

#### 场景2：导入PDF论文

**对话示例**：

```
👤 用户：
我有一篇PDF论文，路径是 ~/Downloads/attention-paper.pdf

🤖 Agent：
正在读取PDF...
✓ 检测到：43页，英文学术论文
✓ 保存到：raw/pdfs/attention-is-all-you-need.pdf
✓ 提取文本：成功

开始编译...
✓ 生成摘要：wiki/sources/paper-012.md
  - 摘要结构：问题背景→方法→实验→结论
✓ 创建新页面：
  - entities/concepts/attention-mechanism.md
  - entities/concepts/transformer.md
✓ 更新相关页面：4个

完成！
```

---

#### 场景3：导入X/Twitter内容

**前置要求**：Chrome以调试模式运行

```bash
# 启动Chrome（macOS）
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 &
```

**对话示例**：

```
👤 用户：
https://x.com/karpathy/status/2040470801506541998

🤖 Agent：
识别为X/Twitter链接，使用baoyu提取器...
✓ 连接Chrome成功（端口9222）
✓ 提取推文内容
✓ 保存到：raw/tweets/karpathy-2040470801.md

内容长度：短推文（<500字），采用简化处理
✓ 生成简要摘要：wiki/sources/tweet-008.md
✓ 更新 [[Karpathy]] 页面

完成！
```

---

#### 场景4：批量导入文件夹

**对话示例**：

```
👤 用户：
请批量处理 ~/Documents/papers/ 下的所有PDF

🤖 Agent：
扫描目录...
✓ 发现12个PDF文件

开始批量处理：
[1/12] attention-paper.pdf ✓
[2/12] bert-paper.pdf ✓
[3/12] gpt3-paper.pdf ✓
...
[12/12] llama-paper.pdf ✓

批量处理完成！
- 新增摘要：12个
- 新增实体页：8个
- 更新主题页：3个
- 总耗时：约15分钟
```

---

### 10.3 查询知识库（Query）

#### 场景1：简单查询

**对话示例**：

```
👤 用户：
RAG和Wiki模式有什么区别？

🤖 Agent：
查询index.md → 找到相关页面...
读取：
- wiki/comparisons/rag-vs-wiki.md
- wiki/entities/concepts/rag.md
- wiki/topics/knowledge-management.md

根据Wiki内容回答：

核心区别：
1. **知识表现**：RAG用向量片段，Wiki用结构化页面
2. **积累性**：RAG无积累，Wiki有复利增长
3. **查询方式**：RAG检索拼凑，Wiki导航阅读
4. **维护**：RAG自动索引，Wiki AI主动维护

详见：[[rag-vs-wiki]]

是否需要将此回答归档为新页面？
```

---

#### 场景2：综合分析查询

**对话示例**：

```
👤 用户：
总结一下2026年AI Agent的发展趋势

🤖 Agent：
查询相关素材...找到18篇相关文章

综合分析中...
✓ 提取关键趋势
✓ 整理时间线
✓ 识别争议点

生成综合分析：wiki/synthesis/ai-agent-trend-2026.md

主要趋势：
1. 从单轮对话到持久化记忆
2. 从工具调用到工作流编排
3. 从文本到多模态
...

已归档到Wiki，可通过 [[AI Agent 2026趋势]] 访问。
```

---

### 10.4 回填归档（File Back）

#### 机制说明

好的查询答案不应消失在对话历史中，应该成为Wiki的一部分。

#### 触发方式

**自动触发**：
- 综合分析类回答
- 对比分析类回答
- 超过500字的深度回答

**手动触发**：

```
请将刚才的回答归档到Wiki
```

#### 归档流程

```
生成回答 → 评估价值 → 创建Wiki页面 → 更新索引 → 记录日志
```

---

### 10.5 健康检查（Lint）

#### 手动触发

```
👤 用户：
请对知识库进行健康检查

🤖 Agent：
开始全库扫描...

检查项目：
✓ 孤立页面检测
✓ 断开链接检测
✓ 信息矛盾检测
✓ 过时内容检测
✓ 缺失页面建议

生成报告：wiki/lint-report-2026-04-08.md

主要发现：
- 3个孤立页面
- 5个断开链接
- 2处信息矛盾
- 建议创建4个新页面

已自动修复：断开链接5个
待人工确认：信息矛盾2处

查看详情：[[lint-report-2026-04-08]]
```

---

#### 自动触发

在`.wiki-schema.md`中配置：

```markdown
## 自动Lint规则
- 每导入10个素材触发一次
- 每周一自动检查
- 发现矛盾时立即标记
```

---

## 11. 素材来源支持

### 11.1 来源分类

#### 11.1.1 核心主线（无外部依赖）

| 类型 | 支持格式 | 特点 |
|------|---------|------|
| **本地文件** | PDF、Markdown、TXT、HTML | 直接读取 |
| **纯文本** | 粘贴内容 | 即时处理 |

**优势**：
- ✅ 无需额外工具
- ✅ 100%可靠
- ✅ 离线可用

---

#### 11.1.2 可选外挂（需额外工具）

| 来源 | 提取器 | 依赖 | 失败回退 |
|------|-------|------|---------|
| **网页文章** | baoyu-url-to-markdown | Chrome调试模式 | 手动复制粘贴 |
| **X/Twitter** | baoyu-url-to-markdown | Chrome调试模式 + 登录 | 手动复制 |
| **微信公众号** | wechat-article-to-markdown | uv | 手动复制 |
| **YouTube** | youtube-transcript | 无 | 手动字幕下载 |
| **知乎** | baoyu-url-to-markdown | Chrome调试模式 | 手动复制 |

---

#### 11.1.3 手动入口

| 来源 | 原因 | 操作 |
|------|------|------|
| **小红书** | 无可靠提取器 | 用户复制内容粘贴给Agent |

---

### 11.2 提取器详解

#### 11.2.1 baoyu-url-to-markdown

**作者**：[JimLiu](https://github.com/JimLiu)  
**项目**：https://github.com/JimLiu/baoyu-skills

**工作原理**：
1. 通过Chrome DevTools Protocol连接Chrome
2. 在真实浏览器环境中渲染页面
3. 提取主要内容
4. 转换为Markdown

**支持的网站**：
- 大部分新闻网站
- 技术博客
- X/Twitter
- 知乎

**环境要求**：
```bash
# Chrome必须以调试模式启动
chrome --remote-debugging-port=9222

# 验证连接
curl http://localhost:9222/json
```

---

#### 11.2.2 wechat-article-to-markdown

**作者**：[jackwener](https://github.com/jackwener)  
**项目**：https://github.com/jackwener/wechat-article-to-markdown

**工作原理**：
1. 解析微信公众号文章URL
2. 提取文章内容和图片
3. 转换为Markdown格式

**环境要求**：
```bash
# 需要uv（Python环境管理器）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 验证安装
uv --version
```

---

#### 11.2.3 youtube-transcript

**功能**：提取YouTube视频的字幕/转写文本

**工作原理**：
1. 解析YouTube视频ID
2. 获取可用的字幕轨道
3. 提取文本内容

**无需额外依赖**

---

### 11.3 失败回退策略

#### 回退流程

```
自动提取尝试 → 失败 → Agent提示回退方案 → 用户手动操作
```

#### 示例对话

```
👤用户：
https://x.com/someone/status/123

🤖 Agent：
尝试自动提取...
✗ 失败：Chrome未启动调试模式

回退方案：
1. 启动Chrome调试模式：
   /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
     --remote-debugging-port=9222 &
   
2. 或者，你可以直接复制推文内容给我，我会处理

请选择一种方式继续。
```

---

## 12. 故障排查

### 12.1 安装问题

#### 问题1：install.sh权限不足

**症状**：
```bash
bash: ./install.sh: Permission denied
```

**解决方案**：
```bash
chmod +x install.sh
bash install.sh --platform claude
```

---

#### 问题2：平台检测失败

**症状**：
```
Error: Cannot detect target platform
```

**原因**：
- 没有找到任何Agent的配置目录
- 使用了`--platform auto`但环境中有多个平台

**解决方案**：
```bash
# 明确指定平台
bash install.sh --platform claude

# 或自定义路径
bash install.sh --platform openclaw --target-dir ~/.openclaw/skills
```

---

#### 问题3：安装到错误的位置

**症状**：
Agent无法识别技能

**检查方法**：
```bash
# 检查实际安装位置
find ~ -name "llm-wiki" -type d

# 检查Agent配置目录
ls -la ~/.claude/skills/
ls -la ~/.codex/skills/
ls -la ~/.openclaw/skills/
```

**解决方案**：
```bash
# 重新安装到正确位置
bash install.sh --platform <你的平台> --target-dir <正确路径>
```

---

### 12.2 提取器问题

#### 问题4：Chrome调试连接失败

**症状**：
```
Error: Cannot connect to Chrome on port 9222
```

**检查**：
```bash
# 检查Chrome是否在调试模式运行
curl http://localhost:9222/json
```

**解决方案**：

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

**Windows**：
```cmd
"C:\Program Files\Google\Chrome\Application\chrome.exe" ^
  --remote-debugging-port=9222
```

---

#### 问题5：X/Twitter提取失败

**症状**：
```
Error: Cannot extract Twitter content
```

**常见原因**：
1. Chrome未登录X/Twitter
2. X网站反爬机制
3. 网络连接问题

**解决方案**：

**方案1**：确保Chrome已登录
```
1. 在Chrome中访问 https://x.com
2. 登录你的账号
3. 保持该标签页打开
4. 重试提取
```

**方案2**：手动复制粘贴
```
👤 用户：
无法自动提取，我直接把内容粘贴给你：
[粘贴推文内容]
```

---

#### 问题6：微信公众号提取失败

**症状**：
```
Error: wechat-article-to-markdown not found
```

**原因**：未安装uv

**解决方案**：
```bash
# 安装uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 重新运行安装器
bash install.sh --platform claude

# 验证
uv --version
```

---

### 12.3 Agent问题

#### 问题7：Agent无法识别技能

**症状**：
Agent说"我不知道怎么使用llm-wiki"

**检查方法**：

**对Agent说**：
```
请列出你当前可用的技能
```

**如果llm-wiki不在列表中**：

1. **检查安装路径**：
```bash
# Claude Code
ls ~/.claude/skills/llm-wiki/

# Codex
ls ~/.codex/skills/llm-wiki/

# OpenClaw
ls ~/.openclaw/skills/llm-wiki/
```

2. **检查配置文件**：
```bash
# Claude Code
cat ~/.claude/skills/llm-wiki/platforms/claude/CLAUDE.md

# Codex
cat ~/.codex/skills/llm-wiki/platforms/codex/AGENTS.md

# OpenClaw
cat ~/.openclaw/skills/llm-wiki/platforms/openclaw/README.md
```

3. **重启Agent**（某些Agent需要重启才能加载新技能）

---

#### 问题8：Agent执行命令失败

**症状**：
```
Error: Command failed: bash install.sh
```

**原因**：
- Agent当前目录不正确
- 权限不足
- 路径中有空格或特殊字符

**解决方案**：

**对Agent说**：
```
请先进入仓库目录：
cd /path/to/llm-wiki-skill

然后运行：
bash install.sh --platform claude
```

---

### 12.4 Wiki使用问题

#### 问题9：Obsidian无法打开Wiki

**症状**：
Obsidian报错或显示空白

**原因**：
- 路径不正确
- 目录结构损坏
- 权限问题

**解决方案**：

1. **验证目录结构**：
```bash
ls -la /your/wiki/path/
# 应该看到 raw/, wiki/, index.md, log.md
```

2. **检查权限**：
```bash
chmod -R 755 /your/wiki/path/
```

3. **在Obsidian中重新打开**：
```
Obsidian → Open folder as vault → 选择正确路径
```

---

#### 问题10：双向链接不工作

**症状**：
点击`[[链接]]`无反应

**原因**：
- 目标页面不存在
- 文件名与链接不匹配
- Obsidian缓存问题

**解决方案**：

1. **检查目标文件是否存在**：
```bash
# 假设链接是 [[Karpathy]]
find /your/wiki -name "*karpathy*.md" -o -name "*Karpathy*.md"
```

2. **清除Obsidian缓存**：
```
Obsidian → Settings → Files and links → Clear cache
```

3. **让Agent修复断链**：
```
请运行健康检查，修复所有断开的链接
```

---

## 13. 最佳实践

### 13.1 初始化最佳实践

#### 实践1：选择合适的根目录

**推荐位置**：

```bash
# 个人知识库
~/Documents/PersonalWiki/

# 项目特定
~/Projects/ProjectName/wiki/

# 研究主题
~/Research/TopicName/
```

**避免**：
- ❌ 系统目录（`/usr/`, `/etc/`）
- ❌ 云盘同步目录根部（可能冲突）
- ❌ 包含空格或特殊字符的路径

---

#### 实践2：规划目录结构

**对Agent说**：

```
请在 ~/Documents/AIResearch/ 初始化知识库，
并为以下主题创建预分类：
- 论文阅读
- 技术文章
- 会议笔记
- 实验记录
```

---

### 13.2 素材导入最佳实践

#### 实践3：保持素材来源可追溯

**好的做法**：

```markdown
# raw/articles/2026-04-08-karpathy-wiki.md

---
来源：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
作者：Andrej Karpathy
日期：2026-04-04
类型：Gist文档
---

[文章内容...]
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
2. 检查生成的Wiki质量
3. 调整.wiki-schema.md配置
4. 再批量导入剩余素材
```

---

#### 实践5：长文分段处理

**对超长文章**（>10000字）：

**对Agent说**：

```
这篇文章很长，请分几部分处理：
1. 先处理摘要和整体结构
2. 再处理关键章节的细节
3. 最后生成完整的Wiki页面
```

---

### 13.3 Wiki维护最佳实践

#### 实践6：定期健康检查

**建议频率**：

| 规模 | 频率 |
|------|------|
| <50篇素材 | 每周一次 |
| 50-200篇 | 每3天一次 |
| >200篇 | 每天一次 |

**自动化**：

在`.wiki-schema.md`中配置：
```markdown
## 自动维护规则
每导入10个素材 → 自动Lint
每周一上午 → 全面健康检查
发现矛盾 → 立即标记提醒
```

---

#### 实践7：人工审核关键页面

**需要人工审核的内容**：

- ✅ 信息矛盾标记
- ✅ 重要人物/概念的实体页
- ✅ 综合分析页面
- ✅ 对比分析结论

**工作流**：

```
1. Agent标记需人工确认项
2. 定期（如每周末）集中审核
3. 确认or修改
4. 更新到Wiki
```

---

#### 实践8：备份策略

**推荐方案**：

**方案1：Git版本控制**

```bash
cd /your/wiki/path
git init
git add .
git commit -m "Initial wiki"

# 每次大规模更新后
git add .
git commit -m "Added 20 new sources"

# 推送到远程（私有仓库）
git remote add origin git@github.com:yourname/private-wiki.git
git push -u origin main
```

**方案2：定期快照**

```bash
#!/bin/bash
# backup-wiki.sh

DATE=$(date +%Y%m%d)
tar -czf ~/Backups/wiki-backup-$DATE.tar.gz /your/wiki/path/
```

**方案3：云端同步**

```
将Wiki根目录放在：
- iCloud Drive
- Dropbox  
- OneDrive
```

**注意**：确保同步工具不会与Obsidian冲突。

---

### 13.4 查询最佳实践

#### 实践9：善用回填机制

**好的查询**：

```
👤：请分析AI Agent在2026年的三大技术突破

🤖：[生成深度分析...]

👤：很好，请将这份分析归档为主题页
```

**避免**：
- ❌ 简单事实查询就归档
- ❌ 重复内容归档
- ❌ 临时性问题归档

---

#### 实践10：构建查询记录

**在log.md中记录查询**：

```markdown
## [2026-04-08] query | AI Agent技术突破分析
- 综合18篇素材
- 生成主题页：topics/ai-agent-2026-breakthroughs.md
- 新增实体页：3个
- 耗时：5分钟
```

**价值**：
- 追溯思考过程
- 避免重复查询
- 识别知识盲点

---

## 14. 常见问题FAQ

### 14.1 一般问题

#### Q1：这个项目和NotebookLM有什么区别？

**A**：

| 维度 | NotebookLM | LLM-Wiki |
|------|-----------|----------|
| **模式** | RAG（检索） | Wiki（编译） |
| **积累性** | 无 | 有 |
| **本地化** | 云端 | 本地 |
| **可定制** | 有限 | 完全可控 |
| **平台** | Google | 多Agent支持 |

**总结**：NotebookLM适合临时查询，LLM-Wiki适合长期积累。

---

#### Q2：我需要编程基础吗？

**A**：

**不需要编程**，但有基础更好：

| 操作 | 编程要求 |
|------|---------|
| 基本使用 | 无需任何编程知识 |
| 安装配置 | 会运行bash命令即可 |
| 高级定制 | 需要编辑Markdown和Shell脚本 |
| 开发扩展 | 需要Python/TypeScript知识 |

---

#### Q3：能用中文吗？

**A**：

**完全支持中文**：

- ✅ 中文素材
- ✅ 中文Wiki页面
- ✅ 中文查询
- ✅ 中文实体名

**建议**：
- 实体页文件名用英文或拼音（避免路径问题）
- 页面内容可以全中文

**示例**：
```
文件名：entities/people/karpathy.md
内容标题：# 安德烈·卡帕西（Andrej Karpathy）
```

---

#### Q4：支持哪些语言模型？

**A**：

取决于你使用的Agent：

| Agent | 支持的模型 |
|-------|----------|
| **Claude Code** | Claude Sonnet 4, Opus 4 |
| **Codex** | GPT-4, GPT-4o |
| **OpenClaw** | 可配置多种模型 |

---

#### Q5：有大小限制吗？

**A**：

**理论上无限制**，但实际：

| 规模 | 性能 | 建议 |
|------|------|------|
| <100篇素材 | 优秀 | 单一知识库 |
| 100-500篇 | 良好 | 考虑使用搜索增强 |
| 500-1000篇 | 可用 | 必须使用搜索工具（qmd/OMEGA） |
| >1000篇 | 需优化 | 考虑分库或企业方案 |

**扩展方案**：
- 安装qmd或OMEGA提供语义搜索
- 按主题分离多个知识库
- 使用外部向量数据库

---

### 14.2 技术问题

#### Q6：为什么用Markdown而不是数据库？

**A**：

**Markdown的优势**：

1. **人类可读**：直接打开编辑
2. **工具无关**：任何编辑器都可用
3. **版本控制**：Git原生支持
4. **未来证明**：永远不会过时
5. **跨平台**：完全可移植

**数据库的问题**：
- ❌ 供应商锁定
- ❌ 迁移困难
- ❌ 需要专门工具
- ❌ 不易人工审核

---

#### Q7：可以用其他前端吗（不用Obsidian）？

**A**：

**完全可以**，因为底层是标准Markdown：

| 工具 | 特点 |
|------|------|
| **VS Code** | 开发者友好，插件丰富 |
| **Typora** | 简洁美观，所见即所得 |
| **Logseq** | 大纲笔记，双向链接 |
| **Notion** | 需导入，有限支持 |
| **自定义Web界面** | 完全可控 |

---

#### Q8：怎么处理敏感信息？

**A**：

**隐私保护策略**：

1. **本地运行**：
   - 所有数据在本地
   - 不上传到云端

2. **敏感内容标记**：
   ```markdown
   ---
   sensitive: true
   ---
   ```

3. **使用本地模型**（OpenClaw）：
   - LLaMA、Mistral等开源模型
   - 完全离线运行

4. **分离敏感知识库**：
   - 公开知识库 vs 私密知识库
   - 不同的根目录

---

#### Q9：能和团队共享吗？

**A**：

**可以，但需注意**：

**方案1：私有Git仓库**
```bash
# 初始化
git init
git add .
git commit -m "Team wiki"

# 推送到私有仓库
git remote add origin git@github.com:team/wiki.git
git push

# 团队成员克隆
git clone git@github.com:team/wiki.git
```

**方案2：共享文件夹**
- 企业网盘
- NAS共享

**注意事项**：
- ✅ 需要访问控制
- ✅ 需要协作规范（避免冲突）
- ✅ 敏感信息分离
- ⚠️ Agent操作可能冲突（建议轮流使用）

---

#### Q10：如何迁移到新电脑？

**A**：

**简单方法**：

```bash
# 旧电脑：打包
tar -czf my-wiki.tar.gz /path/to/wiki/

# 复制到新电脑

# 新电脑：解压
tar -xzf my-wiki.tar.gz

# 重新安装llm-wiki-skill
cd llm-wiki-skill
bash install.sh --platform <你的平台>

# 在Obsidian中打开新Wiki路径
```

**Git方法**：

```bash
# 旧电脑已经git管理
git push

# 新电脑
git clone <repo-url>
```

---

### 14.3 使用问题

#### Q11：导入后怎么验证Agent正确理解了内容？

**A**：

**验证方法**：

1. **查看摘要页**：
```
请给我看一下刚才导入文章的摘要
```

2. **测试查询**：
```
刚才那篇文章的核心观点是什么？
```

3. **检查链接**：
```
刚才生成了哪些实体页？请列出
```

4. **Obsidian图谱**：
   - 打开图谱视图
   - 查看新增节点和连接

---

#### Q12：如何删除错误导入的素材？

**A**：

**步骤**：

1. **删除原始文件**：
```bash
rm /path/to/wiki/raw/articles/wrong-article.md
```

2. **告诉Agent清理**：
```
刚才导入的wrong-article.md是错误的，
请：
1. 删除对应的摘要页
2. 移除相关链接
3. 更新索引
4. 记录到日志
```

3. **可选：Lint检查**：
```
请运行健康检查，清理孤立内容
```

---

#### Q13：能自动去重吗？

**A**：

**部分支持**：

**自动去重**：
- ✅ URL去重（同一URL不会重复导入）
- ✅ 文件名去重（检测同名文件）

**需人工判断**：
- ⚠️ 内容相似但URL不同
- ⚠️ 同一主题的不同文章

**建议工作流**：

```
👤：发现两篇文章内容重复，请合并

🤖：分析中...
文章A和文章B确实高度重复（相似度89%）

建议：
1. 保留文章A（内容更完整）
2. 在文章A摘要中注明"另见文章B"
3. 删除文章B摘要
4. 合并实体页更新

是否执行？
```

---

#### Q14：怎么导出整个Wiki？

**A**：

**导出方法**：

**方法1：直接复制**
```bash
# 整个wiki就是文件夹，直接复制即可
cp -r /your/wiki/ /export/destination/
```

**方法2：打包**
```bash
tar -czf my-wiki-export.tar.gz /your/wiki/
```

**方法3：转换为PDF**（通过Obsidian插件）
1. 安装Pandoc插件
2. 选择要导出的页面
3. Export to PDF

**方法4：生成静态网站**
- 使用Obsidian Publish
- 或使用MkDocs + obsidian-to-mkdocs插件

---

#### Q15：如何处理多语言内容？

**A**：

**策略**：

**策略1：语言分离**
```
wiki/
├── entities-zh/    # 中文实体
├── entities-en/    # 英文实体
├── topics-zh/
└── topics-en/
```

**策略2：双语页面**
```markdown
# Transformer架构

## 中文说明
[中文内容...]

## English Description
[English content...]
```

**策略3：自动翻译（未来功能）**
```
👤：请为所有英文实体页生成中文版本
```

---

## 15. 技术架构

### 15.1 系统架构图

```
┌─────────────────────────────────────────────────────────┐
│                      用户层                             │
│  Obsidian  │  VS Code  │  Web界面  │  命令行           │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                   Agent层                               │
│  Claude Code  │  Codex  │  OpenClaw  │  其他Agent      │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                 LLM-Wiki Skill                          │
│  ┌─────────────────────────────────────────────┐       │
│  │  核心模块                                    │       │
│  │  - 素材路由器                                │       │
│  │  - Wiki编译器                                │       │
│  │  - 索引管理器                                │       │
│  │  - 链接维护器                                │       │
│  │  - 健康检查器                                │       │
│  └─────────────────────────────────────────────┘       │
│  ┌─────────────────────────────────────────────┐       │
│  │  提取器插件                                  │       │
│  │  - baoyu-url-to-markdown                     │       │
│  │  - wechat-article-to-markdown                │       │
│  │  - youtube-transcript                        │       │
│  └─────────────────────────────────────────────┘       │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                 文件系统层                              │
│  raw/  │  wiki/  │  index.md  │  log.md                │
└─────────────────────────────────────────────────────────┘
```

---

### 15.2 数据流

#### 导入流程

```
用户提供URL/文件
    ↓
Agent识别并路由
    ↓
提取器处理
    ↓
保存到raw/
    ↓
Wiki编译器分析
    ↓
生成/更新wiki/页面
    ↓
更新index.md
    ↓
记录到log.md
```

#### 查询流程

```
用户提问
    ↓
Agent读取index.md
    ↓
定位相关页面
    ↓
读取多个页面
    ↓
综合生成回答
    ↓
(可选)归档回wiki/
    ↓
记录到log.md
```

---

### 15.3 文件格式规范

#### 素材文件格式

```markdown
---
source_url: https://example.com/article
author: Author Name
date: 2026-04-08
type: article
extracted_at: 2026-04-08T14:30:00Z
---

# 文章标题

[文章内容...]
```

#### Wiki页面格式

```markdown
---
type: entity | topic | source | comparison | synthesis
created: 2026-04-08
updated: 2026-04-08
tags: [tag1, tag2]
source_count: 5
---

# 页面标题

## 内容

[正文...]

## 相关页面
- [[相关页面1]]
- [[相关页面2]]

## 来源
- [[source-001]]
- [[source-002]]

最后更新：2026-04-08
```

---

### 15.4 扩展接口

#### 自定义提取器

**接口规范**：

```typescript
interface Extractor {
  name: string;
  // 判断是否支持该URL
  canHandle(url: string): boolean;
  
  // 提取内容
  extract(url: string): Promise<{
    title: string;
    content: string;
    author?: string;
    date?: string;
    metadata?: Record<string, any>;
  }>;
}
```

**示例**（概念）：

```typescript
class CustomExtractor implements Extractor {
  name = 'my-custom-extractor';
  
  canHandle(url: string): boolean {
    return url.includes('customsite.com');
  }
  
  async extract(url: string) {
    // 实现提取逻辑
    return {
      title: '...',
      content: '...',
    };
  }
}
```

---

## 16. 贡献与致谢

### 16.1 项目依赖

本项目集成了以下开源项目：

#### 核心依赖

| 项目 | 作者 | 用途 | 许可证 |
|------|------|------|--------|
| **baoyu-url-to-markdown** | [JimLiu](https://github.com/JimLiu) | 网页内容提取 | MIT |
| **wechat-article-to-markdown** | [jackwener](https://github.com/jackwener) | 微信公众号提取 | MIT |
| **youtube-transcript** | - | YouTube字幕提取 | - |

#### 方法论来源

- **Andrej Karpathy** - [llm-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

---

### 16.2 贡献指南

欢迎贡献！

**可以贡献的内容**：

1. **新的提取器**
   - 支持更多网站
   - 改进提取质量

2. **Agent适配**
   - 支持新的Agent平台

3. **功能增强**
   - 多语言支持
   - 性能优化

4. **文档改进**
   - 翻译
   - 教程
   - 最佳实践

**贡献流程**：

```bash
# Fork项目
# 创建分支
git checkout -b feature/your-feature

# 开发和测试
# ...

# 提交PR
git push origin feature/your-feature
# 在GitHub上创建Pull Request
```

---

### 16.3 许可证

**MIT License**

---

### 16.4 联系方式

- **GitHub Issues**: https://github.com/sdyckjq-lab/llm-wiki-skill/issues
- **项目主页**: https://github.com/sdyckjq-lab/llm-wiki-skill

---

## 附录

### A. 快速参考

#### 常用命令

```bash
# 安装
bash install.sh --platform claude

# 检查安装
ls ~/.claude/skills/llm-wiki/

# 更新
cd llm-wiki-skill && git pull
bash install.sh --platform claude
```

#### 常用对话

```
# 初始化
"请在~/Documents/MyWiki初始化知识库"

# 导入
"请导入这篇文章：[URL]"
"请批量处理~/Downloads/papers/下的所有PDF"

# 查询
"RAG和Wiki有什么区别？"
"总结一下AI Agent的发展趋势"

# 维护
"请进行健康检查"
"请将刚才的回答归档到Wiki"
```

---

### B. 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| **原始素材** | Raw Sources | 未经处理的原始文件 |
| **Wiki编译** | Wiki Compilation | 将素材转换为结构化Wiki |
| **实体页** | Entity Page | 关于人物/概念/工具的页面 |
| **主题页** | Topic Page | 主题综述页面 |
| **双向链接** | Backlink | `[[页面]]`语法的互相引用 |
| **Lint** | Lint | 健康检查 |
| **回填** | File Back | 将查询结果归档回Wiki |
| **孤立页面** | Orphan Page | 无入站链接的页面 |

---

### C. 资源链接

#### 官方资源

- **项目主页**: https://github.com/sdyckjq-lab/llm-wiki-skill
- **Karpathy原文**: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

#### 工具下载

- **Obsidian**: https://obsidian.md/
- **uv**: https://github.com/astral-sh/uv
- **bun**: https://bun.sh/

#### 学习资源

- **Markdown语法**: https://www.markdownguide.org/
- **Obsidian文档**: https://help.obsidian.md/
- **Git基础**: https://git-scm.com/book/zh/v2

---

## 文档信息

| 项目 | 内容 |
|------|------|
| **文档版本** | v1.0 |
| **编写日期** | 2026年4月8日 |
| **最后更新** | 2026年4月8日 |
| **字数统计** | 约25000字 |
| **适用版本** | llm-wiki-skill main分支 |

---

**文档完整结束**

祝你构建知识库愉快！🎉📚

如有问题，请提交Issue：https://github.com/sdyckjq-lab/llm-wiki-skill/issues
