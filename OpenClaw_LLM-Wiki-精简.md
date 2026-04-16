# OpenClaw LLM-Wiki 指南

> **更新日期**: 2026年4月15日  
> **适用平台**: macOS + OpenClaw  
> **文档版本**: v1.0
> **完整文档**: [OpenClaw_LLM-Wiki.md](./OpenClaw_LLM-Wiki.md)

本文档精简总结 LLM-Wiki 知识库系统的核心要点。

---

## 更新日志

### 2026-04-15
- 📋 基于 Karpathy 方法论
- 🛠️ 提供完整安装脚本
- ✅ 创建完成详细技术文档

---

## 📋 目录

1. [LLM-Wiki 核心理念](#1-llm-wiki-核心理念)
2. [RAG vs LLM-Wiki](#2-rag-vs-llm-wiki)
3. [系统要求](#3-系统要求)
4. [系统架构](#4-系统架构)
5. [快速部署](#5-快速部署)
6. [使用演示](#6-使用演示)

---

## 1. LLM-Wiki 核心理念

### 🎯 核心思想

> **把碎片化的信息变成持续积累、互相链接的知识库**

**三个关键原则**：

| 原则 | 说明 | 价值 |
|------|------|------|
| **编译而非检索** | 素材编译成Wiki，而非每次检索 | 一次处理，持续使用 |
| **持久化积累** | 每次添加都让Wiki更丰富 | 知识复利增长 |
| **自动化维护** | Agent自动摘要、链接、检查 | 解放人工维护 |

---

### 📊 知识流转

```
原始素材 (raw/)
    ↓ 摄入(Ingest)
AI 分析提取
    ↓ 编译(Compile)
结构化 Wiki (wiki/)
    ↓ 查询(Query)
回答用户问题
    ↓ 回填(Feedback)
优质答案归档回 Wiki
    ↓
循环：知识持续增长
```

---

## 2. RAG vs LLM-Wiki

### 🆚 核心差异对比

| 对比维度 | RAG 模式 | LLM-Wiki 模式 ⭐ |
|---------|---------|-----------------|
| **知识形式** | 向量片段 | 结构化页面 |
| **查询方式** | 每次检索 | 导航阅读 |
| **积累性** | ❌ 无积累 | ✅ 持续积累 |
| **上下文** | ⚠️ 可能碎片化 | ✅ 完整保留 |
| **维护成本** | 查询时高 | 查询时低 |
| **人类可读** | ❌ 向量不可读 | ✅ Markdown 可读 |

---

### 💡 典型场景对比

**场景**: 你阅读了50篇AI Agent相关文章，想要总结发展趋势

| 方案 | 工作流程 | 结果 |
|------|---------|------|
| **RAG** | 每次查询 → 检索片段 → 拼凑答案 | ❌ 答案丢失，下次重新检索 |
| **Wiki** | 素材导入 → 编译成主题页 → 持续更新 | ✅ 答案归档，知识持续积累 |

---

## 3. 系统要求

### 💻 硬件配置

| 项目 | 基础配置 | 推荐配置 |
|------|---------|---------|
| **设备** | M1/M2/M3 Mac | M3 Pro/Max/M4 |
| **内存** | 16GB | 32GB+ |
| **存储** | 50GB | 100GB+ |
| **系统** | macOS 13+ | macOS 14+ |

### 📦 必需组件

| 组件 | 用途 | 检查命令 |
|------|------|---------|
| **Bash** | 运行脚本 | `bash --version` |
| **Git** | 克隆仓库 | `git --version` |
| **OpenClaw** | AI Agent平台 | `openclaw --version` |

### 🔧 可选工具

| 工具 | 用途 | 场景 |
|------|------|------|
| **Chrome调试** | 网页自动提取 | X/Twitter、知乎 |
| **uv** | Python环境 | 微信公众号提取 |
| **bun/npm** | JS依赖 | 网页内容处理 |
| **Obsidian** | 可视化浏览 | 图形化查看Wiki |

---

## 4. 系统架构

### 🏗️ 整体架构

```
┌──────────────────────────────────────────┐
│  用户交互层                               │
├──────────────────────────────────────────┤
│  • OpenClaw CLI                          │
│  • Web UI (http://localhost:3000)       │
│  • 微信 ClawBot                          │
└──────────────┬───────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│  OpenClaw Gateway                        │
├──────────────────────────────────────────┤
│  • LLM-Wiki Skill 加载                   │
│  • 工作流编排                             │
│  • 命令路由                               │
└──────────────┬───────────────────────────┘
               ↓
        ┌──────┴──────┐
        ↓             ↓
┌──────────────┐  ┌──────────────────┐
│ Ollama       │  │ LLM-Wiki Scripts │
│ LLM 引擎     │  │ 工具脚本集        │
├──────────────┤  ├──────────────────┤
│ • Qwen3.5 9B│  │ • init.sh        │
│ • 本地推理  │  │ • ingest.sh      │
│ • 无需联网  │  │ • query.sh       │
│ • 完全私密  │  │ • lint.sh        │
└──────────────┘  └──────────────────┘
        ↓                   ↓
┌──────────────────────────────────────────┐
│  知识库目录结构                           │
├──────────────────────────────────────────┤
│  raw/        - 原始素材（只读）           │
│  wiki/       - AI生成Wiki（可编辑）       │
│  index.md    - 总索引                     │
│  log.md      - 操作日志                   │
└──────────────────────────────────────────┘
```

---

### 📁 目录结构

```
你的知识库/
├── raw/                          # 原始素材（只读）
│   ├── articles/                 # 网页文章
│   │   └── 2026-04-15-xxx.md
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
├── wiki/                         # AI生成Wiki
│   ├── entities/                 # 实体页面
│   │   ├── people/              # 人物
│   │   │   ├── karpathy.md
│   │   │   └── hinton.md
│   │   ├── concepts/            # 概念
│   │   │   ├── rag.md
│   │   │   └── transformer.md
│   │   └── tools/               # 工具
│   │       ├── openclaw.md
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

### 📋 目录说明

| 目录 | 特性 | 用途 |
|------|------|------|
| **raw/** | 🔒 只读、不可变 | 存储所有原始素材 |
| **wiki/** | ✍️ AI自动维护 | 结构化知识库 |
| **entities/** | 人物/概念/工具 | 单个实体详细页 |
| **topics/** | 主题综述 | 领域知识整合 |
| **sources/** | 每个素材一页 | 素材精华摘要 |
| **comparisons/** | 对比分析 | 横向比较 |
| **synthesis/** | 综合分析 | 跨素材总结 |

---

## 5. 快速部署

### 🚀 三步安装

#### 第1步：安装 LLM-Wiki Skill

**让 OpenClaw Agent 自动安装**（推荐）：

```
请帮我安装 llm-wiki-skill：
https://github.com/sdyckjq-lab/llm-wiki-skill

使用OpenClaw平台，运行：
bash install.sh --platform openclaw
```

**或手动安装**：

```bash
cd ~/Downloads
git clone https://github.com/sdyckjq-lab/llm-wiki-skill.git
cd llm-wiki-skill
bash install.sh --platform openclaw
```

**安装位置**: `~/.openclaw/skills/llm-wiki/`

**自定义路径**（可选）：

```bash
# 如果 OpenClaw 使用非标准路径
bash install.sh \
  --platform openclaw \
  --target-dir ~/MyOpenClaw/skills
```

---

#### 第2步：初始化知识库

```
/wiki-init ~/Documents/MyWiki
```

**或自然语言**：

```
请在 ~/Documents/MyWiki 初始化一个知识库，
主题是开发速查手册
```

**Agent自动创建**：
- ✅ 完整目录结构
- ✅ 模板文件
- ✅ 配置文件

---

#### 第3步：开始使用

```
/wiki-ingest https://example.com/article
```

就这么简单！🎉

---

### 🔧 可选配置

#### Chrome调试模式（用于网页提取）

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 &
```

#### 安装 uv（用于微信公众号）

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

---

## 6. 使用演示

### 🎬 完整工作流程

> **场景**: 学习AI Agent，从文章到知识库

---

#### 📝 第1步：初始化知识库

```
👤 用户: "请在 ~/Desktop/xiuery/coding-manual  初始化知识库，
主题是程序开发速查手册"

🤖 OpenClaw:
正在初始化知识库...

✓ 创建目录结构
  - raw/ (原始素材)
  - wiki/ (知识库)

✓ 生成模板文件
  - index.md (总索引)
  - log.md (操作日志)
  - .wiki-schema.md (配置)

✅ 知识库已就绪：~/Desktop/xiuery/coding-manual 

现在可以开始导入素材了！
```

---

#### 📚 第2步：批量导入文档

```
👤 用户: "批量处理 ~/Desktop/xiuery/openclaw/docs 下的所有文档"

🤖 OpenClaw:
扫描目录...
✓ 发现 8 个文件

━━━━━━━━━━━━━━━━━━━━━━━━━━
开始批量处理：

[1/8] python-best-practices.md
  ✓ 提取成功 (Markdown文档)
  ✓ 创建实体：[[Python]], [[Code Style]]
  ⏱️ 用时：1分钟

[2/8] docker-cheatsheet.pdf
  ✓ 提取成功 (15页)
  ✓ 创建实体：[[Docker]], [[Container]]
  ⏱️ 用时：2分钟

[3/8] git-workflow.md
  ✓ 提取成功 (Markdown文档)
  ✓ 更新实体：[[Git]], [[Version Control]]
  ⏱️ 用时：1分钟

[4/8] react-hooks-guide.pdf
  ✓ 提取成功 (25页)
  ✓ 创建实体：[[React]], [[Hooks]]
  ⏱️ 用时：2分钟

[5/8] api-design-patterns.md
  ✓ 提取成功 (Markdown文档)
  ✓ 创建实体：[[API Design]], [[REST]]
  ⏱️ 用时：1分钟

[6/8] database-optimization.pdf
  ✓ 提取成功 (35页)
  ✓ 创建实体：[[Database]], [[SQL Optimization]]
  ⏱️ 用时：3分钟

[7/8] testing-strategies.txt
  ✓ 提取成功 (纯文本)
  ✓ 创建实体：[[Unit Test]], [[TDD]]
  ⏱️ 用时：1分钟

[8/8] kubernetes-deployment.md
  ✓ 提取成功 (Markdown文档)
  ✓ 创建实体：[[Kubernetes]], [[Deployment]]
  ⏱️ 用时：1分钟

━━━━━━━━━━━━━━━━━━━━━━━━━━
批处理完成！

📊 统计：
  - 新增摘要页：8个
  - 新增实体页：15个
  - 更新主题页：5个
  - 总耗时：12分钟

详细报告：wiki/batch-report-2026-04-15.md
```

**展示能力**:
- ✅ 批量处理能力
- ✅ 进度实时显示
- ✅ 自动关联知识
- ✅ 生成处理报告

---

#### 📥 第3步：导入网页文章

```
👤 用户: "请导入这篇文章：
https://manual.xiuery.com/db/mongodb/manual.html"

🤖 OpenClaw:
正在提取网页内容...
✓ 使用 baoyu-url-to-markdown 提取成功
✓ 文章长度：4,200字（长文）

保存到：raw/articles/2026-04-15-mongodb-manual.md

━━━━━━━━━━━━━━━━━━━━━━━━━━
编译到 Wiki...

✓ 生成摘要页：wiki/sources/article-001.md

✓ 识别实体（创建实体页）：
  • [[MongoDB]] - entities/tools/mongodb.md
  • [[Database]] - entities/concepts/database.md
  • [[NoSQL]] - entities/concepts/nosql.md
  • [[Document Store]] - entities/concepts/document-store.md

✓ 创建主题页：
  • topics/database-management.md

✓ 更新索引：index.md
✓ 记录日志：log.md

━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 完成！本次影响 8 个页面。
```

**展示能力**:
- ✅ 自动网页提取
- ✅ 智能分析实体
- ✅ 自动创建链接
- ✅ 生成知识网络

---

#### 🔍 第4步：查询知识库

```
👤 用户: "MongoDB的索引优化有哪些最佳实践？"

🤖 OpenClaw:
查询 index.md...找到相关页面

读取页面：
  • wiki/entities/tools/mongodb.md
  • wiki/topics/database-optimization.md
  • wiki/sources/mongodb-manual.md

━━━━━━━━━━━━━━━━━━━━━━━━━━
根据 Wiki 内容回答：

【MongoDB索引优化最佳实践】

1️⃣ **选择合适的索引类型**
   • 单字段索引：最常用查询字段
   • 复合索引：多字段组合查询
   • 文本索引：全文搜索场景

2️⃣ **索引设计原则**
   • ESR规则：Equality → Sort → Range
   • 高选择性字段优先
   • 避免过多索引（影响写入性能）

3️⃣ **性能优化技巧**
   • 使用覆盖索引减少文档扫描
   • 定期分析慢查询日志
   • 使用explain()分析查询计划

4️⃣ **维护建议**
   • 定期重建索引（compact）
   • 监控索引大小和使用率
   • 删除未使用的索引

详见：[[MongoDB索引优化]]

━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 本回答是否有价值？
如有必要，可归档为新页面。
```

**展示能力**:
- ✅ 智能定位页面
- ✅ 综合多个来源
- ✅ 结构化回答
- ✅ 提供Wiki链接

---

#### ✅ 第5步：健康检查

```
👤 用户: "/wiki-lint"

🤖 OpenClaw:
开始全库扫描...

━━━━━━━━━━━━━━━━━━━━━━━━━━
【检查结果】

⚠️ 孤立页面（2个）：
  • entities/concepts/memex.md - 无入站链接
  • topics/zettelkasten.md - 未被引用
  
  🔧 建议：链接到 knowledge-management 主题页

⚠️ 断开链接（3个）：
  • topics/ai-agents.md → [[GPT-4]] (页面不存在)
  • entities/people/karpathy.md → [[Tesla AI]] (不存在)
  • sources/article-003.md → [[Scaling Laws]] (不存在)
  
  🔧 已自动修复：2个
  ⚠️ 待创建页面：1个

✅ 信息一致性：良好

💡 建议创建的页面（3个）：
  • [[GPT-4]] - 被引用 8 次
  • [[Tesla AI]] - 被引用 5 次
  • [[Scaling Laws]] - 被引用 4 次

━━━━━━━━━━━━━━━━━━━━━━━━━━
修复操作：
  ✓ 自动修复断链：2个
  ✓ 建议孤立页链接位置
  ⏳ 待人工确认：创建3个新页面

详细报告：wiki/lint-report-2026-04-15.md
```

**展示能力**:
- ✅ 全库健康检查
- ✅ 发现问题
- ✅ 自动修复
- ✅ 改进建议

---

#### 📊 第6步：查看状态

```
👤 用户: "/wiki-status"

🤖 OpenClaw:
【知识库状态】

📍 位置：~/Documents/coding-manual

━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 统计数据：

素材 (raw/)：
  • 文章：15 篇
  • 论文：5 篇
  • 推文：3 条
  • 笔记：2 篇
  ─────────────
  • 总计：25 篇

Wiki 页面 (wiki/)：
  • 实体页：18 个
    - 人物：5
    - 概念：10
    - 工具：3
  • 主题页：6 个
  • 摘要页：25 个
  • 对比页：2 个
  • 综合页：1 个
  ─────────────
  • 总计：52 个页面

━━━━━━━━━━━━━━━━━━━━━━━━━━
📈 增长趋势：
  • 近7天新增：12 篇素材
  • 本月累计：25 篇
  • 活跃主题：AI Agents, Knowledge Management

⚠️ 待处理：
  • 孤立页面：2 个
  • 建议创建：3 个页面
  • 上次 Lint：2分钟前

最后更新：2026-04-15 15:30
━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### 🎯 核心命令速查

| 命令 | 用途 | 示例 |
|------|------|------|
| `/wiki-init` | 初始化知识库 | `/wiki-init ~/Documents/MyWiki` |
| `/wiki-ingest` | 导入素材 | `/wiki-ingest https://example.com` |
| `/wiki-query` | 查询知识库 | `/wiki-query RAG是什么？` |
| `/wiki-lint` | 健康检查 | `/wiki-lint` |
| `/wiki-status` | 查看状态 | `/wiki-status` |

---

### 💡 高级技巧

#### 技巧1：多素材综合分析

```
👤: "基于Wiki，总结2026年AI Agent的三大趋势"

🤖: [读取18篇相关素材]
     [生成综合分析页]
     
✅ 已归档到：wiki/synthesis/ai-agent-trends-2026.md
```

#### 技巧2：对比分析

```
👤: "对比GPT-4和Claude的能力差异"

🤖: [读取相关实体页和素材]
     [生成对比页]
     
✅ 已归档到：wiki/comparisons/gpt4-vs-claude.md
```

#### 技巧3：自动回填

```
👤: "这份分析很好，归档到Wiki"

🤖: ✅ 已归档为新页面
     详见：wiki/synthesis/xxx.md
```

---

## 📚 素材来源支持

### ✅ 核心支持（无外部依赖）

| 类型 | 格式 | 操作 |
|------|------|------|
| **本地文件** | PDF/MD/TXT | 直接读取 |
| **纯文本** | 粘贴内容 | 即时处理 |

### 🔧 扩展支持（需配置）

| 来源 | 依赖 | 提取器 |
|------|------|--------|
| **网页** | Chrome调试 | baoyu-url-to-markdown |
| **X/Twitter** | Chrome调试+登录 | baoyu-url-to-markdown |
| **微信公众号** | uv | wechat-article-to-markdown |
| **YouTube** | ❌ 无 | youtube-transcript |
| **知乎** | Chrome调试 | baoyu-url-to-markdown |

### 🔄 手动回退

| 来源 | 操作 |
|------|------|
| **小红书** | 复制粘贴内容 |
| **提取失败** | 复制粘贴内容 |

---

## 🔗 Obsidian 集成

### 📊 可视化浏览

所有Wiki内容都是标准Markdown，可直接用Obsidian打开：

**步骤**：

1. **安装 Obsidian**: https://obsidian.md/
2. **打开 Wiki**: Obsidian → Open folder → 选择Wiki根目录
3. **查看图谱**: Ctrl/Cmd + G

**优势**：

| 功能 | 说明 |
|------|------|
| 📊 **图谱视图** | 可视化知识网络 |
| 🔗 **双向链接** | 点击`[[链接]]`跳转 |
| 🔍 **全文搜索** | 快速定位内容 |
| 🏷️ **标签过滤** | 按主题分类查看 |

---

## 💡 最佳实践

### ✅ 推荐做法

| 场景 | 建议 |
|------|------|
| **初始化** | 明确主题，规划目录 |
| **导入** | 先测试1-2个样本 |
| **查询** | 充分利用Wiki链接 |
| **维护** | 定期健康检查 |
| **备份** | Git版本控制 |

### ⚠️ 注意事项

1. **素材可追溯** - 保留原始链接和元数据
2. **定期Lint** - 发现并修复问题
3. **人工审核** - 关键内容二次确认
4. **Git备份** - 防止数据丢失

---

## 📚 相关资源

### 官方文档
- **完整技术文档**: [OpenClaw_LLM-Wiki.md](./OpenClaw_LLM-Wiki.md)
- **项目主页**: https://github.com/sdyckjq-lab/llm-wiki-skill
- **Karpathy原文**: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

### 工具下载
- **OpenClaw**: https://openclaw.ai/
- **Ollama**: https://ollama.com/
- **Obsidian**: https://obsidian.md/
- **uv**: https://github.com/astral-sh/uv

---

**文档结束**

> 本文档精简总结了 OpenClaw LLM-Wiki 的核心要点  
> 详细技术信息、故障排查请参阅完整版文档
